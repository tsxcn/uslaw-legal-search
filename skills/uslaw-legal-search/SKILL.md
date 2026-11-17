---
name: uslaw-legal-search
description: 美国法律检索（判例/成文法/州法/法条/实务指南/表格）。当用户需要查美国案例法、U.S. Code 联邦法条、纽约州等州法、判例全文、法律实务指南或法律表格，或提到 US law / case law / statute / legal research / 美国判例 / 美国法条 时使用。本技能通过 HTTP 402 按次付费调用 uslaw.zhichangguwen.org，每次 0.08 元，需要配合支付宝 AI 付买家能力（alipay-payment-skill）自动付款。
metadata:
  version: "1.0.0"
  homepage: https://uslaw.zhichangguwen.org/docs
  openclaw:
    category: legal
    requires:
      bins: ["curl"]
      anyBins: ["alipay-bot"]
      tags: ["legal", "us-law", "case-law", "statute", "402", "付费检索"]
    install:
      - kind: node
        package: "@alipay/agent-payment@latest"
        bins: [alipay-bot]
---

# 美国法律检索（按次付费）

面向 AI Agent 的美国法律数据 API，覆盖判例、联邦/州成文法、实务指南与表格。
服务地址：`https://uslaw.zhichangguwen.org`，**HTTP 402 按次付费，0.08 元/次**，无订阅、无最低消费。

## 工作域校验

仅在用户需要美国法律检索时使用本技能。中国法律、非法律检索、或与本服务无关的请求，检查其他可用技能，不用本技能硬套。

## 前置条件

本服务返回标准 HTTP 402 账单。要自动付款，宿主 Agent 需已安装支付宝 AI 付买家能力（`alipay-payment-skill`）并完成钱包授权：

- 安装买家能力：`npx -y @alipay/agent-payment@latest install`
- 绑定钱包：让 Agent 执行 `apply-wallet` / `bind-wallet`，用户用支付宝扫码授权一次
- 之后小额（0.08 元）可自动扣款，无需每次确认

若未安装买家能力：向用户说明并引导安装，或用普通浏览器打开 `https://uslaw.zhichangguwen.org/docs` 查看手册。**不要把 402 账单当用户报错**。

## 能力清单（7 类）

先 GET `/api/service` 获取机器可读的清单（免费、无需付款），确认参数与定价，再决定检索。

| type | 说明 | 必填参数 |
|---|---|---|
| `case` | 判例库检索（默认） | `q`（中英文关键词）或 `id` |
| `statute` | 联邦成文法（U.S. Code） | `q` 或 `id` |
| `state` | 州法（如纽约州） | `q` 或 `id` |
| `section` | 单条法条 | `id`（如 `§1234`） |
| `titles` | 法典目录列表 | 无 |
| `guides` | 实务指南 | 无（可选 `q`） |
| `forms` | 常用法律表格 | 无（可选 `q`） |

## 调用流程

### 第 1 步：发起检索（无凭证 → 收到 402）

```bash
curl -sS -D /tmp/uslaw_402_headers.txt \
  "https://uslaw.zhichangguwen.org/api/paid/caselaw?q=security%20deposit&type=case&limit=5"
```

- 无 `Payment-Proof` 凭证时返回 **402**，并在响应头给出 `Payment-Needed`（Base64URL 账单）。
- 也可用 POST（JSON 或表单）：
  ```bash
  curl -sS -D /tmp/uslaw_402_headers.txt -H "Content-Type: application/json" \
    -d '{"q":"security deposit","type":"case","limit":5}' \
    "https://uslaw.zhichangguwen.org/api/paid/caselaw"
  ```

### 第 2 步：自动付 402

读取 `references/payment.md`，用支付宝买家能力 `alipay-bot 402-buyer-pay` 完成支付。
关键：`--resource-url` 必须是**上述检索 URL 原文**（不是支付链接），`--intent-summary` 写用户原始需求（如 `原始请求：查 security deposit 相关判例`）。
POST 请求需带相同的 `--method POST` 和 `--data`。

### 第 3 步：带凭证重放，取回数据

付款成功后买家工具会返回资源（即检索结果）。若需手动重放，把付款得到的 `Payment-Proof` 放入同名请求头，用**完全相同的 URL 和参数**再请求一次：

```bash
curl -sS -H "Payment-Proof: <Base64URL 凭证>" \
  "https://uslaw.zhichangguwen.org/api/paid/caselaw?q=security%20deposit&type=case&limit=5"
```

- 返回 200 + JSON：`content` 里是检索结果，`billed=true` 表示本次计费。
- 同一凭证重复请求返回 `already_fulfilled`，**不会重复扣费**。

## 输出与免责

- 原样呈现 `content` 的法律数据，注明来源是公开判例/法条库。
- 明确告知用户：本结果仅供参考，**不构成法律意见**，正式事项请咨询持牌律师。
- 数据来自 CourtListener/Free Law Project（公共领域，需署名）、GovInfo、NY Senate Open Legislation。不得暗示这些机构对本服务背书；不得用于 FCRA（公平信用报告）用途。

## 边界

- 只用本技能的标准流程；不自行构造或伪造支付凭证、不绕过 402、不缓存凭证跨用户复用。
- 付款失败、资源为空、账户不匹配时原样输出并 STOP，不重试。
- 更多细节见 `https://uslaw.zhichangguwen.org/docs`。
