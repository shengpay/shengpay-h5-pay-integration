# 申请退款

- **接口路径**：`POST /api/v3/payment/refund`
- **Content-Type**：`application/json; charset=UTF-8`
- **作用**：对已支付的订单发起退款，支持部分退款（累计不超原订单金额）。
- **幂等键**：`outRefundNo`（相同 outRefundNo 重复调用不会产生多笔退款）
- **金额单位**：**分**（整数）

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `appId` | String | **是** | 应用ID |
| `mchId` | String | **是** | 商户号 |
| `nonceStr` | String | **是** | 随机字符串 |
| `sign` | String | **是** | 签名值 |
| `signType` | String | **是** | 签名类型：`RSA` / `RSA2` |
| `transactionId` | String | 二选一 | 盛付通支付订单号（推荐使用） |
| `outTradeNo` | String | 二选一 | 商户订单号 |
| `outRefundNo` | String | **是** | 商户退款单号，需保证唯一 |
| `totalFee` | Integer | 否（已废弃） | 原始订单金额（**分**） |
| `refundFee` | Integer | **是** | 退款金额（**分**），不能超过原订单金额 |
| `refundDesc` | String | 否 | 退款原因/描述 |
| `refundAccount` | String | 否 | 退款资金来源 |
| `notifyUrl` | String | **是** | 退款结果回调地址，最长 255 位 |

> `transactionId` 和 `outTradeNo` 至少填一个来指定原支付订单。

## 响应参数

### 公共字段

| 参数 | 类型 | 说明 |
|------|------|------|
| `returnCode` | String | 通信标识 |
| `returnMsg` | String | 通信描述 |
| `resultCode` | String | 业务结果：`SUCCESS` / `FAIL` |
| `errorCode` | String | 错误代码（失败时有值） |
| `errorCodeDes` | String | 错误描述（失败时有值） |
| `nonceStr` | String | 随机字符串 |
| `signType` | String | 签名类型 |
| `sign` | String | 签名值 |

### 业务字段

| 参数 | 类型 | 说明 |
|------|------|------|
| `appId` | String | 应用ID |
| `mchId` | String | 商户号 |
| `transactionId` | String | 盛付通支付订单号 |
| `outTradeNo` | String | 商户订单号 |
| `refundId` | String | 盛付通退款单号——**用此字段做后续查询和幂等确认** |
| `outRefundNo` | String | 商户退款单号 |
| `totalFee` | String | 订单金额 |
| `refundFee` | String | 退款金额 |

## 请求示例

```json
{
  "appId": "312343123132131331232",
  "mchId": "93762611",
  "transactionId": "WP20260616000001",
  "outRefundNo": "RF202606160001",
  "refundFee": 100,
  "refundDesc": "用户申请退款",
  "notifyUrl": "https://your-server.com/notify/refund",
  "nonceStr": "refundNonce0987654321",
  "signType": "RSA2",
  "sign": "计算得到的签名值..."
}
```

## 响应示例

```json
{
  "returnCode": "SUCCESS",
  "returnMsg": "SUCCESS",
  "resultCode": "SUCCESS",
  "nonceStr": "RefundResponseNonce",
  "signType": "RSA2",
  "sign": "响应签名值...",
  "appId": "312343123132131331232",
  "mchId": "93762611",
  "transactionId": "WP20260616000001",
  "outTradeNo": "ORD202606160001",
  "refundId": "RFWP2026061600001",
  "outRefundNo": "RF202606160001",
  "totalFee": "100",
  "refundFee": "100"
}
```
