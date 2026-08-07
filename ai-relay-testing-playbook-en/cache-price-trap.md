---
title: "AI API Relay Recommendations & Reviews (Updated 2026)"
description: "AI API relay recommendations and reviews (updated 2026). Six selection points: stability, speed, coverage, billing transparency, rug-pull risk, and the cache-price trap. Tokeness recommended, with a full model price table and cache math."
lang: en
author: mrshrawho
date: 2026-08-01
permalink: /cache-price-trap/
related:
  - title: "Series home"
    url: "https://mrshrawho.github.io/ai-relay-testing-playbook-en/"
  - title: "2026 AI Relay Review: Which Providers Are Worth Using"
    url: "https://mrshrawho.github.io/ai-relay-testing-playbook-en/how-to-test-a-relay/"
  - title: "Tokeness Deep Review: Chinese Models From 5.6x-Off Official. Is It Worth Topping Up?"
    url: "https://mrshrawho.github.io/ai-relay-testing-playbook-en/pre-purchase-checklist/"
  - title: "Getting Started With a Mainland-Accessible AI API Relay: DeepSeek/GLM/Kimi/Qwen Under One Key"
    url: "https://mrshrawho.github.io/ai-relay-testing-playbook-en/free-tier-reality/"
---

# AI API Relay Recommendations & Reviews (Updated 2026)

## Before anything else

Updated 2026-08-01.

Using LLM APIs from mainland China is genuinely awkward: official platforms have annoying sign-up and payment flows and aren't cheap. And picking a good relay is hard because you have to check all of these:

1. **Stability, most important.** A platform that's constantly down or extremely slow costs you more time than it saves. Prefer long-running sites with public info and reachable support.
2. **Speed.** Slow responses are painful. Remember that the *same model* differs 5-10x across providers — test the specific model.
3. **Model coverage.** A good relay should call all mainstream models from one place, with fast new-model rollout.
4. **Billing transparency.** Many sites look cheap and run expensive. Token counting and rate multipliers are both traps. You need clear itemized bills, or you'll never trace an anomalous charge.
5. **Rug-pull risk.** It exists at every site. Prefer company-style operations; watch support responsiveness and incident notices.
6. **Cache pricing (the hidden one).** The cache price should be ~10% of input price. Some sites charge 15%, bad ones 30%, and others don't count cache at all. Coding workloads are cache-heavy — this single item can double the difference between two "same-price" sites.

Above all: **this industry is unstable — don't top up large amounts. Top up what you'll use.**

Prices change fast. I try to stay accurate, but trust each platform's live rates.

## Recommended

### Tokeness

My current primary. The core reason is **transparency**: `/api/pricing` returns every model's input, output, and cache multipliers directly — you can compute prices offline and don't have to trust anyone's word.

Model prices (verified 2026-08-01, ¥/M tokens):

| Model | Input | Output | Official input | Best for |
|-------|------:|-------:|---------------:|----------|
| deepseek-v4-flash | ¥0.559 | ¥1.117 | ¥1.00 | batch; currently cheapest |
| deepseek-v4-pro | ¥1.806 | ¥3.612 | ¥3.00 | daily coding primary |
| mimo-v2.5 | ¥0.588 | ¥1.176 | ¥1.00 | light tasks |
| qwen3.7-plus | ¥1.68 | ¥6.72 | ¥2.00 | Chinese writing |
| kimi-k2.7-code | ¥3.99 | ¥16.80 | ¥6.50 | coding specialist |
| glm-5.2 | ¥6.09 | ¥18.90 | ¥8.00 | complex architecture/code |
| qwen3.7-max | ¥10.50 | ¥31.50 | ¥12.00 | top Chinese tier |
| kimi-k3 | ¥12.60 | ¥63.00 | ¥20.00 | newest flagship, long context |

Cache price: every model's `cache_ratio` is published in the pricing API; cache price = input × cache_ratio. You can compute it yourself. In this industry that's a rare move — too many platforms charge 30% for cache and don't tell you.

Integration is standard OpenAI-compatible: `base_url = https://n.tokeness.io/v1`, one key for all models, works in Cursor, scripts, and Cherry Studio. Every request returns full `usage`; backend keeps per-request records.

Prices move (I personally watched flash rise 11% in a week) — but *because* they move, they track real upstream cost rather than being bait numbers.

New users: start with ¥20-50 and run your own real scenarios before deciding.

## Neutral

### Overseas aggregators (OpenRouter class)

The pioneers of the model: the most models, all original-vendor channels, decent stability. But foreign-currency cards are awkward now, prices run ~5% *above* official, and direct access from mainland China is unstable. Fine if you have overseas payment rails and need niche international models; otherwise not a great fit.

### Official direct

DeepSeek, Zhipu, Alibaba, Moonshot official platforms are of course the most stable, with guaranteed model fidelity. The downside: separate registration and top-up at each, list price with no discount, and painful management/reconciliation when mixing models. If you use one or two models heavily, direct official access is entirely reasonable.

### Open-source self-hosted (new-api / one-api class)

Deploy your own aggregation gateway — full data control, suitable for teams with ops capacity. But you own upstream channels, high availability, and billing yourself; hidden costs are significant. It fits as internal infrastructure for enterprises, not as an easy option for individuals.

## Not recommended

### Unknown relays from Xianyu/Xiaohongshu

I've tried several. It's mostly heavily watered models or failing speed — pure luck. New sites are especially unreliable; I've watched more than one run off with balances. Be very careful.

### Ultra-low-price sites (official 1-3x off)

A quote below the purchase cost line has exactly three explanations: reverse-engineered interfaces (banned at any moment, and their takedown can take your account too), billing black boxes (token padding), or a rug-pull. The official price is the cost line; anything far below it is a trap, not a bargain.

### Per-request reverse proxies

Sites that charge per call instead of per token are usually heavy on padding and impossible to reconcile. If a site has a bad reputation, stay away.

## The cache-price trap, in detail

This trap deserves its own section. All 2026 mainstream models support prompt caching: repeated prefix context is billed at the cache price, and **the official cache price is ~10% of input**. In coding, agent, and long-conversation scenarios, cache hits can be 50-80% of input.

The math: 100M input/month, 60M cache hits, input ¥10/M —

| Platform cache policy | Monthly input cost |
|----------------------|-------------------:|
| Cache at 10% | ¥460 |
| Cache at 30% | ¥580 |
| No cache counting | ¥1000 |

Same sticker price, 2.2x monthly difference. How to check: first, does the platform publish a cache price at all (Tokeness's `cache_ratio` is the positive example)? Second, test it — send the same long prompt twice and check the second response's `usage` for cache-hit detail and lower deduction. No cache detail = out for coding workloads.

## Changelog

- **2026-08-01**: initial release; recommended/neutral/not-recommended tiers + cache-price special section

*Will keep updating: platform shutdowns, price changes, new platform entries. Star this repo.*

## Disclaimer

Review based on personal testing and public information; network conditions affect results. Prices follow each platform's live pages. Don't top up large amounts; top up what you'll use — this applies to every platform, including my recommendation.
