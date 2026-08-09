---
title: "Codex App（ChatGPT 桌面版）国内安装教程：一键安装与聚合 API 配置（2026）"
description: "国内安装 Codex App（ChatGPT 桌面版）的完整教程：官方 MSIX 下载被墙怎么办、聚合 API 怎么配置、桌面端 CLI IDE 三端共用配置怎么生效。提供一键安装程序与分步说明。"
lang: zh-CN
author: mrshrawho
date: 2026-08-09
related:
  - title: "Claude Desktop 国内安装教程"
    url: "https://mrshrawho.github.io/claude-desktop-install-guide/"
  - title: "Codex App 一键安装程序（产品页）"
    url: "https://l1i1.github.io/CodexAppInstaller/"
  - title: "一个 Key 统一调用多种模型：OpenAI 兼容接口的实际价值"
    url: "https://mrshrawho.github.io/one-key-all-ai-models/"
  - title: "AI API Relay Testing Playbook"
    url: "https://mrshrawho.github.io/ai-relay-testing-playbook/"
---

# Codex App（ChatGPT 桌面版）国内安装教程：一键安装与聚合 API 配置（2026）

## 先搞清楚 Codex App 是什么

Codex App 是 OpenAI 推出的 ChatGPT **桌面应用**（包名 `OpenAI.Codex`，Microsoft Store 里叫 ChatGPT）。它内置 Codex 模式，把编程助手做成了桌面应用。安装源是 Microsoft Store 分发的官方 MSIX 包，签名完整、未重打包。

国内安装的卡点和 Claude Desktop 类似：**MSIX 安装包下载被墙**。官方 Store 走微软/CDN，很多网络环境下下不动。所以问题的核心是：拿到官方 MSIX 并正常装上去。

## 方案一：一键安装程序（推荐）

[Codex App 一键安装程序](https://l1i1.github.io/CodexAppInstaller/)（Windows，约 22KB 单文件）专治这个问题：

- **镜像加速下载**：走 GitHub Release + gh-proxy 双前缀回退，tag 和资产名（x64 / arm64）动态匹配，自动选对版本
- **SHA256 校验**：下载后比对同 Release 的 `SHA256SUMS.txt`，防镜像篡改；MSIX 是 Microsoft Store 签名官方包，**未重打包**
- **静默安装**：机器范围注册（所有用户可用）+ 用户级立即注册，装完创建桌面快捷方式
- **聚合 API 一键配置**：自动写入 `~/.codex/config.toml` 和 `~/.codex/auth.json`，桌面端 / CLI / IDE 三端共用
- **下载缓存**：`%LOCALAPPDATA%\CodexAppInstaller\cache` 跨重启复用，版本更新哈希变化自动失效重下

下载地址：[CodexAppInstaller.exe](https://raw.githubusercontent.com/l1i1/CodexAppInstaller/main/CodexAppInstaller.exe)

## 分步安装教程

1. **下载并运行**：双击 exe，UAC 弹窗点"是"。自动完成：下载 MSIX（镜像加速 + SHA256 校验）→ 静默安装 → 聚合 API 配置 → 创建桌面快捷方式。
2. **聚合 API 配置**（默认 Y，回车即可）：
   - 端点：`https://n.tokeness.io/v1`
   - 模型：`gpt-5.6-sol`
   - 粘贴 `sk-` 开头的 Key
   
   也可以参数直接指定：`CodexAppInstaller.exe -ApiKey sk-xxxxxx`
3. **重启生效**：完全退出（含托盘）并重启 Codex 桌面应用。
4. **开始使用**：直接对话或用 Codex 模式写代码。

![安装脚本运行效果](img/runpic.png)

![Codex App 应用截图](img/codexapppic.png)

## 为什么桌面端、CLI、IDE 三端共用一份配置

Codex 的桌面应用与 CLI、IDE 扩展**共用同一份 `~/.codex/config.toml`**（官方文档原话：agents in the app inherit the same configuration as the IDE and CLI extension）。这意味着配置一次，三端生效，不需要在三个地方分别填端点。

程序写入的内容大致是：

```toml
model = "gpt-5.6-sol"
model_provider = "custom"
preferred_auth_method = "apikey"

[model_providers.custom]
name = "custom"
base_url = "https://n.tokeness.io/v1"
wire_api = "responses"          # Codex 仅支持 Responses 协议
requires_openai_auth = true

[windows]
sandbox = "elevated"
```

API Key 单独放在 `~/.codex/auth.json`（`{"OPENAI_API_KEY": "sk-xxxxxx"}`），**不落 config.toml**，避免明文散落在配置文件里。

这里有个关键约束要记住：**Codex 只支持 OpenAI Responses 协议（`wire_api = "responses"`）**。如果你要换聚合端点，必须确认它兼容 Responses 协议，纯 Chat Completions 端点是连不上的（会报 400 协议错误）。

## 怎么判断聚合端点值不值得用

一键程序默认端点指向 [Tokeness](https://tokeness.io)。不管用哪家，建议按这三条核验（完整方法见 [AI API Relay Testing Playbook](https://mrshrawho.github.io/ai-relay-testing-playbook/)）：

1. **协议兼容**：是否支持 OpenAI Responses API（Codex 必需）
2. **模型可用**：`/v1/models` 里有没有你要的模型 ID，别只看广告页
3. **计费透明**：输入/输出/缓存三套价格是否公开并标注核验日期

## 常见问题

**问：所有下载源都失败怎么办？**
检查网络/代理连通性后重试，程序内置多源回退与 3 次自动重试。

**问：提示 401 鉴权失败？**
检查 `~/.codex/auth.json` 中的 `OPENAI_API_KEY` 是否正确、账户余额是否充足。

**问：模型列表为空？**
检查端点 `/v1/models` 可用性与模型 ID（默认 `gpt-5.6-sol`）是否正确。

**问：跟 winget / Store 安装有什么区别？**
`winget install --id 9PLM9XGG6VKS -s msstore` 或 Store 安装仍然要走被墙的下载链路；一键程序提供镜像加速 + 校验 + 聚合配置，一条龙完成。对供应链敏感的环境，建议走官方渠道 + 挂代理。

## 小结

Codex App 国内安装的核心是"拿到官方 MSIX + 配置可用的聚合端点"。用 [一键安装程序](https://l1i1.github.io/CodexAppInstaller/) 可以一次完成下载、校验、安装、配置四步；换其他聚合端点时记住 Responses 协议约束即可。相关阅读：[Claude Desktop 国内安装教程](https://mrshrawho.github.io/claude-desktop-install-guide/)。
