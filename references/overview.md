# 接入总览

本文档帮助首次接入的开发者快速理解盛付通 H5 快捷支付的整体流程和关键概念。

## 什么是 H5 快捷支付

H5 快捷支付是盛付通提供的收银台支付产品。商户通过服务端 API 下单获取支付链接（mwebUrl），用户打开链接后在盛付通 H5 收银台页面完成绑卡和支付，盛付通异步通知商户支付结果。

## 核心流程（一句话版）

**商户下单 → 获取 mwebUrl → 用户打开 H5 收银台支付 → 盛付通异步通知商户 → 商户更新订单状态**

## 涉及的接口（9个）

### 商户主动调用（6个）

| 序号 | 接口 | 路径 | 作用 | 详细文档 |
|------|------|------|------|---------|
| 1 | 统一下单 | `POST /api/v3/payment/unifiedorder` | 提交支付订单，获取 H5 收银台链接（mwebUrl） | `references/unified-order.md` |
| 2 | 订单查询 | `POST /api/v3/payment/orderquery` | 主动查询订单支付状态 | `references/order-query.md` |
| 3 | 申请退款 | `POST /api/v3/payment/refund` | 对已支付订单发起退款 | `references/refund.md` |
| 4 | 退款查询 | `POST /api/v3/payment/refundquery` | 主动查询退款处理状态 | `references/refund-query.md` |
| 5 | 获取卡管理URL | `POST /api/v3/quick-sign/getUserCardManageUrl` | 获取卡管理 H5 页面链接 | `references/card-manage.md` |
| 6 | 快捷已绑卡查询 | `POST /api/v3/quick-sign/queryBindCardList` | 按用户身份查询已绑卡列表（不打开 H5） | `references/query-bind-card-list.md` |

### 盛付通异步回调（3个，商户需实现）

| 序号 | 通知 | 触发时机 | 详细文档 |
|------|------|---------|---------|
| 1 | 支付结果通知 | 用户支付完成 | `references/payment-notify.md` |
| 2 | 退款结果通知 | 退款处理完成 | `references/refund-notify.md` |
| 3 | 用户卡变更通知 | 用户绑卡/解绑 | `references/card-notify.md` |

## 关键概念

### 金额单位

**所有金额字段单位为"分"**（整数）。例如 ¥1.00 传入 `100`。

### 幂等性

- 统一下单：相同 `outTradeNo` 重复调用返回同一 `prepayId`
- 退款：相同 `outRefundNo` 重复调用不会产生多笔退款

### 签名机制

- 所有商户请求必须加签（使用**商户私钥**），详见 `references/signing.md`
- 所有盛付通响应和回调必须验签（使用**盛付通公钥**）

### 异步通知 ≠ 前端回跳

- `mwebUrl` 支付完成后的前端回跳 ≠ 支付成功
- **最终状态以异步通知验签 + 主动查单为准**
- 商户回调接口必须返回纯文本 `SUCCESS`（区分大小写）

## 接入步骤

```
第一步：准备 mchId / appId / 商户私钥 / 盛付通公钥 / notifyUrl
第二步：实现统一下单调用，获取 mwebUrl
第三步：引导用户打开 mwebUrl 完成支付
第四步：实现支付结果回调接口（验签 + 处理 + 返回 SUCCESS）
第五步：实现订单查询接口调用（作为回调兜底）
（可选）第六步：实现退款 + 退款回调
（可选）第七步：实现卡管理 + 卡变更回调
（可选）第八步：实现已绑卡查询（服务端拉列表，不打开 H5）
```

## 交互时序

### H5 快捷支付流程

```
商户服务                         盛付通                       盛付通H5收银台          用户
  │                               │                             │                   │
  │──① POST /api/v3/payment/unifiedorder──────────────────────>│                   │
  │   传入订单信息+签名              │                             │                   │
  │                               │──生成prepayId+mwebUrl       │                   │
  │<──返回 prepayId + mwebUrl ────│                             │                   │
  │                               │                             │                   │
  │──② 唤起 mwebUrl ────────────────────────────────────────────────────────────>│
  │                               │                             │                   │
  │                               │<──③ preparePayment ────────│                   │
  │                               │   查单+构建支付上下文         │                   │
  │                               │──返回订单信息───────────────>│                   │
  │                               │                             │──展示收银台页面     │
  │                               │                             │<──用户绑卡/支付     │
  │                               │                             │──支付处理          │
  │                               │<──支付渠道回调──────────────│                   │
  │                               │                             │                   │
  │<──④ POST {notifyUrl} ─────────│  支付结果异步通知             │                   │
  │   验签+返回 SUCCESS            │                             │                   │
  │                               │                             │                   │
  │──⑤ POST /api/v3/payment/orderquery────────────────────────>│（可选，主动查单）   │
  │<──返回订单状态─────────────────│                             │                   │
```

### 退款流程

```
商户服务                         盛付通
  │                               │
  │──① POST /api/v3/payment/refund───────────────────────────>│
  │<──返回退款申请结果──────────────│
  │                               │──异步处理退款
  │<──② POST {notifyUrl} ─────────│  退款结果异步通知
  │   验签+返回 SUCCESS            │
  │──③ POST /api/v3/payment/refundquery─────────────────────>│（可选）
  │<──返回退款状态─────────────────│
```

### 纯绑卡流程

```
商户服务                         盛付通                       盛付通H5收银台          用户
  │                               │                             │                   │
  │──① POST /api/v3/quick-sign/getUserCardManageUrl──────────>│                   │
  │<──返回 userCardManageUrl ──────│                             │                   │
  │──② 唤起 userCardManageUrl ─────────────────────────────────────────────────>│
  │                               │<──③ prepare ───────────────│                   │
  │                               │                             │──展示卡管理页面     │
  │                               │                             │──用户绑卡/解绑     │
  │<──④ POST {notifyUrl} ─────────│  绑卡/解绑事件异步通知        │                   │
  │   验签+返回 SUCCESS            │                             │                   │
```

### 已绑卡查询（服务端拉列表）

```
商户服务                         盛付通
  │                               │
  │──① POST /api/v3/quick-sign/queryBindCardList────────────>│
  │   传入 mchId/appId/unionId + 签名                         │
  │<──返回 cardList（JSON 字符串）────────────────────────────│
  │   验签后 JSON.parse(cardList)                             │
```

## 官方文档索引

| 文档 | 链接 |
|------|------|
| H5 快捷支付 API 列表 | https://docs.shengpay.com/盛付通支付/H5快捷支付/API列表/ |
| 订单状态说明 | https://docs.shengpay.com/盛付通支付/H5快捷支付/订单状态说明/ |
| 快捷支付收单（飞书） | https://qcnr5itfne0r.feishu.cn/wiki/BcxewUZNoiNCJhkwO7lcBBFkndb |
| H5 用户卡管理（飞书） | https://qcnr5itfne0r.feishu.cn/wiki/AUG3wYaj7i7cBykkPR3cSzFfnyc |
| 用户卡变更事件通知（飞书） | https://qcnr5itfne0r.feishu.cn/wiki/SOBrwgzdZiYN4okH5M3cKBUJnuh |
