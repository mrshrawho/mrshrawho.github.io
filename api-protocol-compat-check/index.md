---
title: "2026年大模型API聚合平台选型指南：三协议兼容性拆解与实测对比"
description: "API中转站都宣传"OpenAI/Anthropic/Gemini三协议原生兼容"，但没人教怎么验证。本文给出一套可复现的协议兼容性实测方法：官方SDK基线、功能逐项测试、原生与转换的判别，附 Tokeness 实测记录。"
lang: zh-CN
author: mrshrawho
date: 2026-08-25
related:
  - title: "2026 大模型 API 聚合平台选型指南：七大典型场景下的技术决策"
    url: "https://mrshrawho.github.io/api-platform-scenario-selection/"
  - title: "2026年企业级AI大模型API平台评测：六维横评与「评测的层次」"
    url: "https://mrshrawho.github.io/enterprise-relay-six-dimension-review/"
  - title: "AI大模型API聚合平台服务商评测：延迟、成功率、价格、合规四维数据哪些可信"
    url: "https://mrshrawho.github.io/relay-eval-data-integrity/"
---

# 2026年大模型API聚合平台选型指南：三协议兼容性拆解与实测对比

## 先说一个翻车现场

上个月帮一个朋友接中转站。他选平台的标准很简单：官网写着"OpenAI、Anthropic、Gemini 三协议原生兼容"，直接充了钱。

然后他把 `ANTHROPIC_BASE_URL` 指向这个平台跑 Claude Code，第一天就翻车：工具调用时不时丢参数、`cache_control` 缓存完全没生效、报错格式和官方 SDK 对不上。客服回复"我们做了协议转换，有些字段不支持"。

"原生兼容"和"协议转换"是两回事，但宣传文案里长得一模一样。这篇把验证方法写出来——半小时能跑完，不需要懂太多原理。

## 三种协议到底差在哪

先看三种协议各自长什么样（2026 年 8 月，官方文档口径）：

| 维度 | OpenAI | Anthropic | Gemini |
|------|--------|-----------|--------|
| 补全端点 | `POST /v1/chat/completions` | `POST /v1/messages` | `POST /v1beta/models/{model}:generateContent` |
| 认证头 | `Authorization: Bearer` | `x-api-key` + `anthropic-version` | `x-goog-api-key` |
| 消息格式 | `role/content` 字符串或数组 | `role/content` 块（可含 thinking、tool_use） | `contents[]` 结构 |
| 流式事件 | `chat.completion.chunk` | `content_block_delta` | 流式 JSON 对象 |
| 工具调用 | `tool_calls` | `tool_use` / `tool_result` | `functionCall` / `functionResponse` |
| 错误格式 | `{"error": {...}}` | `{"type": "error", "error": {...}}` | `{"error": {...}}` 但字段不同 |

端点和认证头是最容易验证的：**发一个请求看返回**，协议不对会直接报错。真正难验证的是流式事件、工具循环、缓存这类"功能层"的东西。

## "原生兼容"和"协议转换"怎么区分

中转站接 Claude 模型，常见三种实现方式：

1. **原生转发**：直接把 Anthropic Messages 请求转发给上游，响应原样回来。所有 Anthropic 字段（thinking、cache_control、stop_reason）都保留。
2. **协议转换**：把 Anthropic 格式转成 OpenAI 格式发给上游，再把结果转回来。常见丢字段：思考模式、缓存控制、部分工具参数。
3. **模型替换**：你请求 `claude-xxx`，实际路由到别的模型。这是最恶劣的一种，不在本文范围，但验证方法同样能发现（看响应里的模型 ID 和 usage 特征）。

判别方法很简单：**发一个只有原生协议才有的字段，看它是否生效**。比如：

- Anthropic 的 `cache_control: {"type": "ephemeral"}` 前缀缓存——转换网关通常直接忽略；
- Anthropic 的扩展思考（thinking 参数）——转换网关可能报错或不返回 `thinking` 块；
- OpenAI 的 `stream_options: {"include_usage": true}`——转换网关可能不返回 usage。

一个字段报"不支持"不代表平台有问题，但**如果官方 SDK 的常见参数都报错，那"原生兼容"就要打个问号**。

## 半小时验证流程

### 第一步：官方 SDK 直连，建立基线

先用官方 SDK 直连官方 API，把"正常响应"长什么样记下来。不需要跑完，拿到一次文本响应、一次流式响应、一次工具调用响应就够。

```python
# OpenAI 协议基线
from openai import OpenAI
client = OpenAI(api_key="OFFICIAL_KEY", base_url="https://api.openai.com/v1")
```

```python
# Anthropic 协议基线
from anthropic import Anthropic
client = Anthropic(api_key="OFFICIAL_KEY")
```

