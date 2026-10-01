# What is Simba? --- Bayesian Marketing Mix Modeling Platform

Simba is a no-code Bayesian Marketing Mix Modeling (MMM) platform built on [PyMC-Marketing](../core-concepts/bayesian-modeling.md), the leading open-source Bayesian marketing science framework. [Simba](https://getsimba.ai) makes advanced econometric modeling accessible to marketers, analysts, and agencies without requiring programming skills or a PhD in statistics.

Marketing Mix Modeling answers the fundamental question every marketer faces: **which channels are actually driving results, and how should I allocate my budget?** Simba gives you those answers with full statistical transparency, uncertainty quantification, and actionable optimization recommendations.

---

## The Problems Simba Solves

### Attribution is broken

Last-click and multi-touch attribution models rely on user-level tracking that is increasingly unreliable. Privacy regulations like GDPR, the deprecation of third-party cookies, and platform signal loss (iOS App Tracking Transparency, for example) have made digital attribution less and less trustworthy. MMM sidesteps these issues entirely by working with aggregated data --- no cookies, no pixels, no user-level tracking required.

### Black-box tools erode trust

Many marketing analytics platforms produce a single point estimate with no explanation of how they arrived at it. When a tool tells you that paid search drove 23% of revenue but cannot explain its assumptions, confidence intervals, or methodology, you are making million-dollar decisions on faith. Simba is a fully transparent platform: every model assumption, prior distribution, and posterior (the updated belief about parameters after seeing the data) result is visible and auditable.

### Traditional MMM is slow and expensive

Classic marketing mix modeling engagements involve months of consulting work, custom R or Python code, and six-figure price tags. Simba compresses this into a self-serve workflow that takes days, not months, while preserving the statistical rigor that makes MMM valuable.

### Optimization is disconnected from measurement

Most measurement tools stop at telling you what happened. Simba carries the analysis forward into [scenario planning](../platform-guide/scenario-planning.md) and [budget optimization](../platform-guide/budget-optimization.md), so you move from insight to action in a single platform.

---

## Who Simba is Built For

### Brand Marketers and Marketing Leaders

You need to justify spend, report on channel effectiveness, and make budget decisions with confidence. Simba gives you clear, defensible answers without requiring you to learn Python or Bayesian statistics. Smart defaults and the Data Validator guide you through every step.

### Agencies and Consultancies

You manage multiple clients, each with unique data and channel mixes. Simba supports multi-project environments with isolated data per client, so you can deliver rigorous MMM results faster with transparent methodology that builds trust.

### Data Science and Analytics Teams

You understand the statistics but want to move faster than building custom PyMC-Marketing pipelines from scratch. Simba gives you a configurable UI for setting [priors](../core-concepts/priors-and-distributions.md), [saturation curves](../core-concepts/saturation-curves.md), and [adstock transformations](../core-concepts/adstock-effects.md) while the PyMC-Marketing engine handles the inference. You get the rigor without the boilerplate.

### Performance Marketing Teams

You need to understand the incremental impact of each channel, including those that digital attribution consistently overvalues or undervalues. Simba quantifies true incrementality with Bayesian credible intervals (the Bayesian equivalent of a confidence interval --- a range where the true value most likely falls), giving you a statistically grounded basis for optimization.

---

## How Simba Differs from Traditional MMM Tools

| Aspect | Traditional MMM | Simba |
|---|---|---|
| **Methodology** | Frequentist regression, often opaque | Bayesian inference via PyMC-Marketing, fully transparent |
| **Uncertainty** | Single point estimates | Full posterior distributions with 94% HDI credible intervals |
| **Prior knowledge** | Ignored or ad hoc | Formally incorporated via configurable prior distributions |
| **Coding required** | Yes (R, Python, SAS) | No --- complete no-code interface |
| **Time to results** | Weeks to months | Days |
| **Model transparency** | Black box or consultant-dependent | Every assumption visible and auditable |
| **Optimization** | Separate tool or manual process | Built-in scenario planning and budget optimization |
| **Data validation** | Manual QA | AI-powered Data Validator with 10 specialized checks |
| **Updates** | Expensive re-engagement | Re-run models as new data arrives |

For a detailed comparison against specific tools (Meta's Robyn, Google's Meridian, consulting models), see the Competitor Comparison (ask us at https://getsimba.ai).

---

## The PyMC-Marketing Foundation

Simba is built on [PyMC-Marketing](../core-concepts/bayesian-modeling.md), the open-source Bayesian marketing science library maintained by the PyMC Labs team. This means:

- **Proven methodology.** PyMC-Marketing implements peer-reviewed Bayesian media mix modeling techniques used by leading data science teams worldwide.
- **Active development.** The underlying engine benefits from continuous improvements by a global open-source community.
- **No vendor lock-in.** The statistical methodology is open-source and transparent. Simba adds the no-code interface, workflow automation, AI-driven defaults, security infrastructure, and optimization engine on top.
- **Extensibility.** For teams that want to go deeper, the PyMC-Marketing foundation means your models are compatible with the broader PyMC ecosystem.

Learn more about the Bayesian approach in [Bayesian Modeling in Marketing](../core-concepts/bayesian-modeling.md).

---

## The Simba Workflow

Simba is organized around four main areas, accessible from the sidebar navigation:

### 1. Model Warehouse

Your central hub for data management and model creation. Upload your CSV data, run the **Data Validator** to check for quality issues, configure your model through a 5-step wizard, and manage all your saved models, projects, and portfolios.

Read more: [Model Creation Wizard](../platform-guide/model-creation-wizard.md) | [Data Validator](../platform-guide/data-auditor.md)

### 2. Active Model

The results hub for your fitted model. Explore channel contributions, response curves, ROAS (Return on Ad Spend), coefficients, model diagnostics, and more across multiple analysis tabs.

Read more: [Incremental Measurement](../platform-guide/measurement.md)

### 3. Scenario Planner

Create custom budget plans and forecast their revenue impact. Choose between the Monthly Planner (a guided 6-step wizard) or the Advanced Planner (a manual grid editor for granular control). Generate predictions with 94% HDI (3%-97%) uncertainty bands.

Read more: [Scenario Planning](../platform-guide/scenario-planning.md)

### 4. Budget Optimizer

Algorithmically find the optimal budget allocation across channels. Configure risk tolerance, channel constraints, spend timing, and revenue multipliers through a guided wizard. The optimizer maximizes risk-adjusted expected revenue using the full Bayesian posterior.

Read more: [Budget Optimization](../platform-guide/budget-optimization.md)

### Additional Capabilities

- **Smart Defaults** --- Auto-generated prior configurations from your historical data. Run a credible model immediately, then refine as you learn more. Read more: [Smart Defaults](../platform-guide/smart-defaults.md)
- **VAR Modeling** --- Vector AutoRegression for understanding inter-channel dynamics and long-term brand effects. Read more: [VAR Models](../platform-guide/var-models.md)
- **Portfolio Analysis** --- Cross-brand comparison and optimization with halo and trademark channel support. Read more: [Portfolio Analysis](../platform-guide/portfolio-analysis.md)

### Security and Compliance

Your data is protected with encryption at rest and in transit, access isolated per project, and compliance with applicable data protection standards. Two-factor authentication (2FA) and SSO (Google, Microsoft) are available for account security.

Read more: [Security Overview](../security/README.md)

---

## How Simba Fits Into Your Marketing Stack

Simba is not a replacement for your ad platforms, analytics tools, or BI dashboards. It sits alongside them as your **strategic measurement and optimization layer**:

- **Ad platforms** (Google Ads, Meta, etc.) tell you what happened inside their walled gardens.
- **Web analytics** (GA4, Adobe Analytics) track on-site behavior.
- **Simba** tells you the true incremental impact of each channel on your business KPIs, accounts for offline and external factors, and optimizes your budget allocation across all channels.

Simba works with aggregated, time-series data --- typically weekly or daily summaries of spend, impressions, and KPIs. You do not need to connect APIs or share user-level data.

---

## Next Steps

Ready to get started? Head to the [Quick Start Guide](quick-start-guide.md) and build your first marketing mix model.

Or, if you want to set up your account first, see [Account Setup](account-setup.md).

---

## Related Documentation

- [Quick Start Guide](quick-start-guide.md) --- Build your first model step by step
- [Platform Overview](platform-overview.md) --- Navigate the Simba interface
- [Bayesian Modeling in Marketing](../core-concepts/bayesian-modeling.md) --- Understand the methodology
- [Data Requirements](../data/data-requirements.md) --- What data you need to get started
- [Pricing and Plans](../pricing/README.md) --- Plan comparison and pricing details

**Comparing Simba to alternatives?** See the Competitor Comparison (ask us at https://getsimba.ai) for detailed analysis against open-source libraries, SaaS platforms, and consulting firms.
