# SIMBA — Bayesian Marketing Mix Modeling (MMM) Platform

[![Website](https://img.shields.io/badge/Website-getsimba.ai-blue)](https://getsimba.ai)
[![License](https://img.shields.io/badge/License-All%20Rights%20Reserved-red)](#license)
[![Built With](https://img.shields.io/badge/Built%20on-PyMC-orange)](https://www.pymc.io/)
[![Security](https://img.shields.io/badge/Security-Cyber%20Essentials%20Certified-green)](#security--compliance)

**Simba is a no-code Bayesian Marketing Mix Modeling platform that measures media effectiveness, optimizes budgets, and forecasts marketing ROI.** Replace spreadsheets, fragmented models, and black-box vendors with one transparent, enterprise-ready platform.

Built on the open-source [PyMC](https://www.pymc.io/) framework by [PyMC Labs](https://www.pymc-labs.com/), the team behind [PyMC-Marketing](https://www.pymc-marketing.io/), Simba combines the rigor of Bayesian statistics with an intuitive no-code interface — giving marketing teams enterprise-grade marketing mix modeling without writing a single line of code.

---

## What is Marketing Mix Modeling?

Marketing Mix Modeling (MMM) is a statistical technique that measures the impact of marketing activities on business outcomes like revenue and conversions. Unlike last-click attribution or multi-touch attribution (MTA), MMM uses aggregate data to isolate the **incremental contribution** of each media channel — accounting for diminishing returns, carryover effects, seasonality, and external factors.

Simba makes MMM accessible to marketing teams who need rigorous measurement without hiring a data science team. See [What is Marketing Mix Modeling?](docs/core-concepts/marketing-mix-modeling.md) for a full explanation.

---

## Why Choose Simba for Marketing Mix Modeling?

| Challenge | How Simba Solves It |
|---|---|
| Black-box MMM vendors deliver "trust me" results | Fully transparent — inspect every prior, parameter, and assumption |
| Custom MMM models take months to build and maintain | No-code model configuration with smart defaults — first model in 15 minutes |
| Fragmented tools for measurement, planning, and optimization | End-to-end platform: validate data, measure impact, forecast scenarios, optimize budgets |
| One-size-fits-all models ignore domain expertise | Bayesian priors let you encode business knowledge directly into the model |
| Siloed models across brands and markets | Portfolio modeling for cross-brand and cross-market consistency |

---

## Key Features

### Shared Studies and MCP Integration
Connect compatible AI clients to the same studies, recipe revisions, model runs and quality evidence used by analysts. The generated [tool reference](https://github.com/getsimba-ai/simba-mcp/blob/main/docs/tools.md) lists every tool of the current release, with capability discovery for the connected backend and analyst acceptance in the frontend. See the [Simba MCP integration guide](docs/integrations/simba-mcp.md) for setup, retry safety and connection refresh instructions.

### Media Measurement & Attribution
Measure the true incremental impact of every marketing channel using Bayesian causal attribution. Integrate lift test results as likelihood observations to calibrate and validate your model. See [Incremental Measurement](docs/platform-guide/measurement.md).

### Budget Optimization
Risk-adjusted budget allocation that accounts for saturation (diminishing returns), adstock (carryover effects), and uncertainty. Optimize across channels with configurable risk tolerance. See [Budget Optimization](docs/platform-guide/budget-optimization.md).

### Scenario Planning & Forecasting
Test budget scenarios before spending. Single-scenario prediction with uncertainty bands, what-if analysis, and carryover-aware forecasting. See [Scenario Planning](docs/platform-guide/scenario-planning.md).

### No-Code Model Configuration
Configure Bayesian priors, saturation curves, and adstock decay through an intuitive UI. Smart defaults auto-generate starting points based on your data and industry benchmarks. See [Model Configuration](docs/platform-guide/model-configuration.md).

### Automated Data Validation
An AI-powered Data Validator checks your data across 10 validation categories before modeling — detecting anomalies, missing values, multicollinearity, and data quality issues. See [Data Validator](docs/platform-guide/data-auditor.md).

### Portfolio & Multi-Brand Modeling
Cross-brand and cross-client modeling for agencies and multi-brand organizations. Consistent methodology, comparable KPIs, centralized management. See [Portfolio Modeling](docs/use-cases/portfolio-modeling.md).

### Long-Term Effects (Bayesian VAR)
Measure long-term brand effects using Bayesian Vector Autoregression — impulse response functions, forecast error variance decomposition, and long-run equilibrium effects. See [Long-Term Effects](docs/platform-guide/long-term-effects.md).

---

## How Simba Works

Simba provides a complete workflow for marketing mix modeling:

**1. Validate** — The [Data Validator](docs/platform-guide/data-auditor.md) automatically audits your data for quality issues before modeling.

**2. Configure** — Set up your model using [no-code configuration](docs/platform-guide/model-configuration.md) with [smart defaults](docs/platform-guide/smart-defaults.md) or custom Bayesian priors.

**3. Measure** — Run the model and get [incremental measurement](docs/platform-guide/measurement.md) of every channel's contribution to revenue.

**4. Optimize** — Use [budget optimization](docs/platform-guide/budget-optimization.md) and [scenario planning](docs/platform-guide/scenario-planning.md) to allocate spend for maximum ROI.

---

## The Bayesian Advantage

Simba uses Bayesian Marketing Mix Modeling rather than frequentist regression. This matters because:

- **Uncertainty quantification** — every estimate comes with a 94% HDI (Highest Density Interval), so you know how confident to be in each channel's ROI
- **Prior knowledge** — encode domain expertise (e.g., "TV has longer carryover than paid search") directly into the model
- **Lift test calibration** — integrate experimental results (lift tests, geo tests) as likelihood observations to validate and improve model accuracy
- **Small data friendly** — Bayesian models produce reliable estimates even with limited historical data
- **Fully transparent** — built on open-source [PyMC](https://www.pymc.io/), so every model component is inspectable and auditable

Learn more: [Bayesian Modeling Explained](docs/core-concepts/bayesian-modeling.md) | [Priors & Distributions](docs/core-concepts/priors-and-distributions.md)

---

## Documentation

### Getting Started
- [What is Simba?](docs/getting-started/what-is-simba.md) — Product overview and positioning
- [Quick Start Guide](docs/getting-started/quick-start-guide.md) — Build your first marketing mix model in 15 minutes
- [Account Setup](docs/getting-started/account-setup.md) — Registration, plans, and project configuration
- [Platform Overview](docs/getting-started/platform-overview.md) — UI walkthrough and navigation

### Core Concepts
- [Marketing Mix Modeling](docs/core-concepts/marketing-mix-modeling.md) — What MMM is and why it matters
- [Bayesian Modeling](docs/core-concepts/bayesian-modeling.md) — The Bayesian approach to media measurement
- [Incrementality](docs/core-concepts/incrementality.md) — Causal attribution and incremental measurement
- [Saturation Curves](docs/core-concepts/saturation-curves.md) — Diminishing returns and response curves
- [Adstock Effects](docs/core-concepts/adstock-effects.md) — Carryover, memory decay, and lagged impact
- [Priors & Distributions](docs/core-concepts/priors-and-distributions.md) — Configuring Bayesian priors
- [Seasonality](docs/core-concepts/seasonality.md) — Seasonal patterns and trend modeling

### Platform Guide
- [Data Validator](docs/platform-guide/data-auditor.md) — Automated data validation and quality scoring
- [Model Configuration](docs/platform-guide/model-configuration.md) — Configuring priors, saturation curves, and adstock decay
- [Smart Defaults](docs/platform-guide/smart-defaults.md) — Auto-generated model starting points
- [Incremental Measurement](docs/platform-guide/measurement.md) — Channel attribution and contribution analysis
- [Budget Optimization](docs/platform-guide/budget-optimization.md) — Risk-adjusted budget allocation
- [Scenario Planning](docs/platform-guide/scenario-planning.md) — Forecasting and what-if analysis
- [Long-Term Effects](docs/platform-guide/long-term-effects.md) — Bayesian VAR for brand equity modeling

### Data
- [Data Requirements](docs/data/data-requirements.md) — What data you need and supported formats
- [Data Preparation](docs/data/data-preparation.md) — Cleaning and formatting best practices
- [Data Validation](docs/data/data-validation.md) — How the Data Validator audits your data
- [Supported Channels](docs/data/supported-channels.md) — TV, digital, social, OOH, and more

### Use Cases
- [Brand Marketers](docs/use-cases/brand-marketers.md) — For in-house marketing teams
- [Agencies](docs/use-cases/agencies.md) — Multi-client management and portfolio modeling
- [Portfolio Modeling](docs/use-cases/portfolio-modeling.md) — Cross-brand and cross-market analysis
- [Retail & E-commerce](docs/use-cases/retail-and-ecommerce.md) — Online and omnichannel retail

### Security & Compliance
- [Security Overview](docs/security/README.md) — AES-256 encryption, TLS 1.2+, Cyber Essentials certified, GDPR compliant

### Resources
- [Glossary](resources/glossary.md) — Marketing mix modeling and Bayesian statistics terminology
- [PyMC-Marketing & Simba](resources/pymc-marketing.md) — How the open-source project and platform relate
- [Further Reading](resources/further-reading.md) — Papers, articles, and external resources
- [FAQ](docs/faq/README.md) — Frequently asked questions
- [Pricing & Plans](docs/pricing/README.md) — See [getsimba.ai](https://getsimba.ai) for current plans

---

## Simba vs. Other MMM Solutions

| Capability | Simba | Google Meridian | Meta Robyn | Custom In-House |
|---|---|---|---|---|
| No-code UI | Yes | No (Python) | No (R) | No |
| Bayesian framework | Yes (PyMC) | Yes (lightweight Bayesian) | Ridge regression | Varies |
| Uncertainty quantification | 94% HDI on all outputs | Limited | No | Varies |
| Budget optimization | Built-in, risk-adjusted | Separate | Basic | Build your own |
| Lift test integration | Yes (likelihood observations) | Yes | Yes (calibration) | Build your own |
| Portfolio / multi-brand | Built-in | No | No | Build your own |
| Long-term effects (VAR) | Built-in (Bayesian VAR) | No | No | Build your own |
| Enterprise security | Cyber Essentials, GDPR | Google Cloud | Self-hosted | Self-managed |
| Time to first model | 15 minutes | Days–weeks | Days–weeks | Months |

See [full competitor comparison](sales/competitor-comparison.md) for details.

---

## Getting Help

- **[GitHub Issues](https://github.com/nialloulton/simba-mmm/issues)** — Bug reports, feature requests, and support questions
- **Email**: info@pymc-labs.com
- **Website**: [getsimba.ai](https://getsimba.ai)
- **Start free**: [demo.simba-mmm.com/signup](https://demo.simba-mmm.com/signup)
- **Book a demo**: [Schedule a call](https://calendly.com/niall-oulton)

---

## Built on PyMC

Simba's models are written in [PyMC](https://www.pymc.io/), the open-source probabilistic programming framework, by [PyMC Labs](https://www.pymc-labs.com/), the team behind [PyMC-Marketing](https://www.pymc-marketing.io/). The engine is Simba's own; it does not import PyMC-Marketing. This means:

- **Full transparency** — the probabilistic models driving your ROI are inspectable and auditable
- **Scientific rigor** — built on decades of Bayesian statistics research
- **No vendor lock-in** — your modeling logic is built on open-source foundations
- **Community-driven** — benefit from continuous improvements by the PyMC community

---

## License

Copyright 2026 1749 Ltd. All rights reserved.

This repository and its contents are proprietary. See [LICENSE](LICENSE) for details.

[Privacy Policy](https://getsimba.ai/privacy) | [Terms of Service](https://getsimba.ai/terms)

---

<sub>[Simba](https://getsimba.ai) — Bayesian Marketing Mix Modeling platform. Built on [PyMC](https://www.pymc.io/) by [PyMC Labs](https://www.pymc-labs.com/), the team behind [PyMC-Marketing](https://www.pymc-marketing.io/).</sub>
