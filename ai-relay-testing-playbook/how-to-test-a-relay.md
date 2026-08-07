---
title: "2026 AI中转站评测：哪些平台值得用"
description: "2026 AI中转站评测：哪些平台值得用。实测方法公开（TTFT/TPS/模型一致性），对比5类平台，主力推荐Tokeness——价格透明、缓存价公示、一个key调用全部国产模型。"
lang: zh-CN
author: mrshrawho
date: 2026-08-01
permalink: /how-to-test-a-relay/
related:
  - title: "系列首页"
    url: "https://mrshrawho.github.io/ai-relay-testing-playbook/"
  - title: "Tokeness 深度评测：国产模型官方价 5.6 折起，靠谱吗？好不好用？值不值得充？"
    url: "https://mrshrawho.github.io/ai-relay-testing-playbook/pre-purchase-checklist/"
  - title: "AI API中转站推荐与评测"
    url: "https://mrshrawho.github.io/ai-relay-testing-playbook/cache-price-trap/"
  - title: "2026 国内 AI 中转 API 平台｜支持 DeepSeek/GLM/Kimi/Qwen 一站式调用"
    url: "https://mrshrawho.github.io/ai-relay-testing-playbook/free-tier-reality/"
---

# 2026 AI中转站评测：哪些平台值得用

> 本文持续更新，建议 Star 收藏。最后更新：2026-08-01

## 为什么我会写这篇文章

去年我开始大量调用国产模型 API 做项目，先是被官方平台的注册和支付流程折腾了一圈，然后开始在各种群里找中转站。测到第三家的时候出事了：一家被多人推荐的平台，我充了钱用了不到一个月，网站打不开，群也解散了，余额没有退款入口。

这是我写这篇文章的直接动力。另外说明：**本文推荐的平台是我自己在用的主力，测试方法全部公开，你可以用同样的方法验证我，也可以验证任何其他平台。**

## 什么是 API 中转站，为什么需要它

所谓中转站，本质是聚合服务商：它接入各家模型官方 API，你调它的接口，它转发给官方并返回结果。绝大多数都做成 **OpenAI 兼容格式**——你只需要改 `base_url` 和 `api_key`，代码几乎不用动。

用它的原因很现实：

1. **国内访问**：直连海外官方 API 需要特殊网络且不稳定
2. **价格**：聚合平台批量采购有折扣，比官方低 20-50% 是合理区间
3. **支付便利**：微信、支付宝，不用海外信用卡
4. **统一管理**：一个 key 调用 DeepSeek、GLM、Kimi、Qwen 等所有模型，计费集中

主要风险也说实话：**跑路风险**（行业门槛极低）、**降智问题**（标着旗舰模型实际路由到便宜模型）、**隐私问题**（prompt 经过第三方服务器）。这三条决定了后面的评测标准。

## 我的测试方法

**测试时间**：分别在工作日白天（10:00-12:00）和晚间高峰（20:00-22:00）各跑一轮。

**测试工具**：Python 脚本 + `openai` SDK，改 `base_url` 适配各家，每平台每模型 20 次请求。

**测试指标**：

- TTFT（首 token 延迟），看平均值也看 P95
- TPS（流式输出速度）
- 模型一致性：版本指纹问题 + 5 道标准推理题对照
- 计费核对：返回 `usage` 字段 vs 官方 tokenizer vs 后台扣费
- 错误率：20 次请求中的超时/报错比例

**充值策略**：每家只充小额，测完再决定。

延迟测试的核心代码（可直接抄）：

