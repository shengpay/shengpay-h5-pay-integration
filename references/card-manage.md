# 获取用户银行卡管理页 URL

- **接口路径**：`POST /api/v3/quick-sign/getUserCardManageUrl`
- **Content-Type**：`application/json; charset=UTF-8`
- **作用**：获取用户银行卡管理 H5 页面链接。用户打开后可查看已绑卡、绑定新卡、解绑卡（纯绑卡场景，不涉及支付）。
- **URL 有效期**：默认 **15 分钟**（可配置），过期需重新调用。用户进入页面后通过 ticket 维持状态，不受 URL 过期影响。

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `mchId` | String | **是** | 商户号 |
| `appId` | String | **是** | 应用ID |
| `unionId` | String | **是** | 连尚数字 unionId（标识用户） |
| `nonceStr` | String | **是** | 随机字符串 |
| `signType` | String | **是** | 签名类型：`RSA` / `RSA2` |
| `sign` | String | **是** | 签名值 |
| `subMchId` | String | 否 | 子商户号（服务商模式） |
| `openId` | String | 否 | 连尚数字 openId |
| `notifyUrl` | String | 否 | **绑卡/解绑事件通知地址**。若不传，则用户绑卡解绑后不会通知商户 |
| `returnUrl` | String | 否 | 同步回跳地址（用户在 H5 卡管理页面操作完成后回跳的地址） |

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
| `userCardManageUrl` | String | 用户银行卡管理页完整 URL，格式：`{cashierUrl}?cardManageToken={token}&pkg={pkg}` |

## 请求示例

```json
{
  "mchId": "93762611",
  "appId": "312343123132131331232",
  "unionId": "9A2CE6A3FA1AFB3302F643D3B817AD5B",
  "notifyUrl": "https://your-server.com/callback/bindcard",
  "returnUrl": "https://your-server.com/card/manage/return",
  "nonceStr": "cardManageNonce12345",
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
  "nonceStr": "ResponseNonce67890",
  "signType": "RSA2",
  "sign": "响应签名值...",
  "appId": "312343123132131331232",
  "mchId": "93762611",
  "userCardManageUrl": "https://h5-pay.shengpay.com/card-manage?cardManageToken=xxx&pkg=yyy"
}
```

## 注意事项

1. URL 中的 `cardManageToken` 和 `pkg` 有时效性（默认 15 分钟），过期后需重新调用本接口获取
2. 若传了 `notifyUrl`，用户在 H5 页面上完成绑卡或解绑后，盛付通会向该地址 POST 卡变更通知（详见 `references/card-notify.md`）
3. 此接口用于纯绑卡场景；支付中绑卡的通知地址通过统一下单 `extra.bindCardEventNotifyUrl` 指定
4. 若只需服务端查询已绑卡、不打开 H5，请用 `references/query-bind-card-list.md`
