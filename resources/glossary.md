# Glossary --- Marketing Mix Modeling and Bayesian Statistics Terminology

A comprehensive reference of terms used in marketing mix modeling (MMM), Bayesian statistics, and the Simba platform.

---

## A

### Adstock
The carryover effect of advertising. When a TV ad airs this week, its impact doesn't disappear immediately --- it "stocks up" and decays over time. Simba supports two adstock formulations: **geometric** (exponential decay) and **delayed** (builds to a peak before decaying). Also called **carryover** or **lag effect**. See [Adstock Effects](../docs/core-concepts/adstock-effects.md).

### Attribution
The process of assigning credit for business outcomes (e.g., sales) to specific marketing activities. In MMM, attribution is based on statistical modeling of aggregate data, not individual user tracking.

---

## B

### Base Sales
The level of sales or revenue that would occur without any marketing activity. In MMM, base sales represent organic demand driven by brand equity, distribution, pricing, and other non-media factors. Simba separates base sales from media-driven incremental lift.

### Bayesian Inference
A statistical method that updates probability estimates as new data becomes available. Instead of producing a single "best guess" (like frequentist methods), Bayesian inference produces a full probability distribution over possible values, quantifying uncertainty. Simba uses Bayesian inference for all model fitting. See [Bayesian Modeling](../docs/core-concepts/bayesian-modeling.md).

### Bayesian MMM
Marketing mix modeling that uses Bayesian statistical methods. Advantages over traditional (frequentist) MMM include uncertainty quantification, prior knowledge integration, and better performance with limited data. Simba is a Bayesian MMM platform built on PyMC-Marketing.

---

## C

### Carryover
See **Adstock**. The lagged impact of advertising beyond the period in which it was delivered.

### Causal Attribution
Determining which marketing activities actually *caused* business outcomes, as opposed to merely being correlated with them. Bayesian MMM with proper controls provides causal (not just correlational) attribution.

### Confidence Interval
In frequentist statistics, a range of values that contains the true parameter with a certain probability. In Bayesian statistics, the analogous concept is a **credible interval** (or HDI), which Simba uses.

### Credible Interval
The Bayesian equivalent of a confidence interval. Simba reports the **94% HDI (Highest Density Interval)**, spanning the 3rd to 97th percentile of the posterior. This means there is a 94% probability that the true value falls within this range, given the data and model. More intuitive and directly interpretable than frequentist confidence intervals.

---

## D

### Data Validator
Simba's data validation system. Runs 10 specialized checks across your dataset covering schema integrity, frequency diagnostics, alignment, multiplier logic, controls, coverage, outlier detection, multicollinearity, leakage, and documentation quality. Triggered by clicking **Run Data Validator** in the Model Warehouse configuration screen.

### Decay Rate
The speed at which the impact of advertising diminishes over time. A high decay rate means the effect fades quickly (typical of digital ads); a low decay rate means the effect persists (typical of TV). Configured per channel in Simba via a Beta prior. See [Adstock Effects](../docs/core-concepts/adstock-effects.md).

### Delayed Adstock
An adstock formulation where the peak effect does not occur immediately but builds to a maximum after a configurable delay, then decays. Used for channels where impact takes time to materialize, such as brand-building TV campaigns or sponsorships. See **Adstock**.

### Diminishing Returns
The phenomenon where additional spend on a channel produces progressively less incremental impact. Modeled in Simba through **saturation curves** using the tanh function. See [Saturation Curves](../docs/core-concepts/saturation-curves.md).

### Distribution (Statistical)
A mathematical function describing the probability of different outcomes. In Simba's UI, the available distributions for priors are: **Normal**, **InverseGamma**, **TruncatedNormal**, and **TVP** (Time-Varying Parameter). See [Priors and Distributions](../docs/core-concepts/priors-and-distributions.md).

---

## E

### Exogenous Variable
In a VAR model, a variable that drives the system but is not predicted by it. Media spend is typically treated as exogenous --- you control how much to spend, and it affects revenue and brand metrics, but is not itself an outcome of the system. Contrast with **Endogenous Variable**.

