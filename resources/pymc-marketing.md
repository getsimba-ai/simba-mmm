# PyMC-Marketing & Simba — How the Open-Source Project and Platform Relate

## What is PyMC-Marketing?

[PyMC-Marketing](https://www.pymc-marketing.io/) is the leading open-source Python library for Bayesian marketing analytics. It provides statistical models for:

- Marketing mix modeling (MMM)
- Customer lifetime value (CLV)
- Media saturation and adstock effects
- Bayesian inference for marketing measurement

PyMC-Marketing is built on top of [PyMC](https://www.pymc.io/), one of the most widely used probabilistic programming frameworks in the world.

## How Simba and PyMC-Marketing Relate

Simba is an **enterprise platform built on PyMC by the team behind PyMC-Marketing**. Simba's modeling engine is its own PyMC code; it does not import PyMC-Marketing. The two share the same statistical foundations and conventions (adstock, saturation, lift tests as likelihood observations), and the tutorial notebooks in this repository use PyMC-Marketing to teach them. Think of it this way:

| Layer | What It Is |
|-------|-----------|
| **PyMC** | The probabilistic programming engine (open-source) |
| **PyMC-Marketing** | The marketing-specific modeling library (open-source), by the same team |
| **Simba** | The enterprise platform, with its own PyMC engine, UI, data validation, scenario planning, and optimization |

Simba **writes the PyMC code so you don't have to**. The platform translates your UI configurations (priors, saturation, decay) into a PyMC model, runs Bayesian inference, and presents results through an intuitive interface.

## What This Means for Users

### Full Transparency

Because Simba is built on open-source foundations:

- The modeling methodology is **publicly available and peer-reviewed**
- You can inspect how priors, saturation, and adstock are implemented
- Results are reproducible — there's no proprietary "secret sauce" hiding behind the numbers
- Academic and industry researchers continuously improve the underlying methods

### No Vendor Lock-In

Your modeling logic is built on an open standard. If you ever wanted to:

- Audit the methodology independently
- Reproduce results in a Python environment
- Transition to a custom implementation

...you can, because the foundation is open-source.

### Community-Driven Improvement

PyMC-Marketing benefits from contributions by statisticians, data scientists, and marketing analysts worldwide. Improvements to the open-source library flow into Simba as platform updates.

### Scientific Rigor

PyMC and PyMC-Marketing are used in academic research, published in peer-reviewed journals, and maintained by professional statisticians. This gives Simba a level of methodological credibility that proprietary tools can't match.

## What Simba Adds on Top

While PyMC-Marketing provides the statistical models, Simba adds the enterprise platform layer:

| Capability | PyMC-Marketing | Simba |
|------------|---------------|-------|
| Bayesian MMM models | Yes | Yes (its own PyMC engine) |
| No-code UI | No (Python required) | Yes |
| AI Data Validator | No | Yes |
| Scenario planning | Manual implementation | Built-in |
| Budget optimization | Manual implementation | Built-in, risk-adjusted |
| Multi-model management | Manual | Built-in |
| Portfolio modeling | Not supported | Built-in |
| Smart defaults | Not available | Auto-generated |
| Enterprise security | Self-managed | Encrypted at rest, TLS 1.2+, Cyber Essentials |
| Support | Community | Dedicated support + Managed tier |

## For Technical Users

If you're familiar with PyMC-Marketing and want to understand how Simba maps to the library:

- **Model configuration** in Simba corresponds to defining a PyMC-Marketing `MMM` model with custom priors, saturation functions, and adstock transformations; the concepts map one to one even though the code is Simba's own
- **Model fit** runs Bayesian inference using PyMC's MCMC sampling
- **Smart defaults** are derived from your data and industry benchmarks; PyMC-Marketing's documented parameter ranges are a useful cross-check
- **Optimization** uses the fitted posterior distributions to find optimal budget allocations

## Learn More

- **PyMC-Marketing**: [pymc-marketing.io](https://www.pymc-marketing.io/)
- **PyMC**: [pymc.io](https://www.pymc.io/)
- **Simba**: [getsimba.ai](https://getsimba.ai)

---

*See also: [Bayesian Modeling](../docs/core-concepts/bayesian-modeling.md) | [What is Simba?](../docs/getting-started/what-is-simba.md)*
