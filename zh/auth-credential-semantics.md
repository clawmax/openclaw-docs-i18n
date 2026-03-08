

  配置与操作

  
# 认证凭据语义

本文档定义了在以下场景中使用的规范凭据资格与解析语义：

-   `resolveAuthProfileOrder`
-   `resolveApiKeyForProfile`
-   `models status --probe`
-   `doctor-auth`

目标是保持选择时与运行时的行为一致。

## 稳定的原因代码

-   `ok`
-   `missing_credential`
-   `invalid_expires`
-   `expired`
-   `unresolved_ref`

## 令牌凭据

令牌凭据 (`type: "token"`) 支持内联 `token` 和/或 `tokenRef`。

### 资格规则

1.  当 `token` 和 `tokenRef` 均不存在时，令牌配置文件不具备资格。
2.  `expires` 是可选的。
3.  如果存在 `expires`，它必须是一个大于 `0` 的有限数字。
4.  如果 `expires` 无效（`NaN`、`0`、负数、非有限数或类型错误），则配置文件不具备资格，原因为 `invalid_expires`。
5.  如果 `expires` 已过时，则配置文件不具备资格，原因为 `expired`。
6.  `tokenRef` 不会绕过 `expires` 验证。

### 解析规则

1.  解析器语义在 `expires` 方面与资格语义匹配。
2.  对于具备资格的配置文件，令牌材料可以从内联值或 `tokenRef` 解析。
3.  无法解析的引用会在 `models status --probe` 输出中产生 `unresolved_ref`。

## 向后兼容的消息传递

为了脚本兼容性，探测错误的第一行保持不变：`Auth profile credentials are missing or expired.`。人性化的详细信息和稳定的原因代码可以添加在后续行中。

[认证](./gateway/authentication.md)[密钥管理](./gateway/secrets.md)