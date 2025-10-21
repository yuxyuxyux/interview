你是要在 Spring Boot 里实现 **JWT 登录认证** 吗？我帮你拆成几个关键步骤，给你一个可以直接运行的最小示例。

------

## 🔑 JWT 基本流程

1. 用户提交用户名和密码登录。
2. 服务端校验成功后生成 **JWT Token**，返回给用户。
3. 用户以后请求时在请求头带上 `Authorization: Bearer <token>`。
4. 服务端通过拦截器/过滤器解析 JWT，校验是否合法。

------

## 📦 引入依赖

Maven：

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
```

------

## 🛠 JWT 工具类

```java
import io.jsonwebtoken.*;
import io.jsonwebtoken.security.Keys;

import java.security.Key;
import java.util.Date;

public class JwtUtil {
    // 秘钥（建议放配置文件）
    private static final Key key = Keys.secretKeyFor(SignatureAlgorithm.HS256);
    private static final long EXPIRATION = 1000 * 60 * 60; // 1小时

    // 生成 Token
    public static String generateToken(String username) {
        return Jwts.builder()
                .setSubject(username)  // 存放用户名
                .setIssuedAt(new Date()) // 签发时间
                .setExpiration(new Date(System.currentTimeMillis() + EXPIRATION)) // 过期时间
                .signWith(key) // 签名
                .compact();
    }

    // 解析 Token
    public static Jws<Claims> parseToken(String token) {
        return Jwts.parserBuilder()
                .setSigningKey(key)
                .build()
                .parseClaimsJws(token);
    }

    // 获取用户名
    public static String getUsername(String token) {
        return parseToken(token).getBody().getSubject();
    }
}
```

------

## 🎯 登录接口示例

```java
@RestController
@RequestMapping("/auth")
public class AuthController {

    @PostMapping("/login")
    public Map<String, String> login(@RequestParam String username, @RequestParam String password) {
        // 这里写死，实际需要数据库验证
        if ("admin".equals(username) && "123456".equals(password)) {
            String token = JwtUtil.generateToken(username);
            return Map.of("token", token);
        } else {
            return Map.of("error", "用户名或密码错误");
        }
    }
}
```

------

## 🛡 拦截请求验证 Token

可以用 Spring Filter：

```java
@Component
public class JwtFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain filterChain) throws IOException, ServletException {
        String authHeader = request.getHeader("Authorization");

        if (authHeader != null && authHeader.startsWith("Bearer ")) {
            String token = authHeader.substring(7);
            try {
                String username = JwtUtil.getUsername(token);
                // 这里可以设置 Spring Security 的认证信息
                request.setAttribute("username", username);
            } catch (JwtException e) {
                response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                return;
            }
        }

        filterChain.doFilter(request, response);
    }
}
```

------

## 🚀 使用示例

1. 登录获取 token：

   ```bash
   curl -X POST "http://localhost:8080/auth/login?username=admin&password=123456"
   # {"token":"xxxxxxx"}
   ```

2. 请求业务接口时带 token：

   ```bash
   curl -H "Authorization: Bearer xxxxxxx" http://localhost:8080/hello
   ```

------

要不要我帮你写成 **Spring Security + JWT 的完整登录认证方案**？这样能自动接管权限控制，比自己写 filter 更安全。