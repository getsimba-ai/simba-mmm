# Validation metrics, holdouts and quality policies

Every fitted model reports a table of fit diagnostics, and a model that reserved a holdout also saves its predictions for that window. A Study turns those saved numbers into pass, fail or not-evaluated against a quality policy you declare before the run.

Everything on this page except the holdout fraction itself is read or declared through the API and through [Simba MCP](../integrations/simba-mcp.md). The MCP tool reference is generated from the running server: see [`docs/tools.md` in the simba-mcp repository](https://github.com/getsimba-ai/simba-mcp/blob/main/docs/tools.md) for the exact parameters of `get_model_results`, `create_quality_policy`, `evaluate_study_run`, `assess_study_validation_pair` and `declare_study_holdout_use`.

## Fit diagnostics a model reports

```
get_model_results(model_hash="abc123", sections="model_stats")
```

The `model_stats` section is a list of rows. Each row has a `Test Name`, an `Output`, an `Evaluation` sentence and a `Status` of `success`, `warning`, `error` or `info`. Read `Status`, not the sentence, when you automate on it. The in-sample rows compare the posterior predictive mean with the actual KPI over the fitted window.

| Test Name | What it measures | How the row is judged |
|---|---|---|
| `Mean Absolute Percent Error (%)` | MAPE of the posterior predictive mean, in percent | below 5 is a `warning` for possible overfitting, above 10 a `warning` for possible underfitting, otherwise `Ok` |
| `R-Squared` | share of KPI variance explained | compared against expected bands; outside them it is flagged as potential or significant underfitting or overfitting |
| `Durbin-Watson` | autocorrelation in the residuals | 1.5 to 2.5 is `Ok`, otherwise possible autocorrelation |
| `Max R_hat` | the largest R-hat over every posterior variable; the row also carries `MaxParam` and `Median` | above 1.2 the model has not converged and the status is `error` |
| `Leave-One-Out-CV` | expected log predictive density from leave-one-out cross-validation | relative metric, higher is better; status `info` |
| `Pareto K (% influential)` | share of observations whose Pareto k is above 0.7 | `warning` when above 10%, or when any observation is above 1 |
| `Jarque-Bera (p value)` | normality of the residuals | p above 0.05 means the residuals are normal |
| `Out-of-Sample MAPE (%)`, `Out-of-Sample R-Squared`, `Out-of-Sample RMSE` | the same fit statistics on the holdout window | present only when the model reserved a holdout; out-of-sample MAPE below 10 generalises well, above 20 is a `warning` |

Two more things about convergence:

- The 1.2 threshold applies to the single worst parameter. To find it, request the `r_hat` section, which lists R-hat for every posterior variable including transform parameters such as `{channel}_decay`. The `posterior` section lists each coefficient with its mean, sd, 94% HDI (3% to 97%) and `r_hat`.
- ESS and divergences are not rows in `model_stats`. The fit saves a retained-sampling record with `chains`, `draws_per_chain`, `total_draws`, `ess_bulk_min` and `ess_tail_min` (the minimum over all posterior variables), `divergences` and `divergence_fraction`, with a status of `available`, `partial` or `unavailable`. Today that record is read through a Study: as `sampling_evidence` on a validation-pair assessment, or through the sampling checks of a quality policy.

A prior predictive fit reports R-hat, LOO and Pareto k as not applicable.

## Reserve a holdout window

A holdout is declared with the model's training fraction, `train_period`. The rows are sorted by date, the first `n × train_period` rows (rounded down) train the model, and the remaining rows form the prediction window. The in-sample diagnostics above are computed over those training rows only. `train_period=1` fits on every row and saves no prediction window. Today the fraction is set in the model creation wizard's train/test split; `create_model` over the API and MCP has no holdout parameter, and a study run inherits the fraction from the wizard configuration its recipe froze.

The saved window is opt-in on results:

```
get_model_results(model_hash="abc123", sections="prediction_window")
```

Each row carries `date`, `model`, `fit actual`, `hdi_50_lower`, `hdi_50_upper`, `hdi_95_lower` and `hdi_95_upper`. The response's `sections_available` lists `prediction_window` whenever the model has one, even when you did not ask for it. For a model linked to a Study run whose saved window can be assessed, serving this section appends an access event (`results_json` or `results_csv`) that `get_study_prediction_access` reports.

From those rows a Study computes `prediction_mae`, `prediction_rmse` and `prediction_wape`, and reports `training_end`, `prediction_start`, `prediction_end` and `row_count`. The prediction dates must be unique and strictly after the last training date; otherwise the window is `not_evaluated` with a reason.

## Declare a quality policy

```
create_quality_policy(
    study_id="st_1", name="Weekly sales gate", rationale="Convergence, fit and holdout limits for this project",
    checks=[
        {"metric": "r_hat_max", "maximum": 1.2, "required": True},
        {"metric": "prediction_wape", "maximum": 0.15, "required": True},
        {"metric": "divergences", "maximum": 0, "required": True},
        {"metric": "durbin_watson", "operator": "between", "minimum": 1.5, "maximum": 2.5, "required": False}
    ],
    validation_protocol={
        "kind": "temporal_holdout", "training_end": "2024-09-30",
        "prediction_start": "2024-10-07", "prediction_end": "2024-12-30",
        "min_draws": 2000, "min_tune": 1500, "min_chains": 4,
        "max_r_hat": 1.2, "max_prediction_wape": 0.15,
        "retained_sampling": {"min_ess_bulk": 400, "min_ess_tail": 400, "max_divergences": 0}
    })
```

The numbers above are examples; no default thresholds are assumed. A check names a metric once and is `required` unless you say otherwise. Built-in, diagnostic and sampling checks use `lte` (the default), `gte` or `between` with only the bounds that operator needs; a custom numeric check also states `name`, `units` and an explicit `operator`; provenance, boolean and manual checks use `equals` with an `expected` value. At least one check must be required, at most 20 checks are accepted, and a bound on `r_hat_max` must be at least 1.

| Check family | Metrics | Where the value comes from |
|---|---|---|
| Built-in | `r_hat_max`, `mae`, `rmse`, `wape`, `prediction_mae`, `prediction_rmse`, `prediction_wape` | saved R-hat, the fitted-window actual-versus-model table, the saved prediction window |
| Diagnostic | `r_squared`, `mape`, `durbin_watson`, `pareto_k_pct`, `normality_p`, `loo_cv` | the `model_stats` rows above |
| Sampling | `retained_chains`, `retained_draws_per_chain`, `ess_bulk_min`, `ess_tail_min`, `divergences` | the retained-sampling record, only when it is complete |
| Provenance | `provenance:holdout`, `provenance:prior` | `equals` `review_required`; `blocked` fails, an absent record is `not_collected` |
| Custom | `custom:<slug>` numeric, `kind: boolean`, `kind: manual` | evidence you submit with the evaluation; manual sign-off needs a signed-in reviewer |

WAPE is a fraction: the sum of absolute errors divided by the sum of absolute actuals. `mae`, `rmse` and `wape` describe the fitted window, not the holdout; the `prediction_*` metrics describe the holdout.

The optional `validation_protocol` declares, before both runs launch, the holdout dates (`training_end` before `prediction_start`, which is on or before `prediction_end`), the configured sampling minima (`min_draws`, `min_tune`, `min_chains` of at least 2), `max_r_hat` (at least 1) and `max_prediction_wape`. `retained_sampling` adds limits on what the sampler actually retained, and `require_policy_review` requires current analyst acceptance of each latest same-policy assessment. Policies are immutable: create a new one with `derived_from_policy_id` to change a limit.

## Evaluate a run

```
evaluate_study_run(run_id="run_7", policy_id="qp_3", preview=True)
```

Every check comes back as `pass`, `fail` or `not_evaluated`, with `basis.reason` from a closed set such as `not_collected`, `not_supplied`, `awaiting_human_review`, `unsupported` or `malformed_evidence`. The report's `status` is `fail` when any required check fails, `not_evaluated` when any required check could not be evaluated, and otherwise `review_required`: `business_validity` and `analyst_review` always stay with a person. `not_evaluated` never passes. With `preview=False` the assessment is saved and immutable. VAR runs are `not_evaluated` with scope `unsupported_model_family`.

## Assess a validation pair

```
assess_study_validation_pair(study_id="st_1", full_run_id="run_7", validation_run_id="run_8", policy_id="qp_3")
```

The full run fits with `train_period=1`, the validation run with a strict training subset, both launched under the same policy. Each check is `pass` or `blocked`, and the overall `status` is `pass` only when every check passes.

| Check | What must hold |
|---|---|
| `declared_protocol`, `declared_at_launch` | the policy carries a `validation_protocol`, and both runs launched under it (adopted models cannot qualify) |
| `distinct_runs`, `completed_mmm` | two different completed MMM runs |
| `same_inputs`, `same_settings`, `same_runtime` | frozen data, priors, costs, calibration, settings (apart from `train_period` and display names) and engine manifests match |
| `full_and_holdout_split` | `train_period` is 1 on the full run and strictly between 0 and 1 on the validation run |
| `configured_sampling` | draws, tune and chains on both runs meet `min_draws`, `min_tune` and `min_chains` |
| `full_r_hat`, `validation_r_hat` | every saved R-hat is finite and at most `max_r_hat` |
| `prediction_window`, `prediction_wape` | the validation run's observed dates equal the declared three dates, and its `prediction_wape` is at most `max_prediction_wape` |
| `matching_coverage` | the full run's dates equal the disjoint union of the validation run's training and prediction dates |
| `{full,validation}_retained_{chains,draws_per_chain,ess_bulk_min,ess_tail_min,divergences}` | only when `retained_sampling` was declared; a partial record blocks |

The response also carries `sampling_qualification` (`pass`, `blocked` or `not_declared`), `sampling_evidence` for both runs, `holdout_provenance` and `prior_provenance` (`review_required`, `blocked` or `unavailable`), an `evidence_hash` naming the assessed records, and `decision_grade_ready`, which is always `false` today.

## Declare how the holdout was used

```
get_study_prediction_access(run_id="run_8")
declare_study_holdout_use(run_id="run_8", declaration_id="3f6c2a10-...", source_access_id="acc_12",
                          disposition="review_only", reason="Read once to check the WAPE, no settings changed")
```

`disposition` is `review_only`, `informed_revision` or `uncertain`. `informed_revision` needs `affected_revision_id`, a published revision in the same study, and blocks that revision from champion acceptance until fresh validation. Reuse the same `declaration_id` on a retry. A declaration is recorded as reported by the submitter; it does not certify independence.

## Things to keep in mind

- **A saved prediction window is not certified untouched holdout evidence.** Access history starts when prediction evidence is served by a deliberate action; earlier activity and offline work are not covered, so absence of events never proves untouched status.
- **Fitted-window metrics are not held-out validation.** `mae`, `rmse`, `wape` and `r_hat_max` describe the training window.
- **A policy report never says `pass` overall.** The best outcome is `review_required`; acceptance and champion selection happen in the signed-in frontend, not over the API.
- **Retired policies stay readable** but are refused for new launches, assessments and pair reviews.

## Next steps

- [Measurement](./measurement.md): reading the Model Stats and Actual vs Model tabs in the dashboard.
- [Model creation wizard](./model-creation-wizard.md): the Train/Test Split slider and sampling options.
- [Simba MCP](../integrations/simba-mcp.md): the Studies workflow from recipe to recommendation.
- [Incrementality tests](./incrementality-tests.md): calibrating a model with experiment results.
