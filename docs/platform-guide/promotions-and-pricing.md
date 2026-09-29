# Promotions and pricing as controls: reading their contributions

Promotions, price and distribution are not media, but they move the KPI, so you give them to the model as control columns. Each control gets its own per-period contribution in the results, and on a multiplicative model you choose the level that contribution is measured against.

Controls are available through the API and through [Simba MCP](../integrations/simba-mcp.md). The MCP tool reference is generated from the running server: see [`docs/tools.md` in the simba-mcp repository](https://github.com/getsimba-ai/simba-mcp/blob/main/docs/tools.md) for the exact parameters of `create_model` (`control_columns`, `control_priors`, `control_reference`, `link`, `attribution`) and `get_model_results`.

## Add promotions, price and distribution as controls

Name the columns in `control_columns`. A control column must be unique and cannot be a media, KPI, date or hierarchy column. Prepare the columns as described in [Data preparation](../data/data-preparation.md).

```
create_model(uploaded_file_id=42, date_column="date", kpi_column="units", hierarchy_column="brand",
             channels=[{"name": "TV", "activity_column": "tv_grps", "spend_column": "tv_spend"},
                       {"name": "Search", "activity_column": "search_clicks", "spend_column": "search_spend"}],
             control_columns=["price_index", "promo_flag", "distribution"],
             link="log",
             control_reference={"_default": "auto", "price_index": "average", "promo_flag": "absent"})
```

Over HTTP, `control_columns` and `control_priors` sit at the root of the request body and `control_reference`, `link` and `attribution` sit inside `config`:

```
POST /api/v1/models
{
  "uploaded_file_id": 42, "date_column": "date", "kpi_column": "units", "hierarchy_column": "brand",
  "channels": [{"name": "TV", "activity_column": "tv_grps", "spend_column": "tv_spend"}],
  "control_columns": ["price_index", "promo_flag", "distribution"],
  "control_priors": [{"control": "price_index", "transform": "LOG"}],
  "config": {"link": "log", "control_reference": {"_default": "auto"}}
}
```

With `LOG` on the price, `auto` resolves it to `absent`, because the transformed column is already centred on its mean (see the reference rule below). In the dashboard, controls are chosen in the model creation wizard and their priors are edited in the Prior Builder; see [Model configuration](./model-configuration.md).

## Choose a transform for each control

Each `control_priors` entry names one selected control with `control`, plus any of `transform`, `distribution` (`normal`, `inversegamma`, `truncatednormal`, `halfnormal`), `mean`, `sd`, `lower` and `upper`. Priors are in transformed units, so changing the transform does not convert a coefficient you already set.

| `transform` | The model sees | Refused when |
|---|---|---|
| `N` | the raw values | the column is not finite numeric data (this applies to every transform) |
| `DM` | value divided by the column mean | the mean is zero |
| `STA` | value divided by the sample standard deviation | the standard deviation is zero |
| `DDM` | value divided by the mean KPI | the mean KPI is not positive |
| `LOG` | log of value divided by the column mean | any value is zero or negative |

The "Refused when" checks run before the fit for every control that has a `control_priors` entry; a boolean column is refused there too. The `LOG` positivity check also runs on every fit, whichever way the transform was set.

`LOG` is the transform to use for a price series: with the KPI on its mean-divided scale, the coefficient reads as a constant elasticity, and the contribution is zero in a period where price sits at its mean. A 0/1 promotion flag cannot take `LOG` because of the zeros; `N` keeps it as a flag.

## How control contributions are reported

The `contributions` section of `get_model_results` returns one row per period with one column per control (keyed by the column name) and per media channel (keyed by the activity column), plus `Base`, `Seasonality` and `Event Effect` when the model has them, `Model`, `Fit Actual` and `Actual`.

```
get_model_results(model_hash="abc123", sections="contributions,model_config")
```

A row from a synthetic weekly model, in KPI units, with round numbers:

```json
{"Date": 1722816000000, "tv_grps": 9000, "search_clicks": 4000, "price_index": -3000, "promo_flag": 6000,
 "distribution": 2000, "Base": 60000, "Overlap": -1000, "Model": 77000, "Fit Actual": 78000, "Actual": 78000}
```

What the row tells you:
- **`Date`** is a millisecond epoch integer at native granularity; rows bucketed to week, month or quarter carry `period_start` and `period_end` instead.
- **Control contributions are in KPI units today.** The multiplier is not applied to them, and the per-period revenue table (`coefficients`) covers media channels only, so there is no per-period revenue figure for a control.
- **A control's contribution can be negative**, and once a reference level is set it legitimately runs either side of zero.
- **`Overlap`** appears only on a multiplicative model fitted under the `removal_lift` convention; see below.

`start`, `end` and `granularity` window this section together with `coefficients`, `actual_vs_model` and `channel_summary`; the other sections come back as fitted and are listed in `meta.not_windowed`. See [Report sales and media data](../integrations/report-sales-and-media-data.md).

## Choose the reference level: `control_reference`

On a multiplicative model (`link="log"`), every contribution answers "what would the model predict if this component were removed". For media, removed means zero spend, which is a real scenario. For a price index that only ranges between 0.95 and 1.16, or a distribution level, zero is far outside the data, and measuring against it produces contributions that dwarf the outcome and a negative `Base`. `control_reference` sets, per control, the level the removal is measured against.

| Mode | "Remove it" means | Use when |
|---|---|---|
| `absent` | the control at zero | zero is observed: promotion flags, events |
| `average` | the control at its observed mean | zero is far outside the data: price, distribution |
| `lowest` / `highest` | the control at its lowest / highest observed level | you want a deliberate scenario framing |
| `auto` | detected at fit time | you want the rule below applied per control |

The `auto` rule measures how far zero sits from the observed range of the transformed column the model fitted: the smallest absolute value divided by the range. Below 0.5 it resolves to `absent`, otherwise to `average`. A constant non-zero column resolves to `average`. A `LOG` control is already centred on its mean, so it resolves to `absent`; do not set `average` on top of it, which would centre it twice.

How the mapping is read:
- Omit `control_reference` and every control stays at `absent`.
- Provide it, and any control you do not name falls to `_default`, which itself defaults to `auto`.
- Unknown control names and unknown modes are refused. Any value other than `absent` requires `link="log"`; an additive model's contributions are exact and take no reference.

The fit reports what it resolved in `model_config.control_references`, one entry per control with `requested`, `resolved`, `zero_distance` (`null` for a constant non-zero column, whose distance is unbounded) and the posterior-mean `q_ref`:

```json
{"price_index": {"requested": "average", "resolved": "average", "zero_distance": 5.0, "q_ref": -0.02},
 "promo_flag": {"requested": "absent", "resolved": "absent", "zero_distance": 0.0, "q_ref": 0.0},
 "distribution": {"requested": "auto", "resolved": "average", "zero_distance": 4.0, "q_ref": 0.30}}
```

Mechanically, the referenced control's log-scale effect is measured as its effect minus its effect at the reference level, and the reference level folds into `Base`, which stays positive under every mode. `Model` is unchanged and the media columns are unchanged. In the dashboard the same choice is made per control under Model Details, Advanced Options, with the labels "Auto (detect at fit)", "vs. none / absent", "vs. average conditions", "vs. lowest observed" and "vs. highest observed"; the wizard shows an indicative detection from the upload preview and the fit recomputes it on the full dataset.

## The `Overlap` column on a multiplicative model

In a multiplicative model the components multiply, so removing them one at a time counts each shared lift more than once. Under the `removal_lift` convention (the API default) the `contributions` section carries an `Overlap` column that closes the gap: `Base` + every component + `Overlap` = `Model`, exactly, in every period. `Overlap` is not a channel: never rank it, share it or feed it to the optimizer or a scenario.

One period, with `Base` = 100, TV at a log-scale effect of 0.40 and a promotion at 0.20:

| Quantity | TV | Promotion | Sum |
|---|---|---|---|
| Model | | | 182.21 |
| Joint effect (Model minus Base) | | | 82.21 |
| `removal_lift` contribution | 60.07 | 33.03 | 93.10 |
| `Overlap` | | | -10.89 |

Under `aumann_shapley` (the dashboard default for a multiplicative model), `shapley` or `proportional_normalized`, the shared lift is allocated across the components and the columns sum to `Model` with no `Overlap` column; its absence does not mean the model is additive. An additive model (`link="identity"`) has exact, additive contributions and no convention applies. The convention is chosen at fit time in `attribution` and reported in `model_config`.

## Things to keep in mind

- **A control cannot be named** `Base`, `Model`, `Overlap`, `Seasonality`, `Event Effect`, `Fit Actual`, `Actual` or `Date` (exact, case-sensitive). The fit is refused before sampling; a lowercase variant is accepted.
- **The reference level is fixed at fit time.** Changing it, like changing the convention, needs a refit. Scenario runs reuse the fit-time reference; `average`, `lowest` and `highest` mean training-window levels, never levels from the scenario data.
- **A centred series that crosses zero auto-detects `absent`.** Name it explicitly if you want `average`.
- **`proportional_normalized` can refuse a fit** whose per-period share denominator changes sign or cancels. The error names the remedy: set `control_reference` (for example `{"_default": "auto"}`) or refit under `removal_lift`, `shapley` or `aumann_shapley`, which have no share denominator.
- **Refitting can restate history.** Quote the model you read from.

## Next steps

- [Data preparation](../data/data-preparation.md): cleaning and formatting control columns before upload.
- [Model configuration](./model-configuration.md): the Prior Builder and control priors in the dashboard.
- [Incremental measurement](./measurement.md): the Contributions tab and the other results tabs.
- [Scenario planning](./scenario-planning.md): what-if runs that reuse the fitted reference levels.
- [Report sales and media data](../integrations/report-sales-and-media-data.md): windowing results by date and grain.
