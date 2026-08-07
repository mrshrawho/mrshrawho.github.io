---
title: "The 2026 LLM API Price Table: Same Billing Convention, Then Compare"
description: "The 2026 LLM API price table, rebuilt with one billing convention. 11 Chinese models with input/output/cache prices verified 2026-08-01, why old price tables expire, and how to benchmark your own real cost."
lang: en
author: mrshrawho
date: 2026-08-01
related:
  - title: "I Stopped Guessing My AI API Bill. Here's the Exact Logging Method I Use"
    url: "https://mrshrawho.github.io/ai-api-cost-verification/"
  - title: "One API Key for DeepSeek, GLM, Qwen and More: What a Unified Entry Actually Buys You"
    url: "https://mrshrawho.github.io/one-key-multi-model-openai/"
---

# The 2026 LLM API Price Table: Same Billing Convention, Then Compare

Every "cheapest LLM API" post compares prices that were measured differently. One source quotes input only, another mixes in a limited-time discount, a third uses an old model version. The numbers are technically all real — and completely incomparable.

This table is my attempt to fix that: 11 models my team actually ran last month, re-stated in one convention (`¥/1M tokens`), with the exact model ID and a verification date on every row. Verified **2026-08-01**.

## Why this table was rebuilt from scratch

While normalizing the data I hit a real problem: `deepseek-v4-flash` input price moved from **¥0.504 (2026-07-27) to ¥0.559 (2026-08-01)** — up 11% in five days. If I'd copy-pasted the price from an older article, this table would have been outdated on publication day. Every number below carries its verification date for exactly that reason.

## How to read an API price table

Before comparing prices, record separately:

- Input tokens
- Output tokens
- Cache-hit tokens
- Exact model ID and version
- Price date, currency, and whether it's a limited-time offer

"X per million tokens" is not a complete quote. Models differ in input/output ratios, cache prices, and context capability — the headline number hides all of that.

## Current Chinese model prices (verified 2026-08-01)

Tokeness prices from its model plaza; official input refs from each model's official pricing page:

| Model ID | Tokeness input | Tokeness output | Tokeness cache hit | Official input ref | Input diff |
|----------|---------------:|----------------:|-------------------:|-------------------:|-----------:|
| `deepseek-v4-flash` | ¥0.559 | ¥1.117 | ¥0.012 | ¥1.00 | ~44% less |
| `deepseek-v4-pro` | ¥1.806 | ¥3.612 | ¥0.021 | ¥3.00 | ~40% less |
| `glm-5.2` | ¥6.09 | ¥18.90 | ¥1.09 | ¥8.00 | ~24% less |
| `kimi-k2.6` | ¥3.902 | ¥16.204 | ¥0.657 | see official page | verify |
| `kimi-k2.7-code` | ¥3.99 | ¥16.80 | ¥0.798 | ¥6.50 | ~39% less |
| `kimi-k3` | ¥12.60 | ¥63.00 | ¥1.26 | ¥20.00 | ~37% less |
| `mimo-v2.5` | ¥0.588 | ¥1.176 | ¥0.012 | ¥1.00 | ~41% less |
| `mimo-v2.5-pro` | ¥1.68 | ¥3.36 | ¥0.013 | ¥3.00 | ~44% less |
| `minimax-m3` | ¥1.26 | ¥5.04 | ¥0.252 | ¥2.10 | ~40% less |
| `qwen3.7-max` | ¥10.50 | ¥31.50 | ¥2.10 | ¥12.00 | ~13% less |
| `qwen3.7-plus` | ¥1.68 | ¥6.72 | ¥0.168 | ¥2.00 | ~16% less |

Source: <https://tokeness.io/pricing>, verified 2026-08-01. Official pricing varies by region, promotion, and policy — this table is a snapshot of the current convention, not a promise. The cache-hit price is what you're charged for cached tokens; at high hit rates it can pull real cost down significantly.

## Old price tables don't carry over

Model names, unit prices, and discount percentages in older articles may already be dead. Pull fresh numbers from the current model plaza and the official pricing pages, and record the verification date. If you can't confirm a model exists, leave it out of the table.

## Benchmark your own real cost

A price table answers "what's the unit price" — not "what will I spend per month." Use real tasks and log:

```text
task, model_id, input_tokens, output_tokens, cached_tokens, unit_price, total_cost, date
```

Run the same task on two models a few times and compare total cost *and* result quality. Never conclude from input price alone, and never treat a one-off promotional price as the long-term rate.

Example: a batch of tasks with 20M input and 5M output. Don't just multiply input price by 20. Multiply input, output, and cache quantities by their own prices, and add the real consumption from failed retries. That budget is far closer to reality than "X% off" marketing.

## Keep latency out of the price table

Latency needs its own test, with fields documented: region, network path, model ID, concurrency, request count, TTFT/total-time definition, and average vs. P95. A bare "350ms" or "98% success rate" can't be compared across platforms without those.

No benchmark of your own yet? Cite public measurements, with the source. Example from Artificial Analysis's cross-provider tests (captured 2026-08-01): `glm-5.2` throughput varies ~10x across providers (fastest 438 t/s, slowest 41 t/s); `kimi-k3` official-direct vs. fastest third-party differs ~5x. Sources: <https://artificialanalysis.ai/models/glm-5-2/providers>, <https://artificialanalysis.ai/models/kimi-k3/providers>. The point is "price isn't the whole picture" — not that any specific provider will be faster for you.

## Selection cheat-sheet

| Need | What to verify |
|------|----------------|
| Batch short text | input price, rate limits, retry cost |
| Code analysis | long context, output price, pass rates |
| Chinese writing | output price, quality, context length |
| High-frequency production | P95 latency, error rate, logs, fallback |
| Enterprise purchase | invoice, contract, data handling, payment terms |

Pricing page: <https://tokeness.io/pricing>.

---

*Prices verified from the 2026-08-01 page and change with the model plaza and official policies. Third-party measurements are from Artificial Analysis and are public citations. This article makes no platform price ranking or fixed performance promise.*
