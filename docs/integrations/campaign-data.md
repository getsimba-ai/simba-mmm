# Campaign data

Simba models at channel level. **Campaign data** brings the campaigns and ad sets behind each channel into Simba — spend, impressions, clicks, and the platforms' own conversions and value, one row per campaign per day — so campaign-level reports and, later, campaign-level incremental returns can read them. It arrives through a data pipeline you already have, and the channel each campaign counts towards is something you declare, never something Simba guesses.

## The shape

One row per campaign (or ad set) per day per platform account:

| Column | Required | Notes |
|---|---|---|
| `date` | yes | YYYY-MM-DD |
| `platform` | yes | `meta`, `google_ads`, `tiktok` or `other` |
| `campaign_id` | yes | the platform's id, as text |
| `spend` | yes | account currency, major units |
| `account_id` | | the platform's account id |
| `campaign_name`, `adset_id`, `adset_name` | | ad-set rows roll up to their campaign |
| `currency` | | ISO 4217; one per account |
| `impressions`, `clicks` | | counts |
| `platform_conversions`, `platform_value` | | the platform's attributed conversions and value |
| `last_click_conversions`, `last_click_value` | | from an analytics source such as a GA4 export, if you have one |
| `platform_channel_type` | | the platform's own label (`search`, `video`, `pmax`, `display`, `social`), used for suggestions only |

Column names are matched without regard to case, spaces or dashes. Unknown columns are ignored and listed; nothing is inferred. Rows that share a key (platform, account, campaign, ad set, day) are added together, so an ad-level file rolls up to its ad sets.

## Where it comes from

**From your warehouse.** The dbt project in the integration guide builds `campaign_daily_facts` in exactly this shape from Fivetran's Google Ads, Meta and TikTok tables, keeping ids, names, conversions and value, and labelling Google Ads campaigns by their type (search, video, Performance Max, display) instead of counting them all as search. Point a pipeline's warehouse source at that table.

**From a file.** Upload a CSV in this shape as a pipeline source. The Google Ads, Meta and TikTok exports need a little renaming; the required four columns are all a pipeline needs.

## Register the pipeline

Build the pipeline, run it, then on its Output card press **Use as campaign facts**. The dialog asks one thing: the **restatement window**, 28 days unless you choose otherwise. From then on every successful run of that pipeline refreshes the campaign facts: rows inside the window overwrite what was stored, a day the platform withdrew disappears, and older history is left alone. Platforms restate conversions for weeks, which is what the window is for.

The card says what the last ingest did. If a run's output lacks a required column, the pipeline run itself still succeeds and its version is saved, but the card and the run log say the facts were not ingested and why. **Stop** on the card ends the registration; facts already ingested are kept.

A scheduled pipeline keeps the facts fresh with no extra set-up — see [Refresh data on a schedule](./refresh-data-on-a-schedule.md).

## Map campaigns to channels

Open a saved MMM model and its **Campaigns** tab. Every campaign is listed with its spend in the window, the platform's own label, and a **Model channel** selector. Choose the channel each campaign counts towards and press **Save map**.

- **Declared, never inferred.** A campaign no row covers is *unmapped*: it stays listed with its spend and is not counted towards any channel. A suggestion drawn from the platform's label and the campaign's name may be offered; it counts for nothing until you choose it.
- **One channel per campaign per date.** Map rows can also match campaigns by a name pattern (`YouTube*`) and can carry dates, so a campaign that moved between channels is mapped for each period. A row for an exact campaign id beats any pattern. A map that would count one campaign towards two channels on the same dates is refused, and nothing is saved.
- **Drift is a warning.** The tab compares the spend of the campaigns mapped to each channel with the model's own spend for that channel over the dates both have. A gap beyond the warning threshold (5% unless you change it) is shown, and the map still saves. The usual causes are a campaign that belongs elsewhere, or model data that stops before the campaigns do.

The map belongs to the model, so a refit or a second model has its own.

## Reports

**In the app** the Campaigns tab shows the totals. **Over the API and MCP** the same facts answer questions such as "spend and conversions by campaign for August, weekly": `GET /api/v1/campaigns/report` and the MCP tool `get_campaign_report` run the data-report engine over the facts, grouped by platform, model channel, campaign or ad set, at daily, weekly, monthly, quarterly or yearly grain. Spend of unmapped campaigns appears as its own group, `unmapped`, and is never dropped. These are the platforms' attributed numbers, not Simba's incremental attribution.

```
GET /api/v1/campaigns?model=<hash>                       the campaigns and their channels under this model's map
GET /api/v1/campaigns/report?model=<hash>&group_by=channel&granularity=month&metrics=spend
PUT /api/v1/campaigns/map?model=<hash>                   {"rows": [{"platform": "google_ads", "campaign_id": "1107", "channel": "paid_search_spend"}]}
GET /api/v1/campaigns/map?model=<hash>                   the map, the unmapped campaigns, drift
```

Over MCP: `list_campaigns`, `get_campaign_report` and `set_campaign_mapping`. Registering a pipeline as the source stays in the app. Reads need an API key with `read:models`; the map needs `create:models`.

| Error code | Meaning |
|---|---|
| `campaign_facts_empty` | Nothing matches: no pipeline is registered as the source, or it has not run |
| `campaign_map_invalid` | A channel the model lacks, a malformed row, or a campaign that would count twice (`conflicts` lists them) |
| `campaign_source_invalid` | The registration body is wrong (the pipeline, or a restatement window outside 1–365 days) |

## Next steps

- [Connect your warehouse](./connect-your-warehouse.md) — build the pipeline that reads the campaign mart
- [Refresh data on a schedule](./refresh-data-on-a-schedule.md) — keep the facts current
- [Simba MCP](./simba-mcp.md) — the tools agents use for the same reads and the map
- [Incremental Measurement](../platform-guide/measurement.md) — what channel-level results mean
