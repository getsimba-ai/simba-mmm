# Simba Documentation

**The complete guide to Bayesian Marketing Mix Modeling with Simba** --- from first upload to optimized budget recommendations.

Simba is a no-code MMM platform built on [PyMC-Marketing](https://www.pymc-marketing.io/). Every model is fully transparent, every prior is configurable, and every result includes calibrated uncertainty intervals.

---

## Quick Navigation

| I want to... | Start here |
|---|---|
| Understand what Simba does | [What is Simba?](./getting-started/what-is-simba.md) |
| Build my first model | [Quick Start Guide](./getting-started/quick-start-guide.md) |
| Connect agents to shared studies and models | [Simba MCP integration guide](./integrations/simba-mcp.md) |
| Report sales and media data for any period | [Report sales and media data](./integrations/report-sales-and-media-data.md) |
| Connect my data warehouse or lake | [Connect your warehouse](./integrations/connect-your-warehouse.md) |
| Keep my data refreshed automatically | [Refresh data on a schedule](./integrations/refresh-data-on-a-schedule.md) |
| Prepare and format my data | [Data Requirements](./data/data-requirements.md) |
| Configure model priors and settings | [Model Configuration](./platform-guide/model-configuration.md) |
| Understand my channel results | [Incremental Measurement](./platform-guide/measurement.md) |
| Optimize my media budget | [Budget Optimization](./platform-guide/budget-optimization.md) |
| Forecast a what-if scenario | [Scenario Planning](./platform-guide/scenario-planning.md) |
| Learn the Bayesian methodology | [Bayesian Modeling](./core-concepts/bayesian-modeling.md) |
| Troubleshoot an issue | [Troubleshooting](./platform-guide/troubleshooting.md) |

---

## Getting Started

New to Simba? Start here.

- **[What is Simba?](./getting-started/what-is-simba.md)** --- Platform overview, who it's for, and how it compares to other MMM tools
- **[Quick Start Guide](./getting-started/quick-start-guide.md)** --- Build your first marketing mix model step by step
- **[Your First Model (Tutorial)](./getting-started/first-model-tutorial.md)** --- Hands-on walkthrough with sample data
- **[Account Setup](./getting-started/account-setup.md)** --- Registration, plans, and project configuration
- **[Platform Overview](./getting-started/platform-overview.md)** --- Navigating the Simba interface: Model Warehouse, Active Model, Optimization, and Scenario Planner

---

## Core Concepts

The statistical and marketing foundations behind every Simba model. Read these to understand *why* the platform works the way it does.

| Concept | What you'll learn |
|---|---|
| [Marketing Mix Modeling](./core-concepts/marketing-mix-modeling.md) | What MMM is, how it differs from MTA, and why Bayesian MMM is the standard |
| [Bayesian Modeling](./core-concepts/bayesian-modeling.md) | Priors + likelihood = posterior; uncertainty quantification; 94% HDI |
| [Incrementality](./core-concepts/incrementality.md) | Causal attribution, base vs. incremental revenue, lift test calibration |
| [Saturation Curves](./core-concepts/saturation-curves.md) | Diminishing returns via `tanh` saturation with scalar and alpha parameters |
| [Adstock Effects](./core-concepts/adstock-effects.md) | Carryover and decay --- geometric and delayed adstock (always applied before saturation) |
| [Priors & Distributions](./core-concepts/priors-and-distributions.md) | Normal, InverseGamma, TruncatedNormal, TVP --- configuring what the model believes before seeing data |
| [Seasonality](./core-concepts/seasonality.md) | Fourier seasonality, HSGP trend, GP-smoothed events --- all opt-in |
| [Budget Optimization](./core-concepts/budget-optimization.md) | Marginal equalization, mean-variance optimization, gamma risk aversion, posterior-aware allocation |
| [Halo Effects](./core-concepts/halo-effects.md) | Cross-brand lift: halo channels (fixed 0.005 coefficient) and trademark channels (75% prior reduction) |
| [VAR Modeling](./core-concepts/var-modeling.md) | Bayesian VAR with Minnesota priors --- impulse response, FEVD, and long-run multipliers |

---

## Platform Guide

Step-by-step guides for every feature in the Simba interface.

### Model Setup

- **[Model Creation Wizard](./platform-guide/model-creation-wizard.md)** --- The 5-step wizard: source config, variable selection, prior builder, model setup, and model details
- **[Model Configuration](./platform-guide/model-configuration.md)** --- Deep reference for priors, saturation curves, adstock decay, and variable transformations
- **[Smart Defaults](./platform-guide/smart-defaults.md)** --- How auto-generated starting points are derived from your data and industry benchmarks
- **[Halo & Trademark Channels](./platform-guide/halo-trademark-channels.md)** --- Configuring cross-brand effects for portfolio analysis

### Measurement & Analysis

- **[Incremental Measurement](./platform-guide/measurement.md)** --- Channel contributions, response curves, ROAS, posterior diagnostics, and contribution groups
- **[Long-Term Effects](./platform-guide/long-term-effects.md)** --- Brand equity modeling with Bayesian VAR
- **[VAR Models](./platform-guide/var-models.md)** --- Building and interpreting Vector AutoRegression models
- **[Portfolio Analysis](./platform-guide/portfolio-analysis.md)** --- Cross-brand comparison, portfolio-level optimization, and consistent KPIs

### Planning & Optimization

- **[Budget Optimization](./platform-guide/budget-optimization.md)** --- Risk-adjusted spend allocation with carryover awareness and per-channel constraints
- **[Scenario Planning](./platform-guide/scenario-planning.md)** --- What-if forecasting with uncertainty bands, ROAS analysis, and revenue decomposition

### Data Quality

- **[Data Validator](./platform-guide/data-auditor.md)** --- AI-powered validation across 10 categories: schema, frequency, alignment, outliers, multicollinearity, and more

### Collaboration & Reporting

- **[Exports & Reporting](./platform-guide/exports-reporting.md)** --- PDF reports, CSV data exports, and chart downloads
- **[Sharing & Collaboration](./platform-guide/sharing-collaboration.md)** --- Model sharing, team management, and project organization
- **[Usage Tracking](./platform-guide/usage-tracking.md)** --- Plan limits, billing periods, and usage management

### Help

- **[Troubleshooting](./platform-guide/troubleshooting.md)** --- Data upload issues, model convergence, unexpected results, and optimizer problems

---

## Data

Everything about preparing data for Simba --- from export to upload.

- **[Data Requirements](./data/data-requirements.md)** --- What you need: CSV format (50MB max), date column, target KPI, media + cost pairs, 52+ weeks recommended
- **[Data Preparation](./data/data-preparation.md)** --- Cleaning, formatting, handling missing values, and time alignment
- **[Data Validation](./data/data-validation.md)** --- How the Data Validator's 10 automated checks assess your data quality
- **[Exporting from Ad Platforms](./data/exporting-from-platforms.md)** --- Google Ads, Meta, GA4, TikTok, DV360, TV, and OOH export guides
- **[Supported Channels](./data/supported-channels.md)** --- 15 auto-detected channel categories and 4 metric types (Spend, Impressions, GRPs, Clicks)

---

## Use Cases

How different teams and industries use Simba.

- **[Brand Marketers](./use-cases/brand-marketers.md)** --- In-house teams measuring and optimizing cross-channel ROI
- **[Agencies](./use-cases/agencies.md)** --- Multi-client management with portfolio modeling (includes Growth Dynamics case study)
- **[Portfolio Modeling](./use-cases/portfolio-modeling.md)** --- Cross-brand and cross-market analysis with halo effects
- **[Retail & E-Commerce](./use-cases/retail-and-ecommerce.md)** --- Omnichannel measurement, promotional impact, and seasonal planning

---

## Security & Compliance

- **[Security Overview](./security/README.md)** --- AES-256 encryption at rest, TLS 1.3 in transit, Cyber Essentials certified, GDPR compliant

---

## Additional Resources

- **[Glossary](../resources/glossary.md)** --- Definitions for MMM and Bayesian statistics terminology
- **[PyMC-Marketing & Simba](../resources/pymc-marketing.md)** --- How the open-source foundation and platform relate
- **[Further Reading](../resources/further-reading.md)** --- Academic papers, articles, and external resources
- **[FAQ](./faq/README.md)** --- Frequently asked questions
- **[Pricing & Plans](./pricing/README.md)** --- Current plan details at [getsimba.ai](https://getsimba.ai)

---

## How Simba Works

```
 CSV Upload ──→ Data Validator ──→ Model Configuration ──→ Bayesian Fitting
                 (10 checks)       (priors, adstock,        (PyMC-Marketing)
                                    saturation, trend)
                                                                  │
                ┌─────────────────────────────────────────────────┘
                ▼
         Active Model ──→ Budget Optimization ──→ Scenario Planning
         (contributions,    (risk-adjusted,         (what-if forecasts,
          response curves,   per-channel bounds,     uncertainty bands,
          ROAS, 94% HDI)     portfolio-level)        carryover-aware)
```

**Every output includes uncertainty.** Simba uses the full posterior distribution (~3,000 samples) for optimization and forecasting --- not point estimates.

---

## Getting Help

- **[GitHub Issues](https://github.com/getsimba-ai/simba-mmm/issues)** --- Bug reports, feature requests, documentation feedback
- **Email**: info@pymc-labs.com
- **Website**: [getsimba.ai](https://getsimba.ai)
- **Book a demo**: [Schedule a call](https://getsimba.ai)

---

<sub>Built on [PyMC-Marketing](https://www.pymc-marketing.io/) by [PyMC Labs](https://www.pymc-labs.com/). Documentation for [Simba](https://getsimba.ai) --- Bayesian Marketing Mix Modeling platform.</sub>
