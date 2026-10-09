---
name: shengpay-h5-pay-integration
description: "盛付通H5快捷支付接入AI AGENT：适用于首次接入、H5支付、快捷收单、卡管理、已绑卡查询、订单查询、退款、异步通知、RSA/RSA2签名验签、V3 extra加密扩展、状态枚举、错误码排查、上线检查。"
---

# 盛付通 H5 快捷支付集成

## 什么时候使用

- 第一次接入盛付通 H5 快捷支付，需要了解接入流程和准备要素。
- 已有订单系统，需要增量接入盛付通 H5 收银台、订单查询、退款能力。
- 需要了解商户密钥体系（商户私钥加签、盛付通公钥验签）。
- 需要了解具体接口的请求参数、响应参数，或需要生成代码。
- 需要查询用户已绑卡列表（不打开 H5）。
- 需要了解异步通知（支付通知、退款通知、卡变更通知）的处理规范和验签方式。
- 用户贴出请求参数、返回参数、错误信息或代码片段，需要定位问题。
- 用户询问状态枚举含义（WAIT_PAY、SUCCESS、REFUND 等）。
- 准备上线前需要做检查确认。

## 什么时候不要使用

- 非盛付通支付、非 H5 快捷支付相关问题，不使用本 Skill。
- 涉及盛付通聚合支付、余额支付、网银支付等其他产品线（不在本 Skill 覆盖范围）。

## 快速路由表

路由优先级：用户显式接口问题 > 签名/通知阻塞项 > 快速路由表精确命中 > 首次接入默认流程。

| 用户场景 | 最小 reference 集 |
| --- | --- |
| 首次接入、不了解接入流程 | `references/preparation.md`、`references/overview.md` |
| 需要了解所有接口有哪些 | `references/overview.md` |
| 统一下单——请求参数/响应参数（含 extra 加密） | `references/unified-order.md`、`references/signing.md` |
| 订单查询——请求参数/响应参数 | `references/order-query.md`、`references/signing.md` |
| 支付结果通知——字段含义/处理方式 | `references/payment-notify.md`、`references/async-notify.md` |
| 申请退款——请求参数/响应参数 | `references/refund.md`、`references/signing.md` |
| 退款查询——请求参数/响应参数 | `references/refund-query.md`、`references/signing.md` |
| 退款结果通知——字段含义/处理方式 | `references/refund-notify.md`、`references/async-notify.md` |
| 获取卡管理URL——请求参数/响应参数 | `references/card-manage.md`、`references/signing.md` |
| 查询已绑卡列表——请求参数/响应参数 | `references/query-bind-card-list.md`、`references/signing.md` |
| 卡变更事件通知——字段含义/处理方式 | `references/card-notify.md`、`references/async-notify.md` |
| 签名/验签有问题 | `references/signing.md`、`references/preparation.md` |
| 异步通知收不到或不知道如何处理 | `references/async-notify.md`、`references/signing.md` |
| 状态枚举含义不清楚 | `references/status-enums.md` |
| 收到错误码不知道怎么处理 | `references/error-codes.md`、`references/faq.md` |
| 接入联调中遇到具体问题 | `references/faq.md`、`references/error-codes.md` |
| 存量系统新增盛付通支付 | `references/overview.md`、`references/unified-order.md`、`references/async-notify.md` |
| 上线前检查 | `references/faq.md`、`references/async-notify.md` |

## 产品线裁决

盛付通 H5 快捷支付只有一条主链路：

| 用户目标 | 默认方案 | 主文档 |
| --- | --- | --- |
| H5 页面收银台支付 | H5 快捷支付（tradeType=MWEB） | `references/unified-order.md`、`references/payment-notify.md` |
| 标准服务端下单 → H5 收银台 → 异步通知 | 统一下单 + 支付/退款通知闭环 | `references/unified-order.md`、`references/payment-notify.md`、`references/refund.md` |
| 纯绑卡（不支付） | getUserCardManageUrl + 卡变更通知 | `references/card-manage.md`、`references/card-notify.md` |
| 查询已绑卡（不打开 H5） | queryBindCardList | `references/query-bind-card-list.md`、`references/signing.md` |

## 阶段主路由

