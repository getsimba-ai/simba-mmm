# Recurring automation: a weekly refresh and report from an external agent

An agent outside Simba can run the whole weekly cycle on its own: refresh the data, fit a model on the new version, wait for it, and read the numbers for a report. Any scheduler that can run an MCP client or plain HTTP will do: a scheduled Claude Code job, a script that calls the Claude API with its MCP connector, or a cron job in Python.

Every step below is one MCP tool and one HTTP request on the same API, with the same key and the same scopes. The MCP tool reference is generated from the running server: see [`docs/tools.md` in the simba-mcp repository](https://github.com/getsimba-ai/simba-mcp/blob/main/docs/tools.md) for the exact parameters of `run_pipeline`, `get_pipeline_run`, `create_model`, `get_model_status`, `get_model_results` and `get_data_report`. Connecting a client is covered in [Simba MCP](./simba-mcp.md).

## The job at a glance

| Step | MCP tool | HTTP | Scope |
|---|---|---|---|
| Refresh the data | `run_pipeline`, `get_pipeline_run` | `POST` / `GET /api/v1/pipelines/{pipeline_ref}/runs` | `create:models` |
| Find the new dataset | `list_pipeline_versions`, `list_uploads` | `GET /api/v1/pipelines/{pipeline_ref}/versions`, `GET /api/v1/ingest` | `ingest` |
| Create the model once | `list_models`, `create_model` | `GET` / `POST /api/v1/models` | `read:models`, `create:models` |
| Wait for the fit | `get_model_status` | `GET /api/v1/models/{model_hash}/status` | `read:models` |
| Keep the model | `save_model` | `POST /api/v1/models/{model_hash}/save` | `create:models` |
| Read the report | `get_model_results`, `get_data_report` | `GET /api/v1/models/{model_hash}/results`, `GET /api/v1/datasets/{dataset_id}/report` | `read:results`, `read:models` |

## Refresh the data

Either let the pipeline's own schedule run first (see [Refresh data on a schedule](./refresh-data-on-a-schedule.md)) or start a run yourself with `run_pipeline(pipeline_ref="ab12cd34ef")` and poll `get_pipeline_run(pipeline_ref="ab12cd34ef", run_id=41)` until `status` is `succeeded` or `failed`, as described on [Connect your warehouse](./connect-your-warehouse.md). Only one run of a pipeline happens at a time: if the schedule is already running it, `run_pipeline` answers `409 run_in_progress` with that run's `run_id`, and you poll that one instead.

A successful run saves a new version and registers its output as a dataset named `{pipeline name}_v{version}.csv` with `source_type` `pipeline`; any character in the pipeline name other than a letter, a digit, `-` or `_` becomes `_`, leading and trailing `_` are dropped, and an empty name becomes `pipeline`, so a pipeline named `weekly_media` gives `weekly_media_v12.csv`. Match the run's `version_id` to a row of `list_pipeline_versions` (newest first, so with the schedule alone the first row is the refreshed data) to get the version number, then find the dataset id with `list_uploads(name="weekly_media_v12.csv")`. That id is the `uploaded_file_id` that `create_model` needs.

## Create the model once

Today `create_model` has no submission key: every call starts a new fit and returns a new `model_hash`, so a job retried after a lost response would fit twice. Give each week's model a deterministic `name` (it is honoured verbatim, apart from HTML tags being stripped) and look for it before creating. API-created models start unsaved, so list with `include_unsaved=True`; the listing is newest first.

```
list_models(include_unsaved=True)
create_model(uploaded_file_id=57, name="Weekly MMM 2026-W40",
             date_column="date", kpi_column="revenue", hierarchy_column="brand",
             channels=[{"name": "TV", "activity_column": "tv_activity", "spend_column": "tv_spend"},
                       {"name": "Search", "activity_column": "search_activity", "spend_column": "search_spend"},
                       {"name": "Social", "activity_column": "social_activity", "spend_column": "social_spend"}],
             total_media_effect="Retail")
→ {"model_hash": "f835671a25", "status": "pending", "name": "Weekly MMM 2026-W40"}
```

If the weekly fit belongs to a study, launch it there instead: `launch_study_run(study_id=..., revision_id=..., policy_id=..., submission_key="weekly-2026-W40")` takes a caller-generated `submission_key`. Repeating the call with the same key returns the same run; the same key with a different revision or policy is refused with `409`. A revision on the refreshed data starts from `get_recipe_draft_template(pipeline_version_id=212)`; the rest is in [A Studies workflow](./simba-mcp.md#a-studies-workflow).

## Wait for the fit, then read the report

```
get_model_status(model_hash="f835671a25")   → {"status": "under way", "progress": 40, "error": null, ...}
```

Keep polling while `status` is `pending` or `under way`; `complete` is the outcome you want, `failed` carries the message in `error`, and any other status (a cancelled fit is `revoked`, one that ran out of time is `time exceeded`) also ends the wait. Polling every 10 seconds stays well inside the per-key rate limit. Then file the model with `save_model(model_hash="f835671a25", name="Weekly MMM 2026-W40")` so it appears in the default listing and is not pruned as an unsaved model; at the saved-models cap (the same cap as in the app) the save is refused with `400` and `error_type` `saved_limit`.

```
get_model_results(model_hash="f835671a25", sections="channel_summary,model_stats")
get_data_report(dataset_id=57, start="2026-09-07", end="2026-10-04", granularity="week",
                group_by="channel", metrics=["kpi", "spend"], roles={"revenue": "kpi", "brand": "hierarchy"})
```

`channel_summary` gives one row per channel with `Channel`, `Sales`, `Spend`, `Revenue` and `ROI`; `model_stats` gives the fit diagnostics. The data report gives the actual weekly KPI and spend from the dataset, and its `data_through` says how fresh the data is. Windows, roles and the aggregation rules are on [Report sales and media data](./report-sales-and-media-data.md).

## The key the job needs

Create the key under Profile → API Keys, as the user who owns the pipeline, with the scopes `ingest`, `read:models`, `read:results` and `create:models`; `optimize` and `scenario` are not needed for this job. A key sees only its owner's pipelines and models. Every key expires: 90 days by default, and one year at most, so put the rotation date in the scheduler's calendar. There is no MCP tool for creating or revoking keys. Keep the key in the scheduler's secret store, never in a prompt or in source control.

## A complete example

A cron job in Python, using synthetic data with columns `date`, `revenue`, `brand` and `{channel}_activity` / `{channel}_spend` for TV, Search and Social.

```python
import datetime as dt, json, re, time, requests

BASE = "https://demo.simba-mmm.com/api/v1"
HEADERS = {"Authorization": "Bearer simba_sk_..."}   # from the scheduler's secret store
PIPELINE = "ab12cd34ef"                               # pipeline_hash from list_pipelines
MODEL_NAME = f"Weekly MMM {dt.date.today():%G-W%V}"   # e.g. Weekly MMM 2026-W40

def get(path, **params):
    r = requests.get(f"{BASE}{path}", headers=HEADERS, params=params)
    r.raise_for_status()
    return r.json()
def wait(path, running):
    while (obj := get(path))["status"] in running:
        time.sleep(10)
    return obj

# 1. Refresh the data. A run already going answers 409 with its run_id: follow it.
r = requests.post(f"{BASE}/pipelines/{PIPELINE}/runs", headers=HEADERS, json={})
if r.status_code not in (202, 409): r.raise_for_status()   # 202 queued, 409 run_in_progress
run = wait(f"/pipelines/{PIPELINE}/runs/{r.json()['run_id']}", ("queued", "running"))
if run["status"] != "succeeded":
    raise SystemExit(f"refresh failed: {run['error_code']}: {run['error']}")

# 2. The dataset the new version registered: {pipeline name}_v{version}.csv
versions = get(f"/pipelines/{PIPELINE}/versions")
number = next(v["version"] for v in versions["versions"] if v["id"] == run["version_id"])
safe_name = re.sub(r"[^\w-]", "_", versions["pipeline"]["name"]).strip("_") or "pipeline"
dataset_id = get("/ingest", name=f"{safe_name}_v{number}.csv")["files"][0]["id"]

# 3. Create the model once: a retried job finds this week's model by name instead.
found = [m for m in get("/models", include_unsaved="true")["models"]
         if m["name"] == MODEL_NAME and m["status"] != "failed"]
if found:
    model_hash = found[0]["model_hash"]
else:
    r = requests.post(f"{BASE}/models", headers=HEADERS, json={
        "name": MODEL_NAME, "data_source": {"uploaded_file_id": dataset_id},
        "date_column": "date", "kpi_column": "revenue", "hierarchy_column": "brand",
        "channels": [
            {"name": "TV", "activity_column": "tv_activity", "spend_column": "tv_spend"},
            {"name": "Search", "activity_column": "search_activity", "spend_column": "search_spend"},
            {"name": "Social", "activity_column": "social_activity", "spend_column": "social_spend"}],
        "total_media_effect": "Retail"})
    r.raise_for_status()
    model_hash = r.json()["model_hash"]

# 4. Wait for the fit, then file the model so it is not pruned as an unsaved model.
status = wait(f"/models/{model_hash}/status", ("pending", "under way"))
if status["status"] != "complete":
    raise SystemExit(f"fit ended with status {status['status']}: {status['error']}")
requests.post(f"{BASE}/models/{model_hash}/save", headers=HEADERS, json={"name": MODEL_NAME}).raise_for_status()

# 5. The report: channel ROI from the model, actual weekly KPI and spend from the data.
results = get(f"/models/{model_hash}/results", sections="channel_summary,model_stats")["results"]
for row in results["channel_summary"]:
    print(f"{row['Channel']}: ROI {row['ROI']:.2f} on spend {row['Spend']:,.0f}")
report = get(f"/datasets/{dataset_id}/report", start="2026-09-07", end="2026-10-04",
             granularity="week", group_by="channel", metrics="kpi,spend",
             roles=json.dumps({"revenue": "kpi", "brand": "hierarchy"}))
print(f"model {model_hash}, data through {report['dataset']['data_through']}")
```

## Things to keep in mind

- **A retry is your responsibility on `create_model`.** It has no submission key today; the name check above is what stops a second fit. `launch_study_run` is the one fit submission that dedupes on a key.
- **Unsaved models are pruned.** The app keeps at most 10 unsaved models per user and prunes when a model is created in the app. A fit still in progress, a saved model, a shared one, and one in a portfolio or a study are never pruned. Save the weekly model, or it may be gone by the next report.
- **Each week's model restates history.** Quote the `model_hash` and the `data_through` you read from in every report.
- **Rate limits are per key.** Model status, results, the data report, the model and dataset listings and the version listing allow 60 requests a minute; polling a pipeline run allows 120; starting a pipeline run allows 10; creating a model allows 100.

## Next steps

- [Refresh data on a schedule](./refresh-data-on-a-schedule.md)
- [Connect your warehouse](./connect-your-warehouse.md)
- [Report sales and media data](./report-sales-and-media-data.md)
- [Simba MCP](./simba-mcp.md)
- [Incremental Measurement](../platform-guide/measurement.md)
