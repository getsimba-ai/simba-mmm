# Troubleshooting --- Common Issues and How to Fix Them

This page covers the most common issues users encounter in Simba, organized by category. For each issue: what you see, why it happens, and how to fix it.

---

## Data Upload Issues

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Upload fails with "invalid file" | File is not CSV, or exceeds 10 MB limit | Save as `.csv` (not `.xlsx`). If file is too large, aggregate to weekly granularity or reduce the date range. See [Data Requirements](../data/data-requirements.md). |
| Columns not detected correctly | Column names don't match expected patterns | Rename columns to clear names (e.g., `tv_spend`, `revenue`). The [semantic matcher](smart-defaults.md) works best with descriptive names. |
| Date column not recognized | Date format is unusual or inconsistent | Use `YYYY-MM-DD` format. Simba supports 10+ formats but inconsistent formats within a column will fail. See [Data Preparation](../data/data-preparation.md). |
| "Too few observations" warning | Fewer than ~52 rows of data | The model needs at least 1 year of data (52 weekly rows or equivalent) to estimate seasonality and media effects reliably. Ideally use 2+ years. |

---

## Data Validator Issues

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| High multicollinearity warning | Two or more channels always spend together | Consider combining correlated channels into one column, or remove one. The model cannot distinguish effects of channels that always co-move. See [Data Validation](../data/data-validation.md). |
| Outlier warnings on revenue | Genuine business events (sales, promotions) | Review each flagged outlier. If it's a real event (e.g., Black Friday), keep it — the model handles it. If it's a data error, fix it. |
| Coverage warning | A channel has too many zero-spend periods | If a channel was only active for a few weeks, the model may not have enough data to estimate its effect. Consider removing channels with <10% non-zero periods. |
| Schema validation errors | Blank cells, text in numeric columns, duplicates | Fix the data in your CSV: fill blanks with 0 (for spend) or remove rows, ensure all spend/revenue columns are numeric, remove duplicate dates. |

---

## Model Fitting Issues

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Model fitting takes very long (>2 hours) | Too many channels, daily data, or TVP priors | Reduce the number of channels (combine small ones), switch from daily to weekly data, or switch TVP priors back to static priors. TVP adds significant computation. |
| Model fails / does not converge | Prior-data conflict, extreme outliers, or insufficient data | Check the error message. Common causes: (1) priors too tight — loosen sigma values, (2) extreme outliers distorting the likelihood — remove or cap them, (3) too few data points for the number of parameters. See [Priors and Distributions](../core-concepts/priors-and-distributions.md). |
| Low effective sample size (ESS) warnings | Posterior is difficult to sample | Try increasing the number of MCMC samples (if available in advanced settings), or simplify the model (fewer channels, fewer TVP parameters). This is a sign the model is complex relative to the data. |
| R-hat warnings (>1.05) | Chains have not converged | The model needs more samples or simpler specification. Try: (1) loosening very tight priors, (2) removing highly correlated channels, (3) ensuring data has enough variation. Re-fit after changes. |

---

## Results Look Wrong

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| A channel shows negative contribution | The model estimates that channel is cannibalizing sales | Check if the channel's prior allows negative values (Normal distribution). If you believe the channel must have a positive effect, switch to InverseGamma or TruncatedNormal with lower bound 0. Also check for data issues (lag misalignment, wrong metric). |
| All channels show near-zero effect | Base demand is absorbing everything | This can happen with strong seasonality or trend that explains most variance. Try: (1) enable seasonality/trend controls to separate them from media, (2) check that your media data has enough variation (channels that are always "on" are hard to measure). |
| One channel dominates unreasonably | Prior too strong, or channel is confounded with seasonality | Review the prior for that channel — is sigma too small (overly confident)? Check if the channel's spend pattern closely matches seasonal peaks (e.g., always spending most in Q4 when sales are naturally high). |
| ROAS seems too high or too low | Scale mismatch or prior issue | Verify that spend and revenue are in the same currency and units. A spend column in thousands and revenue in dollars will produce inflated ROAS. Also check the prior — a very narrow InverseGamma can constrain the coefficient. |
| Wide credible intervals on everything | Insufficient data or too many parameters | The model is uncertain. This is actually correct behavior — it's telling you there isn't enough data to be precise. Consider: (1) adding more historical data, (2) incorporating [lift test](../core-concepts/incrementality.md) observations to sharpen key channels, (3) reducing the number of channels. |

---

## Optimizer Issues

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Optimizer puts everything in one channel | That channel has the steepest response curve at current spend | This is the optimizer working correctly — it's found the highest marginal return. If you want diversification, add minimum spend constraints per channel, or increase gamma (risk aversion) to penalize concentration in uncertain channels. |
| Optimizer recommends no change | Current allocation is already near-optimal, or gamma is very high | Try gamma = 0 (risk-neutral) to see pure optimization. If still no change, your current allocation may genuinely be close to optimal. |
| "Infeasible" error | Constraints conflict with each other | Check that your minimum spend constraints across all channels don't exceed the total budget, and that maximum constraints leave room for allocation. Loosen constraints. |
| Optimized revenue lower than current | Possible with high gamma and high uncertainty | The risk-adjusted objective (`mean - gamma * std`) penalizes uncertain channels. Try lowering gamma. Or this may indicate that some channels have very uncertain estimates — the optimizer is being conservative. See [Budget Optimization](../core-concepts/budget-optimization.md). |

---

## Scenario Planner Issues

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Scenario shows no revenue change | The modified channel has a flat [saturation curve](../core-concepts/saturation-curves.md) at current spend | Check the channel's response curve — if it's already in the flat region, adding more spend won't help. Try reducing spend instead to see the downside. |
| Scenario revenue drops when adding spend | Reallocation from a more effective channel, or [adstock](../core-concepts/adstock-effects.md) timing effects | If you increased one channel by reducing another, the net effect depends on which channel has higher marginal returns. Check the marginal ROAS comparison. |
| Can't create a new scenario | Scenario limit reached on Trial plan | The Trial plan includes 10 scenario runs. Upgrade to Enterprise or Managed for unlimited scenarios. See [Pricing](../pricing/README.md). |

---

## Account & Access Issues

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Can't log in | Wrong credentials or SSO not configured | Try password reset. For SSO, contact your admin to ensure your email domain is configured. See [Account Setup](../getting-started/account-setup.md). |
| Features are greyed out / locked | Plan limitations | VAR is enabled on Trial, Pro and Scale, but not Analyst. Portfolio availability and limits also depend on your plan. Check [Pricing](../pricing/README.md) and your account usage for the applicable limits. |
| Model disappeared | Accidental deletion or plan expiry | Check the Model Warehouse archive. If your trial expired, models are preserved in read-only mode — upgrade to regain editing access. |

---

## Still Stuck?

If your issue isn't covered above:

1. **Check the [FAQ](../faq/README.md)** for general questions
2. **Open a GitHub issue** at [github.com/nialloulton/simba-mmm/issues](https://github.com/nialloulton/simba-mmm/issues) for documentation gaps
3. **Email support** at [info@pymc-labs.com](mailto:info@pymc-labs.com) for platform issues
4. **Book a call** at [calendly.com/simba-onboarding](https://calendly.com/simba-onboarding) for hands-on help