### Endogenous Variable
In a VAR model, a variable that both influences and is influenced by other variables in the system. Revenue, brand awareness, and brand search are typically endogenous --- they interact through feedback loops. Contrast with **Exogenous Variable**.

---

## F

### FEVD (Forecast Error Variance Decomposition)
A statistical technique used in VAR models that decomposes the forecast uncertainty of each endogenous variable into the contributions of shocks from all endogenous variables in the system. In marketing, FEVD shows how much of the uncertainty in revenue forecasts is attributable to changes in brand awareness, brand search, and other brand metrics. See **VAR**.

### Fourier Seasonality
The method Simba uses to model seasonal patterns. Pairs of sine and cosine functions at different frequencies capture recurring annual (and optionally weekly) patterns in your data. Inspired by the Prophet approach but implemented natively in PyMC. See [Seasonality](../docs/core-concepts/seasonality.md).

### Fully Transparent
Simba's design principle that every model assumption is visible, inspectable, and auditable --- the opposite of a black-box. Every prior, parameter, saturation curve, and uncertainty interval can be reviewed by stakeholders.

---

## G

### GRP (Gross Rating Point)
A measure of advertising exposure for TV. One GRP represents reaching 1% of the target audience one time. Common input variable for TV in marketing mix models.

---

## H

### Halo Effect
In marketing, the phenomenon where advertising for one brand or product generates incremental lift for related brands or products in the same portfolio. Simba models halo effects through halo channels (fixed coefficient of 0.005) and trademark channels, enabling cross-brand optimization. See [Halo Effects](../docs/core-concepts/halo-effects.md).

### Half-Life
The number of time periods it takes for a channel's adstock (carryover) effect to decay to 50% of its initial value. A channel with a half-life of 2 weeks retains half its impact after two weeks and a quarter after four weeks. A more intuitive alternative to the raw decay rate parameter.

### HDI (Highest Density Interval)
The Bayesian credible interval used by Simba. The 94% HDI spans the 3rd to 97th percentile of the posterior distribution. It represents the narrowest interval containing 94% of the probability mass, giving the most likely range of parameter values. See **Credible Interval**.

### Hierarchy Column
A required column in Simba's data upload that identifies the brand, market, region, or segment for each row. Even single-brand models need a hierarchy column (just one brand name repeated). Recognized keywords: brand, market, region, category, segment, geography.

---

## I

### Impulse Response Function (IRF)
In VAR models, the time path of how a one-unit shock to one variable propagates through the entire system. For example, an IRF for TV spend → Revenue shows the cumulative revenue impact accounting for all indirect pathways (TV → brand awareness → brand search → revenue). See **VAR**.

### Incrementality
The additional business outcome (revenue, conversions, etc.) that would not have occurred without marketing activity. Simba measures incrementality by isolating the causal impact of each media channel from base sales and other factors. See [Incrementality](../docs/core-concepts/incrementality.md).

### InverseGamma Distribution
A continuous probability distribution defined on positive real numbers. In Simba, InverseGamma is the **default distribution for media channel coefficients**, set automatically by the smart priors system. See [Priors and Distributions](../docs/core-concepts/priors-and-distributions.md).

---

## L

### Lift
The incremental increase in a business metric attributable to a specific marketing activity. For example, if TV advertising produces a 15% lift in revenue, it means revenue was 15% higher than it would have been without TV.

### Lift Test
An experiment (often randomized, such as geo-based holdout tests) designed to measure the incremental impact of a marketing activity. In Simba, lift test results are integrated as **likelihood observations** that calibrate the model --- they enter the model as additional data constraints, not as priors. See [Incrementality](../docs/core-concepts/incrementality.md).

### Long-Run Effects
In VAR models, the total cumulative impact of a sustained change in one variable on all other endogenous variables, after the system has fully absorbed the shock. Computed analytically via Psi_inf = (I - A_sum)^{-1}. Long-run effects reveal that brand-building channels (like TV) often have total effects much larger than their short-term MMM ROI suggests. See **VAR**.

