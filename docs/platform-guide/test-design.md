# Design an incrementality test

Simba can propose an incrementality experiment from a saved media mix model: pause or change one channel's spend, everywhere or in some markets, and learn before the test starts how large an effect it could detect, how likely it is to detect the effect the model expects, and how long it should run. The design is calculated by replaying the model's saved posterior. Nothing else is touched: no experiment is run, no model is changed and no budget moves. A design you accept can be saved as a planned test in the project's [incrementality tests](./incrementality-tests.md), where its result is recorded once the measurement window has closed.

In the app the dialog opens from **What to test next** on a saved model's **Media Results** tab (see [What to test next](../integrations/experiment-priorities.md)), one channel at a time. Set-up takes three choices and a date; alpha, target power, the baseline and a named effect wait behind **Advanced settings**. The same calculation is available over the API and [Simba MCP](../integrations/simba-mcp.md).

Every name and number on this page is synthetic.

## What a test design is

A design keeps three quantities apart. They are never shown as one figure, because people confuse them.

| Quantity | What it is | What it rests on |
|---|---|---|
| **What the model expects** (the model-implied effect) | The cumulative effect the saved model implies the intervention would have over the measurement window, with a 94% HDI (3% to 97%) | The saved posterior draws. Parameter uncertainty only. It is not a prediction of what the experiment will measure |
| **The smallest effect the test can detect** (the detectable effect) | The smallest true cumulative effect that the pre-specified analysis would call, at the chosen alpha, with the target power | The noise of the comparison, the duration, alpha and the target power. It does not depend on how much spend changes |
| **The chance of detecting it** (power) | The probability that the pre-specified analysis rejects the null at a given effect: at the model's expected effect, at an effect you name, and averaged over the model's whole uncertainty (assurance) | The detectable effect and the size of the effect in question |

Around them the design fixes the intervention itself: the channel, the spend change (a pause, a signed percentage against a baseline, or an explicit schedule), the start date, the duration selected from the candidates you asked for, the carryover tail the model's adstock implies, the measurement window (intervention plus tail), the signed spend change in currency, and the analysis that will be run when the test ends.

Effects are reported on four estimands and each result says which it uses:

| Estimand | Definition | Units |
|---|---|---|
| Cumulative | The sum over the measurement window of the KPI under the intervention minus the KPI under the baseline | KPI units |
| Mean per period | The cumulative effect divided by the number of periods in the window | KPI units per period |
| Relative | The cumulative effect divided by the counterfactual KPI summed over the window | ratio |
| iROAS | The cumulative effect in revenue units divided by the absolute spend change. For a pause it is reported as KPI lost per unit of spend saved, a positive number, and stated as such | ratio; only when the KPI is revenue or the model stamps a scale to revenue |

A worked example. A weekly revenue model has 156 weeks of history ending 1 November 2026. You ask what pausing TV for 4, 6, 8 or 12 weeks from 2 November would show, at alpha 0.10 two-sided and 80% target power, naming a cumulative effect of £150,000 you would care about. The design that comes back selects 8 weeks:

| | Value | Caption |
|---|---|---|
| What the model expects | minus £310,000 | 94% HDI minus £520,000 to minus £120,000 |
| Smallest effect this test can detect | £286,000 | 3.1% of the window, £26,000 a week |
| Chance of detecting it | 85% | assurance 74%; 37% at your named £150,000 |

The pause runs 2 November to 27 December, is measured through 17 January 2027 (3 weeks of carryover) and saves £240,000 of spend. The 8-week duration is selected because it is the shortest candidate whose power at the model's expected effect reaches 80%; 4 weeks reaches 64% and 6 weeks 76%.

## Supported models

The designer works from what a saved model's artefact already holds, so it supports the models whose forward effect can be replayed exactly without rebuilding the model graph:

