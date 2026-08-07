---
title: "Tokeness Deep Review: Chinese Models From 5.6x-Off Official. Is It Worth Topping Up?"
description: "Tokeness deep review: Chinese models from 5.6x-off official price. Is it legit? Worth topping up? Positioning, 9-model price table, integration tutorial, pros/cons and FAQ — everything verified 2026-08-01."
lang: en
author: mrshrawho
date: 2026-08-01
permalink: /ai-relay-testing-playbook-en/pre-purchase-checklist/
related:
  - title: "Series home"
    url: "https://mrshrawho.github.io/ai-relay-testing-playbook-en/"
  - title: "2026 AI Relay Review: Which Providers Are Worth Using"
    url: "https://mrshrawho.github.io/ai-relay-testing-playbook-en/how-to-test-a-relay/"
  - title: "AI API Relay Recommendations & Reviews (Updated 2026)"
    url: "https://mrshrawho.github.io/ai-relay-testing-playbook-en/cache-price-trap/"
  - title: "Getting Started With a Mainland-Accessible AI API Relay: DeepSeek/GLM/Kimi/Qwen Under One Key"
    url: "https://mrshrawho.github.io/ai-relay-testing-playbook-en/free-tier-reality/"
---

# Tokeness Deep Review: Chinese Models From 5.6x-Off Official. Is It Worth Topping Up?

## What it is — and what it is not

**Tokeness = a mainland-accessible AI API aggregator** focused on Chinese models: DeepSeek, GLM, Kimi, Qwen, MiMo, MiniMax.

How it works in one sentence:

```text
Your code / Cursor / custom app
        ↓ (change base_url, OpenAI-compatible format)
    n.tokeness.io (aggregation layer: auth → route → forward)
        ↓ (to each model vendor's official API)
    official model result → back to you the same way
```

It solves three real problems:

| Pain point | Official direct | How an aggregator helps |
|------------|-----------------|-------------------------|
| Sign-up | register + verify at every vendor | one account calls them all |
| Payment | some vendors reject personal WeChat/Alipay | top up in CNY directly |
| Price | official list price | bulk-purchase discounts, 13-44% below official |

⚠️ **Important caveat:** it is a third-party aggregator, not the model vendor. Your requests technically pass through its servers; assess the data-privacy boundary yourself (more below).

### At-a-glance

| Item | Detail |
|------|--------|
| What it is | third-party API aggregator (you → Tokeness → model vendors) |
| Models | DeepSeek V4 family, GLM-5.2, Kimi K2.6/K2.7/K3, Qwen3.7 Max/Plus, MiMo V2.5, MiniMax M3 |
| API | OpenAI-compatible (`/v1/chat/completions`); change base_url |
| Base URL | `https://n.tokeness.io/v1` |
| Price | 56-87% of official (varies by model); input/output/cache published |
| Price source | `/api/pricing` public endpoint; reconcile offline |
| Billing | per-token; every request returns full usage |

## Is it actually cheaper?

Directly from `/api/pricing`, verified 2026-08-01:

| Model | Tokeness input | Tokeness output | Official input | Saving |
|-------|---------------:|----------------:|---------------:|--------|
| `deepseek-v4-flash` | ¥0.559 | ¥1.117 | ¥1.00 | ~44% |
| `deepseek-v4-pro` | ¥1.806 | ¥3.612 | ¥3.00 | ~40% |
| `glm-5.2` | ¥6.09 | ¥18.90 | ¥8.00 | ~24% |
| `kimi-k2.7-code` | ¥3.99 | ¥16.80 | ¥6.50 | ~39% |
| `kimi-k3` | ¥12.60 | ¥63.00 | ¥20.00 | ~37% |
| `qwen3.7-max` | ¥10.50 | ¥31.50 | ¥12.00 | ~13% |
| `qwen3.7-plus` | ¥1.68 | ¥6.72 | ¥2.00 | ~16% |
| `mimo-v2.5` | ¥0.588 | ¥1.176 | ¥1.00 | ~41% |
| `minimax-m3` | ¥1.26 | ¥5.04 | ¥2.10 | ~40% |

(Unit: ¥/M tokens. Official prices may vary by promo and region — trust the official pages.)

**This discount band is "reasonably cheap," not "suspiciously cheap."** Industry common sense: 5-9x-off-of-official comes from volume-purchase discounts and is sustainable; 1-3x-off can't cover purchase cost and implies reverse-engineered interfaces, billing black boxes, or watered models. Tokeness sits in the sane range — the first reason I trust it.

The second reason is **cache-price disclosure**. All 2026 mainstream models support prompt caching; the official cache price is ~10% of normal input, and plenty of platforms quietly charge 30% or skip caching entirely. Tokeness publishes `cache_ratio` per model in `/api/pricing`, so you can compute the cache price yourself. In coding and long-conversation workloads (cache = 50-80% of input), this single line item can halve your monthly bill.

