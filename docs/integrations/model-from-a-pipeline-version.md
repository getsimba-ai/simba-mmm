# Build a model from a pipeline version

Every pipeline run that produces output saves a version: the output rows, their column names, a row count and a checksum. You can build a model on one specific version over the API or [Simba MCP](./simba-mcp.md), so the model records exactly which data it was fitted on.

Building and running the pipeline itself is covered in [Connect your warehouse](./connect-your-warehouse.md). The MCP tool reference is generated from the running server: see [`docs/tools.md` in the simba-mcp repository](https://github.com/getsimba-ai/simba-mcp/blob/main/docs/tools.md) for the exact parameters of `list_pipelines`, `list_pipeline_versions`, `get_recipe_draft_template`, `create_recipe_draft`, `publish_recipe_draft` and `create_model`.

## What a version pins

A version is saved once and never changes; running the pipeline again saves a new one with the next version number.

| Field | Meaning |
|---|---|
| `id` | The number you pass as `pipeline_version_id` |
| `version` | The version number within the pipeline, counting up from 1 |
| `created_at` | When the run saved it |
| `row_count`, `column_count` | The shape of the output |
| `columns` | The output column names, in order |
| `checksum` | SHA-256 of the output written as CSV |

The `checksum` is the identity a recipe draft records as its source `sha256`: the draft hashes the same stored bytes. A run started with `start_date` / `end_date` stores that range with the version's run parameters. Today those parameters are shown in the app's version detail only; the API listing returns the identity fields above and nothing else.

## List pipelines and their versions

```
GET /api/v1/pipelines?limit=50
GET /api/v1/pipelines/{pipeline_ref}/versions?limit=50
```

Over MCP:

```
list_pipelines()
list_pipeline_versions(pipeline_ref="ab12cd34ef", limit=50)
```

`pipeline_ref` is the pipeline's hash or id. Both calls need the `ingest` scope and list only the key owner's pipelines. Versions come newest first, identity only; the data itself is never in the listing. Paging is opt-in: pass `limit` (1 to 200) to receive a page and a `next_cursor`, and send the cursor back unchanged for the next page. A synthetic example:

```json
{
  "pipeline": {"id": 7, "pipeline_hash": "ab12cd34ef", "name": "Weekly sales"},
  "versions": [
    {"id": 212, "version": 3, "created_at": "2024-09-02T06:00:12", "row_count": 156, "column_count": 7,
     "columns": ["date", "brand", "revenue", "tv_spend", "tv_grps", "search_spend", "search_clicks"],
     "checksum": "9f2c…"}
  ],
  "next_cursor": null
}
```

## Fit a model directly on a version

Each version's output is also registered as a dataset in your workspace, named after the pipeline and the version number, for example `Weekly_sales_v3.csv`. `list_uploads` shows it with `source_type` `pipeline`, and the [data report](./report-sales-and-media-data.md) for that dataset returns its `pipeline_version_id`, so you can confirm which version a dataset is. Pass the dataset's id as `uploaded_file_id` to `create_model`:

```
create_model(uploaded_file_id=57, date_column="date", kpi_column="revenue", hierarchy_column="brand",
             channels=[{"name": "TV", "activity_column": "tv_grps", "spend_column": "tv_spend"},
                       {"name": "Search", "activity_column": "search_clicks", "spend_column": "search_spend"}])
```

The same request over HTTP:

```
POST /api/v1/models
{"data_source": {"uploaded_file_id": 57}, "date_column": "date", "kpi_column": "revenue",
 "hierarchy_column": "brand", "channels": [{"name": "TV", "activity_column": "tv_grps", "spend_column": "tv_spend"}]}
```

The fit is queued and the response carries a `model_hash`; poll `get_model_status` until it is complete. This needs the `create:models` scope. The model's configuration echo (`get_model`) records the dataset it read under `data_source`.

## Freeze the version in a study recipe

A recipe draft records the pipeline identity as well as the bytes, so a revision can later say which pipeline, version and hash it came from. Start from a template for the version:

```
GET /api/v1/recipe-draft-template?family=mmm&pipeline_version_id=212
```

Over MCP:

```
get_recipe_draft_template(family="mmm", pipeline_version_id=212)
```

The response is a complete draft `snapshot` plus a `template_hash` and the envelope schema. Choosing a version fills `snapshot.source` with the frozen bytes and an `origin` of `{"kind": "pipeline_version", "id": 212, "pipeline_id": 7, "version": 3, "sha256": "9f2c…"}`, and seeds the editable preview with the column headers and the first 50 rows. Give either `uploaded_file_id` or `pipeline_version_id`, never both. The template needs `read:models`; it creates, publishes and runs nothing, and its defaults are not a validated model.

Copy the snapshot, edit what you need and keep the rest, then save, publish and launch:

```
create_recipe_draft(study_id="…", draft_id="<uuid>", name="Weekly sales v3", snapshot={…})
publish_recipe_draft(draft_id="<uuid>", expected_version=1, publication_id="<uuid>", reason="Refresh on v3")
launch_study_run(study_id="…", revision_id="…", policy_id="…", submission_key="<uuid>")
```

Over HTTP the draft is `POST /api/v1/studies/{study_id}/recipe-drafts` with a body of `{"id": "<uuid>", "name": "…", "snapshot": {…}}` (201), and publication is `POST /api/v1/recipe-drafts/{draft_id}/publish` with `{"publication_id": "<uuid>", "expected_version": 1, "reason": "…"}`, which returns the new `revisions`. Saving re-checks the source bytes against the recorded origin and refuses a mismatch. Publication freezes the revision; it does not fit. The rest of the Studies flow (quality policies, launching, evaluation) is in [Simba MCP](./simba-mcp.md). After publishing, `get_recipe_revision` reports the recorded origin under `inspection.lineage`, with `available` telling you whether that version is still yours and still hashes the same.

## Which sources run today

| Source | In the standard build |
|---|---|
| CSV upload, PostgreSQL, Amazon Athena, S3 Iceberg | Yes; these run on the hosted service |
| Databricks, weather (Meteostat), economic indicators | Listed; each needs an optional driver a deployment can include |
| Snowflake | Listed; its driver is not part of the build today, so the source is hidden |

A source whose driver is missing is hidden from the builder and refused at preview and run time, with a message naming the driver. Everything a version pins, and everything on this page, is the same for every source: a version is the output rows, whatever produced them.

## Things to keep in mind

- **Quote the version id, not "latest".** A new run saves a new version; the one you built from never changes. The only thing that can remove a version is the scheduled prune below.
- **Scheduled refreshes prune old versions**, keeping the newest 30. A version recorded as the source of a model built in the app or of a recipe revision is never removed. Today `create_model` over the API records the dataset, not the version, so a model fitted that way does not protect its version: the fitted model and its registered dataset stay, but once the version is pruned the dataset's report shows `pipeline_version_id` as null. Runs you start yourself, or from the API, never prune. See [Refresh data on a schedule](./refresh-data-on-a-schedule.md).
- **A version without saved output cannot seed a draft.** The template answers 409 for it, and 404 for a version that is not yours.
- **Draft sources are capped at 10 MB** of stored CSV; a larger version output is refused by the template.
- **Scopes differ by step.** Listing needs `ingest`, the template needs `read:models`, and drafts, publication and `create_model` need `create:models`.
- **`create_model` records the dataset; a recipe records the pipeline.** Use the recipe route when you need the pipeline id, version and hash on the model's lineage.

## Next steps

- [Connect your warehouse](./connect-your-warehouse.md)
- [Refresh data on a schedule](./refresh-data-on-a-schedule.md)
- [Report sales and media data](./report-sales-and-media-data.md)
- [Simba MCP](./simba-mcp.md)
- [Model configuration](../platform-guide/model-configuration.md)
