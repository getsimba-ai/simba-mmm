# Platform Guide

This guide covers every major feature of the Simba MMM platform, organized around the four-step workflow: **Audit, Measure, Predict, Optimize**. Each page explains what a feature does, how to use it, and how to interpret the results.

## The Simba Workflow

### Step 1: Audit

- **[Data Validator](./data-auditor.md)** --- Automated validation and quality scoring for every data source. Detects anomalies, handles missing values, checks schema integrity, and produces a data health score before modeling begins.

### Step 2: Measure

- **[Incremental Measurement](./measurement.md)** --- Causal attribution across all marketing channels. Separates base sales from media lift, integrates lift tests, and isolates the true incremental value of each channel. Includes model lifecycle, cloning, and custom contribution groups.

- **[Validation Metrics, Holdouts and Quality Policies](./validation-and-holdout.md)** --- Fit diagnostics and their thresholds, reserving a holdout, saved prediction windows, and declaring the quality policy a study run is scored against.

- **[Studies](./studies.md)** --- One shared record of a modelling question: drafts, published revisions, diffs, calibration, quality policies, evaluations and the Champion a person finally chooses.

### Step 3: Predict

- **[Scenario Planning](./scenario-planning.md)** --- What-if forecasting with uncertainty bands. Configure monthly or weekly spend plans, adjust control variables, and generate Bayesian predictions with a comprehensive results dashboard including ROAS analysis, revenue decomposition, and efficiency scatter plots.

### Step 4: Optimize

- **[Budget Optimization](./budget-optimization.md)** --- Risk-adjusted spend allocation with multi-week carryover awareness. Delivers precise per-channel recommendations that balance ROI targets with risk appetite. Supports portfolio optimization with halo and trademark effects.

## Model Setup and Configuration

- **[Model Creation Wizard](./model-creation-wizard.md)** --- Step-by-step guide through the 5-stage wizard: source configuration, variable selection, prior builder, model setup, and model details.

- **[Model Configuration](./model-configuration.md)** --- No-code interface for setting priors, saturation curves, decay parameters, variable transformations, and adstock types on a per-channel basis.

- **[KPI Types and Profit Outputs](./kpi-types-and-profit.md)** --- Choosing a likelihood for revenue, units, orders or shares, giving the model an operating margin, and where profit appears in results, planning and the optimizer.

- **[Promotions and Pricing as Controls](./promotions-and-pricing.md)** --- Adding promotions, price and distribution as controls, choosing each control's transform and reference level, and reading its contribution.

- **[Smart Defaults](./smart-defaults.md)** --- Auto-generated starting points derived from historical data and industry benchmarks. Explains when defaults are sufficient and when domain expertise should override them.

- **[Long-Term Effects](./long-term-effects.md)** --- Modeling lasting brand impact beyond immediate performance. Covers memory decay curves, behavioral response, and seasonal patterns.

- **[VAR Models](./var-models.md)** --- Building and interpreting Vector AutoRegression models for understanding inter-channel dynamics and system-level marketing effects.

- **[Halo and Trademark Channels](./halo-trademark-channels.md)** --- Configuring cross-brand marketing effects for portfolio analysis, including halo channels and masterbrand/portfolio/corporate trademark channels.

## Analysis and Collaboration

- **[Portfolio Analysis](./portfolio-analysis.md)** --- Cross-brand and cross-model comparison and optimization. Compare metrics, response curves, and ROI across brands, and optimize budget at the portfolio level.

- **[Exports and Reporting](./exports-reporting.md)** --- PDF report generation, CSV data exports, and chart downloads for offline analysis and stakeholder communication.

- **[Sharing and Collaboration](./sharing-collaboration.md)** --- Model sharing, portfolio sharing, team management, and project organization for collaborative workflows.

## Help and Troubleshooting

- **[Troubleshooting](./troubleshooting.md)** --- Common issues and how to fix them. Covers data upload problems, model convergence, unexpected results, optimizer issues, and account access.

## Account Management

- **[Usage Tracking](./usage-tracking.md)** --- Understanding plan limits, usage tracking, billing periods, and how to manage your monthly allocations.

> **Unfamiliar with a term?** See the [Glossary](../../resources/glossary.md) for definitions.
