# Further Reading — Papers, Articles, and External Resources

A curated collection of resources for deeper learning about marketing mix modeling, Bayesian statistics, and the methodologies behind Simba.

## Marketing Mix Modeling

### Foundational Concepts

- [Marketing Mix Modeling: A Practical Guide](https://www.pymc-marketing.io/) — Overview of how Bayesian approaches are transforming the MMM landscape (see PyMC-Marketing docs for worked examples)
- [The Resurgence of Marketing Mix Modeling](https://www.thinkwithgoogle.com/) — Why MMM is experiencing a renaissance driven by privacy changes and cookie deprecation
- [Lift Test Integration in PyMC-Marketing](https://www.pymc-marketing.io/en/0.16.0/notebooks/mmm/mmm_lift_test.html) — How lift test observations are integrated as likelihood terms in Bayesian MMM

### Industry Reports

- [Google's guide to MMM and Meridian](https://github.com/google/meridian) — Google's perspective on how marketing mix modeling fits into modern measurement
- [Meta's Robyn documentation](https://facebookexperimental.github.io/Robyn/) — Meta's open-source MMM approach (ridge regression, for comparison with Bayesian methods)

## Bayesian Statistics

### Introductory Resources

- **"Bayesian Data Analysis" by Gelman et al.** — The definitive textbook on Bayesian statistics (advanced but comprehensive)
- **"Statistical Rethinking" by Richard McElreath** — An accessible introduction to Bayesian modeling with practical examples
- **"Think Bayes" by Allen Downey** — A beginner-friendly introduction to Bayesian statistics using Python

### PyMC Resources

- **[PyMC Documentation](https://www.pymc.io/projects/docs/en/latest/)** — Official documentation for the PyMC probabilistic programming library
- **[PyMC-Marketing Documentation](https://www.pymc-marketing.io/)** — Documentation for the marketing-specific library that Simba is built on
- **[PyMC Examples Gallery](https://www.pymc.io/projects/examples/en/latest/)** — Worked examples of Bayesian modeling with PyMC

## Media Effectiveness

### Saturation and Diminishing Returns

- [Understanding Media Saturation](https://www.pymc-marketing.io/) — How diminishing returns affect media investment decisions (see PyMC-Marketing saturation docs)

### Adstock and Carryover

- [Advertising Adstock — Broadbent (1979)](https://en.wikipedia.org/wiki/Advertising_adstock) — The foundational concept of how advertising effects persist and decay over time

## Privacy and the Future of Measurement

- The deprecation of third-party cookies, iOS App Tracking Transparency (ATT), GDPR, and similar regulations are making user-level attribution increasingly unreliable. MMM's reliance on aggregate time-series data makes it inherently privacy-compliant.

## Open-Source MMM Tools

- **[PyMC-Marketing](https://www.pymc-marketing.io/)** — The open-source Bayesian marketing analytics library (Simba's foundation)
- **[Robyn by Meta](https://facebookexperimental.github.io/Robyn/)** — Meta's open-source MMM solution (R-based, ridge regression approach)
- **[Meridian by Google](https://github.com/google/meridian)** — Google's open-source Bayesian MMM tool

## Conferences and Communities

- **PyMCon** — The PyMC community conference featuring talks on Bayesian statistics and applications
- **Marketing Science Conference** — INFORMS conference on quantitative marketing research
- **Measure Camp** — Unconference for analytics and measurement professionals

---

*Have a resource suggestion? [Let us know](https://github.com/nialloulton/simba-mmm/issues).*

*See also: [Glossary](glossary.md) | [PyMC-Marketing & Simba](pymc-marketing.md) | [Core Concepts](../docs/core-concepts/README.md)*
