# 订单查询

- **接口路径**：`POST /api/v3/payment/orderquery`
- **Content-Type**：`application/json; charset=UTF-8`
- **作用**：主动查询订单的支付状态。建议所有商户实现此接口作为回调通知的兜底手段。

## 请求参数

| 参数 | 类型 | 必填 | 说明                       |
|------|------|------|----------------------------|
| `appId` | String | **是** | 应用ID                     |
| `mchId` | String | **是** | 商户号                     |
| `transactionId` | String | 二选一 | 盛付通支付订单号，推荐使用 |
| `outTradeNo` | String | 二选一 | 商户订单号                 |
| `nonceStr` | String | **是** | 随机字符串                 |
| `signType` | String | **是** | 签名类型：`RSA` / `RSA2`   |
| `sign` | String | **是** | 签名值                     |

> `transactionId` 和 `outTradeNo` 至少填一个；推荐使用 `transactionId`（更精确）。

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
| `tradeType` | String | 交易类型（如 MWEB） |
| `transactionId` | String | 盛付通交易单号 |
| `outTradeNo` | String | 商户订单号 |
| `openId` | String | 用户 openId |
| `totalFee` | Integer | 订单金额（**分**） |
| `tradeState` | String | 交易状态，详见 `references/status-enums.md`：`WAIT_PAY`、`PAYING`、`SUCCESS`、`CLOSED`、`PAY_ERROR`、`REFUND`、`REVOKED` |
| `tradeStateDesc` | String | 交易状态描述 |
| `bankType` | String | 付款银行编码（如 ICBC） |
| `timeEnd` | String | 支付完成时间，格式 `yyyyMMddHHmmss` |
| `attach` | String | 附加数据（原样返回商户下单时传入的 attach） |
| `couponFee` | Integer | 优惠金额（**分**），有使用优惠券时返回 |
| `couponCount` | Integer | 优惠券数量，有使用优惠券时返回 |
| `couponId0` | String | 第 1 张优惠券 ID |
| `couponFee0` | Integer | 第 1 张优惠券金额（**分**） |
| `couponType0` | String | 第 1 张优惠券类型 |
| `cardNo` | String | 卡号后 4 位（快捷支付时返回） |
| `cardType` | String | 卡类型（快捷支付时返回）：`CC`=信用卡 / `DC`=借记卡 |

> 优惠券可能有多个，字段名以 `couponId{N}`、`couponFee{N}`、`couponType{N}` 编号递增。

## 请求示例

```json
{
  "appId": "312343123132131331232",
  "mchId": "93762611",
  "transactionId": "C1234123213124343123123",
  "nonceStr": "qwerty1234567890",
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
  "nonceStr": "ResponseNonce0987654321",
  "signType": "RSA2",
  "sign": "响应签名值...",
  "appId": "312343123132131331232",
  "mchId": "93762611",
  "tradeType": "MWEB",
  "transactionId": "C1234123213124343123123",
  "outTradeNo": "ORD202606160001",
  "openId": "15",
  "totalFee": 100,
  "tradeState": "SUCCESS",
  "tradeStateDesc": "支付成功",
  "bankType": "ICBC",
  "timeEnd": "20260616120000",
  "cardNo": "1234",
  "cardType": "DC"
}
```
