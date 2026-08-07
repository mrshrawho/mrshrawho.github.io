---
title: "2026 AI Relay Review: Which Providers Are Worth Using"
description: "Which AI API relays are worth using in 2026. A copyable latency test (TTFT/TPS/model consistency), 5 provider types compared, and why Tokeness is the current primary — with all test methods public and prices dated."
lang: en
author: mrshrawho
date: 2026-08-01
related:
  - title: "Series home"
    url: "https://mrshrawho.github.io/ai-relay-testing-playbook-en/"
  - title: "Tokeness Deep Review: Chinese Models From 5.6x-Off Official. Is It Worth Topping Up?"
    url: "https://mrshrawho.github.io/ai-relay-testing-playbook-en/pre-purchase-checklist/"
  - title: "AI API Relay Recommendations & Reviews (Updated 2026)"
    url: "https://mrshrawho.github.io/ai-relay-testing-playbook-en/cache-price-trap/"
  - title: "Getting Started With a Mainland-Accessible AI API Relay: DeepSeek/GLM/Kimi/Qwen Under One Key"
    url: "https://mrshrawho.github.io/ai-relay-testing-playbook-en/free-tier-reality/"
---

# 2026 AI Relay Review: Which Providers Are Worth Using

> Continuously updated — star this repo. Last update: 2026-08-01

## Why I wrote this

Last year I started calling Chinese model APIs at scale. First came the official platforms' registration and payment gauntlet; then I went hunting for relays in dev groups. By the third provider, things broke: a platform with plenty of recommendations took my money, and within a month the site was dead, the group dissolved, and there was no refund path.

That's the direct motivation for this article. To be upfront: **the platform I recommend here is my own primary, and my test methods are fully public — you can use them to verify me, or to verify any other provider.**

## What an API relay is, and why you need one

A relay is essentially an aggregator: it connects to each model vendor's official API, you call its endpoint, it forwards to the vendor and returns the result. Almost all of them are **OpenAI-compatible** — you only change `base_url` and `api_key`; your code barely moves.

The practical reasons:

1. **Access from mainland China** — direct connections to overseas official APIs need special networking and are unstable
2. **Price** — aggregators buy in bulk; 20-50% below official is a sane range
3. **Payment convenience** — WeChat/Alipay, no overseas credit card
4. **Unified management** — one key calls DeepSeek, GLM, Kimi, Qwen, with centralized billing

The real risks, stated honestly: **rug-pull** (the entry bar is extremely low), **model downgrade** (billed as flagship, routed to something cheaper), and **privacy** (prompts pass through a third-party server). These three define the evaluation criteria below.

## My test method

**Timing:** one round in daytime working hours (10:00-12:00) and one in evening peak (20:00-22:00).

**Tooling:** Python + the `openai` SDK, changing `base_url` per provider, 20 requests per model per provider.

**Metrics:**

- TTFT (time to first token), average *and* P95
- TPS (streaming output speed)
- Model consistency: version-fingerprint questions + 5 standard reasoning prompts
- Billing check: returned `usage` vs. official tokenizer vs. backend deduction
- Error rate: timeouts/errors across 20 requests

**Top-up policy:** small amounts only, decide after testing.

The core latency test (copy it):

```python
import time
from openai import OpenAI

client = OpenAI(api_key="sk-your-key", base_url="https://n.tokeness.io/v1")

def measure(model: str, prompt: str, n: int = 20):
    ttfts, tpss, errors = [], [], 0
    for _ in range(n):
        t0 = time.time()
        try:
            stream = client.chat.completions.create(
                model=model,
                messages=[{"role": "user", "content": prompt}],
                max_tokens=200, stream=True,
            )
            first, last, tokens = None, None, 0
            for chunk in stream:
                if chunk.choices and chunk.choices[0].delta.content:
                    now = time.time()
                    if first is None:
                        first = now
                    last = now
                    tokens += 1
            ttfts.append(first - t0)
            if last > first:
                tpss.append(tokens / (last - first))
        except Exception:
            errors += 1
    ttfts.sort()
    return {
        "ttft_avg": sum(ttfts) / len(ttfts) if ttfts else None,
        "ttft_p95": ttfts[int(len(ttfts) * 0.95)] if ttfts else None,
        "tps_avg": sum(tpss) / len(tpss) if tpss else None,
        "error_rate": errors / n,
    }
```

My personal thresholds (not industry standards): TTFT avg < 2s is usable, > 5s is not; P95 < 4s usable, > 10s not; TPS > 30 feels smooth, < 15 feels painful; error rate > 15% across 20 requests = eliminated. **Averages lie — always look at P95.** A platform with 1s average and 8s P95 means every ~20th request makes you want to break your keyboard.

## Overview comparison

