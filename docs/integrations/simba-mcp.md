# Simba MCP: shared workflows for analysts and agents

The server reports its version and its tool list when a client connects. The generated reference at [`docs/tools.md` in the simba-mcp repository](https://github.com/getsimba-ai/simba-mcp/blob/main/docs/tools.md) lists every tool of the current release with its parameters and whether it reads or writes; it is rendered from the running server, so no count or version is typed by hand. The connected backend determines which operations are supported; installing the package does not upgrade that backend.

The [open-source MCP server](https://github.com/getsimba-ai/simba-mcp) connects compatible AI clients to Simba's [Bayesian marketing mix models](../core-concepts/bayesian-modeling.md). Analysts and agents use the same backend services and persisted objects. Studies, recipe revisions, hashes, lineage, quality policies, evaluations and decisions live in the Simba database. MCP does not maintain another study store or run another modelling engine.

## Connect a client

Two ways to authenticate, depending on the client. **A key** (create one under Profile → API Keys; choose its scopes, and it expires within a year) works everywhere. **OAuth** is for clients whose connector settings expect a sign-in flow: you approve the client on a consent screen, choose the scopes it gets, and can revoke it later under Profile → Connected apps. Tokens issued that way expire after an hour and renew for up to 30 days. Both give exactly the same access: the same scopes and the same refusals apply to a key and to an OAuth token.

The hosted server for the demo deployment is `https://demo.simba-mmm.com/mcp`. Keep keys in your client's protected configuration, never in prompts, shared screenshots or source control.

### claude.ai (custom connector, OAuth)

1. In claude.ai, open **Settings → Connectors** and choose **Add custom connector**.
2. Name it (for example "Simba") and enter the server URL `https://demo.simba-mmm.com/mcp`. Leave the OAuth client fields empty: the connector registers itself.
3. Click **Add**, then **Connect**. A Simba page opens. Sign in if you are not signed in (with your second factor, if you have one).
4. On the consent screen, the client name shown is the one the client registered. On a first connection only the read scopes (`read:models`, `read:results`) start ticked; a client you approved before starts with the scopes it already has. Tick the others the connection needs, then click **Approve**.
5. Back in claude.ai the connector shows as connected. In a new chat, enable it and try "List my Simba projects".

To disconnect, revoke it under Profile → Connected apps in Simba, or remove the connector in claude.ai. Either way the client has to connect again.

### ChatGPT (connector, OAuth)

1. In ChatGPT, open **Settings → Connectors** and create a connector.
2. Name it, enter the MCP server URL `https://demo.simba-mmm.com/mcp`, and choose **OAuth** as the authentication. No client ID or secret is needed.
3. Save, then **Connect**. Sign in to Simba if asked, tick the scopes the connection needs on the consent screen (on a first connection only the read scopes start ticked), and click **Approve**.
4. Start a new conversation with the connector enabled.

After a server upgrade, start a new conversation so changed tool metadata is reloaded; if the tools still look stale, disconnect the connector and connect it again.

### Claude Desktop (local, key)

Claude Desktop runs the server as a local process. Python 3.11 or later is required.

1. Create an API key under Profile → API Keys with the scopes you need.
2. Open **Settings → Developer → Edit Config** and add:

```json
{
  "mcpServers": {
    "simba": {
      "command": "uvx",
      "args": ["simba-mcp"],
      "env": {
        "SIMBA_API_URL": "https://demo.simba-mmm.com",
        "SIMBA_API_KEY": "simba_sk_…"
      }
    }
  }
}
```

3. Restart Claude Desktop so it starts the server and reads its tool list.

`uvx simba-mcp` runs the package without installing it; `pip install simba-mcp` with `"command": "simba-mcp"` works the same way, since `simba-mcp` is the package's command and stdio is its default transport. After upgrading the package, restart Claude Desktop: the tool catalog is built when the process starts, so a running process keeps the old one.

### Claude Code (key)

```sh
claude mcp add --transport http simba https://demo.simba-mmm.com/mcp   --header "Authorization: Bearer simba_sk_…"
```

Then `/mcp` in a session shows the connection.

### Cursor (key)

Add to `.cursor/mcp.json` in your project, or the global one under Settings → MCP:

```json
{
  "mcpServers": {
    "simba": {
      "url": "https://demo.simba-mmm.com/mcp",
      "headers": { "Authorization": "Bearer simba_sk_…" }
    }
  }
}
```

### Claude API (MCP connector, key)

The Messages API's MCP connector authenticates with a token passed in the request, so use a key. Pass the server with `url: "https://demo.simba-mmm.com/mcp"` and `authorization_token` set to your key; see the [client configuration examples](https://github.com/getsimba-ai/simba-mcp#quick-start).

### Which method a client uses

| Client | Method | Where to revoke |
|---|---|---|
| claude.ai custom connector | OAuth | Profile → Connected apps |
| ChatGPT connector | OAuth | Profile → Connected apps |
| Claude Desktop | key (local process) | Profile → API Keys |
| Claude Code | key | Profile → API Keys |
| Cursor | key | Profile → API Keys |
| Claude API connector | key | Profile → API Keys |

Hosted clients cannot read a file from your computer through `csv_path`. Use the CSV content upload, or upload through the Simba application and select the existing dataset. Start with `get_data_schema` for the current CSV contract and limits.

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
3. **Declare the quality policy.** Select or create project-specific checks before launch. The built-in metrics are `r_hat_max`, `mae`, `rmse` and `wape` on the fitted window and `prediction_mae`, `prediction_rmse` and `prediction_wape` on the saved prediction window; WAPE is a fraction. `create_quality_policy` also accepts checks on saved diagnostics, custom numeric checks, boolean checks and manual checks (a manual check is signed off by a person in the application, never through a key). There are no universal default pass thresholds.
4. **Launch explicitly.** Pass the frozen revision, a policy from the same study and a caller-generated `submission_key` to `launch_study_run`. A study budget limits attempts; it does not automatically launch that many fits.
5. **Monitor the shared run.** Read `get_study_run` and the existing model progress. A cancellation request is not confirmed cancellation; poll until the run reports its outcome. Missing heartbeat evidence is unknown, and a stall threshold is not an ETA or permission to restart a fit.
6. **Evaluate and compare.** Evaluate saved evidence, then compare candidates against one policy. Missing evidence does not pass. Fitted-window error metrics are not held-out validation; prediction-window checks read saved predictions dated after the training window and do not certify an untouched holdout. Different datasets are flagged rather than ranked together.
7. **Recommend for analyst review.** Record the candidate, evaluation and rationale with `recommend_study_run`. The frontend records analyst acceptance or rejection. A recommendation does not accept or automatically promote a model, and a passing report does not prove business validity.

Analyst-created recipes and runs are visible through the same study objects. Captured wizard recipes can be inspected, launched and edited in place: read the authoring snapshot with `get_recipe_revision_authoring`, save a draft with `create_recipe_draft` and a `target`, and publish it as the recipe's next revision. A recipe imported as a model snapshot is review-only.

## Retry and conflict recovery

Writes are sent once without automatic HTTP retries. If a response is uncertain, inspect the saved objects before repeating a mutation. For a study launch retry, reuse the **same submission key and identical revision/policy inputs**. Do not generate a new key to escape an attempt-budget or state conflict.

A stale recipe version returns **412**: reload the current recipe, reconcile the edit and submit the current `expected_version`. An effective-input hash conflict returns **409**: validate again and review the changed inputs. Error objects retain `error` and `_status_code`, with additive `_error_code` and `_next_action` guidance.

## Retrieve only the evidence you need

For model results, begin with `sections="channel_summary,model_stats"`, then request additional evidence. Filter with `channels` and `max_grid_points` where applicable. Optional `max_response_bytes` returns **413** if the filtered JSON exceeds the limit, rather than presenting partial evidence. This check happens after backend download and excludes MCP envelope overhead. Study listings (runs, recipes, evaluations, decisions) return every row unless you pass `limit`; the response then carries `next_cursor`, which you send back unchanged as `cursor` for the next page.

Releases so far have kept existing tool names, required inputs and default payloads, and have added new tools and response fields rather than changing existing ones; each release states what it keeps. See the [release notes](https://github.com/getsimba-ai/simba-mcp/releases) and the [architecture and compatibility guide](https://github.com/getsimba-ai/simba-mcp/blob/main/docs/architecture.md).

## Try a read-only check

> Inspect my connected Simba backend capabilities and list my studies for the project I select. Summarize available workflow operations and any unknown capabilities. Do not upload data, create a model or launch a run.

For setup problems, include the server version and tool names in a [public MCP issue](https://github.com/getsimba-ai/simba-mcp/issues), without credentials or private data, or contact [info@pymc-labs.com](mailto:info@pymc-labs.com).

## Next steps

### Result views and experiment guidance

These guides describe capabilities introduced in v0.16.0 and their backend and client requirements. Check each guide's availability note and the connected server's tool list:

- [Native result charts](./native-result-charts.md)
- [What to test next](./experiment-priorities.md)

### Platform guides

- [Model configuration](../platform-guide/model-configuration.md)
- [Measurement and diagnostics](../platform-guide/measurement.md)
- [Budget optimization](../platform-guide/budget-optimization.md)
- [Scenario planning](../platform-guide/scenario-planning.md)

### Core concepts

- [Bayesian modeling](../core-concepts/bayesian-modeling.md)
- [Priors and distributions](../core-concepts/priors-and-distributions.md)
- [Optimization](../core-concepts/budget-optimization.md)
