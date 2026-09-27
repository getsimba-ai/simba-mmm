# Connect your warehouse

A **data pipeline** reads your data where it already lives — a database, a data lake, a file — shapes it into Simba's modelling format, and saves each result as a versioned dataset you can build models on. You build a pipeline once in the app; after that it can be run by you, by an agent over the API or [Simba MCP](./simba-mcp.md), or on a schedule (see [Refresh data on a schedule](./refresh-data-on-a-schedule.md)).

## Sources

| Source | What it reads | You provide |
|---|---|---|
| **CSV upload** | A file you upload | The file |
| **PostgreSQL** | A table or a read-only SQL query | Host, port, database, user, password, and a table or query |
| **Amazon Athena** | A Glue Data Catalog table or an Athena query — a good fit for a Fivetran Managed Data Lake | AWS region, an S3 location for query results, a database and table (or a query), and an AWS access key |
| **S3 Iceberg** | An Apache Iceberg table on S3 through the AWS Glue catalog, read directly with no query engine | AWS region, the Glue namespace and table, and an AWS access key |

The builder lists only the sources the service can run. Other connectors (Snowflake, Databricks, weather and economic indicators) appear on deployments that include their drivers; where a driver is missing, the source is hidden rather than failing when it runs. A pipeline that already uses a hidden source still opens, and a run of it stops with a message naming the missing driver.

### Credentials

- **Saved credentials are write-only.** Once saved, a password or secret key comes back as `***`; send `***` back to keep the saved value. Credentials are stored encrypted and are never copied into a pipeline's saved versions.
- **AWS sources use the access key you enter** (optionally assuming a role with it). The hosted service never reads your data with its own AWS identity. Give the key read access to the tables you need, and write access only to the Athena query-results location.

## Build a pipeline

In the app, open **Data Pipelines** and create a pipeline. Drag a source onto the canvas, then add steps — filter, merge, aggregate, calculate, clean — and mark the step whose output is the dataset. Each step previews its output as you edit, so you can see the columns before anything is saved.

## Run it

Press **Run pipeline**. The run happens on the server: the page follows it, shows how long it has been running, and you can leave the page — the run carries on. When it finishes, each step shows its columns, the new version appears in the **Runs** menu, and its output is available in your datasets for model building. If it fails, the step that failed is marked and the message says why and what to change.

Only one run of a pipeline happens at a time. Pressing Run while another run is going — started in another tab, by an agent, or by the schedule — follows that run instead of starting a second one.

The **Runs** menu lists every run: when, who started it (you, the schedule, or an API key by name), the version it saved, and why a run failed.

## Run it from the API or MCP

Both need an API key with the `create:models` scope, and see only the key owner's pipelines. `pipeline_ref` is a pipeline's hash or id (from `list_pipelines`).

```
POST /api/v1/pipelines/{pipeline_ref}/runs          → 202 {"run_id": 41, "status": "queued"}
GET  /api/v1/pipelines/{pipeline_ref}/runs/{run_id} → {"status": "succeeded", "version_id": 212, ...}
```

Over MCP:

```
run_pipeline(pipeline_ref="ab12cd34ef")
get_pipeline_run(pipeline_ref="ab12cd34ef", run_id=41)
```

`run_pipeline` answers at once; poll `get_pipeline_run` every few seconds until `status` is `succeeded` or `failed`. An optional `start_date` / `end_date` (YYYY-MM-DD) limits the source steps to that date range. A successful run returns the `version_id` it saved — pass it as `pipeline_version_id` to `get_recipe_draft_template` to build a model on the refreshed data.

| Status | Meaning |
|---|---|
| `queued` | Waiting for a worker, usually a second or two |
| `running` | Reading and shaping the data |
| `succeeded` | A new version was saved (`version_id`) |
| `failed` | See `error_code` and `error` |

| `error_code` | What to do |
|---|---|
| `execution_failed` | A source or step failed; `error` names it. Fix it in the builder and run again |
| `no_output` | The date range returned no rows; widen it |
| `timeout` | The run passed its 30-minute limit; narrow the query or the date range |
| `interrupted`, `not_started` | The run was lost on the server; start it again |
| `not_queued` | The run could not be queued; try again shortly |
| `owner_blocked` | The pipeline owner's account is blocked, so its runs do not start |

Starting a run while one is going returns `409 run_in_progress` with that run's `run_id`: poll it rather than starting another.

The exact MCP parameters are in the [simba-mcp tool reference](https://github.com/getsimba-ai/simba-mcp/blob/main/docs/tools.md).
