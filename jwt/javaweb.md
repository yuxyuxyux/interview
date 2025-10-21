👌 你问的是 **JavaWeb（不一定用 Spring Boot）里 JWT 的实现方式**，那我给你一个最通用的版本：用 Servlet + Filter 来实现 JWT 登录认证。这样无论你是普通 JavaWeb 项目还是 Spring Boot，都可以用。

------

## 🔑 JWT 基本原理

1. 用户登录成功后，服务端生成 **JWT Token** 并返回给前端。
2. 前端把 token 存在 **localStorage / sessionStorage / cookie**。
3. 前端以后请求时带上 `Authorization: Bearer <token>`。
4. 服务端的 **Filter** 拦截请求，解析并校验 token。

------

## 📦 依赖

如果用 Maven：

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
    private static final Key key = Keys.secretKeyFor(SignatureAlgorithm.HS256);
    private static final long EXPIRATION = 60 * 60 * 1000; // 1小时

    // 生成 Token
    public static String generateToken(String username) {
        return Jwts.builder()
                .setSubject(username)
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + EXPIRATION))
                .signWith(key)
                .compact();
    }

    // 校验 Token
    public static String validateToken(String token) {
        try {
            return Jwts.parserBuilder().setSigningKey(key).build()
                    .parseClaimsJws(token)
                    .getBody()
                    .getSubject();
        } catch (JwtException e) {
            return null; // 无效 token
        }
    }
}
```

------

## 🎯 登录接口（Servlet 示例）

```java
@WebServlet("/login")
public class LoginServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        String username = req.getParameter("username");
        String password = req.getParameter("password");

        // 假设用户名/密码校验
        if ("admin".equals(username) && "123456".equals(password)) {
            String token = JwtUtil.generateToken(username);

            resp.setContentType("application/json;charset=UTF-8");
            resp.getWriter().write("{\"token\":\"" + token + "\"}");
        } else {
            resp.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            resp.getWriter().write("{\"error\":\"用户名或密码错误\"}");
        }
    }
}
```

------

## 🛡 JWT 校验过滤器

```java
@WebFilter("/*")
public class JwtFilter implements Filter {
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse resp = (HttpServletResponse) response;

        String path = req.getRequestURI();

        // 登录接口放行
        if (path.equals("/login")) {
            chain.doFilter(request, response);
            return;
        }

        // 获取 Authorization 头
        String authHeader = req.getHeader("Authorization");
        if (authHeader != null && authHeader.startsWith("Bearer ")) {
            String token = authHeader.substring(7);
            String username = JwtUtil.validateToken(token);

            if (username != null) {
                req.setAttribute("username", username);
                chain.doFilter(request, response);
                return;
            }
        }

        // token 无效
        resp.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        resp.getWriter().write("{\"error\":\"无效的 Token\"}");
    }
}
```

------

## 🚀 测试流程

1. 登录：

   ```bash
   curl -X POST "http://localhost:8080/yourapp/login" -d "username=admin&password=123456"
   ```

   返回：

   ```json
   {"token":"xxxxxx"}
   ```

2. 调用需要认证的接口：

   ```bash
   curl -H "Authorization: Bearer xxxxxx" http://localhost:8080/yourapp/hello
   ```

------

👉 这样就是一个完整的 **JavaWeb JWT 实现**，用 Servlet + Filter 就能跑通。
 要不要我再帮你写一个 **配合 Vue/前端的示例请求流程**，方便你测试？