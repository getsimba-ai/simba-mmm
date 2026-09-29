# Usage Tracking --- Understanding Plan Limits and Usage

Simba tracks usage of key platform actions against your plan's limits. Understanding how usage is tracked helps you plan your modeling activities and avoid hitting limits at critical moments.

> **Latest pricing and plans**: Visit [getsimba.ai](https://getsimba.ai) for current plan details and pricing.

---

## What Is Tracked

Simba meters the following actions:

| Resource | Type | Description |
|---|---|---|
| **Saved models** | Cumulative | Total completed models retained in your account (not per-period) |
| **Model fits** | Per-period | Each new model fitting run |
| **Optimizations** | Per-period | Each budget optimization run (single-brand or portfolio) |
| **Scenarios** | Per-period | Each scenario planning simulation |
| **Refits** | Per-period | Re-running an existing model with updated data or configuration |
| **VAR models** | Per-period | VAR model creation (counts toward model fit limits, tier-dependent availability) |
| **Portfolios** | Cumulative | Total number of portfolios (not per-period) |

**Per-period** resources reset every 30 days at the start of your billing period. **Cumulative** resources count toward a total limit that does not reset.

---

## Plan Limits

| Resource | Trial | Analyst | Pro | Scale |
|---|---|---|---|---|
| **Saved models** | 20 | 1 | 5 | 20 |
| **Model fits/period** | 10 | 1 | 5 | Unlimited |
| **Optimizations/period** | 10 | 6 | 8 | Unlimited |
| **Scenarios/period** | 10 | 6 | 8 | Unlimited |
| **Refits/period** | 0 | 1 | 5 | Unlimited |
| **VAR models** | ✅ Enabled | ❌ Disabled | ✅ Enabled | ✅ Enabled |
| **Portfolios** | 0 | 0 | 1 | 2 |

> **Latest plans and pricing**: Visit [getsimba.ai](https://getsimba.ai) for current details. Plans and limits may change.

---

## Billing Periods

Usage counters for per-period resources (model fits, optimizations, scenarios, refits) reset every **30 days** from your subscription start date. Your billing period aligns with when you first subscribed or upgraded.

To see your current billing period and days until reset, navigate to **Profile > Billing**.

---

## Trial Details

### 28-Day Free Trial

Every new account starts with a **28-day free trial** that includes full platform access:

- 20 saved models, 10 model fits, 10 optimizations, 10 scenarios
- VAR models enabled
- Full access to [Data Validator](./data-auditor.md), [Smart Defaults](./smart-defaults.md), and all analysis tabs
- No credit card required

### 3-Day Grace Period

When your 28-day trial expires, you enter a **3-day grace period**:

- You can **view** existing models and results
- Actions are still permitted with **strong warning messages** urging you to upgrade
- All data and model configurations are preserved

After the grace period expires, model creation, optimizations, scenarios, and refits are **blocked** until you upgrade to a paid plan. Your data and models are retained — nothing is deleted.

---

## Checking Your Usage

To view your current usage:

1. Navigate to **Profile > Billing** (the Subscription tab in your profile).
2. The usage dashboard shows:
   - **Usage meters** for each resource: current count, plan limit, remaining allocation, and percentage bar
   - **Days Until Reset** for the current billing period
   - **Plan Features** summary showing all limits for your current tier
3. The **Usage** tab shows historical activity: total counts, today's activity, completion rates, and 30-day trend charts.

---

## What Happens When You Hit a Limit

Simba uses **soft enforcement** for most limits:

- When you reach a per-period limit, actions are **still allowed** but you receive a **warning message** indicating you've exceeded your allocation
- The warning appears in the API response and is visible in the usage meters ("at limit" indicator)
- You can still use other features that are within their limits
- Usage resets automatically at the start of your next billing period
- Alternatively, **upgrade your plan** to immediately increase your limits

**Hard enforcement** only applies when a trial (and its grace period) has fully expired — in that case, model creation, optimizations, scenarios, and refits are blocked until you upgrade.

---

## Unsaved Model Cleanup

Simba automatically manages unsaved models to keep your account organized:

- A maximum of **10 unsaved models** can exist at any time
- When an 11th unsaved model is created, the **oldest unsaved model is automatically deleted**
- **Saved models** (marked as saved by you) are never automatically deleted
- This cleanup only affects unsaved/draft models, not your finalized work

---

## Tips for Managing Usage

- **Use [Smart Defaults](./smart-defaults.md) and prior predictive checks** to validate your configuration before fitting. This reduces wasted fits on misconfigured models.
- **Save [optimization](./budget-optimization.md) runs** for when you have finalized your model. Run [scenarios](./scenario-planning.md) first to explore "what if" questions before committing to a formal optimization.
- **Clone models** instead of creating new ones from scratch when iterating on configurations.
- **Save models you want to keep** — unsaved models are automatically cleaned up after 10 accumulate.
- **Monitor your usage** via Profile > Billing, especially mid-period when you may need to pace your remaining allocations.

---

## Next Steps

**Platform guides:**
- [Model Creation Wizard](./model-creation-wizard.md) --- Building models (uses model fit quota)
- [Budget Optimization](./budget-optimization.md) --- Optimization runs (uses optimization quota)
- [Scenario Planning](./scenario-planning.md) --- Scenario simulations (uses scenario quota)
- [Sharing and Collaboration](./sharing-collaboration.md) --- Team and sharing features

**Getting started:**
- [Account Setup](../getting-started/account-setup.md) --- Registration and plan management
