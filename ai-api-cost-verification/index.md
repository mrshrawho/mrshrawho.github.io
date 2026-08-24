---
title: "I Stopped Guessing My AI API Bill. Here's the Exact Logging Method I Use"
description: "How to verify AI API costs and stability using two months of request logs — without trusting any provider's marketing. Includes exact model IDs, input/output/cache pricing verified 2026-08-01, and a reproducible latency test."
lang: en
author: mrshrawho
date: 2026-08-01
related:
  - title: "The 2026 LLM API Price Table: Same Billing Convention, Then Compare"
    url: "https://mrshrawho.github.io/llm-api-pricing-2026/"
  - title: "One API Key for DeepSeek, GLM, Qwen and More: What a Unified Entry Actually Buys You"
    url: "https://mrshrawho.github.io/one-key-multi-model-openai/"
---

# I Stopped Guessing My AI API Bill. Here's the Exact Logging Method I Use

My first month with multi-model API access, I made three mistakes that cost me a weekend of backtracking — and a bill that didn't match anything I'd planned.

- **I compared only input prices and topped up.** A platform advertised `deepseek-v4-flash` input at a great rate. I ignored output and cache pricing and ran a batch workload for a week. My output tokens were 3x my input tokens, and the real bill was ~60% higher than my estimate.
- **I estimated costs from wallet balance.** The platform had no per-request logs. At month-end all I could see was "¥200 gone" — no way to trace which model or which job spent it.
- **I judged latency from one request at peak hour.** Wednesday 9 PM, TTFT measured 6 seconds. I wrote the platform off. Same model, next morning: 1.8 seconds. One request proves nothing.

The second month I started logging every request. That's when the numbers finally lined up. This article is that method — it's about how to *verify*, not which platform I rank first.

## Why verifying matters more than picking

Once a team wires up several models, the hard part isn't the first line of integration code. It's everything after:

- Base URLs and model IDs differ per provider
- Input, output, and cache prices get mixed up easily
- When you swap models for the same task, nobody can trace what it actually cost
- One request can't tell you whether evenings will be stable

So I stopped reading ranking posts and started keeping records I could re-check months later.

## Step 1: Fix the model and price baseline

Before comparing anything, pin down four things: the **exact model ID**, the **direction** (input vs. output), the **billing unit**, and the **verification date**. Example models verified 2026-08-01 from Tokeness's model plaza, in `¥/1M tokens`:

| Model ID | Input | Output | Cache hit | Official input ref |
|----------|------:|-------:|----------:|-------------------:|
| `deepseek-v4-flash` | ¥0.559 | ¥1.117 | ¥0.012 | ¥1.00 |
| `deepseek-v4-pro` | ¥1.806 | ¥3.612 | ¥0.021 | ¥3.00 |
| `glm-5.2` | ¥6.09 | ¥18.90 | ¥1.09 | ¥8.00 |
| `qwen3.7-max` | ¥10.50 | ¥31.50 | ¥2.10 | ¥12.00 |

Source: <https://tokeness.ai/pricing>, verified 2026-08-01. Official input refs come from each model's official pricing page. **Pricing changes:** `deepseek-v4-flash` input rose from ¥0.504 (2026-07-27) to ¥0.559 (+11%). Nothing else moved. This is exactly why every price table must carry a verification date — copy-pasting from an older article gives you stale numbers.

The same model name can also point to different versions, routes, and cache policies. Never keep reusing a retired model ID or an old quote as today's price.

## Step 2: Reconcile usage and billing from logs

Base month-end checks on request logs, not wallet balance. Keep at least:

```text
timestamp, model_id, input_tokens, output_tokens, cached_tokens, status, cost
```

Example: a batch job using `deepseek-v4-flash` with 80M input + 20M output per month, at the 2026-08-01 prices:

```text
80 × ¥0.559 + 20 × ¥1.117 = ¥44.72 + ¥22.34 = ¥67.06
```

That's an illustration of the math, not a promise of your bill. Real numbers also depend on cache hits, retries, model switches, and the platform's price on the day.

## Step 3: Don't judge stability from one request

I test in two time windows and record the conditions in every log entry:

| Field | What I record |
|-------|---------------|
| Date & timezone | e.g. 2026-08-01, Asia/Shanghai |
| Network path | home broadband / cloud server / office |
| Model ID | exact string from the platform |
| Volume | request count and concurrency per model |
| Success rate | % with HTTP success + complete result |
| Latency | TTFT and total time, average or P95 |
| Errors | status codes, error type, retried? |

Sample size, region, concurrency, and metric definitions all have to be public for "98% success" and millisecond numbers to be comparable. Without raw logs, you can write up personal experience — but you can't present it as an SLA.

## Step 4: If you can't benchmark yourself, cite public data properly

No test setup yet? You can cite third-party measurements, but state the source and methodology — don't dress them up as your own. Example: Artificial Analysis's cross-provider test of `kimi-k3` (captured 2026-08-01):

| Provider type | Throughput | First-chunk latency |
|---------------|-----------:|--------------------:|
| Official direct | 35 t/s | 3.94 s |
| Fastest third-party | 172 t/s | 1.25 s |
| Second-fastest | 164 t/s | 1.13 s |

Source: <https://artificialanalysis.ai/models/kimi-k3/providers>. The correct takeaway: **the same model can be 5x faster at one provider than another, so choosing a relay on price alone is incomplete.** It cannot prove that any particular platform will be fast for *you* — your region, time of day, concurrency, and prompt length all change the result.

## Step 5: Verify token billing

Don't assume one tokenizer describes every model. Different models use different tokenizers and cache rules. A safer routine:

1. Read the `usage` field in the model's response
2. Cross-check with the model's official tokenizer (if published)
3. Compute cost for input, output, and cached tokens separately
4. Run short, Chinese, and long texts
5. Save raw JSON, billing records, and your calculation sheet

If you can only see wallet balance — no `usage`, no per-token breakdown — don't publish a "padding percentage" conclusion. You don't have the data.

## The unified-entry benefit, honestly stated

Tokeness's docs publish an OpenAI-compatible Base URL:

```text
https://n.tokeness.dev/v1
```

A unified entry means: some OpenAI-compatible clients can be reused, model IDs/volume/cost can be logged in one place, and switching between already-integrated models requires fewer code changes. It does *not* mean every model and native feature is compatible. Before shipping, check the current status, protocol type, and response fields for each model in the plaza.

## My routine now

1. Test with a small amount of non-sensitive data first
2. Save model IDs, price dates, and request logs
3. Test in your target region and your target time windows
4. Keep a fallback entry point for production
5. Recompute the bill whenever a model or price changes

---

*Price examples verified on the 2026-08-01 page; models, prices, and service capabilities are subject to the live official pages. Third-party measurements are from Artificial Analysis and are public citations, not a performance promise from any platform. This article describes a verification method, not a fixed savings or performance guarantee.*
