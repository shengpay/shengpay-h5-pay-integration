# 状态枚举

本文档列出盛付通 H5 快捷支付涉及的所有状态枚举值及其含义。

> 官方文档：[订单状态说明](https://docs.shengpay.com/%E7%9B%9B%E4%BB%98%E9%80%9A%E6%94%AF%E4%BB%98/H5%E5%BF%AB%E6%8D%B7%E6%94%AF%E4%BB%98/%E8%AE%A2%E5%8D%95%E7%8A%B6%E6%80%81%E8%AF%B4%E6%98%8E/)

## 订单状态（tradeState）

| 状态值 | 说明 |
|--------|------|
| `WAIT_PAY` | 未支付——订单已创建，等待用户支付 |
| `PAYING` | 用户支付中——用户已在收银台进行支付操作 |
| `SUCCESS` | 支付成功——终态 |
| `CLOSED` | 已关闭——订单超时或主动关闭 |
| `PAY_ERROR` | 支付失败——银行返回失败或其他原因 |
| `REFUND` | 转入退款——支付成功后发生退款 |
| `REVOKED` | 已撤销——刷卡支付场景下的撤销 |

### 流转关系

```
WAIT_PAY ──→ PAYING ──→ SUCCESS ──→ REFUND
    │           │
    │           ├──→ PAY_ERROR
    │
    └──→ CLOSED

REVOKED（刷卡支付撤销，独立分支）
```

## 退款状态（refundStatus）

| 状态值 | 说明 |
|--------|------|
| `PROCESSING` | 退款处理中 |
| `SUCCESS` | 退款成功——终态 |
| `REFUND_CLOSE` | 退款关闭 |
| `CHANGE` | 退款异常——原路退款失败（如银行卡作废或冻结），需人工处理 |

### 流转关系

```
PROCESSING ──→ SUCCESS
     │
     ├──→ REFUND_CLOSE
     │
     └──→ CHANGE（异常终态）
```

## 卡变更事件类型（eventType）

| 值 | 说明 |
|----|------|
| `BIND` | 绑卡 |
| `UNBIND` | 解绑 |

## 卡类型

| 值 | 说明 | 使用场景 |
|----|------|---------|
| `DR` | 借记卡（储蓄卡） | 请求传入时使用（如 `extra.payRestriction.cardType`） |
| `CR` | 贷记卡（信用卡） | 请求传入时使用 |

## 响应公共字段

| 字段 | 值 | 说明 |
|------|-----|------|
| `returnCode` | `SUCCESS` | 通信成功 |
| `resultCode` | `SUCCESS` / `FAIL` | 业务成功 / 业务失败 |

## 签名类型（signType）

| 值 | 说明 |
|----|------|
| `RSA` | SHA1withRSA |
| `RSA2` | SHA256withRSA |
