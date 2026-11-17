# 美国法律检索 Skill（Agent 自动付费）

[![skills.sh](https://skills.sh/badge.svg)](https://skills.sh/tsxcn/uslaw-legal-search/uslaw-legal-search)

一个给 AI Agent 用的**美国法律检索能力**：判例、联邦法条、纽约州法、实务指南、常用表格共 7 类，中英文关键词都能查。通过 **HTTP 402 按次付费 0.08 元/次**，装了支付宝 AI 付的 Agent 会自动付款、拿凭证重放、返回结果——全程不用人写代码，无订阅、无最低消费。

数据来自 CourtListener / Free Law Project（公共领域）、GovInfo、NY Senate Open Legislation。仅供检索参考，不构成法律意见。

## 安装

```bash
npx skills add tsxcn/uslaw-legal-search --skill uslaw-legal-search
```

需要 Node.js 22.20+。支持 Claude Code、Codex、Cursor、OpenCode 等 75+ Agent。

## 前置条件（买家付款能力）

本服务返回标准 HTTP 402 账单。要自动付款，宿主 Agent 需先装支付宝 AI 付买家能力并绑定钱包一次：

```bash
npx -y @alipay/agent-payment@latest install
# 然后让 Agent 执行 apply-wallet / bind-wallet，用支付宝扫码授权一次
```

之后 0.08 元的小额调用可自动扣款，无需每次确认。**不要把 402 账单当用户报错**——那是正常的付款请求。

## 能查什么（7 类）

| type | 说明 |
|---|---|
| `case` | 美国判例（CourtListener，30,597 条） |
| `statute` | 联邦成文法 U.S. Code（GovInfo，52,232 条） |
| `state` | 纽约州法（4,038 条） |
| `section` | 单条法条（配 `id`） |
| `titles` | 法典目录 |
| `guides` | 实务指南 |
| `forms` | 常用法律表格 |

## 调用示例

先读免费能力清单（无需付款）：

```bash
curl -sS https://uslaw.zhichangguwen.org/api/service
```

发起检索（无凭证会返回 402 + `Payment-Needed` 账单头）：

```bash
curl -sS -D /tmp/uslaw_402_headers.txt \
  "https://uslaw.zhichangguwen.org/api/paid/caselaw?q=security%20deposit&type=case&limit=5"
```

Agent 用 `alipay-bot 402-buyer-pay` 付款后，带 `Payment-Proof` 头重放同一 URL 即可拿到 JSON 结果。同一凭证重放返回 `already_fulfilled`，不会重复扣费。完整流程见 [Skill 说明](skills/uslaw-legal-search/SKILL.md) 和 [付款参考](skills/uslaw-legal-search/references/payment.md)。

## 链接

- 接入手册：https://uslaw.zhichangguwen.org/docs
- Agent 发现入口：https://uslaw.zhichangguwen.org/llms.txt
- OpenAPI：https://uslaw.zhichangguwen.org/openapi.json
- 技术文章：https://uslaw.zhichangguwen.org/blog/http-402
- 免费能力清单：https://uslaw.zhichangguwen.org/api/service

## 免责

数据仅供检索参考，不构成法律意见；正式事项请咨询持牌律师。数据来自公开库，本 Skill 不代表这些机构，不得用于 FCRA（公平信用报告）用途。

## License

MIT