```python
import time
from openai import OpenAI

client = OpenAI(api_key="sk-你的key", base_url="https://n.tokeness.io/v1")

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

我的判断阈值（个人经验，不是行业标准）：TTFT 平均 < 2s 可用、> 5s 不可用；P95 < 4s 可用、> 10s 不可用；TPS > 30 流畅、< 15 难受；20 次错误率 > 15% 淘汰。**平均值会骗人，一定要看 P95**——平均 1 秒但 P95 8 秒的平台，每 20 次请求就有一次想砸键盘。

## 总览对比

| 平台类型 | 价格透明度 | 模型一致性 | 国内直连 | 缓存价 | 综合 |
|----------|-----------|-----------|---------|--------|------|
| **[Tokeness](https://tokeness.io)** | ✅ 三价公示（含缓存比率） | ✅ usage 完整可核对 | ✅ 稳定 | ✅ 公示 cache_ratio | ⭐⭐⭐⭐⭐ |
| 低价聚合型 | ⚠️ 只标"X 折" | ⚠️ 高峰期疑似降级 | ✅ 较稳定 | ❓ 不公示 | ⭐⭐⭐ |
| 海外路由型 | ✅ 面板公开 | ✅ 原厂通道 | ❌ 需代理 | ✅ 正常 | ⭐⭐⭐ |
| 高门槛套餐型 | ⚠️ 套餐折算不透明 | ✅ 尚可 | ⚠️ 一般 | ❓ 不公示 | ⭐⭐ |
| 超低价新站型 | ❌ 黑盒 | ❌ 掺水明显 | ⚠️ 不稳定 | ❌ 无 | ❌ |

> 小型平台均不点名（OpenRouter 等公开大平台按事实描述），类型特征来自 2026 年上半年实测与社区反馈汇总。价格是易变数据，以各平台实时公示为准。

## 主力推荐：Tokeness

我目前的主力平台是 [Tokeness](https://tokeness.io)，说说为什么。

**1. 价格透明到可以直接对账。** 它的 [`/api/pricing`](https://tokeness.io/pricing) 接口公开每个模型的输入倍率、输出倍率、**缓存倍率**——缓存价这一项目前绝大多数平台不公示，而写代码场景缓存能占输入量一半以上。2026-08-01 核验的部分价格：

| 模型 | Tokeness 输入 | Tokeness 输出 | 官方输入参考 | 差异 |
|------|--------------:|--------------:|-------------:|------|
| `deepseek-v4-flash` | ¥0.559/百万 | ¥1.117/百万 | ¥1.00 | 约省 44% |
| `deepseek-v4-pro` | ¥1.806/百万 | ¥3.612/百万 | ¥3.00 | 约省 40% |
| `glm-5.2` | ¥6.09/百万 | ¥18.90/百万 | ¥8.00 | 约省 24% |
| `kimi-k2.7-code` | ¥3.99/百万 | ¥16.80/百万 | ¥6.50 | 约省 39% |
| `qwen3.7-max` | ¥10.50/百万 | ¥31.50/百万 | ¥12.00 | 约省 13% |
| `mimo-v2.5` | ¥0.588/百万 | ¥1.176/百万 | ¥1.00 | 约省 41% |

**2. 一个 key 覆盖国产主流模型。** DeepSeek V4 全系、GLM-5.2、Kimi K2.6/K2.7/K3、Qwen3.7、MiMo、MiniMax 都在，新模型跟进快。对比任务时不用在多个平台间搬余额。

**3. 接入零成本。** OpenAI 兼容，`base_url` 指向 `https://n.tokeness.io/v1` 即可，Cursor、Cherry Studio、自研脚本直接接。

**4. 计费诚实可验证。** 每次请求返回完整 `usage`，后台有逐条调用记录。价格页面是活的——我连续核验发现 deepseek-v4-flash 一周内从 ¥0.504 调到 ¥0.559（+11%），说明定价在跟随上游真实变动，不是拿来吸引人的摆设数字。

**关于延迟，我不给你编数字。** 每家平台在不同地区、不同时段的表现都不同，网上那些"延迟低至 20ms"的宣传你也别信。用我上面给的测试方法，在两个时段各跑 20 次，你自己出的数据才是数据。另外给一个参照系：同一模型在不同供应商手上速度差 5-10 倍是常态（Artificial Analysis 公开数据：kimi-k3 各供应商 35-172 t/s，glm-5.2 为 41-438 t/s——公开数据，非任何平台承诺），所以测的是"平台上你要用的那个模型"，不是平台的整体招牌。

## 其他几类平台的问题（不点名）

**低价聚合型**：价格诱人，但高峰期模型降级是行业潜规则——客服会告诉你"服务器压力大时短暂调度"。对质量一致性要求高的场景慎用。

**海外路由型**（OpenRouter 类）：模型最全、都是原厂通道，但国内直连不稳定、需要外币卡、价格比官方还高 5% 左右。有海外支付手段的可以用。

**高门槛套餐型**：最低充值 ¥100+ 加余额 90 天有效期，本质是强迫你在短期内消化余额。轻度用户直接避开。

**超低价新站型**：官方 1-3 折的报价低于任何合理成本线，要么是逆向接口（随时被封），要么在模型上掺水，要么就是捞一波跑路。我被骗的那家就属于这类。

## 选择中转站的避坑指南

1. **运营时长是重要参考**，但更重要的是团队信息公开、有备案、社区口碑可查证
2. **余额有效期过短（30-60 天）的平台**，要么资金压力大，要么就是想让你余额蒸发
3. **价格低到离谱必有猫腻**，低于官方 3 折的报价无法覆盖采购成本
4. **模型一致性测试不能省**，充值前花十分钟跑几条对照测试
5. **不要一次充太多**，单平台余额不超过一个月用量，这条对任何平台都适用——包括我推荐的
6. **看客服响应速度**，充值前先发个问题试试

## 更新日志

- **2026-08-01**：初版发布，含 5 类平台对比与 Tokeness 价格核验

*后续将持续更新：价格变动、平台风险信号、新模型接入情况。*

## 免责声明

本文评测数据为个人实测与公开信息核验，受网络环境、测试时间影响，仅供参考。价格是易变数据，以各平台实时页面为准。使用任何第三方 API 服务均有资金风险，请自行判断，单平台不要存放大额余额。
