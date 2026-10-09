# 支付结果通知（异步回调）

- **说明**：此接口是**盛付通向商户推送**的异步通知，不是商户主动调用的接口。支付完成后盛付通向统一下单时传入的 `notifyUrl` POST 通知。
- **重试机制**：支持，指数退避重试（商户未返回 `SUCCESS`、返回非 200、超时 10s 时触发重试）
- **商户返回要求**：返回**纯文本字符串** `SUCCESS`（区分大小写），不要返回 JSON

## 通知参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `returnCode` | String | 固定 `SUCCESS` |
| `returnMsg` | String | 固定 `SUCCESS` |
| `resultCode` | String | 业务结果：`SUCCESS`（支付成功）/ `FAIL`（支付失败） |
| `appId` | String | 应用ID |
| `mchId` | String | 商户号 |
| `nonceStr` | String | 随机字符串 |
| `signType` | String | 签名类型（RSA / RSA2） |
| `sign` | String | 签名值（使用**盛付通公钥**验签） |
| `openId` | String | 用户 openId |
| `tradeType` | String | 交易类型 |
| `transactionId` | String | 盛付通交易单号——**用此字段做幂等去重** |
| `outTradeNo` | String | 商户订单号 |
| `totalFee` | Integer | 订单金额（**分**） |
| `bankType` | String | 付款银行编码 |
| `timeEnd` | String | 支付完成时间，格式 `yyyyMMddHHmmss` |
| `attach` | String | 附加数据（原样返回） |
| `couponFee` | Integer | 优惠金额（**分**），有优惠券时返回 |
| `couponCount` | Integer | 优惠券数量，有优惠券时返回 |
| `couponId0`...`N` | String | 第 N 张优惠券 ID |
| `couponFee0`...`N` | Integer | 第 N 张优惠券金额（**分**） |
| `couponType0`...`N` | String | 第 N 张优惠券类型 |
| `cardNo` | String | 卡号后 4 位（快捷支付场景） |
| `cardType` | String | 卡类型：`CC`=信用卡 / `DC`=借记卡（快捷支付场景） |

## 商户处理流程

```
1. 收到 POST 请求
2. 【验签】用盛付通公钥验证 sign → 不通过则丢弃
3. 【检查】returnCode=SUCCESS 且 resultCode=SUCCESS
4. 【幂等】按 transactionId 去重 → 已处理则直接返回 SUCCESS
5. 【查单确认】可选：调用订单查询接口二次确认
6. 【业务处理】更新订单状态、发货/开通权益
7. 【返回】纯文本字符串 "SUCCESS"
```

## 验签步骤

1. 取出通知中的 `sign` 字段
2. 将**除 sign 外的所有字段**按 key 字典序排序，拼接为 `key1=value1&key2=value2&...`
3. 使用**盛付通公钥**验签（算法与 `signType` 对应）
4. 验签不通过 → 丢弃该通知，记录告警日志

## 通知示例

```json
{
  "returnCode": "SUCCESS",
  "returnMsg": "SUCCESS",
  "resultCode": "SUCCESS",
  "appId": "312343123132131331232",
  "mchId": "93762611",
  "nonceStr": "NotificationNonce1234567890",
  "signType": "RSA2",
  "sign": "通知签名值...",
  "openId": "15",
  "tradeType": "MWEB",
  "transactionId": "WP20260616000001",
  "outTradeNo": "ORD202606160001",
  "totalFee": 100,
  "bankType": "ICBC",
  "timeEnd": "20260616120000",
  "attach": "{\"cardType\":\"DR\"}",
  "cardNo": "1234",
  "cardType": "DC"
}
```