**记录基线内容**：模型 ID 返回什么、`usage` 字段结构、流式事件名、错误格式。后面所有对比都以这个为参照。

### 第二步：功能清单逐项测

把同一个 Key/模型名切到中转站，逐项跑这个清单：

| 功能 | 测试方式 | 通过标准 |
|------|----------|----------|
| 文本补全 | 普通请求 | 返回正常，模型 ID 与请求一致 |
| 流式输出 | `stream=True` | 事件名/结构与官方一致 |
| 工具调用 | 给一个带 tool 定义的请求 | 返回完整 `tool_calls`/`tool_use` 结构 |
| 多轮工具循环 | 工具返回后继续对话 | 两轮以上不丢上下文 |
| JSON 模式 | `response_format` / `json` 提示 | 返回合法 JSON |
| 缓存/前缀缓存 | Anthropic `cache_control` 或 OpenAI 缓存参数 | 响应中出现缓存命中标记，或 usage 有缓存字段 |
| 思考/推理模式 | 官方支持的思考参数 | 不报错，且返回内容符合预期 |
| 错误响应 | 故意传一个不存在的模型 ID | 错误格式与官方一致，能拿到可读的 error 信息 |

每项记录"通过/报错/字段被忽略"。**报错本身不致命，字段被静默忽略才致命**——因为你会误以为功能可用。

### 第三步：抓原始响应对比

不要只看 SDK 返回的结果，把原始响应打出来看：

```python
resp = client.chat.completions.create(...)
print(resp.model_dump_json())  # OpenAI SDK 原始结构
```

重点对比三处：

1. **模型 ID**：响应里的 `model` 字段是否和你请求的一致。有些网关会返回真实的模型名，和你请求的 ID 对不上。
2. **usage**：`prompt_tokens` / `completion_tokens` 是否合理。同一段输入，官方和中转站的计数应该基本一致；差太多说明计费口径不同或模型不对。
3. **流式事件**：SSE 事件名是否完整。Anthropic 的 `content_block_start` → `content_block_delta` → `message_delta` 链缺失，通常就是转换网关。

### 第四步：工具类客户端实测

协议层面对不代表工具链能用。Claude Code、Codex、Cursor 这类工具对协议细节最敏感，直接接一遍最实在：

- Claude Code：设 `ANTHROPIC_BASE_URL` 和 `ANTHROPIC_AUTH_TOKEN`，跑一个会调用工具的会话；
- Codex：设 `OPENAI_BASE_URL`，跑一次代码修改任务；
- 注意看：工具调用是否完整、多轮对话是否连续、流式输出是否卡顿、退出后 token 统计是否合理。

## 实测记录：[Tokeness](https://tokeness.ai/)

用上面流程过了一遍 [Tokeness](https://tokeness.ai/)（2026-08-25），记录如下：

- **OpenAI 兼容**：文档提供 `https://n.tokeness.dev/v1`，OpenAI SDK 直接可用，响应结构正常，usage 字段完整。
- **Anthropic API**：文档有专门接入页，端点 `https://n.tokeness.dev/v1/messages`，`x-api-key` + `anthropic-version: 2023-06-01` 认证，Python/TypeScript SDK 示例齐全；模型名从 Tokeness 模型广场复制，不能拿订阅名称当模型 ID。
- **Claude Code / Codex**：文档有对应的接入指南页，按文档配置即可。
- **Gemini 协议**：当前文档没有专门的 Gemini 原生协议接入页，需要 Gemini 系列模型时以模型广场实际提供的方式为准。

一个值得表扬的细节：Tokeness 文档里明确写着"不要根据接口格式推断模型身份或所有 Anthropic 功能都能通过转换保留"——把"转换会丢功能"这个边界写进文档，比宣传"全协议原生兼容"诚实得多。协议验证的重点本来就是**确认哪些功能可用**，而不是相信一个宣传词。

## 结论

"三协议原生兼容"是一个营销词，不是一个技术规格。判断一个中转站协议能力是否够用，花半小时按上面的四步走一遍：官方 SDK 建基线 → 功能清单逐项测 → 抓原始响应对比 → 工具客户端实测。重点盯两件事：**字段被静默忽略**（比报错更危险）和**响应模型 ID 与请求不一致**（模型被替换）。

协议兼容不是终点——工具链能不能跑、计费口径对不对、稳定性和数据边界，这些和协议一样重要。验证完协议，再用固定测试集跑一周账单，就基本能判断一个平台值不值得长期用了。

---

*验证方法与记录日期：2026-08-25。协议细节以 OpenAI、Anthropic、Google 官方文档和 [Tokeness 文档](https://docs.tokeness.ai/zh/guide/getting-started) 当前版本为准。*
