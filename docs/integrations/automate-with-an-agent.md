# Recurring automation: a weekly refresh and report from an external agent

An external scheduler can refresh a pipeline, freeze its output in a Studies recipe, launch an authorised fit and read its results. The scheduler supplies the cadence, credentials, durable job state and model-authoring rules. Simba does not automatically approve the resulting model or select it as Champion.

This guide describes the current pipeline-version workflow. New pipeline runs store their output as a `PipelineVersion`; they do not create an uploaded-file dataset. Do not discover a fresh version by searching `list_uploads` for a generated filename.

## Prepare the job

Use a synthetic pipeline, a study in a project you own, and an explicit active quality policy for your rehearsal. Replace every example id with an id returned from your account. Author and validate the model configuration before enabling a recurring fit. A source template provides bytes and defaults, not a validated specification.

Keep an API key in the scheduler's secret store. The workflow needs `ingest` for pipeline discovery, `read:models` for templates and study reads, `create:models` for pipeline runs, drafts, publication and model launches, and `read:results` for the report. Keys expire and require rotation. Creating a key, choosing scientific thresholds and authorising recurring compute are setup decisions, not actions the example performs.

## Refresh and pin the exact version

```text
run_pipeline(pipeline_ref="<your pipeline hash>")
get_pipeline_run(pipeline_ref="<your pipeline hash>", run_id=<returned run_id>)
```

Poll until `succeeded` or `failed`. A `409 run_in_progress` can identify an existing run to follow; other refusals must be handled according to their code. Persist the successful run's `version_id`. Do not substitute the newest listed version, which another job may have created.

After an ambiguous pipeline-start response without a run id, stop the scheduler and ask a signed-in person to inspect the pipeline history before starting another run. This history is currently exposed in the builder, not as an MCP or API-key history operation. Pipeline starts do not have the Studies launch submission key. Configure the external scheduler to prevent overlapping executions and keep its checkpoint across retries.

## Author, freeze and launch

```text
get_recipe_draft_template(family="mmm", pipeline_version_id=<persisted version_id>)
create_recipe_draft(study_id="<your study id>", draft_id="<persisted UUID>",
                    name="Weekly synthetic refresh", snapshot=<complete authored snapshot>)
publish_recipe_draft(draft_id="<same UUID>", expected_version=<returned draft version>,
                     publication_id="<persisted publication UUID>", reason="Weekly refresh")
get_launch_eligibility(study_id="<your study id>", revision_id="<returned revision id>",
                       policy_id="<your active policy id>")
launch_study_run(study_id="<your study id>", revision_id="<same revision id>",
                 policy_id="<same policy id>", submission_key="weekly-synthetic-2026-W40")
```

Copy the complete template snapshot. Configure the outcome, date, hierarchy, media roles, units, priors and sampling using your reviewed authoring rules. Preserve the source bytes and origin of the pinned version. A recurring integration must check schema drift and stop when those rules no longer apply. The template's first rows are a preview, not the full source. See [Build a model from a pipeline version](./model-from-a-pipeline-version.md) and [Studies](../platform-guide/studies.md).

Persist the draft UUID, exact authored snapshot, publication UUID, returned revision ids, policy id and launch key before the associated writes. Publication can return multiple revisions, one per prepared brand: select and record the intended revision explicitly, or manage each as a separate job. Do not silently choose the first.

A repeated draft creation with the same UUID and content returns the existing draft; changed content conflicts. Replaying the same publication request returns the same revisions. Launch retries with the same study, submitter, revision, policy and `submission_key` return the existing run without reserving a second fit. A different revision or policy under the same key returns `409 submission_key_conflict`. Do not generate a fresh key merely because a response was lost. A model-name lookup is not an idempotency guarantee.

## Runnable launch and report stage

The following Python example requires `requests`, an already published synthetic revision and an active policy. It **starts a new fit** on its first successful submission. Run it only after authorising that compute. It is the launch/report stage, not a complete unattended authoring integration. The pipeline refresh and version-to-snapshot authoring steps above must be implemented and validated for your data before scheduling the whole cycle.

Set `SIMBA_API_URL` to the service origin, `SIMBA_API_KEY` from a secret store, and `SIMBA_STUDY_ID`, `SIMBA_REVISION_ID`, `SIMBA_POLICY_ID`, `SIMBA_SUBMISSION_KEY` to your persisted job values. Keep the same values on retry.

```python
import json
import os
import time
import requests

base = os.environ["SIMBA_API_URL"].rstrip("/") + "/api/v1"
study = os.environ["SIMBA_STUDY_ID"]
payload = {
    "revision_id": os.environ["SIMBA_REVISION_ID"],
    "policy_id": os.environ["SIMBA_POLICY_ID"],
    "submission_key": os.environ["SIMBA_SUBMISSION_KEY"],
}
session = requests.Session()
session.headers["Authorization"] = "Bearer " + os.environ["SIMBA_API_KEY"]

def request(method, path, **kwargs):
    response = session.request(method, base + path, timeout=60, **kwargs)
    response.raise_for_status()
    return response.json()

# Inspect blockers before a first submission. For an ambiguous launch response,
# replay the identical POST: it may already own a reservation or completed run.
run = request("POST", f"/studies/{study}/runs", json=payload)
model_hash = run["model_hash"]
if not model_hash:
    raise RuntimeError("The run has no model hash; inspect the study run")

deadline = time.monotonic() + 3600
while True:
    status = request("GET", f"/models/{model_hash}/status")
    if status["status"] not in ("pending", "under way"):
        break
    if time.monotonic() >= deadline:
        raise TimeoutError("Polling stopped; the fit may continue. Resume with the same job key.")
    time.sleep(10)
if status["status"] != "complete":
    raise RuntimeError(f"Fit ended: {status['status']}; {status.get('error')}")

results = request("GET", f"/models/{model_hash}/results",
                  params={"sections": "channel_summary,model_stats"})
print(json.dumps({"study_id": study, "revision_id": payload["revision_id"],
                  "run_id": run["id"], "model_hash": model_hash,
                  "results": results["results"]}, indent=2))
```

No blanket HTTP retry is configured. Read the refusal code, preserve the checkpoint and recover the specific operation. A timeout ends this script's wait; it does not cancel the fit. Study-owned fits are retained by the study, so this stage does not call `save_model`.

## Reporting and acceptance

Quote the exact pipeline version, revision, run and model hash alongside the reporting window. `channel_summary` describes the model's fitted window, and each new fit can restate historical estimates. Read `model_stats` and evaluate the run against the chosen quality policy before recommending it for use. Completion is not proof of convergence, causal validity or held-out accuracy.

The current dataset report accepts uploaded-file dataset ids, including historical registered pipeline outputs. It does not take a new pipeline version id. Do not send a version id as `dataset_id` or claim a newly refreshed actual-data report from that endpoint. Report actuals through a separately supported source workflow with explicit provenance.

A recurring run can prepare results and a recommendation. Acceptance, manual sign-off and Champion selection require a signed-in person in the application. A synthetic demonstration validates orchestration and contracts; it does not validate a client's model or certify unattended production operation.

## Next steps

- [Refresh data on a schedule](./refresh-data-on-a-schedule.md)
- [Build a model from a pipeline version](./model-from-a-pipeline-version.md)
- [Studies](../platform-guide/studies.md)
- [Simba MCP](./simba-mcp.md)
