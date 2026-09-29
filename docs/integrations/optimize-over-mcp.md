# Run the optimizer from an agent: constraints, baseline and the profit objective

An agent can run the budget optimizer on a completed model, constrain it per channel and per group of channels, choose revenue or profit as the objective, and read the result back as a table that also carries the historical plan it was compared against. Every run is saved with a stable `run_id`, so it can be listed, pinned and annotated later.

Everything on this page is available through the API and through [Simba MCP](./simba-mcp.md). The MCP tool reference is generated from the running server: see [`docs/tools.md` in the simba-mcp repository](https://github.com/getsimba-ai/simba-mcp/blob/main/docs/tools.md) for the exact parameters of `run_optimizer`, `get_optimizer_results`, `list_runs`, `update_run` and `set_run_pinned`. The wizard that does the same in the app is described in [Budget optimization](../platform-guide/budget-optimization.md).

## Start a run

Call `get_scenario_template` first: it returns the exact channel names the model uses (results are keyed by the activity column, for example `tv_activity`), the average cost per unit for `period_cpm`, and the model's stored `operating_margin` if it has one. Then start the run:

```
run_optimizer(model_hash="abc123", total_budget=500000, num_periods=4, gamma=0.05, currency="USD",
              bounds={"tv_activity": {"lower": 10, "upper": 50}, "search_activity": {"lower": 10, "upper": 50},
                      "social_activity": {"lower": 0, "upper": 30}},
              laydown_weights={"tv_activity": [1, 1, 1, 1], "search_activity": [1, 1, 1, 1], "social_activity": [1, 1, 1, 1]},
              period_cpm={"tv_activity": [20, 20, 20, 20], "search_activity": [2, 2, 2, 2], "social_activity": [8, 8, 8, 8]},
              group_bounds=[{"name": "digital", "channels": ["search_activity", "social_activity"], "lower": 40, "upper": 70}])
```

The same body goes to the API with a key that has the `optimize` scope. Add `objective` and `forward_margin` for a profit run:

```
POST /api/v1/models/{model_hash}/optimize
{"total_budget": 500000, "num_periods": 4, "gamma": 0.05, "currency": "USD",
 "bounds": {...}, "laydown_weights": {...}, "period_cpm": {...},
 "objective": "profit", "forward_margin": 0.3}
```

The run is queued and the response is `202`:

```json
{"model_hash": "abc123", "run_id": "opt_…", "optimizer_status": "pending", "message": "…"}
```

`total_budget`, `num_periods`, `gamma`, `currency`, `bounds`, `laydown_weights` and `period_cpm` are required. The same channel keys must appear in all three per-channel objects, `laydown_weights` and `period_cpm` are arrays of length `num_periods`, and `num_periods` is capped at 104 native periods. The model must be complete. An unknown channel name is refused and the error lists the available names.

## Per-channel and group bounds

You send bounds as percentages of `total_budget`; Simba converts them to currency before solving and stores the currency form on the run. A missing `lower` means 0% and a missing `upper` means 100%. Group bounds use the same percent convention and constrain the sum of the members' spend.

| Form | Where you see it | Example at a 500,000 budget |
|---|---|---|
| Per-channel percent (what you send) | `bounds` | `{"lower": 10, "upper": 50}` is 50,000 to 250,000 |
| Group percent (what you send) | `group_bounds[].lower` / `.upper` | `lower: 40, upper: 70` is 200,000 to 350,000 across the members |
| Per-channel currency (stored) | the run's `inputs.lower_bounds` / `inputs.upper_bounds` | `{"tv_activity": 50000}` / `{"tv_activity": 250000}` |
| Group currency (stored) | the run's `inputs.group_bounds` and the `GroupBounds` results column | `{"name": "digital", "channels": [...], "lower": 200000, "upper": 350000}` |

Groups must be disjoint, each group's range must overlap what its members' own bounds allow, and the effective minimums must fit the budget. Under the revenue objective the effective maximums must also reach the budget, because a revenue run spends all of it. A violation is refused before the run is queued, with the reason in the error.

Group bounds force the `slsqp` engine. If you asked for `optimizer_engine="marginal"`, the run still completes and an `EngineForcedReason` column says why. Results with groups gain a `GroupBoundsReport` column: for each group, `binding` is `"lower"`, `"upper"` or `null`, with the group's `total`, `lower` and `upper` in currency. Members of a binding group legitimately sit off the global marginal, because they share the group's own shadow price.

## Revenue or profit

`objective` defaults to `"revenue"`: the solver maximizes the risk-adjusted predicted revenue and spends exactly `total_budget`. With `objective="profit"` it maximizes margin-weighted response minus spend, and the budget becomes a ceiling: the solver may leave money unspent when no channel returns more than one unit of margin-weighted response per unit spent. Profit runs carry four extra columns: `Objective` (`"profit"`), `ForwardMargin` (the per-period margin list used), `UnspentBudget` (one value, repeated on every row) and `ExpectedProfit` (each period's predicted revenue times that period's margin, minus the channel's spend).

A profit run needs a margin. If the model was built with an operating margin, that stored margin (the trailing-year mean derived at fit time) is used for every period. Otherwise you must pass `forward_margin`, either one fraction in `(0, 1]` such as `0.3`, or a list of `num_periods` fractions for a margin that changes over the horizon. A model with no stored margin and no `forward_margin` is refused, and so is `forward_margin` on a revenue run. The resolved per-period list is stored in the run's `inputs.forward_margin`.

## What "baseline" means

The optimum is compared against a historical plan: the same `total_budget`, spread in the model's own historical spend mix. The mix is taken from the most recent `num_periods` rows of the model's training spend (the whole history if that window has no spend), across optimizable channels only, so a channel with a 0% upper bound gets no baseline spend. That plan is the `HistoricalSpend` column.

Both plans are then predicted through the fitted model under the same convention as the model's Contributions panel, which gives four comparison columns:

| Column | Meaning |
|---|---|
| `HistoricalSpend` | The baseline plan's spend per channel, in currency |
| `HistoricalRevenue` / `HistoricalROI` | The baseline plan's predicted revenue and revenue divided by spend |
| `OptimizedEvalRevenue` / `OptimizedEvalROI` | The optimized plan through the same pass; the revenue mirrors the table's `Revenue`, and ROI is `null` at zero spend |

On the `Total` row the revenues are sums and ROI is total revenue over total spend. `OptimizedEvalRevenue` minus `HistoricalRevenue` on that row is the `revenue_uplift_vs_historical` figure that `list_runs` reports in `key_metrics`. When the model has no spend history the baseline columns are absent and no comparison is invented.

## Read the results table

`results` is a list of one row per channel plus a final row whose `Channel` is `"Total"`. The per-period response and revenue placeholders (`PeriodResponse`, `PeriodRevenue` and their `Weekly` aliases) are dropped from a row when they are still all-null arrays.

| Column | What it holds |
|---|---|
| `OptimalSpend`, `SpendShare`, `PeriodSpend` | Allocated spend, its share of the allocated total (not of `total_budget`, which differs on an under-spending profit run), and the per-period laydown (length `num_periods`) |
| `ExpectedResponse`, `ResponseShare`, `Revenue`, `ROI` | The optimizer's decision math: response and revenue at the allocated spend |
| `Saturation`, `CPM` | Saturation level at the allocated spend, and the average of the `period_cpm` you sent |
| `ObjectiveMarginal`, `KktLambda`, `KktInteriorSpread` | The marginal return at the optimum per channel, the shadow price the solver equalizes, and the spread among freely funded channels |
| `ConvergenceWarning` | `null` on a clean run; otherwise a sentence saying why the allocation may be improvable |
| `MroiAtOptimized`, `MroiAtOptimizedHdi3`, `MroiAtOptimizedHdi97` | Posterior mROI at the optimized spend, with its 94% HDI (3% to 97%) |
| `MroiEvaluationSpend`, `MroiEvaluationPoint`, `MroiStatistic` | The per-period spend the mROI was quoted at (the mean over the active periods of the channel's laydown), the constant `"optimized_spend"`, and `"mean"` |
| `MroiProfitAtOptimized` (+ `Hdi3`, `Hdi97`) | Profit mROI, present only when the model has a stored operating margin |
| `ResponseCurve` | Per channel, a spend grid with predicted revenue and the operating point, `null` on the `Total` row and for a channel whose upper bound is 0% |

`Revenue` / `ROI` and the `HistoricalRevenue` / `OptimizedEvalRevenue` pair come from different conventions; do not mix them in one summary. `ObjectiveMarginal` and `MroiAtOptimized` are also different quantities and can differ by several times: the first is what the solver equalized, the second is the posterior mean of the per-draw marginal ROI at the mean spend of the active periods in the optimized laydown. The profit mROI columns use the model's stored margin and discount rate, never the run's `forward_margin`.

## Poll, list, pin and annotate runs

Poll the run you started, not the model, because a newer run overwrites the model-level state:

```
get_optimizer_results(model_hash="abc123", run_id="opt_…")
```

```
GET /api/v1/models/{model_hash}/optimize/runs/{run_id}
```

Either returns `status`, `inputs` and `results`; `results` stays `null` until `status` is `"complete"`. `list_runs(artifact="optimizer", model_hash="abc123")` returns run summaries ordered pinned first, then newest first; `limit` is clamped to 1 to 200 and `count` is the length of the page, so page until a short one. The objective is not in a summary: read the run's `inputs`, where profit runs carry `objective: "profit"` and revenue runs omit the key.

```
update_run(artifact="optimizer", model_hash="abc123", run_id="opt_…", name="Q4 digital 40-70%", tags=["q4"])
set_run_pinned(artifact="optimizer", model_hash="abc123", run_id="opt_…", pinned=True)
```

Over the API these are `PATCH /api/v1/models/{model_hash}/optimize/runs/{run_id}` with any of `name`, `notes`, `tags` (an unknown field is refused) and `POST .../runs/{run_id}/pin` with `{"pinned": true}`. Renaming a run permanently turns off its automatic naming. Passing `notes: ""` clears the notes. A pin request with no body toggles the state.

## Things to keep in mind

- **Percentages in, currency out.** `bounds` and `group_bounds` are percentages of `total_budget`; every stored and reported bound is in currency.
- **Profit needs a margin and may under-spend.** Check `UnspentBudget` on a profit run before reading the allocation as a full plan.
- **Profit is single-model today.** The portfolio optimizer supports the revenue objective only.
- **The baseline is a mix, not a history.** `HistoricalSpend` is your past spend shares applied to this run's budget, not your past spend totals.
- **Older runs may hold a median mROI.** A stored run without a `MroiStatistic` stamp was written before the mean convention; treat its values as posterior medians.
- **Runs count against your plan's optimization allowance,** and the read routes allow 60 requests a minute, so poll with a pause between calls.

## Next steps

- [Budget optimization](../platform-guide/budget-optimization.md) for the wizard, gamma and the results page in the app
- [Scenario planning](../platform-guide/scenario-planning.md) to test a plan before optimizing it
- [Exports and reporting](../platform-guide/exports-reporting.md) for the optimization CSV exports
- [Report sales and media data](./report-sales-and-media-data.md) to check the spend history the baseline is built from
- [Optimization concepts](../core-concepts/budget-optimization.md) for the theory behind the objective
