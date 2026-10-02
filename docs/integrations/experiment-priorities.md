# What to test next

> **Available in [Simba MCP v0.16.0](https://github.com/getsimba-ai/simba-mcp/releases/tag/v0.16.0), published on [PyPI](https://pypi.org/project/simba-mcp/0.16.0/).** Your connected MCP server must provide v0.16.0 or a later compatible release, and its backend must support the priority endpoint. Package publication does not upgrade a hosted server: check its version and tool list. Native-client acceptance has not yet been observed and is recorded separately.

`recommend_incrementality_tests(model_hash, budget=None, hurdle=1.0, limit=5)`
reads a saved model's marginal-return summaries and ranks channels for further
experiment investigation. It never fits a model, changes spend or starts a test.
It requires a backend with `/api/v1/models/{hash}/test-priorities` support.

See the [v0.16.0 tool reference](https://github.com/getsimba-ai/simba-mcp/blob/v0.16.0/docs/tools.md) for the versioned contract and the [connection guide](./simba-mcp.md) for setup. On a compatible Simba backend, the model's Media Results can expose **What to test next** using the same server-computed ranking; backend deployment and MCP package availability are separate prerequisites.

Example with a synthetic model you can access:

```json
{"model_hash":"YOUR_SYNTHETIC_MODEL_HASH","budget":10000,"hurdle":1.0,"limit":5}
```

The tool needs `read:results`. Test history is explicitly unavailable: registry
records do not establish compatible model and geographical coverage. No registry
information is disclosed or inferred.

## Meaning of the score

The server computes an approximate, local binary expected value of perfect
information. It compares two hypothetical marginal actions: expose an amount
of spend to a channel's marginal return, or retain its hurdle-valued alternative.
The score is expected best utility with perfect information minus the best
expected utility with current information. It uses the stored posterior **mean**,
not the median, and approximates a normal standard deviation from the width of
the stored 94% highest-density interval.

For mean `mu`, standard deviation `sigma`, hurdle `h` and exposure `S`:

```text
a = abs(mu - h) / sigma
score = S * (sigma * normal_density(a) - abs(mu - h) * normal_tail(a))
```

Zero uncertainty or zero exposure gives zero information value. Current funding
status does not determine the score. This follows the optimal-current-action
definition of information value, rather than regret from retaining a known bad
action. See [Heath, Manolopoulou and Baio, section 2](https://discovery.ucl.ac.uk/id/eprint/1467157/2/Heath_Estimating%20the%20expected%20value%20of%20partial%20perfect%20information%20in%20health%20economic%20evaluations.pdf).

`budget` is an exposure scale, **not a recommended experiment budget**. The
server distributes that scale in proportion to recorded current spend. Those
values use each channel's mean active period and need not represent one common
calendar period. Without an explicit budget, their sum is used. Zero-spend
channels receive zero exposure. No hypothetical allocation is invented.

Scores use the model's revenue units. `currency` can be absent, in which case
clients must not invent a currency symbol. This local linear approximation is
not a prediction of changing an entire channel budget. Channels are assessed
separately, ignoring joint uncertainty and portfolio constraints. A real test
provides less than perfect information, incurs costs, and can be infeasible.
Scores are not expected test benefit, detectable lift or a mandate to test.

## Inspectable evidence and limitations

Each row includes the mean, available median, approximate standard deviation,
stake, spend share, hurdle-crossing probability, original interval and reason
codes. The response always discloses the normal and HDI-width approximations.
An asymmetric posterior is not made normal by this approximation. Missing
posterior means or intervals are excluded explicitly, not replaced with zero.

Native scalar coefficient contraction is `1 - posterior variance / prior variance`
when an exact parameter match and supported prior distribution exist. Half-normal
and truncated-normal scales are converted to actual variance. Unsupported priors,
time-varying coefficients and unmatched names yield `prior_unavailable`. Contraction
only breaks ties, never multiplies the score. Recency is unavailable in this
version, so it does not alter the ranking.

Contraction is a variance comparison, not evidence that the prior dominates the
result. Values from zero to below 0.1 receive `limited_variance_contraction`;
negative values receive `posterior_variance_expanded`, because the posterior
variance exceeds the prior variance. Neither reason changes the score or the
existing contraction tie-break. The tool does not infer prior-data agreement,
conflict or identification from this single diagnostic.

Results describe one model. A recorded hierarchy identity can be a brand rather
than a geographical market, so the tool does not infer geographical coverage or
pool independent market fits. Registry history is unknown because its scope cannot
be established. `design_hint.available` is true when the model passes the designer's support
checks and false, with a reason, otherwise; no minimum detectable effect is ever
placed in the card. The design itself is calculated separately: see
[Design an incrementality test](../platform-guide/test-design.md).

## Architecture

The incrementality tool
owns only HTTP transport and public descriptions. The application owns the
scientific method, permissions and stored-artifact reads. MCP returns server
fields without recalculation. Existing registry tools and result contracts remain
unchanged. No new runtime dependency is required.
