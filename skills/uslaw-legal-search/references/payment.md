# 美国法律检索 · 402 付款参考

本页只处理 uslaw.zhichangguwen.org 的 402 付款。付款能力来自 `alipay-payment-skill`（`alipay-bot` CLI）。

## 发起付款（新 402）

1. 保存本轮 402 响应的 `Payment-Needed` 头原始值到安全文件（仅当前用户可读），不解码、不改写。
2. 保存本轮检索的 URL、method、POST body（如有）。
3. 按当前会话的 sessionId，执行**一条**命令：

```bash
# GET 检索
alipay-bot 402-buyer-pay \
  --session-id '<当前会话id>' \
  --file '<Payment-Needed安全文件>' \
  --resource-url 'https://uslaw.zhichangguwen.org/api/paid/caselaw?q=security%20deposit&type=case&limit=5' \
  --intent-summary '原始请求：查 security deposit 相关美国判例'

# POST 检索（追加 --method 和 --data）
alipay-bot 402-buyer-pay \
  --session-id '<当前会话id>' \
  --file '<Payment-Needed安全文件>' \
  --resource-url 'https://uslaw.zhichangguwen.org/api/paid/caselaw' \
  --intent-summary '原始请求：查 security deposit 相关美国判例' \
  --method POST \
  --data '{"q":"security deposit","type":"case","limit":5}'
```

- `--resource-url` 永远是检索地址，不是支付链接。
- `--intent-summary` 写用户原始需求，不要只写 URL 或方法名。

## 查询付款状态

用户表示"已支付/开通好了/继续/查一下"时：

```bash
# 有订单号（32位、第11-14位8282）
alipay-bot 402-query-payment-status --out-shake-no '<订单号>' \
  --resource-url '<本轮检索URL>' [--method POST --data '<body>']

# 否则用交易号
alipay-bot 402-query-payment-status --trade-no '<交易号>' \
  --resource-url '<本轮检索URL>' [--method POST --data '<body>']
```

## 履约回执（仅异常情况）

仅当 buyer-pay/query 已取得资源、且真实输出明确说"发送履约回执失败"并给出交易号，且用户在**后续消息**要求恢复时：

```bash
alipay-bot 402-fulfillment-ack --trade-no '<同次输出的交易号>'
```

## 安全边界

- 只执行上述 `alipay-bot` 子命令，不构造支付凭证、不绕过 402。
- 不展示 token、支付凭证、授权数据、本地凭据。
- 成功、资源为空、业务失败后 STOP；仅超时/网络失败可重试一次。
- 账户不匹配原样输出，不重试、不换号。
