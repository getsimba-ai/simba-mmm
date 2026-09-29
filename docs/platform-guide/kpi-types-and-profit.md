# KPI types and profit outputs

A Simba model fits whichever KPI column you point it at: revenue, units, orders, sign-ups or a share. The likelihood you choose tells the model what kind of number that KPI is, and an operating margin turns the revenue the model attributes to media into profit.

Both settings are available through the API and through [Simba MCP](../integrations/simba-mcp.md). The MCP tool reference is generated from the running server: see [`docs/tools.md` in the simba-mcp repository](https://github.com/getsimba-ai/simba-mcp/blob/main/docs/tools.md) for the exact parameters of `create_model`, `get_model_results` and `run_optimizer`.

## Choose a likelihood for your KPI

The `likelihood` parameter names the observation model. Names are case-insensitive.

| `likelihood` | Use it for | What the model does |
|---|---|---|
| `normal` (default) | A continuous KPI such as revenue or volume | Fits a Normal observation and learns its noise scale |
| `studentt` | A continuous KPI with outliers | Fits a Student-t observation and learns its tail weight and noise scale |
| `lognormal` | A strictly positive KPI with right skew | Fits a log-normal observation; a zero or negative KPI value is refused at build time |
| `logit` | A KPI that is a fraction between 0 and 1 | Fits a logit-normal observation |
| `poisson` | A count KPI such as orders or sign-ups | Fits a Poisson observation |
| `negativebinomial` | A count KPI with more spread than Poisson allows | Fits a negative binomial observation and learns the dispersion |
| `quantile` | Not available today | The name is accepted, but the fit stops with an error asking for normal, studentt or lognormal |

Over MCP, with a count KPI and a price column so that revenue is in currency:

```
create_model(uploaded_file_id=42, date_column="date", kpi_column="orders",
             hierarchy_column="brand", multiplier_column="avg_order_value",
             channels=[{"name": "TV", "activity_column": "tv_grps", "spend_column": "tv_spend"},
                       {"name": "Search", "activity_column": "search_clicks", "spend_column": "search_spend"}],
             likelihood="negativebinomial", operating_margin=0.30)
```

The same request over HTTP puts the likelihood inside `config` and the margin at the top level:

```
POST /api/v1/models
{
  "data_source": {"uploaded_file_id": 42},
  "date_column": "date", "kpi_column": "orders", "hierarchy_column": "brand",
  "multiplier_column": "avg_order_value",
  "channels": [{"name": "TV", "activity_column": "tv_grps", "spend_column": "tv_spend"},
               {"name": "Search", "activity_column": "search_clicks", "spend_column": "search_spend"}],
  "config": {"likelihood": "negativebinomial"},
  "operating_margin": 0.30
}
```

Contributions stay in KPI units. Revenue in the per-period media table is the KPI contribution times the multiplier column, so give a price or value per unit when the KPI is a count. A multiplicative model (`link="log"`) accepts only `lognormal`, `normal` and `studentt`, and defaults to `lognormal` when you leave the likelihood out. In the app the Likelihood Function select on the Model Details step offers normal, lognormal and studentt (and quantile on an additive model); logit, poisson and negativebinomial are set through the API or MCP today. See [Model Configuration](./model-configuration.md).

## Give the model an operating margin

A margin is a fraction of revenue in (0, 1]. You can supply it in two ways, and the API refuses a request that uses both:

| Parameter | Shape | Rule |
|---|---|---|
| `operating_margin` | One number, for example `0.30` | Applied to every date in the dataset |
| `operating_margin_column` | Name of a column in the uploaded CSV | Values must be uniformly fractions in (0, 1] or uniformly percentages in (1, 100]; percentages are normalised; mixed units are refused |

In the app you pick the margin column in Variable Selection; yearly margins can simply repeat across each year's rows.

At fit time the model stores the per-date series and derives one scalar from it: the mean of the most recent year of entries (52 weekly, 12 monthly or 365 daily periods). The series prices historical periods; the scalar is the forward margin for planning. The margin does not change the fit. It is applied when results are served.

## Where profit appears

Every profit figure is revenue times margin. Marginless models omit these fields entirely rather than assuming a margin of 1.

| Where | Field | Definition |
|---|---|---|
| `get_model_results` section `financials` | `operating_margin`, `operating_margin_series` | The stored scalar and the date-keyed series `{"2024-01-01": 0.30, ...}` |
| `channel_summary` rows | `Profit` | Sum over the per-period rows of Revenue_t x margin_t when `coefficients` is in the same response; otherwise Revenue times the forward scalar |
| `channel_summary` rows | `Net Profit` | Profit minus Spend |
| `channel_summary` rows | `Profit ROI` | Profit divided by Spend; null when spend is zero |
| `coefficients` rows (per period) | `Profit` | Revenue_t x margin_t, using the series entry for that date |
| `mroi_summary` (margin models) | `mroi_profit_median`, `mroi_profit_mean`, `mroi_profit_hdi_3`, `mroi_profit_hdi_97`, `pv_kernel_mass_*` | Marginal revenue x margin x PV kernel mass, computed per posterior draw with the forward scalar |
| Optimizer results with `objective="profit"` | `ExpectedProfit`, `ForwardMargin`, `UnspentBudget`, `Objective` | ExpectedProfit is the sum over planning periods of Revenue_t x margin_t minus OptimalSpend; `Revenue`, `ROI` and `ExpectedResponse` stay on the revenue basis |
| `get_scenario_template` | `operating_margin` | The stored scalar, for your own profit math on scenarios |
| App, Media Results tab | Profit ROI, Overall Profit ROI | ROI times the effective margin, applied at display time |

```
get_model_results(model_hash="abc123", sections="financials,channel_summary")
```

```json
{
  "financials": {"operating_margin": 0.30, "operating_margin_series": {"2024-01-01": 0.30, "2024-01-08": 0.30}},
  "channel_summary": [
    {"Channel": "tv_grps", "Spend": 120000, "Revenue": 480000, "ROI": 4.0,
     "Profit": 144000, "Net Profit": 24000, "Profit ROI": 1.2}
  ]
}
```

With only these two sections requested, Profit is Revenue times the forward scalar; add `coefficients` to the request to price each period with its own margin. Over HTTP the same call is `GET /api/v1/models/{model_hash}/results?sections=financials,channel_summary`.

## Optimize for profit

The optimizer's default objective is revenue. With `objective="profit"` it maximises the margin-weighted response minus spend, risk-adjusted by `gamma` in the same way as the revenue objective. A stored margin is used automatically; otherwise pass `forward_margin`, or the run is refused.

```
run_optimizer(model_hash="abc123", total_budget=1000000, num_periods=4, gamma=0.0, currency="USD",
              bounds={"tv_grps": {"lower": 5, "upper": 60}, "search_clicks": {"lower": 5, "upper": 60}},
              laydown_weights={"tv_grps": [1, 1, 1, 1], "search_clicks": [1, 1, 1, 1]},
              period_cpm={"tv_grps": [10, 10, 10, 10], "search_clicks": [2, 2, 2, 2]},
              objective="profit", forward_margin=0.30)
```

- `forward_margin` overrides the stored margin and is only accepted with `objective="profit"`; a revenue run that includes it is refused.
- Over MCP `forward_margin` is one number. The HTTP endpoint also accepts a list with one margin per planning period, for seasonal margin assumptions.
- Carryover that lands after the last planning period is priced at the last planning period's margin.
- Under the profit objective budget can stay unspent when the marginal profit of the next unit is negative.
- The profit objective is single-model only today; a portfolio run that asks for it is refused.

The wizard steps and result columns are covered in [Budget Optimization](./budget-optimization.md).

## Things to keep in mind

- **The margin keys live at the request root.** A margin placed inside `config` is refused: the request fails with an error naming the unknown config key. Put `operating_margin` or `operating_margin_column` next to `kpi_column`, not inside `config`.
- **Per-date lookup is step-forward.** A period uses the most recent series entry on or before its date; dates before the first entry use the first entry.
- **Windowed results keep per-period pricing.** Profit is priced with each period's own margin before summing, and Net Profit and Profit ROI come from the sums. See [Report sales and media data](../integrations/report-sales-and-media-data.md).
- **Marginal ROI uses the forward scalar.** The profit basis of `mroi_summary` applies the trailing-year scalar, not the per-date series, and the profit mean is taken from the profit draws rather than as a product of means.
- **`annual_discount_rate` is display-time only.** It sets the per-period discount factor for the profit-basis marginal ROI and the cohort ledger. At a rate of 0 the PV kernel mass is exactly 1 and profit reduces to revenue times margin.
- **Count likelihoods keep the additive model form.** `poisson` and `negativebinomial` cannot be paired with `link="log"` today.

## Next steps

- [Incremental Measurement](./measurement.md) for the Media Results tab where Profit ROI is shown.
- [Budget Optimization](./budget-optimization.md) for the optimizer wizard and result columns.
- [Scenario Planning](./scenario-planning.md) for forward predictions; the scenario template returns the stored margin.
- [Model Configuration](./model-configuration.md) for the Model Details step where the likelihood is set.
- [Simba MCP](../integrations/simba-mcp.md) to run these calls from an assistant.
