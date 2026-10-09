# 退款结果通知（异步回调）

- **说明**：此接口是**盛付通向商户推送**的异步通知。退款处理完成后，盛付通向退款申请时传入的 `notifyUrl` POST 通知。
- **重试机制**：支持，指数退避重试
- **商户返回要求**：返回**纯文本字符串** `SUCCESS`（区分大小写），不要返回 JSON

## 通知参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `returnCode` | String | 固定 `SUCCESS` |
| `returnMsg` | String | 固定 `SUCCESS` |
| `resultCode` | String | 业务结果：`SUCCESS`（退款成功）/ `FAIL`（退款失败） |
| `appId` | String | 应用ID |
| `mchId` | String | 商户号 |
| `nonceStr` | String | 随机字符串 |
| `signType` | String | 签名类型 |
| `sign` | String | 签名值（使用**盛付通公钥**验签） |
| `transactionId` | String | 盛付通支付订单号 |
| `outTradeNo` | String | 商户订单号 |
| `refundId` | String | 盛付通退款单号——**用此字段做幂等去重** |
| `outRefundNo` | String | 商户退款单号 |
| `totalFee` | String | 原订单金额 |
| `settlementTotalFee` | String | 应结订单金额（订单金额 - 非充值代金券金额） |
| `refundFee` | String | 申请退款金额 |
| `settlementRefundFee` | String | 实际退款金额（申请退款金额 - 非充值代金券退款金额） |
| `refundStatus` | String | 退款状态：`SUCCESS` / `PROCESSING` / `REFUND_CLOSE` / `CHANGE`。详见 `references/status-enums.md` |
| `refundSuccessTime` | String | 退款成功时间，格式 `yyyyMMddHHmmss` |
| `refundRequestSource` | String | 退款请求来源，固定 `API` |

## 商户处理流程

```
1. 收到 POST 请求
2. 【验签】用盛付通公钥验证 sign → 不通过则丢弃
3. 【检查】returnCode=SUCCESS 且 resultCode=SUCCESS
4. 【幂等】按 refundId 去重 → 已处理则直接返回 SUCCESS
5. 【查单确认】可选：调用退款查询接口二次确认
6. 【业务处理】更新退款状态、恢复库存/权益等
7. 【返回】纯文本字符串 "SUCCESS"
```

## 通知示例

```json
{
  "returnCode": "SUCCESS",
  "returnMsg": "SUCCESS",
  "resultCode": "SUCCESS",
  "appId": "312343123132131331232",
  "mchId": "93762611",
  "nonceStr": "RefundNotifyNonce",
  "signType": "RSA2",
  "sign": "通知签名值...",
  "transactionId": "WP20260616000001",
  "outTradeNo": "ORD202606160001",
  "refundId": "RFWP2026061600001",
  "outRefundNo": "RF202606160001",
  "totalFee": "100",
  "settlementTotalFee": "100",
  "refundFee": "100",
  "settlementRefundFee": "100",
  "refundStatus": "SUCCESS",
  "refundSuccessTime": "20260616130000",
  "refundRequestSource": "API"
}
```
