# Account Setup --- Registration, Plans, and Project Configuration

This guide covers everything you need to get your Simba account up and running: creating your account, understanding the available plans, and configuring your project.

---

## Creating Your Account

### Sign Up

1. [Start free](https://demo.simba-mmm.com/users/signup): enter your email address and create a password, or sign up with **Google** or **Microsoft**. No approval step and no credit card. Prefer a walkthrough first? [Book a demo](https://calendly.com/niall-oulton).
2. Click the confirmation link sent to your inbox. That activates your account.
3. Sign in. Your Default Project already holds a **sample model fitted on synthetic data**, so you can read results, run the optimizer and [connect an AI assistant](../integrations/try-the-mcp.md) before you upload anything.
4. Upload your own data when you are ready and start modeling.

### The Free Trial

Every new account starts with a **28-day free trial**. During this period, you have access to core Simba features so you can evaluate the platform with your own data. No credit card is required to begin.

The trial includes:

- Data upload and the [Data Validator](../data/data-validation.md)
- Model configuration with [smart defaults](../core-concepts/priors-and-distributions.md)
- Model fitting and results interpretation
- [Budget optimization](../core-concepts/budget-optimization.md)
- [Scenario planning](../core-concepts/saturation-curves.md)
- [VAR modeling](../core-concepts/var-modeling.md)

At the end of your trial, you can choose a plan to continue. Your data and model configurations are preserved when you upgrade.

---

## Plans

Simba currently offers two plans. For current pricing and to discuss which is right for your team, visit [getsimba.ai](https://getsimba.ai) or contact **info@pymc-labs.com**.

### Enterprise

**Best for:** Organizations that need the full Simba platform with dedicated support.

- Full platform access --- data validation, model fitting, optimization, scenario planning, VAR modeling
- Multi-project support for managing separate brands or markets
- Team collaboration
- Dedicated onboarding support
- Custom SLAs and support terms

### Managed

**Best for:** Organizations that want hands-off, expert-driven marketing mix modeling.

- Simba's team of **PhD statisticians** builds, validates, and maintains your models
- Full platform access plus expert model configuration and interpretation
- Regular reporting cadence tailored to your business
- Strategic consultation on media optimization
- Ideal for teams without in-house data science resources or those wanting expert oversight alongside their own analysis

### Self-Service Tiers (Coming Soon)

Self-service tiers for individual analysts and smaller teams are coming soon. These will offer different levels of access at various price points. Visit [getsimba.ai](https://getsimba.ai) for updates.

---

## Project Configuration

A **project** is your dedicated environment in Simba. It contains your datasets, model configurations, and results.

### Setting Up Your Project

After account creation, your default project is ready to use. To configure it:

1. Navigate to **Settings** from the main menu.
2. Under **Project Settings**, you can:
   - **Rename your project** to reflect your brand or client (e.g., "Acme Corp Q1 2026" or "Client: Greenfield Media").
   - **Set your default currency** for spend data display.
   - **Configure your date format** preference.

### Managing Multiple Projects

On plans that support multiple projects, you can create and manage separate environments:

1. From the project switcher in the top navigation, click **Create New Project**.
2. Name the project and configure its settings independently.
3. Each project has **isolated data storage** --- data uploaded to one project is never accessible from another.
4. Switch between projects using the project dropdown.

This is particularly useful for agencies or organizations that need strict data separation between clients or brands.

---

## Account Security

### Two-Factor Authentication (2FA)

For additional account security, Simba supports two-factor authentication. To enable 2FA:

1. Go to **Settings > Security**.
2. Click **Enable Two-Factor Authentication**.
3. Scan the QR code with your authenticator app (Google Authenticator, Authy, 1Password, or similar).
4. Enter the verification code to confirm setup.

Once enabled, you will need to provide a code from your authenticator app each time you log in.

### Single Sign-On (SSO)

Simba supports sign-in via **Google** and **Microsoft** accounts.

- **New to Simba?** Signing in with Google or Microsoft creates your account; no password is needed. If your provider has not verified your email address, we send a confirmation link first.
- **Already have an account?** Connect a provider to it yourself: go to **Profile > Sign-in methods** and click **Link** next to Google or Microsoft. If you sign in with a provider whose email matches an existing account instead, we email that address a link to connect the provider; nothing is connected until you click it.
- One provider can be connected per account. **Unlink** it from the same tab at any time, as long as the account has a password.
- If two-factor authentication is on, you enter your authenticator code after signing in with a provider too.

For full details on Simba's security practices, see [Security and Compliance](../security/README.md).

---

## Billing

- Plans are billed monthly or annually (annual billing includes a discount).
- Manage payment methods and view invoices in **Settings > Billing**.
- For billing inquiries, contact your account manager or email **info@pymc-labs.com**.

---

## Troubleshooting

| Issue | Solution |
|---|---|
| Did not receive verification email | Check your spam folder. If not found, click "Resend verification" on the login page. |
| Trial expired before evaluation was complete | [Open a GitHub issue](https://github.com/nialloulton/simba-mmm/issues) or email **info@pymc-labs.com** to discuss an extension. |
| Need to transfer project ownership | [Open a GitHub issue](https://github.com/nialloulton/simba-mmm/issues) or email **info@pymc-labs.com**. |

---

## Next Steps

- [Quick Start Guide](./quick-start-guide.md) --- Build your first model.
- [Platform Overview](./platform-overview.md) --- Learn the interface.
- [Data Requirements](../data/data-requirements.md) --- Prepare your data for upload.
- [Security and Compliance](../security/README.md) --- Detailed security documentation.

For any account-related questions, [open a GitHub issue](https://github.com/nialloulton/simba-mmm/issues) or email **info@pymc-labs.com**.