---

## M

### Media Lift
The incremental revenue or outcome attributable to media spending, separated from base sales. In Simba's Active Model view, media lift shows exactly how much value each channel is driving above what would happen organically.

### Media Mix
The combination of marketing channels used in a campaign or marketing plan. Typically includes TV, digital, social, search, OOH, and other channels.

### Minnesota Prior
A structured prior for VAR models, originally developed at the Federal Reserve Bank of Minneapolis. It encodes three beliefs: each variable's best predictor is its own recent past (own-lag mu=0.9), cross-variable effects are small (shrunk toward zero with lambda=0.2), and higher-order lags are less informative than recent ones. Essential for preventing overfitting in marketing VAR models where the number of parameters can easily exceed the number of observations. See [VAR Modeling](../docs/core-concepts/var-modeling.md).

### MMM (Marketing Mix Modeling)
A statistical analysis technique that uses aggregate time-series data to measure the impact of marketing activities on business outcomes. MMM quantifies the contribution of each channel while accounting for saturation, carryover, and seasonality. See [Marketing Mix Modeling](../docs/core-concepts/marketing-mix-modeling.md).

### Model Fit
The process of running Bayesian inference on your data to estimate model parameters. Simba uses NUTS (No U-Turn Sampler) via PyMC for posterior sampling.

### MTA (Multi-Touch Attribution)
A user-level attribution method that tracks individual customer journeys across touchpoints. Unlike MMM, MTA relies on cookies/device tracking and struggles with iOS privacy changes, walled gardens, and offline channels. MMM and MTA are complementary approaches.

### Multiplier Variable
A required column in Simba's data upload. If your target KPI is units or volume, the multiplier (e.g., average price) converts it to revenue. If your KPI is already revenue, set the multiplier column to all 1s. Recognized keywords: price, avg_price, multiplier, conversion.

---

## O

### Optimization
The process of determining the ideal budget allocation across channels to maximize expected response subject to constraints. Simba's budget optimizer uses a **mean-variance** framework with a risk-aversion parameter (gamma) that controls the tradeoff between maximizing return and minimizing uncertainty. See [Budget Optimization](../docs/core-concepts/budget-optimization.md).

---

## P

### Portfolio Model
A model that spans multiple brands, markets, or clients --- enabling cross-entity comparison and optimization, including halo effects and trademark channels.

### Posterior Distribution
In Bayesian statistics, the distribution of a parameter *after* observing data. The posterior combines the prior (what you knew before) with the likelihood (what the data tells you). All of Simba's results are posterior distributions.

### Prior (Prior Distribution)
In Bayesian statistics, a probability distribution representing what you believe about a parameter *before* seeing the data. Priors encode domain knowledge --- for example, TV advertising typically has a positive effect on sales. Simba lets you configure priors per channel via the UI. See [Priors and Distributions](../docs/core-concepts/priors-and-distributions.md).

### Prior Predictive Check
A validation step for Bayesian models where sample forecasts are generated from the prior distributions (before seeing data) to confirm that the priors produce sensible ranges. Used in VAR modeling to validate prior configurations before committing to a full model fit.

### PyMC
An open-source probabilistic programming library for Python. PyMC is the computational engine behind Bayesian inference in Simba and PyMC-Marketing.

### PyMC-Marketing
The open-source library for Bayesian marketing analytics that Simba is built upon. PyMC-Marketing provides the statistical models for media mix modeling, including saturation, adstock, and Bayesian inference.

---

## R

### ROAS (Return on Ad Spend)
Revenue generated per unit of ad spend. ROAS = Revenue / Spend. In MMM, ROAS typically refers to *incremental* ROAS --- the additional revenue attributable to media, divided by spend.

### ROI (Return on Investment)
A broader measure of investment efficiency. ROI = (Gain - Cost) / Cost. In the context of MMM, ROI captures the net return from marketing investment after accounting for the cost.

---

## S

