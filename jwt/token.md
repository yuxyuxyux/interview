👌 我来帮你把 **JWT token 的结构与细节**讲清楚。

------

# 🔑 JWT Token 详解

JWT（JSON Web Token）是一个**基于 JSON 的开放标准（RFC 7519）**，用来在客户端和服务端之间安全地传递信息。
 它是 **字符串格式**，通常长这样：

```
xxxxx.yyyyy.zzzzz
```

由 **三部分**组成：`Header.Payload.Signature`

------

## 1️⃣ Header（头部）

头部一般有两个字段：

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

- `alg`：签名算法，例如 HS256（HMAC + SHA256）、RS256（RSA 签名）。
- `typ`：类型，固定是 `"JWT"`。

编码方式：`Base64UrlEncode(header)`
 👉 结果是第一段字符串。

------

## 2️⃣ Payload（载荷）

存放实际要传递的数据（Claims）。JWT 规定了几种 **标准字段**（可选）：

| 字段  | 含义                                 |
| ----- | ------------------------------------ |
| `iss` | 签发者 (issuer)                      |
| `sub` | 主题 (subject，一般存用户名或用户ID) |
| `aud` | 接收方 (audience)                    |
| `exp` | 过期时间 (expiration)                |
| `nbf` | 生效时间 (not before)                |
| `iat` | 签发时间 (issued at)                 |
| `jti` | JWT 的唯一标识（防止重复使用）       |

你也可以加自定义字段，比如：

```json
{
  "sub": "admin",
  "role": "ROLE_USER",
  "exp": 1715670000
}
```

编码方式：`Base64UrlEncode(payload)`
 👉 结果是第二段字符串。

------

## 3️⃣ Signature（签名）

签名的作用是防篡改。
 计算方法：

```
HMACSHA256(
  Base64UrlEncode(header) + "." + Base64UrlEncode(payload),
  secret
)
```

- 如果是 HS256：使用一个固定的 **秘钥字符串（secret key）**。
- 如果是 RS256：使用私钥签名，公钥验签。

👉 结果是第三段字符串。

------

## 🔗 组合

最终 Token：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJhZG1pbiIsInJvbGUiOiJST0xFX1VTRVIiLCJleHAiOjE3MTU2NzAwMDB9.RvWxVQ_0g6W8sfTb6GlC6YfBfA1cR_TmFmvUlO6jF1w
```

拆开就是：

1. Header：`{"alg":"HS256","typ":"JWT"}`
2. Payload：`{"sub":"admin","role":"ROLE_USER","exp":1715670000}`
3. Signature：根据秘钥算出来的哈希

------

## ⚙️ Token 生命周期

- **签发**：用户登录成功后，服务端生成 JWT。

- **传输**：前端存储在 `localStorage` / `sessionStorage` / `cookie`，以后请求时带在 HTTP Header 里：

  ```
  Authorization: Bearer <token>
  ```

- **校验**：服务端拿到 token，解码前两部分（Header + Payload），用秘钥计算签名比对。

- **过期**：过了 `exp` 就自动失效。

------

## 🔒 优缺点

✅ 优点：

- 跨语言通用（JSON + Base64）
- 无状态（服务端不用保存 session）
- 轻量化，传输效率高

⚠️ 缺点：

- 一旦签发，无法在服务端单方面注销（除非做黑名单）
- Payload 是明文（虽然不能伪造，但能被解码），不要放敏感信息（比如密码）

------

## 🎯 总结

一个 JWT token：

1. `Header`（算法 + 类型）
2. `Payload`（用户数据 + 标准字段）
3. `Signature`（防篡改）

结构就是：

```
Base64Url(header) + "." + Base64Url(payload) + "." + HMACSHA256(...)
```

------

要不要我给你画一个 **图解版 JWT 结构流程**，一眼就能看懂 Header / Payload / Signature 是怎么拼起来的？