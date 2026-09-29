# Simba MCP: shared workflows for analysts and agents

The server reports its version and its tool list when a client connects. The generated reference at [`docs/tools.md` in the simba-mcp repository](https://github.com/getsimba-ai/simba-mcp/blob/main/docs/tools.md) lists every tool of the current release with its parameters and whether it reads or writes; it is rendered from the running server, so no count or version is typed by hand. The connected backend determines which operations are supported; installing the package does not upgrade that backend.

The [open-source MCP server](https://github.com/getsimba-ai/simba-mcp) connects compatible AI clients to Simba's [Bayesian marketing mix models](../core-concepts/bayesian-modeling.md). Analysts and agents use the same backend services and persisted objects. Studies, recipe revisions, hashes, lineage, quality policies, evaluations and decisions live in the Simba database. MCP does not maintain another study store or run another modelling engine.

## Connect locally or to a hosted server

You need a Simba account, an API key and the backend URL for your deployment. Keep the key in your client's protected configuration, never in prompts, shared screenshots or source control.

### Local stdio clients

Python 3.11 or later is required. Install the verified release:

```sh
pip install --upgrade "simba-mcp==0.4.1"
```

Configure your client to run `simba-mcp` with `SIMBA_API_URL` set to your backend URL and `SIMBA_API_KEY` set to your own key. Clients supporting `uvx` can instead run `uvx simba-mcp==0.4.1`. See the [client configuration examples](https://github.com/getsimba-ai/simba-mcp#quick-start).

Restart the MCP connection after upgrading. An already-running process or conversation can retain the old tool catalog. In a fresh connection, inspect the server version and look for `get_backend_capabilities` and `list_studies`.

### Hosted Streamable HTTP clients

Use the MCP endpoint supplied for your deployment. The hosted demo endpoint is `https://demo.simba-mmm.com/mcp`; local configuration uses the backend base URL without `/mcp`.

The hosted service authenticates each caller using `Authorization: Bearer <your Simba API key>`. Your client or configured connection must support sending that credential. Tool annotations do not grant access, and a successful tool listing does not prove that authenticated calls will succeed.

Hosted clients cannot read a file from your computer through `csv_path`. Use supported CSV content upload or upload through the Simba application and select the existing dataset. Local file paths refer to the machine running the MCP process and are disabled by default on hosted transports. Start with `get_data_schema` for the current CSV contract and limits.

### Refresh an existing ChatGPT connection

For a developer-mode MCP connection, open its connection settings in ChatGPT Plugins, select **Refresh**, check that the new tools appear, and start a new conversation. This reloads changed tool metadata after a server upgrade. Published plugins have a separate continuous-review process for tool changes; server deployment alone does not confirm that a published listing has refreshed. Follow [OpenAI's connection and refresh guidance](https://developers.openai.com/plugins/deploy/connect-chatgpt#refresh-metadata).

## Discover what your backend supports

Call `get_backend_capabilities` before choosing model families, transformations, [coefficient priors](../core-concepts/priors-and-distributions.md) or workflow operations. It reads the connected backend's advertisements. An absent advertisement means **unknown**, not supported and not necessarily unsupported. Check the deployed backend version or contact your administrator before using an unadvertised feature.

The tools cover these groups (examples, not the full list):

| Task | Example tools |
|---|---|
| Inspect data and projects | `get_data_schema`, `list_uploads`, `list_projects` |
| Create and inspect models | `create_model`, `create_var_model`, `get_model_status`, `get_model_results` |
| Plan budgets and scenarios | `run_optimizer`, `run_scenario`, `list_runs` |
| Organize shared studies | `list_studies`, `create_study`, `get_study` |
| Freeze and inspect recipes | `validate_study_recipe`, `create_study_recipe`, `get_recipe_revision` |
| Launch and monitor study runs | `launch_study_run`, `get_study_run`, `cancel_study_run` |
| Review evidence and decisions | `evaluate_study_run`, `compare_study_runs`, `recommend_study_run`, `list_study_decisions` |

Later releases add tools beyond these groups; for example, recorded incrementality tests (`list_incrementality_tests`, `get_incrementality_test`, `create_incrementality_test`, `import_incrementality_tests`) and `create_model`'s `calibration` parameter are described in [Incrementality tests](../platform-guide/incrementality-tests.md). Use the connected server's `tools/list` response for exact required inputs and available tools. Standard titles, descriptions and read-only/destructive/idempotency annotations help clients select tools; they are hints, never authorization controls.

## A Studies workflow

1. **Inspect first.** Discover backend capabilities, select the project, and read the study question, state, attempt budget and concurrency limit. Shared project access permits study reads; mutations require ownership.
2. **Validate a recipe.** Resolve the dataset, model settings and priors with `validate_study_recipe`. Save an immutable revision with a rationale. The optional `expected_content_hash` checks that the effective inputs still match the preview.
3. **Declare the quality policy.** Select or create project-specific checks before launch. Available metrics are `r_hat_max`, `mae`, `rmse` and `wape`; WAPE is a fraction. There are no universal default pass thresholds.
4. **Launch explicitly.** Pass the frozen revision, a policy from the same study and a caller-generated `submission_key` to `launch_study_run`. A study budget limits attempts; it does not automatically launch that many fits.
5. **Monitor the shared run.** Read `get_study_run` and the existing model progress. A cancellation request is not confirmed cancellation; poll until the run reports its outcome. Missing heartbeat evidence is unknown, and a stall threshold is not an ETA or permission to restart a fit.
6. **Evaluate and compare.** Evaluate saved evidence, then compare candidates against one policy. Missing evidence does not pass. Current error metrics describe the fitted window, not held-out predictive validation. Different datasets are flagged rather than ranked together.
7. **Recommend for analyst review.** Record the candidate, evaluation and rationale with `recommend_study_run`. The frontend records analyst acceptance or rejection. A recommendation does not accept or automatically promote a model, and a passing report does not prove business validity.

Analyst-created recipes and runs are visible through the same study objects. Captured wizard recipes can be inspected and launched; changing a captured wizard configuration requires another wizard capture or a separately validated API recipe. Imported historical recipes with incomplete provenance can be review-only.

## Retry and conflict recovery

Writes are sent once without automatic HTTP retries. If a response is uncertain, inspect the saved objects before repeating a mutation. For a study launch retry, reuse the **same submission key and identical revision/policy inputs**. Do not generate a new key to escape an attempt-budget or state conflict.

A stale recipe version returns **412**: reload the current recipe, reconcile the edit and submit the current `expected_version`. An effective-input hash conflict returns **409**: validate again and review the changed inputs. Error objects retain `error` and `_status_code`, with additive `_error_code` and `_next_action` guidance.

## Retrieve only the evidence you need

For model results, begin with `sections="channel_summary,model_stats"`, then request additional evidence. Filter with `channels` and `max_grid_points` where applicable. Optional `max_response_bytes` returns **413** if the filtered JSON exceeds the limit, rather than presenting partial evidence. This check happens after backend download and excludes MCP envelope overhead. Study histories are not yet backend-paginated.

Existing tool names, required inputs and default payloads are preserved across releases; new tools and response fields are additive. See the [release notes](https://github.com/getsimba-ai/simba-mcp/releases) and the [architecture and compatibility guide](https://github.com/getsimba-ai/simba-mcp/blob/main/docs/architecture.md).

## Try a read-only check

> Inspect my connected Simba backend capabilities and list my studies for the project I select. Summarize available workflow operations and any unknown capabilities. Do not upload data, create a model or launch a run.

For setup problems, include the server version and tool names in a [public MCP issue](https://github.com/getsimba-ai/simba-mcp/issues), without credentials or private data, or contact [info@pymc-labs.com](mailto:info@pymc-labs.com).

## Next steps

### Platform guides

- [Model configuration](../platform-guide/model-configuration.md)
- [Measurement and diagnostics](../platform-guide/measurement.md)
- [Budget optimization](../platform-guide/budget-optimization.md)
- [Scenario planning](../platform-guide/scenario-planning.md)

### Core concepts

- [Bayesian modeling](../core-concepts/bayesian-modeling.md)
- [Priors and distributions](../core-concepts/priors-and-distributions.md)
- [Optimization](../core-concepts/budget-optimization.md)
