# 退款查询

- **接口路径**：`POST /api/v3/payment/refundquery`
- **Content-Type**：`application/json; charset=UTF-8`
- **作用**：主动查询退款单的处理状态。

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `appId` | String | **是** | 应用ID |
| `mchId` | String | **是** | 商户号 |
| `refundId` | String | 四选一 | 盛付通退款单号（推荐使用，最精确） |
| `outRefundNo` | String | 四选一 | 商户退款单号 |
| `transactionId` | String | 否（已废弃） | 盛付通支付订单号 |
| `outTradeNo` | String | 否（已废弃） | 商户订单号 |
| `nonceStr` | String | **是** | 随机字符串 |
| `signType` | String | **是** | 签名类型：`RSA` / `RSA2` |
| `sign` | String | **是** | 签名值 |

> 建议优先使用 `refundId`（盛付通退款单号，精确度最高）或 `outRefundNo`（商户退款单号）。

## 响应参数

### 公共字段

| 参数 | 类型 | 说明 |
|------|------|------|
| `returnCode` | String | 通信标识 |
| `returnMsg` | String | 通信描述 |
| `resultCode` | String | 业务结果：`SUCCESS` / `FAIL` |
| `errorCode` | String | 错误代码 |
| `errorCodeDes` | String | 错误描述 |
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
| `refundId` | String | 盛付通退款单号 |
| `outRefundNo` | String | 商户退款单号 |
| `totalFee` | String | 原订单金额 |
| `refundFee` | String | 申请退款金额 |
| `settlementRefundFee` | String | 实际退款金额（申请退款金额 - 非充值代金券退款金额） |
| `refundStatus` | String | 退款状态：`SUCCESS` / `PROCESSING` / `REFUND_CLOSE` / `CHANGE`。详见 `references/status-enums.md` |
| `refundSuccessTime` | String | 退款成功时间，格式 `yyyyMMddHHmmss` |
| `refundRequestSource` | String | 退款请求来源，固定 `API` |

## 请求示例

```json
{
  "appId": "312343123132131331232",
  "mchId": "93762611",
  "refundId": "RFWP2026061600001",
  "nonceStr": "refundQueryNonce",
  "signType": "RSA2",
  "sign": "计算得到的签名值..."
}
```
