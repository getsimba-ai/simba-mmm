# Campaign budget recommendations

Campaign budget recommendations explore how to divide a channel budget between its mapped campaigns or ad sets. They combine [campaign incrementality](./campaign-incrementality.md) with the channel's [response curve](../core-concepts/saturation-curves.md). The result is a scenario for review. It does not change advertising-platform budgets, fit a new model or independently measure campaign saturation.

This contract requires a backend and MCP version that provide the campaign marginal and daily-budget routes below. A completed model alone does not establish compatible evidence.

## What the figures mean

| Figure | Meaning |
|---|---|
| Current average daily spend | Observed campaign spend divided by the inclusive calendar days in the evidence window. This is not the platform's configured budget. |
| Suggested daily budget | A flat daily equivalent that respects the channel total and campaign limits. |
| Marginal ROAS | Estimated extra response per extra unit of spend at the proposed allocation, using the inherited channel shape. |
| Incremental ROAS | Attributed incremental revenue divided by observed spend. An average return, distinct from marginal return. |
| Unavailable | The evidence or constraints cannot support this calculation. It does not mean zero return. |

The model's planning curve includes [carryover](../core-concepts/adstock-effects.md) after the spending period. Dividing a budget by days does not produce a forecast of revenue realised on each day. The feature does not choose different budgets for Mondays, weekends or individual future dates.

## Required evidence

Use a newly generated, completed MMM whose stored posterior mean curves include explicit currency and generation provenance. The model must represent revenue, either directly or through an explicit revenue multiplier. Older curves without this evidence remain unavailable; their units are not guessed or retrospectively certified.

The supported currency codes are GBP, USD, EUR, CAD, AUD, NZD, CHF, SGD, HKD, JPY, KWD, BHD and OMR. JPY uses zero decimal places; KWD, BHD and OMR use three; the others use two. Model currency and every relevant campaign-fact currency must match.

The backend checks the curve fingerprint, posterior mean statistic, transform order, reference convention and period basis. Daily and weekly models are supported. Weekly models must explicitly declare `period_date_semantics: period_start` when configured; a weekly date is not guessed to mean the start or end of its period. A daily row represents its date-only day. Other periods are refused, including monthly data where a fixed conversion of 30 days would be misleading. Log-link models require an explicit supported reference level.

[Campaign facts](./campaign-data.md) must cover the observation window and reconcile to model spend for every selected period:

- Supply complete, contiguous model periods, with date-only period-start anchors.
- Supply an explicit daily row for each participating fact identity, including zero-spend rows on inactive days. Missing dates are not treated as zero.
- Use consistent platform, account, campaign and ad-set identities. Do not mix campaign-total rows with their ad-set breakdown.
- Map each participating campaign to its intended model channel. Unmapped spend is reported separately.
- Every allocated item needs positive observed spend and positive relative efficiency. An ineligible mapped item makes its channel unavailable rather than silently redistributing its money.
- Platform value must be complete within the channel or absent throughout. Partially missing values refuse the calculation. When all are absent, the disclosed spend-share method gives each item the same relative efficiency.

Account IDs are needed for allocation identities even though the general campaign-facts import accepts them as optional.

## How the campaign curves are constructed

Let `R_ch(X)` be the stored channel response at model-period spend `X`. For campaign `c`, define:

```text
s_c = campaign observed spend / total mapped channel observed spend
e_c = campaign incremental ROAS / aggregate mapped-channel incremental ROAS

r_c(x) = e_c × s_c × R_ch(x / s_c)
m_c(x) = e_c × R'_ch(x / s_c)
```

The aggregate channel incremental ROAS is summed campaign incremental revenue divided by summed mapped spend, on the same window. Campaign efficiencies inherit the attribution assumptions, including any compatible test override. Brand and retargeting warnings still matter: inheriting a channel curve does not remove attribution bias.

Spend share rescales the horizontal axis. Relative efficiency rescales the marginal return. Increasing one campaign's budget therefore moves it along its inherited diminishing-return curve; the method does not hold its marginal return constant.

With a common complete attribution basis, `sum(s_c × e_c) = 1`. The constructed campaign responses sum to the channel curve at proportional allocation. This is a property of the construction, not evidence that the planning curve equals historical attributed revenue or that campaign causality has been measured.

For daily-equivalent budget `d` and `q` days per model period:

```text
q = 1 for daily models; q = 7 for weekly models

r_daily,c(d) = e_c × s_c × R_ch(q × d / s_c) / q
m_daily,c(d) = e_c × R'_ch(q × d / s_c)
```

The daily response expression is a unit conversion of the planning response, including its carryover convention. It is not a daily realised-revenue forecast.

## Bounds, allocation and rounding

The [allocation method](../core-concepts/budget-optimization.md) uses the shared bounded marginal-return allocator. The default maximum change is 20% above or below observed average daily spend. `max_step_fraction` can explicitly change this between zero and one. The interface offers 10% and 20% scenarios.

An item's lower bound combines its explicit minimum, change limit and supported curve minimum. Its upper bound combines its explicit maximum, change limit and supported curve maximum. Use an explicit minimum to represent a known platform or learning floor; Simba invents no such floor. The calculation stays within the stored curve domain and does not repair increasing marginal curves or extrapolate them.

Interior campaigns approach a common marginal return. Campaigns at bounds can have different marginal returns because the constraints prevent further movement. Flat segments and ties have deterministic treatment using stable identity order. If the total is infeasible, the response explains the refusal and supplies a feasible range where one exists. It does not silently relax limits or change the channel total.