| 阶段 | 推荐阅读 |
| --- | --- |
| 接入准备 | `references/preparation.md`、`references/overview.md` |
| 接入判断 | `references/overview.md` |
| 接口联调——统一下单 | `references/unified-order.md`、`references/signing.md` |
| 接口联调——订单查询 | `references/order-query.md`、`references/status-enums.md` |
| 接口联调——申请退款 | `references/refund.md` |
| 接口联调——退款查询 | `references/refund-query.md`、`references/status-enums.md` |
| 异步通知 | `references/payment-notify.md`、`references/refund-notify.md`、`references/async-notify.md` |
| 绑定卡管理 | `references/card-manage.md`、`references/query-bind-card-list.md`、`references/card-notify.md` |
| 问题排查 | `references/error-codes.md`、`references/faq.md` |
| 上线检查 | `references/faq.md`、`references/async-notify.md` |

## 决策流程

1. 从用户表达中提取 5 个标签：用户类型（首次接入/存量接入）、阶段（准备/联调/上线）、技术栈、当前目标、是否已有订单系统。
2. 首次接入优先输出接入判断卡，帮助用户理清准备要素和接入步骤。
3. 存量接入（已有订单系统、支付入口、回调等）优先给增量接入建议，不重写原系统。
4. 签名问题是最高优先级阻塞项，优先排查。
5. 异步通知问题优先确认验签和返回格式。
6. 状态枚举问题直接查 `references/status-enums.md` 官方口径。

## 检查点机制

### 🔴 CHECKPOINT · HARD STOP 硬检查点

以下情况必须等待用户确认后再继续：

1. 用户要求可直接联调或生产可用代码，但缺少：
   - 商户号（mchId）
   - 应用ID（appId）
   - 商户 RSA/RSA2 私钥（安全来源）
   - 盛付通公钥
   - 测试/生产环境地址
   - 回调 notifyUrl
2. 用户要求跳过验签或收到通知即改成功——必须拒绝，给出安全替代方案。

硬检查点输出首行固定为 `🔴 CHECKPOINT · HARD STOP：硬检查点。`，包含当前判断、为什么不能直接继续、唯一确认问题。

### 失败模式表

| 触发条件 | 一线处理 |
| --- | --- |
| 签名/验签错误 | 输出签名排查步骤，逐项检查拼接规则、密钥对、signType |
| 生产/联调配置缺失却要求可上线代码 | 输出硬检查点，只问最高优先级缺口 |
| 用户要求跳过验签、伪造成功或收到通知即改成功 | 拒绝不安全代码，给验签、幂等、查单/异步通知最终确认替代方案 |
| 回调收不到 | 输出通知排查卡（URL可达性、返回格式、WAF拦截、主动查询兜底） |

## 输出卡片

| 用户意图 | 输出卡片 |
| --- | --- |
| 不知道怎么接 | 接入判断卡（当前判断、推荐方案、还缺配置、下一步） |
| 已有系统要增量接 | 存量改造建议卡（建议新增、建议保留、需要人工确认、风险点） |
| 签名/验签报错 | 签名排查卡（签名字符串构造、密钥匹配、算法选择） |
| 回调收不到 | 通知排查卡（URL可达性、返回格式、验签、幂等） |
| 状态/错误码不明 | 枚举/错误码速查 |
| 上线前确认 | 上线检查卡（必测项、回调确认、日志脱敏） |

每次输出必须显式列出 `本轮实际使用的 references`，数量控制在 2-5 份。

## 全局边界

- 不猜测 `mchId`、`appId`、`notifyUrl`、`outTradeNo` 等运行时值。
- 不把商户私钥、盛付通私钥写入前端、仓库或示例常量。
- 前端 `mwebUrl` 回跳不等于支付成功；最终状态必须经异步通知验签 + 查单确认。
- 不回答费率、合规、通道准入结论。
- 不提供绕过验签、伪造支付成功的代码。
- 生产问题不定责，只整理升级人工材料。
- 本 Skill 不能主动联网检查官方文档更新。
- 接口字段以本地 references 为准，URL 链接只用于来源追溯和人工刷新。

## 当前版本事实

| 项目 | 当前口径 |
| --- | --- |
| Skill 包版本 | `1.0.0` |
| 接口基准 | 盛付通 H5 快捷支付 V3（`/api/v3/payment/*`、`/api/v3/quick-sign/*`） |
| 签名算法 | RSA（SHA1withRSA）、RSA2（SHA256withRSA），推荐 RSA2 |
| 金额单位 | **分**（整数） |
| 幂等键 | 统一下单 = `outTradeNo`；退款 = `outRefundNo` |
| 官方文档入口 | https://docs.shengpay.com/盛付通支付/H5快捷支付/API列表/ |
