# 常见错误码

本文档列出盛付通 H5 快捷支付接口常见的错误码及处理建议。

## 公共错误码

| 错误码 | 说明 | 处理建议 |
|--------|------|---------|
| `LACK_PARAMS` | 缺少必要参数 | 检查请求中必填参数是否完整，对照官方文档逐一核对 |
| `PARAMS_ERROR` | 参数格式错误 | 检查参数值是否符合要求（格式、长度、类型） |
| `SIGN_ERROR` | 签名错误 | 检查签名算法实现和密钥是否正确，详见签名机制文档 |
| `ORDER_NOT_EXIST` | 订单不存在 | 检查 `transactionId` 或 `outTradeNo` 是否正确 |
| `MCH_NOT_FOUND` | 商户不存在 | 检查 `mchId` / `appId` 是否正确，是否已开通 API 白名单 |
| `MEMBER_NOT_EXIST` | 会员不存在 | 已绑卡查询按 appId + unionId + 实际商户号未查到会员。核对身份字段是否与下单/绑卡一致 |
| `SERVICE_PROVIDER_NO_AUTH` | 服务商模式权限未开通 | 传入 `subMchId` 时确认 mchId 已在服务商白名单 |
| `BIZERR_NEED_RETRY` | 业务繁忙，请重试 | 使用退避策略（如 1s/3s/5s）稍后重试 |

## 退款相关错误码

| 错误码 | 说明 | 处理建议 |
|--------|------|---------|
| `REFUND_AMOUNT_ERROR` | 退款金额错误 | 检查退款金额是否超过原订单金额；确认金额单位为**分** |
| `ORDER_NOT_REFUNDABLE` | 订单状态不可退款 | 检查订单是否已支付成功（只有 SUCCESS 状态可退款） |

## 排查建议

1. **先看 errorCode + errorCodeDes**：大多数错误信息已包含具体原因
2. **SIGN_ERROR 最常见**：优先排查签名，用 `references/signing.md` 的排查步骤逐项检查
3. **BIZERR_NEED_RETRY**：不是代码问题，稍等后重试即可
4. **LACK_PARAMS**：对比官方文档逐字段确认

如果以上无法解决，联系盛付通技术支持并提供：
- 请求的完整参数（脱敏后）
- 返回的 errorCode 和 errorCodeDes
- 请求时间
- mchId 和 appId
