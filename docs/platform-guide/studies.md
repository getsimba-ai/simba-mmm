# Studies: drafts, published revisions, diffs, calibration, quality policies and the Champion

A study is a shared record of one modelling question: the recipes that were tried, every run they produced, how each run scored against a quality policy, and which model a person finally chose. Analysts in the application and agents over the API or MCP read and write the same study objects, so nothing about a study lives only in a chat.

The MCP tool reference is generated from the running server: see [`docs/tools.md` in the simba-mcp repository](https://github.com/getsimba-ai/simba-mcp/blob/main/docs/tools.md) for the exact parameters of every tool named below. The short [Studies workflow](../integrations/simba-mcp.md#a-studies-workflow) on the MCP page gives the seven-step summary; this page walks the lifecycle end to end.

## Create a study

A study belongs to a project. Creating one launches nothing and consumes no attempts.

```
create_study(project_id=7, name="Search and Social contribution",
             question="How much do Search and Social contribute to weekly sales after price and seasonality?",
             max_attempts=5, max_concurrent=1)
```

The `question` and optional `context` are text for people. They are stored and shown on the study overview and nothing reads them as a rule; acceptance thresholds belong in a quality policy, model settings in recipe revisions, and run limits in `max_attempts` (1 to 100, default 5, counting failed fits) and `max_concurrent` (1 to 4, default 1). `get_study_overview(study_id=...)` returns the study, its budget, each recipe with its latest revision, run counts, active policies, the champion summary and the last decision in one call.

## Draft a recipe and publish a revision

A draft is unvalidated authoring state, saved encrypted, that you can edit as many times as you like. Publishing turns it into immutable revisions; it does not fit, does not consume an attempt and does not choose a champion. Start from the shared wizard defaults over a dataset you own, copy the returned `snapshot` into a draft, then publish:

```
get_recipe_draft_template(family="mmm", uploaded_file_id=42)
create_recipe_draft(study_id=..., draft_id="<uuid you generate>", name="Baseline", snapshot={...})
publish_recipe_draft(draft_id=..., expected_version=1, publication_id="<uuid you generate>", reason="First frozen baseline")
```

What a published revision records:

- **The frozen inputs.** The compiled settings, the input files with their SHA-256 hashes, the prior rows and the engine identity the revision was frozen on. `get_recipe_revision(recipe_id=..., number=1)` shows which settings were authored and which are fitter defaults, and flags prior fields that are stored but never read for the chosen adstock or saturation family.
- **A content hash.** A 64-character hash of the frozen inputs. Two revisions with the same hash froze the same inputs on the same engine; a fit of either still needs the original seed and engine environment to reproduce exactly. `create_study_recipe` and `revise_study_recipe` accept `expected_content_hash` from `validate_study_recipe` so a save is refused if the effective inputs moved since the preview.
- **Lineage.** Publishing with `target={recipe_id, expected_version}` writes revision N+1 of that recipe and records the draft's base revision as its source. Publishing without a target creates a new recipe, one per prepared brand; pass `source_revision_id` on `create_recipe_draft` to record the published revision it branched from. `get_recipe_revision_authoring` reopens a published revision's snapshot for an in-place edit or a branch.
- **Smart priors.** MMM priors the draft leaves unset are filled with the Prior Builder's calculation over the whole source and frozen at publication. Replaying the same `publication_id` returns the same revisions and does not rebuild them.
- **Calibration.** Lift tests enabled in the draft are attached as likelihood observations, and the revision records their count, units, channels and a hash of the rows. Every calibration row must name a channel that is a selected nonlinear media variable, or publication fails. VAR drafts with calibration enabled fail publication, since VAR does not support it. See [Incrementality tests](./incrementality-tests.md) for the test records themselves.

## Diff two revisions

```
diff_recipe_revisions(recipe_id=..., base=1, other=2)
```

| Field | Contents |
|---|---|
| `settings[]` | `key`, `section`, `label`, `from`, `to` for each setting that changed; blank and absent count as the same value |
| `priors[]` | `row`, `parameter`, `column`, `label`, `from`, `to`; rows are matched by variable and role |
| `data` | The dataset origin and input-hash change, or `null` when the data is the same |
| `counts` | `settings`, `priors`, `data`, `total` |

## Check eligibility, then launch

`launch_study_run` fits a new model from a frozen revision; it never reopens existing results. Read the verdict first, because the launch route applies the same rules and refuses with the first blocker. Over HTTP the read is `GET /api/v1/studies/{study_id}/launch-eligibility?revision_id=...` and the launch is `POST /api/v1/studies/{study_id}/runs`, which answers with the run and status 202.

```
get_launch_eligibility(study_id=..., revision_id=...)
launch_study_run(study_id=..., revision_id=..., policy_id=..., submission_key="baseline-2024-08-01-a")
```

The eligibility read returns `can_launch`, `blockers`, the `budget`, the `policy` used (the newest active one when `policy_id` is omitted) and the `revision` with its engine state.

| Blocker code | Next action |
|---|---|
| `permission_denied` | Ask the project owner to launch; this study is shared read-only |
| `policy_not_found`, `policy_retired` | Choose an active policy that belongs to this study |
| `study_inactive` | Resume the study; a paused study blocks new reservations without cancelling running work |
| `revision_not_found` | Choose a saved revision of a recipe in this study |
| `attempts_exhausted`, `concurrency_exhausted` | Raise the attempt limit in study settings, or wait for a running fit to finish |
| `unsupported_family` | Only MMM and VAR recipes launch from a study |
| `engine_changed` | Call `refreeze_recipe_revision(recipe_id=..., number=...)` and launch the new revision |
| `snapshot_not_executable` | Author a validated API recipe from the review-only snapshot first |

The engine identity is part of the frozen revision. If the engine changes between reservation and the worker picking the run up, the run closes as `engine_changed`; it is terminal but not counted against the attempt budget. Re-freezing creates a new revision with the same specification on the current engine and leaves the old one untouched.

## Evaluate a run against a quality policy

A quality policy is the study's own, immutable set of checks. There are no default thresholds: declare at least one required check, use each metric once, and set the R-hat maximum at 1 or above.

```
create_quality_policy(study_id=..., name="Weekly sales v1",
                      rationale="Converged fit with in-sample error under 15%",
                      checks=[{"metric": "r_hat_max", "maximum": 1.2, "required": true},
                              {"metric": "wape", "maximum": 0.15, "required": true}])
evaluate_study_run(run_id=..., policy_id=..., preview=true)
evaluate_study_run(run_id=..., policy_id=...)
```

Built-in metrics are `r_hat_max`, `mae`, `rmse`, `wape`, `prediction_mae`, `prediction_rmse` and `prediction_wape`; WAPE is a fraction. Artifact-backed checks read saved diagnostics, retained-sampling counts and provenance status; custom numeric, boolean and manual checks are described in the tool reference. To change a policy, create a new one with `derived_from_policy_id` and read `diff_quality_policies` to see what moved; `retire_quality_policy` stops a policy taking new launches and assessments while keeping it readable everywhere it is already named.

An assessment is immutable once saved. Each check reports `pass`, `fail` or `not_evaluated`. The report status is `fail` when any required check failed, `not_evaluated` when any required check had no evidence or the policy has no required checks, and otherwise `review_required`. A report never says "pass": the best outcome is `review_required`, because built-in errors describe the fitted window, not held-out validation, and business validity is a human judgement. VAR runs are reported as `not_evaluated` with scope `unsupported_model_family` today, whatever policy is named; quality-policy checks establish nothing for a VAR run. `compare_study_runs` ranks 2 to 20 runs under one policy and refuses to rank rows whose family, dataset, outcome, units, window or output kind differ.

## Recommend, then a person decides

```
recommend_study_run(study_id=..., run_id=..., evaluation_id=..., reason="Lowest WAPE of the three converged fits")
```

This records a decision of kind `recommend` with the evaluation it rests on. It does not accept or promote anything. The human steps happen in the application, signed in:

1. **Accept.** The project owner accepts a run against a specific assessment; shared read-only access cannot record a decision. Acceptance requires the evidence hash to be unchanged since that assessment, the report status `review_required` and a complete model; otherwise it is refused. The same request over an API key is refused with 403.
2. **Select the Champion.** The project owner selects an accepted candidate, giving a reason and optionally the intended use and a validation reference, and can revoke it later; the history keeps every event. Over an API key the request is refused with 403 and the code `manual_signoff_requires_session`. Nothing in the backend selects a champion on its own; this write is the only place a champion event is created.

`get_study_champion(study_id=...)` returns the incumbent, its blockers, the accepted candidates and the full history. A champion whose model evidence or validation review changed since selection is reported with status `review_required` rather than `current`. `decision_grade_ready` is always `false` today: validation references are reviewer-declared, not independently verified.

## Adopt an existing model

```
adopt_model_into_study(study_id=..., model_hash="…", reason="Fitted before the study existed", confirm=true)
```

Without `confirm` you get a preview and an import report; nothing is written. With `confirm=true` a recipe is created whose revision 1 carries the exact fit inputs and, for a model built after the wizard began capturing snapshots, the wizard snapshot captured at build; older models import with their unrecorded settings marked as such. The fitted result is attached as an adopted run. Adopted runs do not count against the attempt or concurrency budget, and adoption never refits. The request is refused with 409 when the model is not complete or is already attached to a study.

## Retry and conflict semantics

Every launch carries a `submission_key` you generate (8 to 128 characters), unique per study and submitter. After an ambiguous response, send the same key with the same revision and policy: the existing run is returned and no second fit is reserved. The general rule for writes is on the MCP page under [Retry and conflict recovery](../integrations/simba-mcp.md#retry-and-conflict-recovery).

| Situation | Response | What to do |
|---|---|---|
| Same key, same revision and policy | 202 with the existing run | Nothing; poll `get_study_run` |
| Same key, different revision or policy | 409 `submission_key_conflict` | Inspect the run that owns the key; use a new key only for a genuinely new launch |
| Stale `expected_version` on a study, recipe, draft or publish target | 412 `stale_version` | Reload the object, reconcile, submit the current version |
| `expected_content_hash` no longer matches | 409 | Validate again and review the changed inputs |
| Evidence changed since the assessment preview | 409; the code is `stale_evidence` when a `carry_forward` source's evidence basis changed, otherwise `conflict` | Preview again and submit the new `expected_basis_hash` |

Over HTTP a refusal is `{"error": <message>, "code": <code>}`. Codes are stable and additive; branch on the code and show the message.

## Things to keep in mind

- **A frozen revision is not exact reproducibility.** Every revision records that exact sampler reproducibility needs the original seed and engine environment.
- **Fitted-window metrics are not held-out validation.** Prediction-window checks need saved actuals and predictions after the training window, and even then do not certify untouched holdout provenance.
- **Newest is information, not a recommendation.** Choose `policy_id` explicitly; the newest active policy is only the default for the eligibility read.
- **The Champion is a human decision.** MCP and API keys can validate, publish, launch, evaluate, compare and recommend. Acceptance, champion selection and manual sign-off need a signed-in session in the application today.

## Next steps

- [Simba MCP](../integrations/simba-mcp.md) for connecting a client and the seven-step summary
- [Model configuration](./model-configuration.md) for the settings a recipe freezes
- [Incrementality tests](./incrementality-tests.md) for the lift-test records a recipe can calibrate on
- [Measurement and diagnostics](./measurement.md) for R-hat and the error metrics a policy checks
