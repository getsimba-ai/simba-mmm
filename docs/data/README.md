# Data Documentation

Simba requires marketing and business data to build accurate models. This section covers everything you need to know about preparing, formatting, and validating your data.

---

## Guides

### [Data Requirements](./data-requirements.md)
What data you need, supported formats (CSV only, 10 MB max), date formats, and minimum requirements. Covers the six variable types: date, target KPI, media, cost, multiplier, and hierarchy --- with Simba's semantic matcher auto-detecting columns from their names.

### [Data Preparation](./data-preparation.md)
Best practices for cleaning and formatting. Covers time alignment, missing value handling (zeros for no-spend, resolve all NaN before fitting), negative value rules, and the Data Validator's 10 automated checks.

### [Data Validation](./data-validation.md)
How Simba's Data Validator assesses your data across 10 categories: schema, frequency, alignment, multiplier, controls, coverage, outliers, multicollinearity, leakage, and documentation. Triggered via the **Start Validator Agent** button.

### [Exporting Data from Ad Platforms](./exporting-from-platforms.md)
How to export the right data from Google Ads, Meta Ads Manager, GA4, TikTok, DV360, TV, and OOH platforms. Covers what columns to download, what granularity to use, and how to combine everything into one CSV.

### [Supported Channels](./supported-channels.md)
The 15 channel categories Simba auto-detects (TV, digital, social, search, video, radio, print, OOH, email, influencer, affiliate, direct mail, mobile, cinema, sponsorship), the 4 metric types (Spend, Impressions, GRPs, Clicks), and typical saturation/decay profiles by channel type.

---

## Quick Checklist

Before uploading data to Simba, ensure you have:

- [ ] A **single CSV file** (not Excel) under 10 MB
- [ ] A **date column** with a recognized name (date, week, month, or period)
- [ ] A **target KPI** column (revenue, sales, units, or volume)
- [ ] A **multiplier column** (average price, or all 1s if KPI is already revenue)
- [ ] A **hierarchy column** (brand name, or a single repeated value for single-brand models)
- [ ] **Media + cost variable pairs** for each channel (e.g., `tv_impressions` + `tv_spend`)
- [ ] At least 52 weeks of data (104+ recommended)
- [ ] Consistent time granularity across all variables (weekly recommended, daily supported)
- [ ] Zero spend entered as **0** (not blank), no NaN/blank cells remaining
- [ ] No negative values in spend, price, multiplier, or distribution columns

---

> **Unfamiliar with a term?** See the [Glossary](../../resources/glossary.md) for definitions.

> **Next**: [Data Requirements](./data-requirements.md)
