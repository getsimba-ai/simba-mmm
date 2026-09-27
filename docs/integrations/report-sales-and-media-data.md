# Report sales and media data

Simba can report the data you uploaded, not only a fitted model's results. You can ask for any date window, at any grain, and by brand, channel or market. For example: "store sales and TV spend in the North region for August, by week." Fitted model results can also be cut to a date window. Their channel summary is recalculated for that window, not filtered.

Both features are available through the API and through [Simba MCP](./simba-mcp.md). The MCP tool reference is generated from the running server: see [`docs/tools.md` in the simba-mcp repository](https://github.com/getsimba-ai/simba-mcp/blob/main/docs/tools.md) for the exact parameters of `get_data_report`, `upload_data` and `get_model_results`.

## Declare what each column is

A report needs to know what each column means: the KPI, a brand, a spend line, a price. **Simba never guesses this from a column name.** You declare column roles once, when you upload the data, or with each report request.

Only three names are recognised without a declaration, because the data schema itself defines them:

- a column named `date`;
- `{channel}_spend`, which is spend for that channel;
- `{channel}_activity`, which is activity for that channel.

Every other column reports as `unknown` and is not aggregated until you declare it.

| Role | How it aggregates over a period | Unit |
|---|---|---|
| `kpi` | sum | your KPI |
| `spend`, `activity` | sum, per channel | currency / count |
| `outcome:online_sales`, `outcome:store_sales`, `outcome:margin` | sum | currency |
| `outcome:orders`, `outcome:new_customers` | sum | count |
| `media:impressions`, `media:clicks`, `media:grps` | sum, per channel | count / GRPs |
| `control:price` | mean | currency |
| `control:rate`, `control:index` | mean | ratio / index |
| `control:stock` | each brand's last value in the period, summed across brands | count |
| `multiplier` | mean | ratio |
| `hierarchy`, `dimension:market`, `dimension:product`, `dimension:campaign` | keys for filtering and grouping | — |

A declaration is either a role name, or a role plus a channel for media columns:

```json
{
  "revenue": "kpi",
  "brand": "hierarchy",
  "region": "dimension:market",
  "tv_grps": {"role": "media:grps", "channel": "tv"},
  "avg_price": "control:price",
  "orders": "outcome:orders"
}
```

Pass it as `roles` when you upload, for example `upload_data(roles={...})` over MCP. You can also pass it as `roles` on a report request to override what was stored. An unknown role, or a column the file doesn't have, is refused.

## Ask for a report

```
GET /api/v1/datasets/{dataset_id}/report?start=2024-08-01&end=2024-08-31&granularity=week&group_by=channel&metrics=kpi,spend
```

Over MCP:

```
get_data_report(dataset_id=42, start="2024-08-01", end="2024-08-31",
                granularity="week", group_by="channel", metrics=["kpi", "spend"])
```

| Parameter | Values |
|---|---|
| `start`, `end` | Dates as `YYYY-MM-DD`, inclusive |
| `granularity` | `native` (the rows as stored), `week`, `month` or `quarter` |
| `group_by` | `hierarchy`, `channel`, or a dimension role such as `dimension:market` |
| `hierarchy` | Keep one brand or region |
| `metrics` | Roles or role families: `kpi`, `spend`, `outcome`, `outcome:orders`, `control`, … By default, every metric role is included |

**Periods:**
- `week` is the ISO week starting Monday.
- `month` and `quarter` are calendar periods.
- A row belongs to the period of its own date, so a weekly row counts in the month its week starts.

An example response, with a synthetic dataset and round numbers:

```json
{
  "dataset": {"id": 42, "source": "upload", "sha256": "…", "data_through": "2024-12-30"},
  "granularity": "week",
  "rows": [
    {"period_start": "2024-08-05", "period_end": "2024-08-11", "group": "tv", "metric": "spend", "value": 12000, "unit": "currency"},
    {"period_start": "2024-08-05", "period_end": "2024-08-11", "group": "all", "metric": "kpi", "value": 85000, "unit": "kpi"}
  ],
  "meta": {"basis": "dataset", "aggregation": {"spend": "sum", "kpi": "sum", "calendar": "…"}, "roles": {"…": "…"}}
}
```

What the response tells you:
- **`data_through`** is the last date in the dataset, so you can see how fresh it is. It doesn't depend on the window you asked for.
- **`sha256`** identifies the exact file the numbers came from.
- **`meta.aggregation`** states every rule that was applied.

A report is capped at 10,000 rows. Past that you get an error: narrow the window or use a coarser grain.

## Cut model results to a window

`get_model_results`, and the API's model results, accept the same `start`, `end` and `granularity`. They window the per-period sections: `contributions`, the per-period media table (`coefficients`) and `actual_vs_model`.

The channel summary is **recalculated** for the window:

- **ROI is total revenue divided by total spend** over the window, per channel. It is never an average of weekly ROIs, which would weight a quiet week the same as a heavy one. Revenue comes from each period's own revenue figure.
- **Profit uses each period's own margin** before summing. A margin that changed mid-year is respected.
- **Per-unit figures**, such as cost per media unit, are recalculated from totals.
- **Marginal ROI is never added up.** The per-period marginal ROI series comes back exactly as fitted.
- **Prediction intervals** on actual-versus-model are per-period percentiles and cannot be added. When periods are grouped they are dropped; at native granularity they are kept.

The response gains a `meta` block with the window, the basis (each period's fitted contribution), `data_through` and the rules applied.

## Things to keep in mind

- **Model results cover the model's training period.** For dates outside it, or columns the model didn't use, report on the dataset instead.
- **Contributions are in KPI units.** Revenue figures come from the per-period media table, where each period's multiplier has already been applied.
- **Refitting a model can restate history.** Quote the model you read from, and the `data_through` you saw.
