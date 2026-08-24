---
title: "Getting Started With a Mainland-Accessible AI API Relay: DeepSeek/GLM/Kimi/Qwen Under One Key"
description: "2026 getting-started guide for a mainland-accessible AI API relay: DeepSeek/GLM/Kimi/Qwen under one key, CNY pay-as-you-go billing, transparent prices 13-44% below official. Complete registration + API key tutorial with Cursor setup."
lang: en
author: mrshrawho
date: 2026-08-01
permalink: /ai-relay-testing-playbook-en/free-tier-reality/
related:
  - title: "Series home"
    url: "https://mrshrawho.github.io/ai-relay-testing-playbook-en/"
  - title: "2026 AI Relay Review: Which Providers Are Worth Using"
    url: "https://mrshrawho.github.io/ai-relay-testing-playbook-en/how-to-test-a-relay/"
  - title: "Tokeness Deep Review: Chinese Models From 5.6x-Off Official. Is It Worth Topping Up?"
    url: "https://mrshrawho.github.io/ai-relay-testing-playbook-en/pre-purchase-checklist/"
  - title: "AI API Relay Recommendations & Reviews (Updated 2026)"
    url: "https://mrshrawho.github.io/ai-relay-testing-playbook-en/cache-price-trap/"
---

# Getting Started With a Mainland-Accessible AI API Relay: DeepSeek/GLM/Kimi/Qwen Under One Key

📌 This guide solves three problems for mainland developers — registering at every model vendor separately, official prices running high, and multi-model management chaos — by sharing Tokeness, an aggregator I've tested with genuinely transparent billing. It fits coding, content creation, and batch processing.

🔗 Platform: [tokeness.ai](https://tokeness.ai)

## The three pain points

In 2026, using LLM APIs from mainland China means three recurring annoyances:

1. **Registration sprawl:** DeepSeek, Zhipu, Alibaba, Moonshot — every vendor registers, verifies, and tops up separately. Four models = four account systems.
2. **High cost:** official list prices have no discount, and with multiple models you pre-pay at every vendor.
3. **Management chaos:** keys, balances, and bills scattered across platforms; month-end reconciliation is painful.

Tokeness addresses all three with one account: full Chinese-model coverage, an OpenAI-compatible interface, CNY pay-as-you-go billing, and prices 13-44% below official.

## Why this platform, not "just another relay"

Its differentiation is **transparency and low friction**:

- **Full model coverage:** DeepSeek V4 family, GLM-5.2, Kimi K2.6/K2.7 Code/K3, Qwen3.7 Max/Plus, MiMo V2.5, MiniMax M3 — one key for all
- **Verifiable pricing:** `/api/pricing` publishes input/output/cache multipliers per model; cache pricing isn't hidden — rare in this industry
- **Sane discounts:** 56-87% of official, from real volume-purchase discounts — not below-cost-line bait
- **Zero integration cost:** OpenAI-compatible; change one `base_url` line; Cursor, Cherry Studio, and custom scripts all work
- **Mainland direct access:** no special networking required; register and use
- **Security control:** create multiple API tokens grouped by purpose; a single leaked token doesn't expose everything
- **Clear bills:** every request returns full `usage`; backend keeps per-request records — you always know where the money went

## Full registration + API key tutorial

### 3.1 Account registration

1. Open [tokeness.ai](https://tokeness.ai)
2. Fill in registration: username, password, email, captcha
3. Log in to the console

### 3.2 Create an API token

1. In the console, find **API Tokens** → token management page
2. Click **Add Token**, give it a name (e.g. "Cursor dev")
3. Optionally bind a model group to isolate permissions per tool
4. Create — you get an `sk-xxxxxx` key
5. **Copy and save it immediately** — the key is shown only once

Tip: create separate keys for testing and production so you can revoke one without touching the other.

### 3.3 Code integration

```python
from openai import OpenAI

client = OpenAI(
    api_key="sk-your-token",
    base_url="https://n.tokeness.dev/v1",
)

resp = client.chat.completions.create(
    model="deepseek-v4-flash",
    messages=[{"role": "user", "content": "Hello"}],
)
print(resp.choices[0].message.content)
```

Command-line verification:

```bash
curl https://n.tokeness.dev/v1/chat/completions \
  -H "Authorization: Bearer sk-your-token" \
  -H "Content-Type: application/json" \
  -d '{"model":"deepseek-v4-flash","messages":[{"role":"user","content":"hi"}]}'
```

### 3.4 Tool configuration

- **Cursor / Cherry Studio / Cline:** in settings, set the API address to `https://n.tokeness.dev/v1`, paste the token, and use the exact model ID from the plaza (e.g. `glm-5.2`)
- **Custom scripts:** use any OpenAI SDK; change `base_url`

## Price reference

Verified 2026-08-01 (¥/M tokens):

| Model | Input | Output | Official input | Position |
|-------|------:|-------:|---------------:|----------|
| deepseek-v4-flash | ¥0.559 | ¥1.117 | ¥1.00 | cheapest; batch |
| deepseek-v4-pro | ¥1.806 | ¥3.612 | ¥3.00 | daily coding |
| mimo-v2.5 | ¥0.588 | ¥1.176 | ¥1.00 | light tasks |
| qwen3.7-plus | ¥1.68 | ¥6.72 | ¥2.00 | Chinese writing |
| kimi-k2.7-code | ¥3.99 | ¥16.80 | ¥6.50 | coding specialist |
| glm-5.2 | ¥6.09 | ¥18.90 | ¥8.00 | complex tasks |
| kimi-k3 | ¥12.60 | ¥63.00 | ¥20.00 | flagship long text |

The math: daily coding at 1M input + 0.3M output — about **¥2.9/day** on deepseek-v4-pro, about **¥0.9/day** on flash. Start with a small ¥20-50 top-up for a trial.

## Scenario picks

- 💻 **Development:** Cursor codegen, error fixing, refactoring (deepseek-v4-pro / kimi-k2.7-code)
- 📝 **Content creation:** copywriting, logic outlining (qwen3.7-plus / qwen3.7-max)
- 🧠 **Deep reasoning:** long-document parsing, complex architecture (glm-5.2 / kimi-k3)
- 🔧 **Batch processing:** data cleaning, batch translation, automation scripts (deepseek-v4-flash / mimo-v2.5)

## Common configuration issues

- **401 on calls:** check the API key was copied fully, has no stray spaces, and isn't disabled
- **"Model not found":** the model name must exactly match the plaza ID (case-sensitive, version suffix included)
- **Slow responses:** rule out local network and proxy issues first; the same model can route through different upstream channels at different speeds — try another time window

## Two pieces of advice

1. **Start small:** top up ¥20-50, run one day of real tasks, verify usage and billing are clean, then scale.
2. **Don't put sensitive data here:** it's a third-party aggregator — prompts pass through its servers. Personal code and public content are fine; customer data and production DB fields belong on official channels.

---

*Platform: [tokeness.ai](https://tokeness.ai) ｜ Prices verified 2026-08-01, subject to the live page.*
