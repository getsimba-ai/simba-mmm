# Try the MCP: from sign-up to a first answer in five minutes

No sales call, no waiting for approval. Every new Simba account starts with a **sample model fitted on synthetic data**, so an AI assistant connected to your account has something real to answer about before you upload anything. This page walks the whole path; [Simba MCP](./simba-mcp.md) has the full connection details for every client.

## 1. Start free

1. Go to [demo.simba-mmm.com/signup](https://demo.simba-mmm.com/signup). Enter your email address and a password, or sign up with **Google** or **Microsoft**.
2. Click the confirmation link sent to your inbox. That activates your account.
3. Sign in. Under **Models → Saved Models**, your Default Project holds **Sample model (synthetic data)**. Its results, curves, optimizer and scenarios all work; the numbers describe a synthetic dataset, and the model says so wherever its name appears.

## 2. Connect an assistant

Open **Profile → Connected apps**. The **Connect an assistant** block at the top has everything a client needs.

- **claude.ai and ChatGPT:** copy the **MCP server URL** (`https://demo.simba-mmm.com/mcp`), paste it into the assistant's connector settings and approve the connection on Simba's consent screen. The connection then appears in the table below the block. Step by step: [claude.ai](./simba-mcp.md#claudeai-custom-connector-oauth), [ChatGPT](./simba-mcp.md#chatgpt-connector-oauth).
- **Claude Code and Cursor:** create a key on the **API Keys** tab, then add the server to the client's MCP config. The config block is inside the prompt below, with the key as a placeholder. Step by step: [Claude Code](./simba-mcp.md#claude-code-key), [Cursor](./simba-mcp.md#cursor-key).

## 3. Copy the set-up and test prompt

Under the connection options is a **Set-up and test prompt** with a **Copy prompt** button. Paste it into your assistant. It does two things:

1. **Set-up.** It tells the assistant how it is connected (the URL, or the key and config) and asks it to confirm the connection by listing your models, where it should find the sample model.
2. **A six-step workflow on the sample model.** Results (which channels drove revenue, ROI and contribution per channel, fit and convergence), response curves (which channel is nearest saturation, which has headroom), marginal ROI at today's spend, an optimizer run at the current budget, one scenario that moves 10% of the largest channel's spend to the channel with most headroom, and a description of the dataset behind the model.

It ends by asking for a short report: one summary per step with the numbers used, anything a tool refused, quoted as returned, and whether the results and the optimizer agree where they say the same thing.

The prompt is generated for your deployment, so the URL and config in it are already right. Copy it again if the deployment changes.

## What the trial allows

The trial lasts 28 days and includes, per 30 days, 10 model fits, 10 optimizations, 10 scenarios and up to 20 saved models, with one model fitting at a time. The assistant runs under the same account and the same limits: when a limit is reached, its tool call is refused with the same message you would see in the app. See [Pricing](../pricing/README.md) for the plans.

## Next

- Upload your own data: [Quick Start Guide](../getting-started/quick-start-guide.md).
- Give the assistant more to do: [Run the optimizer from an agent](./optimize-over-mcp.md), [Read model results](./read-model-results.md), [Recurring automation](./automate-with-an-agent.md).
- Prefer a walkthrough with a person? [Book a demo](https://calendly.com/niall-oulton).
