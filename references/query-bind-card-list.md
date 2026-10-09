# 快捷已绑卡查询

- **接口路径**：`POST /api/v3/quick-sign/queryBindCardList`
- **Content-Type**：`application/json; charset=UTF-8`
- **作用**：按 `appId` + `unionId` + 实际商户号定位会员，返回该用户已绑定的快捷银行卡列表。只查询，不打开 H5、不绑卡、不解绑。
- **列表字段形态**：`cardList` 是 **JSON 字符串**（不是 JSON 数组），验签时按字符串原值参与拼接，验签后再 `JSON.parse`。
- **对公开启**：`/api/v3/quick-sign` 路径需向运维申请白名单。

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `mchId` | String | **是** | 商户号，最长 20。服务商模式下为服务商商户号 |
| `appId` | String | **是** | 应用ID，最长 32 |
| `unionId` | String | **是** | 联尚数字 unionId（标识用户） |
| `nonceStr` | String | **是** | 随机字符串，最长 32 |
| `signType` | String | **是** | 签名类型：`RSA` / `RSA2` / `SM2` |
| `sign` | String | **是** | 签名值（详见 `references/signing.md`） |
| `subMchId` | String | 否 | 子商户号（服务商模式）。传入时 mchId 须在服务商白名单且通过从属关系校验；定位会员时用 subMchId。最长 20 |
| `openId` | String | 否 | 连尚数字 openId，最长 32。本接口定位会员不使用该字段 |

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
| `appId` | String | 回显 appId |
| `mchId` | String | 回显 mchId |
| `cardList` | String | 已绑卡列表 JSON 字符串。无卡时为 `"[]"`。失败时为空 |

### cardList 元素（解析 JSON 字符串后）

| 参数 | 类型 | 说明 |
|------|------|------|
| `payerName` | String | 付款人姓名掩码。2 字「张*」；3 字及以上「张*三」 |
| `cardType` | String | `DC`=借记（源值 `DR`/`DC`），其余（含 `CR`）=`CC` 贷记 |
| `bankCode` | String | 银行编码 |
| `bankName` | String | 银行名称 |
| `cardNoMask` | String | 卡号掩码，固定前 6 后 4，中间 10 个 `*`，如 `622202**********1234` |
| `sdpAgreementNo` | String | 盛付通快捷协议号 |

## 请求示例

```json
{
  "mchId": "93762611",
  "appId": "312343123132131331232",
  "unionId": "9A2CE6A3FA1AFB3302F643D3B817AD5B",
  "nonceStr": "queryBindCardNonce123",
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
  "nonceStr": "QSQa1b2c3d4e5",
  "signType": "RSA2",
  "sign": "响应签名值...",
  "appId": "312343123132131331232",
  "mchId": "93762611",
  "cardList": "[{\"payerName\":\"张*三\",\"cardType\":\"DC\",\"bankCode\":\"ICBC\",\"bankName\":\"工商银行\",\"cardNoMask\":\"622202**********1234\",\"sdpAgreementNo\":\"578174\"}]"
}
```

无卡时 `cardList` 为 `"[]"`，`resultCode` 仍为 `SUCCESS`。

## 注意事项

1. **先验签再 `JSON.parse(cardList)`**。把 `cardList` 当数组参与签名会导致验签失败。
2. **无卡 ≠ 会员不存在**。会员存在未绑卡 → `SUCCESS` + `"[]"`；会员中心查不到 → `MEMBER_NOT_EXIST`。
3. 定位会员只用 `appId` + `unionId` + 实际商户号（有 `subMchId` 用子商户号），`openId` 不参与。
4. 本接口不返回完整卡号、完整姓名，也不触发绑卡/解绑通知。用户侧管理银行卡请用 `references/card-manage.md`。
5. `cardType` 对外是 `DC`/`CC`，卡变更通知里是 `DR`/`CR`，接入时不要混用。