### Saturation
The point at which additional spend on a channel produces diminishing incremental returns. Simba uses the **tanh** (hyperbolic tangent) saturation function with two parameters: **scalar** (fixed from data maximum) and **alpha** (estimated, Gamma prior mu=1.7). See **Saturation Curve**.

### Saturation Curve
A mathematical function that models the relationship between media input (spend, impressions) and output (revenue, conversions). Simba uses tanh(x / (scalar * alpha)), where scalar is fixed from data and alpha is estimated. The saturation parameter controls how quickly a channel reaches its effectiveness ceiling. See [Saturation Curves](../docs/core-concepts/saturation-curves.md).

### Scenario Planning
The process of modeling different budget allocation strategies to forecast their likely outcomes. Simba's Scenario Planner lets you compare different projections with uncertainty bands.

### Seasonality
Regular, predictable patterns in business outcomes that repeat over time --- such as holiday shopping spikes, summer slowdowns, or back-to-school periods. Simba models seasonality using Fourier series. Seasonality is opt-in (disabled by default), with a frontend default of n=2 Fourier terms. See [Seasonality](../docs/core-concepts/seasonality.md).

### Semantic Matcher
Simba's automatic column classification system. When you upload a CSV, the semantic matcher reads column names and classifies them into categories (date, target, media, cost, control, multiplier, hierarchy) using keyword matching across 13 channel categories and 8 metric types.

### Smart Defaults
Simba's auto-generated model starting points. Based on your data characteristics, smart defaults provide sensible initial values for priors, saturation, and decay parameters. For media channels, the default distribution is InverseGamma. See [Priors and Distributions](../docs/core-concepts/priors-and-distributions.md).

---

## T

### Trademark Channel
A virtual channel in Simba that represents masterbrand, portfolio-level, or corporate advertising benefiting multiple brands simultaneously. Unlike halo channels (which are brand-specific), trademark channels are always global and affect all brands in the portfolio equally, applying a 25% reduction to the parent brand coefficient. See [Halo Effects](../docs/core-concepts/halo-effects.md).

### Trend
A long-term upward or downward shift in your baseline outcome over months or years. Simba's UI exposes **smooth_lltrend** (HSGP Matern52 kernel), which captures gradual changes without attributing them to media. Trend is opt-in (disabled by default). See [Seasonality](../docs/core-concepts/seasonality.md).

### TruncatedNormal Distribution
A normal (Gaussian) distribution that is truncated to a specific range. Used as a prior distribution in Bayesian models when you want to constrain a parameter to a specific range (e.g., positive values only, or between 0 and 1 for decay rates). Available in Simba's model configuration.

### TVP (Time-Varying Parameter)
A prior distribution option in Simba that allows a parameter to change over time rather than being fixed for the entire modeling period. Useful when you believe a channel's effectiveness has shifted (e.g., due to creative changes, market maturation, or competitive dynamics).

---

## U

### Uncertainty Quantification
The process of expressing how confident you are in a result. In Bayesian MMM, every estimate comes with a full posterior distribution --- not just a single number. Simba reports the 94% HDI (3rd--97th percentile) for all parameter estimates and channel contributions.

---

## V

### VAR (Vector AutoRegression)
A Bayesian multi-equation time series model that captures dynamic interactions between endogenous variables (revenue, brand awareness, brand search) driven by exogenous inputs (media spend). Unlike standard MMM (which models a single target), VAR models the feedback system between business outcomes and brand metrics simultaneously. Uses Minnesota priors for regularization and LKJ Cholesky covariance. See [VAR Modeling](../docs/core-concepts/var-modeling.md).

---

## W

### What-If Analysis
A form of scenario planning where you change one or more input variables (e.g., "what if I increase TV spend by 20%?") and observe the predicted impact on outcomes. Simba's Scenario Planner supports what-if analysis with full uncertainty quantification.

---

*Missing a term? [Open a GitHub issue](https://github.com/nialloulton/simba-mmm/issues) or email [info@pymc-labs.com](mailto:info@pymc-labs.com) and we'll add it.*

*See also: [Core Concepts](../docs/core-concepts/README.md) | [FAQ](../docs/faq/README.md)*