Results retain both the continuous solution and the amount rounded to the declared currency minor unit. Rounded amounts reconcile exactly to the accepted total in integer minor units and stay inside bounds. Solver diagnostics report continuous and post-rounding residuals. Rounding does not imply exactly equal marginal returns. A total with unsupported decimal precision is refused.

### Synthetic example

Suppose two campaigns spend £600 and £400 per day, with relative efficiencies 1.05 and 0.925. Let the illustrative channel curve be `R(X) = 1000 × log(1 + X / 100)`.

Their marginal curves are `630 / (60 + x_A)` and `370 / (40 + x_B)`. For a total of £1,000, the continuous allocation is £633 and £367, and both marginal returns are approximately 0.909. Both allocations satisfy the default 20% change limits. A £1,300 total is unavailable because the combined upper limit is £1,200.

These are synthetic figures explaining the method, not a recommendation for an account.

## Read through the API or MCP

Both routes require `read:models` and enforce model ownership. Neither starts a fit, creates an optimiser run, changes mappings nor applies platform budgets. The marginal route reads point evidence without starting a posterior interval job.

```text
GET /api/v1/campaigns/marginal?model=<hash>&start=2026-09-01&end=2026-09-28&level=campaign
```

The response includes `context_key`, `window`, `level`, `currency`, `minor_digits`, `channels`, `rows`, `unmapped`, assumptions and provenance. Channel entries state `status` and a `reason` when unavailable. Ready rows include composite identity, observed daily spend, `miroas`, `method: channel_shape_scaled` and the spend basis.

Over MCP, use `get_campaign_marginal_returns(model_hash, start, end, level)`. MCP requires explicit start and end dates; the HTTP route can default to the model's observed period span.

To calculate with explicit daily channel totals:

```json
{
  "model": "example-model-hash",
  "observation_window": {"start": "2026-09-01", "end": "2026-09-28"},
  "level": "campaign",
  "currency": "GBP",
  "max_step_fraction": 0.2,
  "channel_daily_budgets": {"meta_activity": 1000},
  "bounds": [
    {"platform": "meta", "account_id": "example-account", "campaign_id": "example-campaign", "adset_id": null, "min": 480, "max": 720}
  ]
}
```

Send this body to `POST /api/v1/campaigns/daily-budgets`. Use the exact model channel key, not an assumed display name. Bounds use the full identity `(platform, account_id, campaign_id, adset_id)`; campaign-level bounds use null `adset_id`. Do not identify a campaign by name or campaign ID alone.

Pass the GET response's 64-character `context_key` as optional `expected_context_key` to calculate against the evidence reviewed. If the evidence changes, HTTP 409 with `code: campaign_context_changed` requests a refresh. Review the new scenario rather than blindly resubmitting without the key.

Alternatively, replace `channel_daily_budgets` with a positive integer `optimizer_run_id`. Exactly one budget source is required. The saved run must belong to the same owner and model, be complete and retain compatible currency, model identity, explicit planning dates and cadence. Channel `PeriodSpend` totals are divided by the actual planning calendar days, then rounded to the currency minor unit. Provenance retains the unrounded daily equivalents. The observation window and the saved run's future planning window are different concepts.

MCP uses `recommend_campaign_budgets` with the same fields, except `model_hash` replaces the HTTP body's `model`. The tools preserve the backend's refusals and provenance. See the [public MCP tool reference](https://github.com/getsimba-ai/simba-mcp/blob/main/docs/tools.md) for the installed contract.

### Reading the calculation response

Each requested channel returns a ready or unavailable result. Ready results include `total_daily_budget`, item `recommended_daily_budget`, `continuous_budget`, `current_daily_spend`, `delta`, `miroas_at_recommended`, binding constraints, explanation and assumptions. Provenance records the model curve basis, model results, fact versions and fingerprints, mapping fingerprint and attribution evidence. The request inputs are echoed.

| Refusal | Response |
|---|---|
| Malformed request, duplicate bounds or both budget sources | HTTP 400; correct the request rather than removing constraints silently. |
| Unknown or unowned model/run | HTTP 404. |
| Changed evidence context | HTTP 409, `campaign_context_changed`; refresh and review. |
| Unverified model currency | `currency_basis_unavailable`. |
| Saved run lacks a compatible basis | `run_basis_unavailable`. |
| Unsupported channel evidence or infeasible bounds | Channel `status: unavailable`, with a reason and a feasible range when meaningful. |

## Uncertainty and interpretation

The initial method returns `miroas_hdi: null` and `interval_status: unavailable`. Differentiating separate response-quantile curves does not produce a [posterior marginal credible interval](../core-concepts/bayesian-modeling.md). No interval is invented from those derivatives, and absence of an interval does not imply certainty.

Use the result to review a bounded reallocation under an explicit assumption that relative campaign efficiency persists as spend changes. Check attribution warnings and operational constraints before acting. A well-reconciled allocation is not proof of causal campaign effects or a guarantee of commercial uplift.

## Next steps

### Platform guides and integrations

- [Campaign data](./campaign-data.md): ingest daily facts and declare mappings.
- [Campaign incrementality](./campaign-incrementality.md): understand factors and attribution warnings.
- [Budget optimisation](../platform-guide/budget-optimization.md): plan channel-level budgets.
- [Simba MCP](./simba-mcp.md): connect an assistant to the shared API.

### Core concepts

- [Saturation curves](../core-concepts/saturation-curves.md)
- [Marginal-return optimisation](../core-concepts/budget-optimization.md)
- [Incrementality](../core-concepts/incrementality.md)
