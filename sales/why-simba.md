# Why Simba? — Value Proposition and Differentiators

## The Problem Simba Solves

Marketing teams are drowning in data but starving for insight. The typical measurement landscape looks like this:

- **Platform metrics lie** — Every ad platform claims credit for the same conversions, inflating reported ROAS
- **Fragmented models** — Different teams run different models with different methodologies, producing conflicting results
- **Black-box tools** — Existing MMM solutions produce numbers without explaining the methodology, making results hard to trust and impossible to audit
- **Slow, expensive studies** — Traditional econometric studies take months and cost hundreds of thousands
- **Decision paralysis** — Without trusted measurement, budget decisions are based on gut feel, politics, or whoever shouts loudest

## How Simba Is Different

### 1. Fully Transparent (Not Black-Box)

Simba is the **only enterprise MMM platform** that exposes the actual probabilistic models driving your results. Built on the open-source PyMC-Marketing framework, every prior, parameter, and assumption is inspectable.

**Why it matters**: When your CFO asks "why should I trust these numbers?", you can show them the actual model — not just a dashboard. This builds internal credibility and makes results defensible.

### 2. Bayesian Modeling (Not Regression)

Traditional MMM tools use ordinary least squares (OLS) regression, which produces single-point estimates with no uncertainty quantification. Simba uses **full Bayesian inference**:

- **Uncertainty bands** on every result — know how confident you should be
- **Prior knowledge integration** — encode domain expertise into the model
- **Better with limited data** — Bayesian priors stabilize estimates when data is sparse
- **Probabilistic forecasting** — scenarios include realistic uncertainty, not false precision

### 3. No-Code Configuration

Simba writes the PyMC code so you don't have to. Configure complex Bayesian models — priors, saturation functions, adstock parameters — through an intuitive grid interface.

**Why it matters**: Teams without in-house statisticians can build rigorous models. Teams with statisticians can move faster and focus on interpretation rather than implementation.

### 4. AI-Powered Data Validation

Most MMM tools trust whatever data you give them. Simba's Data Validator validates your data before modeling — detecting anomalies, flagging quality issues, and scoring data health automatically.

**Why it matters**: Prevents the "garbage in, garbage out" problem that undermines so many marketing analytics initiatives.

### 5. End-to-End Platform

Simba covers the full workflow in one platform: **Audit → Measure → Predict → Optimize**. No handoffs, no fragmented toolchain, no broken chain of custody from data to decision.

### 6. Open-Source Foundation

Built on PyMC-Marketing, the leading open-source library for Bayesian marketing analytics. This means:

- No vendor lock-in
- Peer-reviewed methodology
- Continuous community-driven improvements
- Full scientific rigor

## Who Simba Is For

| Audience | Key Value |
|----------|-----------|
| **Brand marketers** | Transparent measurement that builds internal credibility |
| **Marketing agencies** | Scalable, consistent methodology across client portfolios |
| **Data/analytics teams** | Bayesian rigor without the coding overhead |
| **CMOs** | Defensible numbers for budget justification and board reporting |
| **CFOs** | Auditable methodology that finance teams can trust |

## The Simba Advantage — Summary

| Capability | Traditional MMM | Black-Box Tools | Simba |
|------------|----------------|-----------------|-------|
| Transparency | Varies | No | Fully transparent |
| Methodology | OLS regression | Proprietary | Bayesian (PyMC) |
| Uncertainty | None or limited | None | Full posterior distributions |
| Data validation | Manual | Basic | AI-powered validator |
| Code required | Yes (R/Python) | No | No |
| Scenario planning | Separate tool | Limited | Built-in with uncertainty |
| Optimization | Separate tool | Basic | Risk-adjusted, carryover-aware |
| Time to first model | Weeks/months | Days | Hours |

→ [Competitor Comparison](competitor-comparison.md) | [Pricing](../docs/pricing/README.md)

---

*See also: [What is Simba?](../docs/getting-started/what-is-simba.md) | [Case Studies](case-studies/README.md)*
