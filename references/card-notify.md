# 用户卡变更事件通知（异步回调）

- **说明**：此接口是**盛付通向商户推送**的异步通知。用户在 H5 页面完成绑卡或解绑后，盛付通向商户指定的地址 POST 通知。
- **通知来源**：
  - 方式一：统一下单 `extra.bindCardEventNotifyUrl`（支付中绑卡）
  - 方式二：获取卡管理 URL 接口 `notifyUrl` 参数（纯绑卡）
- **重试机制**：**当前版本不重试**，商户需确保回调接口高可用
- **商户返回要求**：返回**纯文本字符串** `SUCCESS`（区分大小写），不要返回 JSON

## 通知参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `mchId` | String | 商户号 |
| `subMchId` | String | 子商户号（可空） |
| `appId` | String | 应用ID |
| `unionId` | String | 连尚数字 unionId（可空） |
| `eventType` | String | 事件类型：`BIND`（绑卡）/ `UNBIND`（解绑） |
| `bankCode` | String | 银行编码（如 `ICBC`） |
| `bankName` | String | 银行名称（如 "中国工商银行"） |
| `cardType` | String | 卡类型：`DR`=借记卡 / `CR`=贷记卡 |
| `cardNoBack4` | String | 银行卡号后 4 位 |
| `agreementNo` | String | 快捷支付协议号 |
| `resultCode` | String | 固定 `SUCCESS` |
| `nonceStr` | String | 随机字符串 |
| `signType` | String | 签名类型 |
| `sign` | String | 签名值（使用**盛付通公钥**验签） |

## 商户处理流程

```
1. 收到 POST 请求
2. 【验签】用盛付通公钥验证 sign → 不通过则丢弃
3. 【幂等】按 agreementNo + eventType 去重 → 已处理则直接返回 SUCCESS
4. 【业务处理】更新用户卡绑定状态、记录绑卡/解绑日志
5. 【返回】纯文本字符串 "SUCCESS"
```

## 通知示例

```json
{
  "mchId": "93762611",
  "appId": "312343123132131331232",
  "unionId": "9A2CE6A3FA1AFB3302F643D3B817AD5B",
  "eventType": "BIND",
  "bankCode": "ICBC",
  "bankName": "中国工商银行",
  "cardType": "DR",
  "cardNoBack4": "1234",
  "agreementNo": "AGR2026061600001",
  "resultCode": "SUCCESS",
  "nonceStr": "BindCardNotifyNonce",
  "signType": "RSA2",
  "sign": "通知签名值..."
}
```

## 注意事项

1. **当前版本无重试**：建议记录通知日志，必要时通过查询签约关系接口主动确认卡绑定状态做补偿
2. **双重来源**：卡变更通知可能来自两个地址——支付中绑卡和纯绑卡。两个地址可以指向同一回调接口统一处理
3. **幂等去重**：建议按 `agreementNo + eventType` 组合去重，防止重复处理
