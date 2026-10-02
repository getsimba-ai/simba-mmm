# Core Concepts

These pages explain the statistical and marketing foundations behind Simba's Bayesian Marketing Mix Modeling platform. Whether you are a marketing analyst building your first model or a data scientist fine-tuning priors, these guides will give you the conceptual grounding you need.

---

## Concept Guides

### [Marketing Mix Modeling (MMM)](./marketing-mix-modeling.md)
What Marketing Mix Modeling is, how it differs from multi-touch attribution, and why Bayesian MMM built on PyMC is the standard for measuring media effectiveness across channels.

### [Bayesian Modeling](./bayesian-modeling.md)
The statistical foundation behind Simba. How prior distributions combine with observed data via the likelihood to produce posterior estimates, and why uncertainty quantification matters for marketing decisions.

### [Incrementality](./incrementality.md)
How Simba decomposes revenue into base demand, seasonality, controls, and incremental media contributions. Covers the contribution formula (`coefficient x tanh(adstocked / (scalar x alpha))`), lift test calibration, and the difference between causal and correlational attribution.

### [Saturation Curves](./saturation-curves.md)
Understanding diminishing returns in media spend. The tanh saturation function uses two parameters --- scalar (anchored to data scale) and alpha (shape, Gamma prior with mean 1.7) --- to model how each channel's effectiveness decreases as spend increases.

### [Adstock Effects](./adstock-effects.md)
How advertising impact carries over time. Simba supports geometric decay (peak at exposure, exponential decline) and delayed adstock (bell-shaped peak at a configurable lag). Adstock is always applied before saturation.

### [Priors and Distributions](./priors-and-distributions.md)
The four user-selectable distributions (Normal, InverseGamma, TruncatedNormal, TVP) plus internally used Beta and Gamma priors. Covers smart defaults derived from industry benchmarks, halo and trademark channel adjustments, and how lift tests enter as likelihood observations rather than priors.

### [Seasonality](./seasonality.md)
Fourier-based seasonality (default n=2 terms), three trend types (smooth HSGP is the only one in the UI), and GP-smoothed event effects with hierarchical weights. Both seasonality and trend are opt-in features.

### [VAR Modeling](./var-modeling.md)
Bayesian Vector AutoRegression for capturing long-term brand effects. Models the endogenous system (revenue, brand awareness, brand search) driven by exogenous media spend, with Minnesota prior shrinkage to stabilize estimates from noisy brand data. Produces impulse response functions, variance decomposition, and long-run multipliers that enhance MMM ROI estimates.

### [Budget Optimization](./budget-optimization.md)
How to allocate budget across channels to maximize total return. Covers the marginal equalization principle, mean-variance optimization with risk aversion (gamma), posterior-aware allocation using draws retained by the fitted model, per-channel constraints, and portfolio-level optimization across brands.

### [Halo Effects](./halo-effects.md)
How advertising for one brand can lift sales of related brands in a portfolio. Halo channels receive a fixed small coefficient (0.005), while trademark channels receive a 75% reduction in their calculated prior.

Each concept page is self-contained, but reading them in the order listed above will give you the most coherent understanding of how the pieces fit together.

> **Unfamiliar with a term?** See the [Glossary](../../resources/glossary.md) for definitions.
