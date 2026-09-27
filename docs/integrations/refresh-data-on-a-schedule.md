# Refresh data on a schedule

A pipeline can refresh itself daily or weekly, so the data your models are built on stays current without anyone pressing Run. Set up the pipeline first — see [Connect your warehouse](./connect-your-warehouse.md).

## Set a schedule in the app

In the pipeline builder, open **Schedule** in the header. Choose **Daily** or **Weekly** (and the day), then the time. Times are shown in your time zone, with the UTC hour alongside — the service schedules whole UTC hours. The popover shows when the next refresh will happen before you save.

Choose **Off** to pause a schedule; its settings are kept for when you turn it back on. The Schedule button always says what is set, for example "Daily · 07:00", and the Pipelines list shows each pipeline's schedule and how its last run went.

## Set a schedule from the API or MCP

Needs an API key with the `create:models` scope.

```
PUT /api/v1/pipelines/{pipeline_ref}/schedule
{"cadence": "weekly", "hour_utc": 6, "weekday": 0, "enabled": true}

GET /api/v1/pipelines/{pipeline_ref}/schedule
→ {"cadence": "weekly", "hour_utc": 6, "weekday": 0, "enabled": true, "next_run_at": "2026-10-05T06:00:00"}
```

Over MCP:

```
set_pipeline_schedule(pipeline_ref="ab12cd34ef", cadence="weekly", hour_utc=6, weekday=0, enabled=True)
```

| Field | Values |
|---|---|
| `cadence` | `daily` or `weekly` |
| `hour_utc` | A whole hour, 0–23, in UTC |
| `weekday` | 0 = Monday … 6 = Sunday; weekly only (omit it for daily) |
| `enabled` | `false` pauses the schedule and keeps its settings |

`next_run_at` is in UTC. A pipeline that has never been scheduled returns `enabled: false` and nulls.

## How scheduled runs behave

- **A scheduled run is an ordinary run.** It appears in the Runs menu marked *scheduled*, and you poll it like any other (`get_pipeline_run`). It runs as the pipeline's owner, with the pipeline's saved connections.
- **One run at a time.** If a run of the pipeline is still going when a slot comes round, that slot is skipped, not stacked.
- **Missed slots are not replayed.** If the service was unavailable over several slots, it runs once when it is back, then continues on schedule.
- **Failures are visible.** If a scheduled refresh fails, the pipeline says so when you next open it — what failed and how to fix it — and the previous version stays the current one.
- **Old versions are pruned after scheduled refreshes.** After each successful scheduled run, the pipeline keeps its newest 30 versions. A version that a model or a study recipe was built from is never removed. Runs you start yourself, or from the API, never remove anything.
- **A blocked account's schedules stop**, the same way its sign-in does.
