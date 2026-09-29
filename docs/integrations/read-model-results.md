# Read model results: actual KPI, spend and media units per period

A fitted model stores its results as named sections, and you read only the sections you ask for. Three of them are per-period tables: `actual_vs_model` (the KPI and the fitted value for each period), `coefficients` (spend, media units, sales and revenue per channel per period) and `contributions` (the decomposition of each period's KPI).

Both the API and [Simba MCP](./simba-mcp.md) serve the same sections from the same stored results. The MCP tool reference is generated from the running server: see [`docs/tools.md` in the simba-mcp repository](https://github.com/getsimba-ai/simba-mcp/blob/main/docs/tools.md) for the exact parameters of `get_model_results` and the full list of sections it describes.

## Ask for the sections you need

```
GET /api/v1/models/{model_hash}/results?sections=actual_vs_model,coefficients,contributions
```

Over MCP:

```
get_model_results(model_hash="f835671a25", sections="actual_vs_model,coefficients,contributions")
```

`sections` is a comma-separated list; leave it empty for the default payload. Results are served only for a model whose status is `complete`; any other status is refused with an error naming the status. The key or token needs the `read:results` scope.

The response is an envelope around the sections:

```json
{
  "model_hash": "f835671a25",
  "name": "Brand A weekly",
  "status": "complete",
  "model_type": "mmm",
  "periodicity": "weekly",
  "sections_available": ["actual_vs_model", "coefficients", "contributions"],
  "results": {"actual_vs_model": [], "coefficients": [], "contributions": []}
}
```

`sections_available` lists the sections in this response, plus the opt-in sections this model can serve that the payload omits (`mroi_periods` when the model stores it, `prediction_window` when a saved prediction window exists). Trust it over any list written in documentation.

## Actual KPI per period: `actual_vs_model`

One row per period, in KPI units. `fit actual` is the KPI the model was fitted to and `model` is the mean of the posterior predictive draws. The four `hdi_` fields are percentile bands of those draws for that period: the 25th to 75th percentile (`hdi_50_`) and the 2.5th to 97.5th percentile (`hdi_95_`).

| Field | Meaning |
|---|---|
| `date` | Period date, as a millisecond epoch integer |
| `fit actual` | Actual KPI for the period |
| `model` | Mean predicted KPI |
| `hdi_50_lower`, `hdi_50_upper` | 50% prediction band |
| `hdi_95_lower`, `hdi_95_upper` | 95% prediction band |

```json
{"date": 1722816000000, "fit actual": 85000, "model": 84000,
 "hdi_50_lower": 82000, "hdi_50_upper": 86000, "hdi_95_lower": 78000, "hdi_95_upper": 90000}
```

## Spend and media units per period: `coefficients`

One row per channel per period. This is the only per-period table in revenue space: `Sales` is the channel's contribution in KPI units, and `Revenue` is that contribution priced with the period's own multiplier. `Media Units` is the channel's activity value for that period, in the units of its activity column.

| Field | Meaning |
|---|---|
| `Date` | Period date, as a millisecond epoch integer |
| `Channel` | The channel's activity column name, for example `tv_activity` |
| `Sales`, `Revenue`, `Spend`, `Media Units` | Per-period totals |
| `ROI` | `Revenue / Spend` |
| `Cost per Media Unit`, `Revenue per Media Unit`, `Sales per Media Unit` | Per-unit ratios for the period |
| `Profit` | `Revenue` times the period's margin; present only on models with a margin |

```json
{"Date": 1722816000000, "Channel": "tv_activity", "Sales": 4000, "Revenue": 40000, "Spend": 12000,
 "Media Units": 500, "ROI": 3.333333, "Cost per Media Unit": 24, "Revenue per Media Unit": 80, "Sales per Media Unit": 8}
```

On models with a margin, `channel_summary` rows also gain `Profit`, `Net Profit` and `Profit ROI`.

## Decomposition per period: `contributions`

One row per period, in KPI units; the multiplier is not applied. Each row has `Date`, one column per channel and per control (keyed by column name), and the components below. Channels, controls and components add up to `Model`.

| Column | Present when |
|---|---|
| `Base` | Always |
| `Seasonality` | The model has seasonality |
| `Event Effect` | The model has event dummies |
| `Overlap` | Multiplicative model fitted with the `removal_lift` attribution convention. A negative reconciliation term so that `Base + components + Overlap = Model`; it is not a channel |
| `Model`, `Fit Actual`, `Actual` | Always |

```json
{"Date": 1722816000000, "tv_activity": 4000, "search_activity": 3000, "price": -1000,
 "Seasonality": 2000, "Base": 76000, "Model": 84000, "Fit Actual": 85000, "Actual": 85000}
```

Under the `aumann_shapley`, `shapley` and `proportional_normalized` conventions the pieces sum exactly and there is no `Overlap` column, so detect it by column presence and read the convention from `model_config.attribution_convention`. Control columns are measured against the reference point resolved at fit time, so a control's series can legitimately span zero.

## Coefficients and convergence: `posterior`, `r_hat`, `model_stats`

`posterior` has one row per fitted coefficient (each channel and control in the model's prior table) with `Variable`, `mean`, `sd`, `hdi_3%`, `hdi_97%` and `r_hat`. The interval is the 94% highest density interval (3% to 97%); quote it next to the mean. The same columns are shown on the [Coefficients tab](../platform-guide/measurement.md#coefficients).

`r_hat` has one row per parameter, `{"Parameter", "R_hat"}`, over all posterior variables, including transform parameters such as `tv_activity_decay` that the `posterior` rows do not cover; multi-dimensional variables are labelled with their coordinate, and when a model has more parameters than the stored table holds, the rows with the highest R-hat are kept. `model_stats` is a list of rows with `Test Name`, `Output`, `Evaluation` and a `Status` flag; its `Max R_hat` row reads "Model has converged." when the maximum is 1.2 or below, and "Model has not converged, simplify the model." above 1.2. Use `r_hat` to find which parameter block is responsible.

## Join on channel names

Per-period sections are keyed by the channel's activity column name (`tv_activity`), not the clean channel name (`TV`). The `channel_map` section is the join key: one record per channel with `channel`, `activity_column` and `spend_column`. `channel_summary` rows carry the legacy `Channel` field (activity column) plus `channel` (clean name) and `activity_column`. Matching is case-sensitive and space-sensitive.

Over MCP, `channels=["TV"]` filters `coefficients`, `channel_summary`, `mroi_summary`, `mroi_periods`, the curve sections, `decay_curves` and `saturation`; that match is case- and space-insensitive and tolerates the `_activity` and `_spend` suffixes. `contributions` is never filtered, because its control columns cannot be told apart from channels client-side.

## Download everything as CSV

```
GET /api/v1/models/{model_hash}/results?format=csv
```

The body is one `# section` header followed by a CSV block per section, blocks separated by a blank line, served as a file named `{model_hash}_results.csv`. Empty sections are skipped, a section that is a single object becomes a one-row block, and nested sections are split into several blocks (`cohort_ledger` into `# cohort_ledger_metadata`, `# cohort_ledger_rows` and `# cohort_ledger_channel_summary`; `mroi_periods` into its rows). Over MCP, `get_model_results(model_hash="f835671a25", format="csv")` returns `{"format": "csv", "content": "..."}` with the same blocks; the `channels` and grid filters apply to JSON only.

## Group drivers for the contributions view

`PUT /api/v1/models/{model_hash}/contribution-groups` stores the driver grouping that the [Contributions tab](../platform-guide/measurement.md#custom-contribution-groups) renders, and `GET` on the same path reads it back. Each group has a `name`, a list of `drivers` (media, control, halo or trademark column names), an optional `color` and optional `baseAdjustments` (`min`, `max` or `none`, keyed by the group's own drivers). Writing needs `create:models` and reading needs `read:results`. A driver name the model does not have is refused, with a did-you-mean hint when a close match exists; a driver may belong to at most one group; and the stored grouping is validated at write time only, so a grouping saved from the dashboard is read back as stored.

## Things to keep in mind

- **Results cover the model's training period.** `start`, `end` and `granularity` cut the per-period sections to a window and recompute `channel_summary`; that behaviour is described in [Cut model results to a window](./report-sales-and-media-data.md#cut-model-results-to-a-window). Bucketed rows carry `period_start` and `period_end` instead of `Date`, bucketed `coefficients` rows add `Periods` (native periods in the bucket), and bucketed `actual_vs_model` rows drop the `hdi_` bands.
- **Dates in the per-period sections are millisecond epoch integers**, not strings.
- **Contributions are in KPI units; revenue lives in `coefficients`.** Do not multiply a contribution by a single multiplier yourself: each period has its own.
- **`Overlap` is not a channel.** Never rank it, share it, or feed it to the optimizer or a scenario.
- **Profit fields exist only on models with a margin.** Their absence means no margin source, not zero profit.
- **A full pull is large.** Curve sections alone are 100 grid points per channel with five band columns. Over MCP, request only the sections you need, pass `channels` and `max_grid_points`, and set `max_response_bytes` to get an error instead of partial evidence; see [Retrieve only the evidence you need](./simba-mcp.md#retrieve-only-the-evidence-you-need).
- **Some sections are opt-in.** `mroi_periods` and `prediction_window` are never in the default payload; name them in `sections`. Serving `prediction_window` for a study-linked model whose saved window is available records an access event, which is why the generated reference marks `get_model_results` as a tool that writes. `mroi_periods` on a model fitted before that artifact existed comes back as `{"available": false}` with a reason; refit to enable it.
- **VAR models carry only `actual_vs_model` per period.** Their other sections are returned unchanged when a window is given.

## Next steps

- [Report sales and media data](./report-sales-and-media-data.md) for dataset reports and date windows on results
- [Simba MCP](./simba-mcp.md) to connect a client and choose scopes
- [Measurement and diagnostics](../platform-guide/measurement.md) for the same sections in the dashboard
- [Exports and reporting](../platform-guide/exports-reporting.md) for CSV and PDF downloads from the UI
- [Budget optimization](../platform-guide/budget-optimization.md) to act on the channel keys you read here
