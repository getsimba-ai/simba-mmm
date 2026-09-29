# Incrementality tests

Simba keeps a record of every incrementality test a project has run: geo tests, owned-media A/B tests (a leaflet or email split, for example) and platform lift studies such as a Meta Conversion Lift. Each record holds what was tested, when, what it found and what it cost, and a completed test can calibrate any model in the project. The calibration row is derived from the record against the chosen model, and every step of that derivation is shown.

Tests are analysed in your own tool. Simba imports the result; it does not run geo or lift analyses itself.

You can work with tests in the app (**Warehouse → Experiments → Incrementality tests**, and the model wizard), through the API, and through [Simba MCP](../integrations/simba-mcp.md). The MCP tool reference is generated from the running server: see [`docs/tools.md` in the simba-mcp repository](https://github.com/getsimba-ai/simba-mcp/blob/main/docs/tools.md) for the exact parameters of `list_incrementality_tests`, `get_incrementality_test`, `create_incrementality_test`, `import_incrementality_tests` and `create_model`.

## What a test record holds

| Field | What it is |
|---|---|
| `type` | `geo`, `owned_media_ab` or `platform_lift`, each with its own block: treatment and control regions and the method for a geo test; the medium, unit and arm sizes for an owned-media split; the platform, study and cell for a lift study |
| `name`, `status` | `planned`, `running`, `completed` or `invalid`. Only completed tests calibrate a model |
| `channel` | The channel as the business names it, e.g. "Paid Social" |
| `model_channel` | The model column the test calibrates, e.g. `meta_impressions`. Optional; you can choose the column when you use the test |
| `kpi` | `revenue`, or an outcome by name such as `orders` or `conversions` |
| `start_date`, `end_date`, `measured_through` | The treatment window, and optionally the end of the carryover window the result was measured through |
| `result` | `lift_abs`, the total incremental outcome over the test and its carryover; an interval with its `low`, `high`, `level` and `sides`; the tool's own `sd` if it reports one; whether the interval is on the response or on incremental ROAS; a `p_value` for display |
| `spend` | The incremental spend and its currency. Needed to calibrate |
| `source` | The tool the result came from, with the tool's own fields kept verbatim |

**Null and negative results are valid records.** A test that found nothing is still a test, and the list shows it like any other. A result reported only as a percentage is refused at import, because a calibration row needs the absolute incremental outcome.

Every edit creates a new version of the record. A model that used the test records which version it used.

## Record and import tests

**Warehouse → Experiments → Incrementality tests** lists a project's tests with their type, channel, dates, result and status, and how many model revisions each one calibrated. Anyone who can read the project can read its tests; adding, editing and deleting need project ownership.

**Add test** records one test by hand, with a form per type.

**Import** reads another tool's output. Every import starts as a dry run that shows each row and any problem with it. Fields the file doesn't carry, such as the channel a GeoX result belongs to, are filled in under *Applies to every test*, and single rows can be corrected before the import creates them.

| Source | What you upload | What you add |
|---|---|---|
| Meta Conversion Lift | The study's results JSON from the Conversion Lift API. One record per cell, dated by the study's active period and measured through its observation end | Nothing for a conversions model. A revenue model asks you to confirm the KPI |
| Google GeoX (meridian-geox) | The analysis result JSON. The lift, its interval and the tool's standard deviation are read per cell | The channel, the KPI and the incremental spend per cell |
| Meta GeoLift (R) | The summary as JSON or CSV, with the object's totals. Run it with `ConfidenceIntervals = TRUE`, otherwise the interval is missing and the import says so | The incremental spend |
| CausalPy | The effect summary's cumulative row, or the lift rows its MMM helper produces | The incremental spend for a summary. Lift rows already state the calibration row and are used as given |
| pymc-marketing | Lift rows | Used as given |
| CSV template | One row per test, in a template you can download from the import dialog | Nothing |

Results from any other tool come in through the CSV template. Files up to 10 MB.

## Use a test with a model

Each test's page has **Use with a model**. Pick a saved model and Simba shows the calibration row the test gives that model, with the steps that produced it, or the reason it can't be used.

In the model wizard, the lift-test section has **Add from a recorded test**. Picked tests appear as read-only rows under the grid with their derived values for the model you are building. Only references to the tests are stored; the row is derived again when the model is fitted, from the model's own data.

Lift tests enter the model as **likelihood observations**, not priors: see [Incrementality](../core-concepts/incrementality.md) for the methodology.

### How the calibration row is derived

The row is `{channel, x, delta_x, delta_y, sigma}`: the channel's baseline level per period, the change in level per period, the change in outcome per period, and the uncertainty of that change. Levels are in the channel's own units, so a channel measured in impressions gets a row in impressions, not in currency.

1. **Status.** Only a completed test calibrates.
2. **Channel.** The model column is the record's `model_channel`, or the column you choose. It must be a media channel the model can calibrate.
3. **KPI and units.** A revenue result gives a row in revenue units. An outcome result must be the model's own KPI; otherwise the test is refused, unless you confirm that the outcome really is the model's KPI. A confirmation is written into the steps.
4. **Periods.** `T` is the number of model periods (weeks or days) that overlap the test window, through `measured_through` when it is set.
5. **Outcome per period.** `delta_y = lift_abs ÷ T`. Dividing by the test's own periods, carryover included, gives the steady-state per-period effect the model compares against.
6. **Uncertainty.** With `z` for the interval's level (1.645 for 90%):
   - a lopsided interval, where one side is more than 1.5 times the other, becomes a two-sided uncertainty (`sigma_low` and `sigma_high`) solved so that each tail of the observation holds its share of the interval's mass; the reported shape is kept rather than averaged away;
   - otherwise the tool's own standard deviation, when it reports one;
   - otherwise `sigma = (high − low) ÷ (2 · z)`, the normal approximation for recovering a standard error from a confidence interval (Cochrane Handbook for Systematic Reviews of Interventions, section 6.5.2.2);
   - a one-sided interval, or no interval and no standard deviation, is refused.
   An interval reported on incremental ROAS is multiplied by the incremental spend first. Each result is then divided by `T`.
7. **Spend per period** is the incremental spend divided by `T`.
8. **Cost per unit** is the channel's spend divided by its media units over the baseline window. It is 1 for a channel whose activity is its spend.
9. **Levels.** `delta_x` is the spend per period divided by the cost per unit. For a geo test, `x` is the channel's average level over the `T` periods before the test started. For a platform lift study `x` is 0, because the study compares ads on with ads off.
10. **Sign.** A lift that moves against the change in activity is kept and reported, but refused for calibration: the model's response curve can't produce it.
11. **Holdout.** Inside a study, a test that ran during the validation holdout is refused, so held-out data can't leak into the fit.

A worked example, with synthetic numbers. A five-week geo test on a spend channel found a lift of 50,000 with a 90% interval of 20,000 to 80,000, on 25,000 of incremental spend. For a weekly model whose baseline over the five weeks before the test averaged 9,062 a week, the row is:

| `x` | `delta_x` | `delta_y` | `sigma` |
|---|---|---|---|
| 9,062 | 5,000 | 10,000 | 3,648 |

`delta_y` is 50,000 ÷ 5. `sigma` is (80,000 − 20,000) ÷ (2 × 1.645) ÷ 5. Both `delta_x` and `x` are in the channel's units, which for a spend channel are currency.

### Reasons a test can't be used

| Reason | Message |
|---|---|
| `test_not_completed` | This test is planned or running. Only completed tests calibrate a model. |
| `owned_media_not_calibratable` | Owned-media tests are recorded and reported, but can't calibrate a model yet. |
| `channel_not_in_model` | This model has no calibratable media channel for the test. |
| `kpi_mismatch` | The test measured a different outcome from this model's KPI. |
| `no_spend` | No incremental spend is recorded. Add it to use the test with a model. |
| `ci_one_sided` | The interval is one-sided, so its uncertainty can't be read. Re-run the analysis two-sided. |
| `no_interval` | No interval or standard deviation was recorded. |
| `sign_conflict` | The lift moves against the change in activity. The model's response curve can't produce that. |
| `window_overlaps_holdout` | The test ran during this study's validation holdout, so using it would leak held-out data. |
| `units_conflict` | Tests picked together must be in the same units (revenue or the model's outcome). |

A refusal is an answer, not an error: the test stays recorded, and the message says what would change the outcome. Two warnings can ride along with a row: a partial-geo test applied to a national model, and a baseline taken from the training window because the data starts after the test's baseline period.

## Over the API and MCP

| Route | What it does |
|---|---|
| `GET /api/v1/projects/{project_id}/incrementality-tests` | List a project's tests, filtered by `type`, `status` or `channel`, paged with `limit` and `cursor` |
| `POST /api/v1/projects/{project_id}/incrementality-tests` | Record one test. Returns the record with its version and content hash |
| `POST /api/v1/projects/{project_id}/incrementality-tests/import` | Import a file: `source`, `content`, `dry_run` (default true), `defaults` and per-row `overrides` |
| `GET /api/v1/incrementality-tests/{id}` | One test, current or a given `version`, with what used it |
| `GET /api/v1/incrementality-tests/{id}/calibration?model_hash=…` | The row the test gives that model, `{status: "ok", row, units, steps, warnings}`, or `{status: "refused", reason, message, steps}`. `channel` picks the model column and `confirm_kpi=true` confirms the outcome |

Reads need the `read:models` scope and writes `create:models`. Over MCP the same operations are `list_incrementality_tests`, `create_incrementality_test`, `import_incrementality_tests` and `get_incrementality_test`, which returns the calibration row alongside the record when given a `model_hash`:

```
get_incrementality_test(test_id="…", model_hash="…")
```

To fit a calibrated model, pass `calibration` to `create_model` in one of two forms:

```json
{"calibration": {"tests": [{"test_id": "…", "channel": "tv_grps", "confirm_kpi": false}]}}
```

```json
{"calibration": {"units": "revenue", "observations": [{"channel": "tv_grps", "x": 9062, "delta_x": 5000, "delta_y": 10000, "sigma": 3648}]}}
```

Recorded tests are derived against the request's own data. If any of them can't calibrate the model, the request fails with `calibration_refused` and the reason per test, and nothing is created. A fitted model says what calibrated it: reading the model returns `model_config.calibration` with the rows, their units and the test versions used.

Study recipe drafts can carry the same test references; they are derived when the draft is published, and the revision records the tests it used.

## Things to keep in mind

- **Analysis stays in your tool.** Simba records and uses results; it doesn't design or run tests.
- **Owned-media tests are reported, not yet calibrated.** They need a channel measured in activity rather than spend.
- **A one-sided interval can't calibrate.** Re-run the analysis two-sided, or record the tool's standard deviation.
- **Deleting is blocked while a study uses the test.** Study revisions and published drafts keep a link to the test versions they used, so their inputs can always be reproduced. A model fitted directly through `create_model` keeps no link back, but does record what calibrated it.
- **The wizard's "Auto-calculate all uncertainties" button is a placeholder.** It sets each row's uncertainty to 25% of its response change. A recorded test's interval is the better source.
- **Limits.** Import files up to 10 MB; up to 50 recorded tests, or 200 rows given directly, per model.