The third detail: its price page is **alive**. I verified deepseek-v4-flash input at ¥0.504 on 07-27 and ¥0.559 on 08-01 (+11%). Prices track real upstream changes — evidence the pricing mechanism is cost-driven, not marketing numbers.

## How to use it

### Step 0: Sign up

Open [tokeness.io](https://tokeness.io), register, and enter the console.

### Step 1: Create an API token

Console → API Tokens → Create. Create separate tokens per purpose (one for testing, one for production). The token is shown only once — save it immediately.

### Step 2: Code integration (OpenAI SDK)

```python
from openai import OpenAI

client = OpenAI(
    api_key="sk-your-token",
    base_url="https://n.tokeness.io/v1",
)

resp = client.chat.completions.create(
    model="deepseek-v4-flash",
    messages=[{"role": "user", "content": "Write quicksort in Python"}],
)
print(resp.choices[0].message.content)
print(resp.usage)  # full usage — first step of billing verification
```

### Step 3: Wire up Cursor / Cherry Studio / Cline

Point the tool's API address (`OPENAI_BASE_URL` or equivalent) at `https://n.tokeness.io/v1`, paste the token, and use the exact model ID shown in the plaza (e.g. `glm-5.2`, `kimi-k2.7-code`).

### Step 4: Top-up advice

Start small (¥20-50), run the flow, confirm model quality and billing are clean before scaling. This advice applies to every platform, including this one.

## Typical scenarios

### A: Daily coding (Cursor + deepseek-v4-pro)

~1M input + 0.3M output/day ≈ **¥2.9/day** (1×1.806 + 0.3×3.612). Note this is a cost estimate only — don't trust anyone's "measured experience" (including mine) for quality and latency; verify with your own script using the method above.

### B: Batch data processing (script + deepseek-v4-flash)

80M input + 20M output/month ≈ **¥67/month** (80×0.559 + 20×1.117). Flash is currently the cost-efficiency floor — great for batch.

### C: Complex tasks (GLM-5.2 / Kimi K3)

GLM-5.2 ranks 5th globally on aitier.net's 2026-06 leaderboard; Qwen3.7 Max 8th — Chinese flagships are no longer "alternatives" but "options" for everyday business. Via an aggregator you save a further 13-24% over official.

### On latency

I don't cite numbers without measurement conditions. Speed is half-determined by the platform's upstream channels — Artificial Analysis public data shows the same model differs 5-10x across providers (kimi-k3: official ~35 t/s, fastest third-party 172 t/s; glm-5.2: 41-438 t/s; public data, no platform promise). **Test it yourself:** 20 requests in each of two windows, and look at TTFT P95, not the average. Method in the first article of this series.

## Value vs. risk

### ✅ Core value

| Value | Detail |
|-------|--------|
| Price transparency | input/output/cache published; reconcile via `/api/pricing` |
| Sane discount | 56-87% of official; volume-purchase logic checks out |
| One key | all Chinese mainstays; zero-cost switching |
| Easy integration | OpenAI-compatible; one base_url line |
| Verifiable billing | full usage + per-request records |

### ⚠️ Things to note

| Risk | What it looks like | Advice |
|------|--------------------|--------|
| Third-party relay | prompts pass through platform servers | fine for personal code; don't put customer data / prod DB fields |
| Price volatility | follows upstream (flash +11% in a week) | re-check prices before heavy usage |
| No public latency guarantee | no SLA numbers | test yourself; keep a fallback for production |
| Industry-wide risk | any aggregator can fold | single-platform balance ≤ one month of usage |

## FAQ

**Q: Is Tokeness the model vendor?**
No. It's a third-party aggregator; requests are forwarded to each vendor's official API.

**Q: How do I confirm I'm getting the real model?**
Three ways: ask version-fingerprint questions (knowledge cutoff) ~10 times and watch for drift; run 5 standard reasoning prompts and compare known outputs; check the returned `usage` field's completeness.

**Q: Which tools work?**
Anything with a custom OpenAI-compatible endpoint: Cursor, Cherry Studio, Cline, custom scripts.

**Q: How fast are new models added?**
Kimi K3, GLM-5.2, Qwen3.7 — all 2026 models are already in the plaza. Check the live model list to be sure.

**Q: Is it worth topping up?**
Top up ¥20-50, test latency and consistency with the first article's method, run one day of real tasks to compute daily cost, then decide. This process is more reliable than any review article, including this one.

## Summary

Tokeness solves a very concrete set of problems: the sign-up, payment, and price barriers of Chinese models, flattened into one account. What won me over isn't the discount — it's **verifiability**: prices, cache ratios, and usage are all public and cross-checkable, which is rare quality in an industry of "official X% off" black boxes.

But remember the third-party boundary: personal development, prototypes, and non-sensitive work are fine; for customer data and compliance-heavy production, assess the data boundary first — or just use the official API.

---

*Prices verified 2026-08-01; models and prices follow the live [tokeness.io/pricing](https://tokeness.io/pricing) page.*
