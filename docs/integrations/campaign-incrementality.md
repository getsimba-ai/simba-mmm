# Campaign incrementality

Simba measures incrementality at **channel** level. **Campaign incrementality** pushes that result down to the campaigns and ad sets behind each channel, so an incremental ROAS sits beside the platform's own ROAS and last-click ROAS for every campaign, on one window and one currency. It needs [campaign data](./campaign-data.md) and a saved campaign map.

## The assumption, first

For each model channel over a window, Simba computes an **incrementality factor**:

```
factor(channel) = the model's incremental revenue for the channel
                  ÷ the platform-attributed value of the campaigns mapped to it
```

Each campaign's incremental ROAS is **that factor × its own platform ROAS**, so campaign incremental revenue always adds up to the channel's. The platform supplies the split between campaigns; the model supplies the level.

**This assumes the platform over-credits every campaign in a channel equally. It does not.** Platforms over-credit retargeting and brand search more than prospecting, so a single factor flatters them: a brand campaign's incremental ROAS here is optimistic. Simba warns when brand or retargeting campaigns (by name) share a channel with others, and every row says how it was made. Nothing on this page is a causal per-campaign measurement.

Three things make the number honest where it matters:

1. **Split the channel.** Map brand or retargeting campaigns to their own model channel (the model needs that channel as a media column). Each then gets its own factor.
2. **Calibrate with a test.** A completed [incrementality test](../platform-guide/incrementality-tests.md) that names the channel and overlaps the window replaces the model's factor with the test's: its measured lift over the channel's platform value during the test. The model's factor stays beside it.
3. **Read the labels.** Every row carries `method` and `factor_source`; the factors card says where each channel's number came from and what to doubt.

## A worked example

A four-week window. Paid Search earned 120,000 of incremental revenue in the model. Three campaigns are mapped to it:

| Campaign | Spend | Platform value | Platform ROAS | Incremental revenue | Incremental ROAS |
|---|---|---|---|---|---|
| Brand exact | 20,000 | 100,000 | 5.00 | 60,000 | 3.00 |
| Generic kitchen | 30,000 | 60,000 | 2.00 | 36,000 | 1.20 |
| Performance Max | 10,000 | 40,000 | 4.00 | 24,000 | 2.40 |
| **Channel** | 60,000 | 200,000 | | **120,000** | |

The factor is 120,000 ÷ 200,000 = 0.60. Brand exact keeps the highest incremental ROAS because the single factor flatters it; that is exactly the warning the page shows.

## When there is no platform value

A channel whose campaigns carry no platform value (a CSV without conversions, or a platform that reports none) has no split to scale. Its incremental revenue is shared across its campaigns **by spend**, labelled `spend share` on every row; add `platform_value` to the pipeline's output for the attribution-scaled numbers. A campaign without platform value in a channel that has some gets no incremental ROAS and is named; it is never given a share.

## Credible intervals

The 94% HDI (3%–97%) on each factor comes from the model's posterior draws. The first time a window is asked for, a background job summarises the draws for that window; until then the page says the intervals are on their way and shows the point estimates, then refreshes itself when they arrive. A model whose draws cannot be loaded says so and shows point estimates only. No band is ever estimated in their place. When a channel's interval spans zero the factor is marked **uninformative**: the model is not sure the channel is incremental at all, and a per-campaign number for it says little.

## In the app

Open a saved MMM model, its **Campaigns** tab, and map campaigns to channels as in [campaign data](./campaign-data.md). Once a campaign counts to a channel the tab shows:

- three columns per campaign: **Incremental ROAS** (with its interval and, under it, the method and the factor used), **Platform ROAS** and **Last-click ROAS**; hover a column heading for what it means;
- a **Channel factors** card: the model's revenue, the platform value, the factor, its interval, its source (the model, or a named test) and notes in words;
- a grain selector for **ad sets**, which inherit their campaign's channel and factor and add up to it;
- the warnings in words: the campaigns one factor flatters, intervals on their way, no interval, spend share, a test override, an uninformative factor.

An unmapped campaign still shows its platform and last-click ROAS, with "map it to a channel to get one" where the incremental ROAS would be.

## Over the API and MCP

```
GET /api/v1/campaigns/incrementality?model=<hash>&start=2026-09-01&end=2026-09-28&level=campaign
```

`level` is `campaign` (default) or `adset`. Without `start` and `end` the window is the overlap of the model's data and the campaign facts. The response carries `window`, `currency`, `interval` (`pending`, `ready` or `unavailable`, with `interval_reason`), one entry per channel (`factor`, `factor_source`, `factor_mmm`, `factor_interval`, `method`, `mmm_revenue`, `platform_value`, `spend`, `test`, `warnings`), one row per campaign or ad set (`spend`, `platform_value`, `last_click_value`, `platform_roas`, `last_click_roas`, `incremental_revenue`, `iroas`, their 94% bands, `method`, `factor_source`), the `unmapped` campaigns with their spend, the `warnings`, and `provenance` (the facts versions, when they were ingested, the map version and the model's attribution convention).

Over MCP the tool is `get_campaign_incrementality(model_hash, start, end, level)`; it reads the same route and needs an API key with `read:models`.

| Warning code | Meaning |
|---|---|
| `retargeting_shares_channel_factor` | brand or retargeting campaigns share a channel's factor with prospecting; one factor flatters them |
| `platform_value_missing` | the channel is on spend share, or named campaigns carry no platform value |
| `kpi_not_revenue` | the model's KPI is valued through a multiplier; incremental ROAS is in that currency |
| `currency_mismatch` | the facts carry more than one currency |
| `uninformative` | the channel factor's interval spans zero |
| `unmapped_spend` | spend in the window counts to no channel |
| `interval_unavailable` | the model's draws could not be loaded (the reason is given) |
| `test_override_skipped` | a test named the channel but could not be used (the reason is given) |

| Error code | Meaning |
|---|---|
| `campaign_facts_empty` | no campaign facts, or none in the window |
| `invalid_window` | an empty or reversed window; the body gives the model's and the facts' spans |
| `model_not_mmm` | a VAR model has no per-channel revenue rows |
| `model_incomplete` | the model has not finished fitting |

## Next steps

- [Campaign budget recommendations](./campaign-budget-recommendations.md): explore a bounded allocation using inherited channel response shapes.

- [Campaign data](./campaign-data.md) — bring the campaigns in and map them to channels
- [Incrementality tests](../platform-guide/incrementality-tests.md) — record a test that can calibrate a channel's factor
- [Simba MCP](./simba-mcp.md) — the tools agents use for the same reads