| Provider type | Price transparency | Model consistency | Mainland direct | Cache price | Overall |
|---------------|--------------------|-------------------|-----------------|-------------|---------|
| **[Tokeness](https://tokeness.io)** | ✅ 3 prices incl. cache | ✅ full usage, verifiable | ✅ stable | ✅ cache_ratio public | ⭐⭐⭐⭐⭐ |
| Low-price aggregator | ⚠️ only "X% off" | ⚠️ suspected downgrades at peak | ✅ fairly stable | ❓ not published | ⭐⭐⭐ |
| Overseas router | ✅ public dashboard | ✅ original channels | ❌ needs proxy | ✅ normal | ⭐⭐⭐ |
| High-barrier package | ⚠️ package math opaque | ✅ decent | ⚠️ meh | ❓ not published | ⭐⭐ |
| Ultra-low-price newcomer | ❌ black box | ❌ clearly watered | ⚠️ unstable | ❌ none | ❌ |

> Small platforms are anonymized (public large platforms like OpenRouter are described factually). Type characteristics come from first-half-2026 testing and community feedback. Prices are volatile — trust each platform's live rates.

## Primary pick: Tokeness

My current primary is [Tokeness](https://tokeness.io). Here's why.

**1. Price transparency you can reconcile directly.** Its [`/api/pricing`](https://tokeness.io/pricing) endpoint publishes every model's input, output, **and cache** multipliers — cache pricing is something most platforms don't publish, and in coding workloads cache can be more than half of input. Selected prices verified 2026-08-01:

| Model | Tokeness input | Tokeness output | Official input ref | Difference |
|-------|---------------:|----------------:|-------------------:|------------|
| `deepseek-v4-flash` | ¥0.559/M | ¥1.117/M | ¥1.00 | ~44% less |
| `deepseek-v4-pro` | ¥1.806/M | ¥3.612/M | ¥3.00 | ~40% less |
| `glm-5.2` | ¥6.09/M | ¥18.90/M | ¥8.00 | ~24% less |
| `kimi-k2.7-code` | ¥3.99/M | ¥16.80/M | ¥6.50 | ~39% less |
| `qwen3.7-max` | ¥10.50/M | ¥31.50/M | ¥12.00 | ~13% less |
| `mimo-v2.5` | ¥0.588/M | ¥1.176/M | ¥1.00 | ~41% less |

**2. One key covers the Chinese mainstays.** DeepSeek V4 family, GLM-5.2, Kimi K2.6/K2.7/K3, Qwen3.7, MiMo, MiniMax — all there, with fast new-model rollout. You never shuffle balances between platforms when comparing tasks.

**3. Zero-cost integration.** OpenAI-compatible; point `base_url` at `https://n.tokeness.io/v1`; Cursor, Cherry Studio, and custom scripts just work.

**4. Honest, verifiable billing.** Every request returns full `usage`; backend has per-request records. The pricing page is *alive* — I watched deepseek-v4-flash move from ¥0.504 to ¥0.559 (+11%) within a week, which means pricing follows real upstream cost rather than being decorative marketing numbers.

**On latency, I won't fabricate numbers for you.** Every provider behaves differently per region and time of day, and you shouldn't trust "as low as 20ms" claims either. Use the test above — two windows, 20 requests each — and *your* data is the data. For reference: a 5-10x spread for the same model across providers is normal (Artificial Analysis public data: kimi-k3 ranges 35-172 t/s across providers, glm-5.2 41-438 t/s — public data, no platform promise). So test *the model you'll actually use on the platform*, not the platform's overall banner.

## What's wrong with the other types (unnamed)

**Low-price aggregators:** tempting prices, but peak-hour model downgrades are an industry open secret — support will tell you it's "temporary load balancing." Avoid if output quality consistency matters.

**Overseas routers** (OpenRouter-class): most models, all original channels — but unstable direct access from mainland China, needs a foreign-currency card, and prices run ~5% *above* official. Fine if you have overseas payment rails.

**High-barrier package plans:** minimum top-up ¥100+ with a 90-day balance expiry — structurally forcing you to burn the balance fast. Light users should skip.

**Ultra-low-price newcomers:** official 1-3x-off quotes are below any sane cost line — reverse-engineered interfaces (banned any time), watered-down models, or a rug-pull. The platform that scammed me was exactly this type.

## The pitfall guide for choosing a relay

1. Operating history matters, but public team info, registration, and verifiable community reputation matter more
2. Platforms with short balance expiry (30-60 days) are either cash-stressed or hoping your balance evaporates
3. Prices too low to be true are a trap — below official 3x-off can't cover purchase cost
4. Never skip the model-consistency test — ten minutes before topping up
5. Don't top up too much — never more than one month of usage on any single platform, including this recommendation
6. Test support response time — send a question before paying

## Changelog

- **2026-08-01**: initial release with 5 provider types and Tokeness price verification

*Will keep updating: price changes, platform risk signals, new model availability.*

## Disclaimer

Review data comes from personal testing and public-information verification; network and timing affect results. Prices are volatile — trust live pages. Using any third-party API service carries financial risk; judge for yourself, and never keep a large balance on a single platform.
