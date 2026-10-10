# 签名机制

本文档说明盛付通 H5 快捷支付的签名和验签规则。这是接入过程中最常见的卡点，请仔细阅读。

## 核心原则

- 商户请求 → 用**商户私钥**加签 → 盛付通用商户公钥验签
- 盛付通响应/回调 → 用**盛付通私钥**加签 → 商户用盛付通公钥验签

## 签名算法

| 算法 | signType 值 | 说明 |
|------|-------------|------|
| RSA | `RSA` | SHA1withRSA （若同时使用其它产品，选择此方式）|
| RSA2 | `RSA2` | SHA256withRSA（推荐） |

## 签名字符串构造

**步骤**：

1. 获取所有请求参数（除 `sign` 字段本身）
2. 按参数名（key）的**字典序（ASCII 序）**排序
3. 拼接为 `key1=value1&key2=value2&...&keyN=valueN`
   - **空值不参与**签名
   - 对于 JSON 字符串字段（如 `extra`、`detail`），使用其 JSON 字符串原值参与拼接
4. 使用**商户私钥**对签名字符串签名
5. Base64 编码后作为 `sign` 参数值

### 伪代码

```
签名步骤：
1. params.remove("sign")          // 排除 sign 字段
2. 按 key 字典序排序所有参数
3. 过滤空值、拼接 key=value&...
4. sign = RSA_Sign(content, privateKey, SHA256withRSA)
5. params.put("sign", base64(sign))
```

### 示例（Java）

```java
// 生成签名字符串
public String getSignatureContent(Map<String, Object> params) {
    params.remove("sign");
    StringBuilder sb = new StringBuilder();
    params.entrySet().stream()
        .filter(e -> e.getValue() != null && !"".equals(e.getValue()))
        .sorted(Map.Entry.comparingByKey())
        .forEach(e -> sb.append(e.getKey()).append("=").append(e.getValue()).append("&"));
    if (sb.length() > 0) sb.setLength(sb.length() - 1);
    return sb.toString();
}

// 签名
String content = getSignatureContent(params);
String sign = SignatureUtil.sign(content, merchantPrivateKey, "RSA2");

// 验签（验证盛付通响应/回调）
boolean valid = SignatureUtil.verify(content, shengpayPublicKey, sign, "RSA2");
```

## ⚠️ 重要：extra 字段参与签名

`extra` 字段（JSON 格式字符串）直接以 JSON 字符串原值参与签名拼接，**不做内部排序**。

例如 `extra={"payRestriction":{"cardType":"DR"},"payer":{"name":"xxx"}}` 整体作为一个值参与拼接：

```
appId=312343123132131331232&body=商品&extra={"payRestriction":{"cardType":"DR"}}&mchId=93762611&...
```

## 常见验签失败原因

1. **拼接规则错误**：未按字典序排序、未排除 sign 字段、空值参与了拼接
2. **密钥不匹配**：用错了密钥对（商户私钥 vs 盛付通公钥）
3. **signType 不一致**：算法声明与实际使用不符
4. **密钥格式问题**：换行符丢失、多了空格、PEM 格式不正确
5. **extra 处理错误**：对 extra 内部做了排序或二次转义
6. **字符编码**：未使用 UTF-8

## 排查步骤

1. 打印签名字符串原文，确认格式为 `key1=value1&key2=value2&...`
2. 确认 `sign` 字段未出现在签名字符串中
3. 确认空值字段未出现在签名字符串中
4. 确认使用的密钥与 signType 一致
5. 用同一签名字符串和盛付通公钥手动验签，定位是加签还是验签出错