- a media mix model with saved posterior draws (a VAR model has no channel response to replay);
- the identity link, which is the default Model Form; a model with no link stamped is treated as identity;
- no time-varying coefficient on the chosen channel;
- a Normal likelihood, or a StudentT likelihood whose tail is heavy enough to have a finite variance (the 3% HDI bound of its degrees of freedom must exceed 2.5; draws at 2 or below are excluded and counted, and the model is refused if they are more than 1% of the draws);
- the chosen channel among the model's nonlinear media channels, with its adstock, saturation and transform settings resolvable, and not on the `LOG` variable transform;
- a cost per unit that can be resolved from the training data (a spend channel's cost per unit is 1), or a schedule given in activity units;
- a start date at least one period after the model's last training date and at most 13 weekly, 91 daily or 3 monthly periods after it.

Both transform orders, the three adstock types and the four saturation types are supported.

Everything else is refused with a reason, never with a number. The log link is outside the supported designs because under it the channel's effect depends on the level of every other term over the test window. Time-varying coefficients, LogNormal (a log-link likelihood), heavy-tailed StudentT fits and prior-only fits are refused in the same way. When a model is refused the app shows one calm card with the title **This model is outside the supported designs**, one sentence and one next step. For a log-link model the sentence is "This model uses the log link. The designer currently supports identity-link models." and the next step is "Fit an identity-link model on the same data, save it, and design the test from there. This model's results are unaffected." A model that is in the supported family but lacks something the calculation needs (carryover kernels, for example) gets the title **This model can't support a time-holdout design yet** instead, with the sentence for the missing piece; the codes and their sentences are listed under [What you get back](#what-you-get-back).

Before anything is calculated, the **What to test next** card already knows whether a model passes these checks and says so under each channel's **Design a test** button.

## How the model-implied effect is computed

The model-implied effect is a paired replay of the saved posterior. For every posterior draw, the channel's contribution is replayed twice over the training history plus the measurement window: once with the baseline spend schedule and once with the intervention schedule. Both replays use the same draw's parameters and the same warm-up history, so the adstock state at the first intervention period is identical in both arms and the only difference is the spend change itself. The per-period differences are summed over the measurement window, giving one cumulative effect per draw. The result reports the mean over draws and the 94% HDI (3% to 97%), labelled "model-implied effect, 94% HDI, parameter uncertainty only".

Under the identity link that difference depends only on the channel's own terms, which is what makes the replay exact: the saved adstock, [saturation](../core-concepts/saturation-curves.md) and coefficient draws are evaluated with the model's own transforms, in the model's own order. Spend is converted to activity at the training-window mean cost per unit, then into model space through the stored variable-transform factor; a spend channel short-circuits to a cost per unit of 1. KPI units are recovered by reversing the outcome scaling, and revenue for future periods uses the model's stamped scale to revenue. The result records which conversions applied as assumption codes.

Nothing about the other channels, the controls or the baseline changes between the two arms. The effect is what the model implies this intervention would do, averaged over its own parameter uncertainty. It is not a forecast of what the experiment will measure, because the experiment also sees observation noise and whatever the model has wrong; those enter the detectable effect and power instead.

## How detectable effect and power are computed

### The pre-specified analysis

The design states in advance how the test will be analysed, and the detectable effect and power are properties of that analysis, not of some better one that might be chosen later. For a time holdout the counterfactual is the model's forecast of the KPI with the baseline schedule for the channel. The analysis is: observed KPI minus the posterior-mean counterfactual forecast, summed over the measurement window, tested against zero using the standard error fixed at design time, not one estimated from the test data. The result names it as `forecast_contrast_v1`.

### The noise basis

The noise is the posterior-predictive standard deviation of that summed contrast under the null. It has two parts, and where they come from depends on the evidence the model has:

1. **Observation noise**: the model's own per-period observation variance, the posterior mean over draws of the likelihood's sigma in KPI units (for a StudentT likelihood, scaled by its degrees of freedom to a variance). In-sample residuals are never used for this.
2. **Forecast-level uncertainty**: how far the model's own forecast level could be off over a window of that length.

**Held-out evidence.** When the model has a saved [held-out window](./validation-and-holdout.md) with actual KPI, posterior-mean prediction and per-period predictive SD, strictly after the training window, at the model's cadence, with no gaps and at least 8 periods, the design uses it. The forecast-level term is the held-out predictive variance over and above the observation variance, treated as a common shift across the window. A **calibration ratio** compares the held-out forecast errors with what the model claimed they would be; a ratio above 1 inflates the noise, a ratio below 1 is never allowed to shrink it. The review step says "Noise is measured on 13 validation weeks the model did not fit." and the Evidence section labels the basis as held out (level A). Reading the held-out window counts as a use of it under the holdout governance described on the validation page.

**Model-conditional evidence.** When there is no held-out window the design still comes back, labelled **model-conditional** (level B). The forecast-level term is then the posterior variance of the model's own fitted level over the most recent training periods of the same length, an in-sample proxy. For a model with a Gaussian-process baseline that proxy is a **lower bound**, because the baseline's uncertainty grows out of sample, so every such result carries the warnings `no_out_of_sample_calibration` and `forecast_uncertainty_lower_bound`, and the review step says "Noise is the model's own estimate; no held-out window was available to check it." No calibration ratio is computed, because there is nothing independent to calibrate against. A model-conditional design cannot detect misspecification: if the model is wrong, the design is wrong in the same direction.

The cumulative standard error is the calibration ratio (floored at 1) times the square root of the observation variance, summed over the window with its autocorrelation, plus the forecast-level variance. In the worked example the observation sigma is £21,000 a week, the forecast-level term is £48,000 cumulative SD (in-sample level proxy, lower bound) and the cumulative SE over the 11-week window is £115,000.

### Autocorrelation

Period-to-period noise in a KPI series is rarely independent, and treating it as independent would overstate how much a longer window helps. The design estimates the lag-1 autocorrelation of its evidence series (the held-out errors when present, otherwise the in-sample residuals, which a smooth baseline understates, and the label says so), corrects it upwards for the small-sample bias of that estimate, floors it at 0 and caps it at 0.9 (warning `autocorrelation_clipped` when the cap binds), and uses it to inflate the variance of the window sum. In the example the raw estimate is 0.31 and the adjusted value 0.43.

### Critical value, power and the detectable effect

Alpha is two-sided (default 0.10) and the target power defaults to 80%. With held-out evidence the critical value comes from a t distribution whose degrees of freedom reflect the effective length of the evidence; with model-conditional evidence it is the normal quantile (1.645 at alpha 0.10). Power at a cumulative effect is the probability that the contrast falls beyond the critical value on either side when the true effect is that size. The cumulative detectable effect is the effect at which that power equals the target, found by bisection; the mean-per-period, relative and iROAS detectable effects follow from it.

Three facts hold by construction and are tested: the cumulative detectable effect does not change when the spend change changes; power rises with the size of the named effect; the iROAS detectable effect falls as the spend change grows.

**Assurance** is power averaged over the whole posterior distribution of the model-implied effect, rather than evaluated at its mean. Power is concave near the target, so assurance is usually below the power at the mean, which is why both are shown. It is always reported when the model-implied effect is available.

### What the power statement means

Every result carries the sentence that says what its power figure is, and it differs by evidence level.

With held-out evidence: "The probability, over observation noise and forecast error of the size measured on the held-out window, that the pre-specified analysis rejects the null at the named effect."

With model-conditional evidence: "If the model is correctly specified and its posterior calibrated: the probability, averaged over the model's uncertainty in its own forecast and over observation noise of the model's stated size, that the pre-specified analysis rejects the null at the named effect. Not the probability that this experiment will detect the effect."

Alpha and power therefore hold in the posterior-averaged sense, integrating over the model's own uncertainty about its counterfactual. Conditional on one fixed truth the false-positive rate is not exactly alpha. The acceptance simulations under [Validation](#validation) are run in the same posterior-averaged sense.

### Carryover tail

An intervention's effect does not stop when the spend change does. The measurement window extends past the intervention by the smallest number of periods at which the channel's posterior-mean cumulative [adstock](../core-concepts/adstock-effects.md) kernel weight reaches 0.95, so the tail of the effect is measured rather than left in the control period. A model saved without carryover kernels cannot set the window and is refused with `carryover_missing`; refitting with the reporting kernel on fixes it. In the example the tail is 3 weeks.

### Duration selection

You give up to eight candidate durations. Each is designed in full, and the table **Compare durations** shows for every one the measurement end, the model's expected effect over that window (it grows with the window), the detectable effect, power at the expected effect, power at the effect you named, assurance, and the power at the end of the model's HDI nearer zero, the cautious end. The shortest duration whose power at the model's expected effect reaches the target is selected. The selection is deterministic given the posterior, so there is no selection-over-noise bias. When no candidate qualifies the result state is **no feasible design** and the table is still returned so the gap is visible, with the two levers that close it: a larger spend change or a longer duration, and in **Advanced settings** a lower target power or a higher alpha.

A named cumulative effect is fixed while the cumulative detectable effect grows with the window, so with a named cumulative effect the shortest qualifying duration is always the one selected. A relative or per-period effect scales with the window, so longer durations can qualify.

## Geo split

A geo split changes the channel's spend in some markets and compares them with the markets left alone. It needs a **market panel**: an uploaded file or a [pipeline version](../integrations/model-from-a-pipeline-version.md) you are authorised to read, with one row per market and period and columns you map for the date, the market, the outcome and, for iROAS, the channel's spend per market. The panel must be at the model's cadence and cover the three windows below without gaps for every eligible market. Its content hash is checked, so a panel that has changed since you chose it is refused.

**Three windows.** The pre-period before the start date is split into three disjoint windows, counted back from the start date, with defaults of 26, 26 and 26 periods at any cadence. The validation window needs at least 26 periods: with fewer, the estimate of the forecast error is itself too noisy and the test would reject more often than its stated level, so the design is refused instead:

- **Matching** finds the controls that track the treated markets. It is used only to screen candidate markets.
- **Calibration** fits the forecast of treated from control, and chooses the arms.
- **Validation** checks that forecast on unseen weeks. Its error is the noise. It is used once, for the chosen arms only.

**Counterfactual.** The counterfactual is a time-based regression (Kerman, Wang and Vaver, *Estimating Ad Effectiveness using Geo Experiments in a Time-Based Regression Framework*): the treatment markets' aggregate outcome is regressed on the control markets' aggregate over the calibration window, and the fitted relationship, applied to the controls during the test, is what the treated markets are compared with. The pre-specified analysis is the cumulative contrast with the design-time standard error, named `tbr_cumulative_v1`. The noise has two parts kept separate so neither is counted twice: the period noise of the validation residuals with their autocorrelation, and the error in the fitted relationship itself, which shifts the whole window together. The critical value is a t quantile with degrees of freedom from the validation window's effective length. A validation window too short for its autocorrelation is refused.

**Arms.** You may restrict the eligible markets; otherwise every market in the panel is eligible. Treatment and control are never empty and never overlap. Among the allowed splits whose treatment share of pre-period outcome lies in your bound (default 20% to 50%), the design chooses the one with the smallest calibration-window forecast error. Up to 12 eligible markets the search is exhaustive; above that it is a greedy forward selection on matching-window correlation (Au, *A Time-Based Regression Matched Markets Approach for Designing Geo Experiments*) followed by the same calibration check. The validation window is then evaluated once for the chosen arms. Markets excluded for data reasons are listed with the reason, for example "Excluded: Islands (3%), because it has 41 weeks of history and the three windows need 78." Fewer than four eligible markets raises the warning `small_panel`.

**Spend change.** The treatment markets' spend change is read from the panel's spend column under the chosen mode, or from the explicit schedule. National lift is never allocated by market share.

**Model-implied effect.** A geo design reports what the model expects only when the model is a per-market fit for every treatment market, replayed per market and summed. For a national model the tile says **Not available for this model**, with the national model's relative effect for the same intervention beside it, labelled "For reference only, the national model implies minus 3.4%" in the example. There is then no power at the model's expected effect and no assurance; name an effect (a relative or per-period one scales with the window) and the design reports the chance of detecting it for every duration, selecting the shortest that reaches the target at your named effect. The design is still available without a named effect if what you want is the detectable effect.

## What you get back

The calculation is a job. Its operational status is `queued`, `running`, `complete`, `failed` or `cancelled`. A complete calculation carries a result whose own state is one of four. Consumers gate on the state, never on which keys are present: an unsupported or missing-evidence result never contains design numbers.

| State | App title | What it means |
|---|---|---|
| `available` | **Review design** | The design was calculated. Every number on this page is present, with its assumptions, warnings and provenance |
| `unsupported` | **This model is outside the supported designs** | The model is not in the supported family. One sentence says why and one next step says what to do; the model's own results are unaffected |
| `insufficient_evidence` | **This model can't support a time-holdout design yet** (or geo-split) | The model is supported but something the calculation needs is missing. The sentence names it and the next step says how to supply it. Your set-up is kept |
| `no_feasible_design` | **No duration reaches 80% power at the model's expected effect** | "At the model's expected effect of a TV pause, none of the four durations you asked for would detect it often enough. The table shows the gap." followed by "Try a larger spend change or a longer duration. A lower target power or a higher alpha also changes the answer; both sit in Advanced settings." The candidate table is returned |

Codes, ids and hashes live only inside **Evidence and provenance**. The sentences below are what the app shows for each code and the meaning an API or MCP consumer should read into it. Channel names, counts and figures are substituted; the examples use the synthetic TV pause.

### Reason codes

A result that is not `available` carries one or more reasons, each a code with a short machine detail.

| Code | State | What the app says | Next step |
|---|---|---|---|
| `posterior_missing` | unsupported | The saved model has no posterior draws to replay. | Save the model again from a completed fit, then design the test. |
| `unsupported_model_family` | unsupported | This model uses the log link. The designer currently supports identity-link models. (For a time-varying fit: This model has time-varying media effects. The designer currently supports fixed-effect identity-link models.) | Fit an identity-link model on the same data, save it, and design the test from there. This model's results are unaffected. |
| `scaling_missing` | insufficient evidence | The saved model has no scale to revenue, so spend cannot be converted into the KPI's units. | Set scale to revenue in Configuration, refit and save, then design the test again. |
| `carryover_missing` | insufficient evidence | The saved model has no carryover kernels for TV, so the measurement window can't be set. | Refit this model with the reporting kernel on (Configuration, Cohort horizon, "complete"), save it, then design the test again. Your set-up is kept. |
| `heldout_missing` | insufficient evidence | The model has no held-out window to measure its forecast noise against. | Refit with a held-out window, save it, then design the test again. |
| `heldout_irregular` | insufficient evidence | The held-out window has gaps or irregular periods, so its forecast noise cannot be measured. | Refit with a continuous held-out window at the model's cadence. |
| `heldout_too_short` | insufficient evidence | The held-out window is too short to measure forecast noise. | Refit with a longer held-out window, save it, then design the test again. |
| `holdout_reuse_unknown` | insufficient evidence | Simba cannot tell whether the held-out window was already used to choose this model. | Record how the held-out window was used, or refit with a fresh one. |
| `panel_mapping_missing` | insufficient evidence | The market panel is missing a required column: date, market or outcome. | Map every required column in the set-up, then calculate again. |
| `panel_history_insufficient` | insufficient evidence | No market has enough history for the matching, calibration and validation windows. | Supply a panel with longer history; the validation window cannot be shorter than 26 periods. |
| `local_effect_evidence_missing` | available, on the model-implied effect only | This is a national model. A market-level model is needed for a local estimate. | Name an effect in Advanced settings; the design then reports the chance of detecting it. |
| `validation_failed` | insufficient evidence | The control markets' forecast failed its check on the validation window. | Allow other markets, or lengthen the calibration window in Advanced settings. |
| `zero_variance` | insufficient evidence | The outcome shows no variation over the evidence window, so noise cannot be measured. | Check the KPI column; a constant series cannot support a design. |
| `invalid_schedule` | refused at request time | The start date or the durations fall outside the horizon the model can forecast. | Choose a start date soon after the training history ends, and durations between 1 and 52 periods. |
| `spend_baseline_zero` | refused at request time | TV's recent average spend is zero, so a percentage change has nothing to act on. | Choose a pause, a longer baseline window, or a schedule through the API. |
| `no_feasible_design` | no feasible design | At the model's expected effect of a TV pause, none of the four durations you asked for would detect it often enough. The table shows the gap. | Try a larger spend change or a longer duration. A lower target power or a higher alpha also changes the answer; both sit in Advanced settings. |

A market excluded from a geo design carries one of `panel_history_insufficient` (it does not have enough history for the three windows), `zero_variance` (its outcome shows no variation) or `validation_failed` (its forecast failed the validation check).

### Warning codes

A warning rides along with an available result. The design stands; the warning says what to doubt.

| Code | What the app says |
|---|---|
| `no_out_of_sample_calibration` | No out-of-sample calibration was possible for this model. |
| `forecast_uncertainty_lower_bound` | The forecast uncertainty term is a lower bound. |
| `forecast_horizon_long` | The test starts well after the training history ends, so the forecast carries more uncertainty than the noise figure shows. |
| `holdout_reuse_unknown` | The held-out window may already have been used to choose this model. |
| `evidence_short` | The evidence window is short, so the noise estimate rests on few periods. |
| `small_panel` | The panel is small, so the matching relationship rests on few pairs. |
| `seasonality_overlap` | The test window overlaps a seasonal peak, so the baseline forecast is less certain there. |
| `calibration_ratio_high` | The control forecast's error on the validation window is well above its in-sample error. |
| `autocorrelation_clipped` | The estimated autocorrelation was clipped to its allowed range. |
| `control_level_stand_in` | The control markets' level during the test was stood in by the validation window's level, because the panel does not cover the same dates one year earlier. |

### Assumption codes

Each assumption is a code with its parameters, so the number the design rested on travels with it.

| Code | What the app says |
|---|---|
| `cpu_training_mean` | Spend converts to activity at the training-window mean cost per unit. |
| `revenue_scale_stamped` | KPI units convert to revenue at the model's stamped scale to revenue. |
| `controls_at_realised_values` | Every other channel and control follows the model's forward-prediction conventions for the plan horizon. |
| `counterfactual_total_recent_training_mean` | The relative effect is taken against the model's fitted KPI over the most recent training periods of the window's length. |

Every available result also states, in words: the baseline (for example "The baseline is the mean TV spend over the last 13 training weeks, £30,000, repeated in every intervention week."); the carryover ("Carryover follows the posterior-mean adstock kernel; 3 weeks cover 95% of the tail."); for a geo split, "Treatment markets are paused together; control markets keep their baseline Display spend.", "The counterfactual is the control markets' outcome scaled by the matching relationship (slope 0.92, intercept 1,200), as validated on the validation window." and "The treatment spend change comes from the panel's spend column; national lift is never allocated by market share."; and the four [limitations](#limitations) below.

### Calculation failures

A failed calculation is not a result state and carries no numbers; it has an error code and a message, and you can calculate again with the same set-up.

| Code | What the app says |
|---|---|
| `artifact_unreadable` | The saved model's artefact could not be read. |
| `timeout` | The calculation ran out of time before it finished. |
| `worker_lost` | The worker running the calculation was lost. |
| `owner_blocked` | The model's owner cannot run calculations at the moment. |
| `queue_unavailable` | The calculation queue was unavailable, so nothing was started. |
| `internal` | Something went wrong inside the calculation. |

## Saving a planned test

Saving is a separate, explicit step. Nothing is saved until you choose to, a calculation nobody saves completes on its own and leaves no record, and saving launches nothing. **Save as planned test** appears only when the result is available and writes one record to the model's project; a model that is not saved to a project cannot save a design until it is. Saving is idempotent: saving the same calculation twice returns the same record.

The record is built by Simba from the stored result, not sent by the client. It holds:

- the type: `time_holdout`, a type that exists only for designed tests and cannot be recorded by hand or imported; or `geo`, the existing type with the design's treatment and control markets, which must not overlap;
- a name such as "TV pause, 8 weeks from 2 Nov 2026 (planned)", the status `planned`, the channel and model channel, the KPI;
- the start and end dates and the measured-through date at the end of the carryover tail;
- the signed incremental spend and its currency;
- a source of "Designed from model …" carrying the calculation id, the model hash and artefact hash, the method and estimator version, and the whole design result under its own key, so no reader can mistake the model-implied effect for a measured lift;
- for a time holdout, the counterfactual (the model's forecast), the analysis method and the carryover periods; for a geo split, the method and partial coverage.

**A planned record has no measured lift and cannot calibrate a model.** Its **Use with a model** card says "This test has no result yet, so it gives a model nothing. Record the result when the measurement window has closed." The record shows the design's detectable effect, model-implied effect and chance under a **Time-holdout design** section, with ids and hashes under **From the source, as reported**.

The lifecycle runs through the existing status changes, each saving a new version of the record:

1. **Planned.** Saved from the dialog. "No result yet. Designed to detect £286,000 at 80% power from minus £240,000 incremental spend."
2. **Running.** **Mark as running** when the test starts.
3. **Record result.** When the measurement window has closed, analyse the test outside Simba or compare actuals with the model's forecast, and enter the measured cumulative lift in KPI units for the whole test plus carryover, with its interval or standard deviation and the measured-through date. Null and negative results are kept like any other.
4. **Completed.** The completed view puts the measured lift beside the design's model-implied effect, both intervals labelled: in the example minus £275,000 measured (94% HDI minus £410,000 to minus £140,000) against minus £310,000 implied at design time.
5. **Use with a model.** A completed geo test calibrates a model exactly as any recorded geo test does (see [How the calibration row is derived](./incrementality-tests.md#how-the-calibration-row-is-derived)). A completed time holdout is kept and reported but does not calibrate: it compares the outcome with the model's own forecast, so its result is not an independent lift observation, and the derivation refuses it with `type_not_calibratable`.

Saving is refused when the result is not available, when the model has been refitted since the calculation (its artefact no longer matches) and when you do not own the target project. A design saved from a model keeps its provenance through every later status change.

## Over the API and MCP

All routes sit under `/api/v1`. Anyone who can read the model can design and read; saving needs ownership of the target project.

| Route | Scope | What it does |
|---|---|---|
| `POST /models/{model_hash}/test-designs` | `create:models` | Submit a design request. Answers 202 with `{calculation_id, model_hash, status: "queued", submitted_at}` at once; 200 with the same body when the `submission_key` repeats identical inputs; 409 `submission_key_conflict` when it repeats with different inputs; 400 `invalid_request` with field errors; 404 for a model you cannot read; 422 `model_not_complete`; 503 `queue_unavailable` when nothing could be started |
| `GET /models/{model_hash}/test-designs/{calculation_id}` | `read:results` | Poll the calculation: its operational `status`, the validated `request` with defaults filled, and the `result` once complete |
| `GET /models/{model_hash}/test-designs?limit=` | `read:results` | The model's calculations, newest first (1 to 50, default 10) |
| `POST /models/{model_hash}/test-designs/{calculation_id}/cancel` | `create:models` | `cancelled` when the job had not started, `cancellation_requested` while it runs; 409 `already_complete` |
| `POST /models/{model_hash}/test-designs/{calculation_id}/save` | `create:models` | Write the planned record: 201 `{id, version, record, content_hash}`; 200 with the same body when already saved; 409 `result_not_available` or `artifact_changed`; 403 when you do not own the project; 400 when the model has no project and none is given in `{project_id?, name?}` |

A key with only `create:models` can submit but not poll; give it `read:results` too.

A request names the channel, the design type, the intervention and the inference settings, with a `submission_key` of 8 to 128 characters that makes resubmission safe:

```json
{
  "submission_key": "design-2026-10-02-tv-pause-a1",
  "channel": "TV",
  "design_type": "time_holdout",
  "intervention": {
    "start_date": "2026-11-02",
    "durations": [4, 6, 8, 12],
    "spend_change": {"mode": "pause"},
    "baseline": {"mode": "recent_average", "periods": 13}
  },
  "inference": {"alpha": 0.10, "target_power": 0.80, "named_effect": {"value": 150000, "estimand": "cumulative"}}
}
```

`spend_change.mode` is `pause`, `percent` (with `pct` from minus 100 to 500, not 0) or `schedule` (explicit baseline and intervention arrays in currency). `baseline.mode` is `recent_average` over 4 to 52 training periods, or `schedule`. `durations` are 1 to 8 distinct whole numbers of periods from 1 to 52. A geo split adds a `geo` block with the panel, its column mappings, the eligible markets, the treatment share bound and the three windows. Unknown keys are refused, and nothing is defaulted silently except the stated defaults, which the result echoes.

Poll the calculation every few seconds until `status` is `complete`, `failed` or `cancelled`; a calculation usually finishes in under a minute. The result's `state`, `reasons`, `warnings` and `assumptions` carry the codes above; `model_implied_effect`, `detectable_effect`, `power`, `noise`, `candidates` and `planned_record` carry the numbers when the state is `available`. `power.meaning` is the sentence from [What the power statement means](#what-the-power-statement-means) for the evidence in use.

Over [Simba MCP](../integrations/simba-mcp.md) the same three actions are three tools; cancel and list are not exposed, because an unpolled calculation completes on its own and saves nothing, and the agent holds the id it submitted:

| Tool | Does |
|---|---|
| `design_incrementality_test` | Submits the request and returns the calculation id at once. A result is `available`, `unsupported`, `insufficient_evidence` or `no_feasible_design`; none of these is an error. Nothing is saved or launched |
| `get_incrementality_test_design` | Returns the calculation. `model_implied_effect.available` may be false |
| `save_incrementality_test_design` | Writes the planned record. It has no measured lift and cannot calibrate a model |

The MCP tool reference is generated from the running server: see [`docs/tools.md` in the simba-mcp repository](https://github.com/getsimba-ai/simba-mcp/blob/main/docs/tools.md) for the exact parameters. `create_incrementality_test` and `import_incrementality_tests` cannot create a designed record; the `type` filters on `list_incrementality_tests` and `get_incrementality_test` include `time_holdout` so planned designs list and read like any other test.

## Validation

The estimator is accepted by simulation before its results are shown, and the specification it implements is frozen before any acceptance simulation runs. Each replication builds a synthetic data-generating process, runs the production estimator end to end on finite data exactly as the product does (replay, evidence-series estimation, standard error, detectable effect, duration choice), then simulates the experiment and the pre-specified analysis and records whether it rejected. Noise is never generated from the standard-error formula and then checked against it. Within a scenario the posterior is held fixed and each replication draws the truth from it, so the posterior mean differs from the truth by a posterior-scale error every time; this is the posterior-averaged sense in which power is stated, and it is the primary pass criterion. A truth-conditional variant is reported beside it to show the size of the counterfactual-error term.

Scenarios cover autocorrelated noise, forecast gaps of 1, 4 and 13 periods, held-out windows of 8, 12 and 26 periods and none, and misspecified variants (a biased or overconfident posterior, a level shift in the held-out window, heteroscedastic and heavy-tailed noise, a seasonality mismatch, a changepoint after training); geo scenarios cover 6, 10 and 20 markets of unequal size with common shocks, spillover into controls, a structural break and a missing block of periods. Effects are null, the detectable effect, half and one and a half times it, and the negative of it. At least 5,000 replications per scenario and effect level. A scenario that fails is not rescued by changing the estimator afterwards: the fix is a domain restriction or a new frozen version with a fresh acceptance run.

The design is calibrated to reject at or below alpha under correct specification; the measured rate is given in the table below.

**Acceptance simulation results**

| Scenario | Evidence | Effect level | Replications | Predicted power | Empirical rate | 95% half-width | Pass |
|---|---|---|---|---|---|---|---|
| *To be filled from the acceptance report* | | | | | | | |

## Limitations

Stated with every result:

- The model-implied effect is model-implied. It is not a forecast of what the experiment will measure.
- Power assumes the pre-specified analysis and the stationarity of the evidence series into the test window.
- The design does not model spillover, competitor response or novelty effects. Controls and other channels are assumed at their realised values over the window; future controls at unknown values are not simulated.
- Under level B the noise basis is the model's own predictive distribution, its forecast-level term is a lower bound, and the power statement is posterior-averaged. It cannot detect misspecification.

And by scope: the designer proposes a design and does not run the experiment or analyse its result (record the result as described above, or in your own tool); it does not build synthetic controls; it offers no decision analysis beyond posterior-averaged assurance; and the log link, time-varying coefficients and VAR models are outside the supported family.

## References

- Kerman, J., Wang, P. and Vaver, J. (2017). *Estimating Ad Effectiveness using Geo Experiments in a Time-Based Regression Framework*.
- Au, T. (2018). *A Time-Based Regression Matched Markets Approach for Designing Geo Experiments*.

## Next steps

**Platform guides:**

- [Incrementality tests](./incrementality-tests.md): the register a planned design is saved to, and how a completed test calibrates a model
- [Validation metrics, holdouts and quality policies](./validation-and-holdout.md): reserving the held-out window that gives a design its strongest noise basis
- [Incremental Measurement](./measurement.md): the Media Results tab the design dialog opens from

**Integrations:**

- [What to test next](../integrations/experiment-priorities.md): the ranking that suggests which channel to design a test for
- [Simba MCP](../integrations/simba-mcp.md): connecting an agent that designs and saves tests

**Core concepts:**

- [Incrementality](../core-concepts/incrementality.md): why experiments and models need each other
- [Bayesian Modeling](../core-concepts/bayesian-modeling.md): posterior draws, the 94% HDI and what parameter uncertainty is
- [Adstock Effects](../core-concepts/adstock-effects.md): the carryover the measurement window waits for
