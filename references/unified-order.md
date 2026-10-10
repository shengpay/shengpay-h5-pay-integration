# 统一下单（快捷支付收单）

- **接口路径**：`POST /api/v3/payment/unifiedorder`
- **Content-Type**：`application/json; charset=UTF-8`
- **作用**：提交 H5 快捷支付订单，获取 H5 收银台链接（mwebUrl），用户在收银台完成支付。
- **幂等键**：`outTradeNo`（相同 outTradeNo 重复调用返回同一 prepayId）
- **金额单位**：**分**（整数）

## 请求参数

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `appId` | String | **是** | 应用ID，由盛付通分配 |
| `appName` | String | **是** | 应用名称 |
| `mchId` | String | **是** | 商户号 |
| `outTradeNo` | String | **是** | 商户订单号，支持字母和数字，最长 64 位，需保证在商户侧唯一 |
| `totalFee` | Integer | **是** | 订单总金额，**单位：分**（例如 100 表示 ¥1.00） |
| `notifyUrl` | String | **是** | 支付结果回调地址，最长 255 位，支付完成后盛付通向此地址 POST 通知 |
| `nonceStr` | String | **是** | 随机字符串，最长 32 位（建议使用 32 位字母+数字） |
| `body` | String | **是** | 商品描述/订单标题 |
| `detail` | String | **是** | 交易详情，JSON 字符串。可为空字符串 `""`；不为空时必须为以下格式（详见下方 [detail 字段说明](#detail-字段说明)） |
| `tradeType` | String | **是** | 交易类型。H5 快捷支付填 `MWEB` |
| `signType` | String | **是** | 签名算法类型：`RSA` / `RSA2` |
| `sign` | String | **是** | 签名值（按签名规则计算，详见 `references/signing.md`） |
| `unionId` | String | **是** | 数字 unionId，MWEB 场景下必填 |
| `openId` | String | 否 | 数字 openId（连尚会员 + 接入连尚应用 = openId） |
| `timeStart` | String | 否 | 交易起始时间，格式 `yyyyMMddHHmmss`，如 `20260616120000` |
| `timeExpire` | String | 否 | 交易失效时间，格式 `yyyyMMddHHmmss` |
| `goodsTag` | String | 否 | 订单优惠标记 |
| `attach` | String | 否 | 附加数据，JSON 字符串，原样返回。支持 `cardType`（`CR`=贷记卡 / `DR`=借记卡） |
| `limitPay` | String | 否 | 指定支付方式（如 `no_credit` 限制使用信用卡） |
| `clientIp` | String | 否 | 客户端 IP 地址 |
| `subMchId` | String | 否 | 子商户号（服务商模式下使用） |
| `productId` | String | 否 | 商品 ID |
| `authCode` | String | 否 | 授权码（刷卡支付场景使用） |
| `sceneInfo` | String | 否 | 场景信息，JSON 格式 |
| `profitSharing` | String | 否 | 是否分账：`Y` 需要分账 / `N` 不需分账 |
| `extra` | String | 否 | **V3 扩展字段**（JSON 字符串），仅在 tradeType=MWEB 下生效。详细说明见下方 [extra 扩展字段详解](#extra-扩展字段详解v3) |
| `returnUrl` | String | 否 | H5 支付完成后的同步回跳地址（仅在 tradeType=MWEB 下生效） |

### detail 字段说明

`detail` 为 JSON 格式字符串，描述订单中的商品明细。

- **可以为空**：传空字符串 `""` 即可
- **不为空时**必须为以下格式：

```json
{
  "goodsDetails": [
    {
      "goodsId": "001",
      "goodsName": "矿泉水",
      "quantity": 1,
      "price": 100
    }
  ]
}
```

| 子字段 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `goodsDetails` | Array | 是 | 商品明细数组 |
| `goodsDetails[].goodsId` | String | 是 | 商品ID |
| `goodsDetails[].goodsName` | String | 是 | 商品名称 |
| `goodsDetails[].quantity` | Integer | 是 | 数量，必须大于 0 |
| `goodsDetails[].price` | Integer | 是 | 单价，**单位：分**，必须大于 0 |

### extra 扩展字段详解（V3）

`extra` 为 JSON 格式字符串，仅在 `tradeType=MWEB` 场景下生效，**该字段参与签名计算**（直接使用其 JSON 字符串原值拼接）。

#### 什么时候使用

- 需要向盛付通 H5 收银台传递付款人身份信息（姓名、身份证号）以进行身份一致性校验
- 需要限制用户只能使用特定类型的银行卡支付
- 需要接收用户在支付流程中绑卡/解绑的事件通知

#### JSON 结构

```json
{
  "payer": {
    "name": "<盛付通公钥加密的密文>",
    "idNo": "<盛付通公钥加密的密文>",
    "idType": "ID_CARD",
    "needLiveness": false
  },
  "payRestriction": {
    "cardType": "DR",
    "bankCodes": ["ICBC", "CMB"],
    "cardNoSuffixes": ["1234"]
  },
  "bindCardEventNotifyUrl": "https://your-server.com/callback/bindcard"
}
```

#### 字段说明

**payer（付款人身份信息）**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `payer.name` | String | 否 | 付款人姓名密文（使用**盛付通公钥**加密，算法与 signType 对应） |
| `payer.idNo` | String | 否 | 付款人身份证号密文（同上加密方式） |
| `payer.idType` | String | 否 | 证件类型，如 `ID_CARD`（身份证） |
| `payer.needLiveness` | Boolean | 否 | 是否需要活体认证，默认 false |

**payRestriction（支付卡限制）**

| 字段 | 类型 | 说明 |
|------|------|------|
| `payRestriction.cardType` | String | 允许的卡类型：`DR`=借记卡 / `CR`=贷记卡，不传不限制 |
| `payRestriction.bankCodes` | String[] | 允许的银行编码列表，如 `["ICBC","CMB"]`，不传不限制 |
| `payRestriction.cardNoSuffixes` | String[] | 仅展示指定尾号的已绑卡（不限制新绑卡），不传不限制 |

**bindCardEventNotifyUrl（绑卡通知地址）**

| 字段 | 类型 | 说明 |
|------|------|------|
| `bindCardEventNotifyUrl` | String | 用户绑卡/解绑后盛付通向此地址 POST 通知（详见 `references/card-notify.md`）。不传则不通知 |

#### extra 加密链路

```
商户侧                                             盛付通侧

1. 使用「盛付通公钥」对 payer 中的
   name/idNo 分别进行非对称加密
   （RSA/RSA2，与 signType 一致）

2. 组装 extra JSON

3. 整体请求加签后发送 ── POST ──>
                                    4. 验签（使用商户公钥）
                                    5. 使用「盛付通私钥」解密 name/idNo
                                       得到明文
                                    6. 加密后存入数据库
```

#### 安全要点

- 商户用盛付通公钥加密 → 只有盛付通能解密（私钥不对外）
- 盛付通解密后以国密 SM4 加密存储 → 数据库即使泄露也无法还原明文
- `extra` 参与签名 → 防止传输过程中被篡改
- 若 `extra` 为空或解析失败 → 跳过身份校验，不影响正常下单（降级策略）

#### 注意事项

1. `extra` 字段直接使用其 JSON 字符串原值参与签名，**不要**对 extra 内部做二次排序
2. 非对称加密后建议 Base64 编码放入 JSON
3. 加密使用盛付通公钥，不是商户自己的公钥
4. 加密算法与 signType 保持一致（RSA → RSA 加密，RSA2 → RSA2 加密）

## 响应参数

### 公共字段

| 参数 | 类型 | 说明 |
|------|------|------|
| `returnCode` | String | 通信标识，`SUCCESS` 表示通信成功 |
| `returnMsg` | String | 通信描述，`SUCCESS` / 具体错误信息 |
| `resultCode` | String | 业务结果，`SUCCESS` / `FAIL` |
| `errorCode` | String | 错误代码（resultCode=FAIL 时有值） |
| `errorCodeDes` | String | 错误描述（resultCode=FAIL 时有值） |
| `nonceStr` | String | 随机字符串 |
| `signType` | String | 签名类型（RSA / RSA2） |
| `sign` | String | 签名值（使用盛付通公钥验签） |

### 业务字段

| 参数 | 类型 | 说明 |
|------|------|------|
| `appId` | String | 回显 appId |
| `mchId` | String | 回显 mchId |
| `tradeType` | String | 交易类型 |
| `prepayId` | String | 预支付 ID（盛付通侧的交易唯一标识，即 transactionId） |
| `mwebUrl` | String | H5 收银台地址，将此链接提供给用户打开即可进入支付页面 |

## 请求示例

```json
{
  "appId": "312343123132131331232",
  "appName": "示例应用",
  "mchId": "93762611",
  "outTradeNo": "ORD202606160001",
  "totalFee": 100,
  "notifyUrl": "https://your-server.com/notify/payment",
  "nonceStr": "abc123def456ghi789jkl012mno345pq",
  "body": "测试商品",
  "detail": "{\"goodsDetail\":\"xxx\"}",
  "tradeType": "MWEB",
  "signType": "RSA2",
  "sign": "计算得到的签名值...",
  "timeExpire": "20260617000000",
  "extra": "{\"payRestriction\":{\"cardType\":\"DR\"},\"bindCardEventNotifyUrl\":\"https://your-server.com/callback/bindcard\"}"
}
```

## 响应示例

```json
{
  "returnCode": "SUCCESS",
  "returnMsg": "SUCCESS",
  "resultCode": "SUCCESS",
  "nonceStr": "XyzAbCdEfGhIjKlMnOpQrStUvWxYz123",
  "signType": "RSA2",
  "sign": "响应签名值...",
  "appId": "312343123132131331232",
  "mchId": "93762611",
  "tradeType": "MWEB",
  "prepayId": "WP20260616000001",
  "mwebUrl": "https://h5-pay.shengpay.com/cashier?prepayId=WP20260616000001"
}
```
