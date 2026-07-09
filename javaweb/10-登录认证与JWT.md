# 10 - 登录认证与 JWT

## 1. 登录功能

### 1.1 流程

```
用户输入用户名密码 → 后端校验 → 生成令牌(JWT) → 返回令牌给前端
                                                    │
                                          前端存 localStorage
                                                    │
                                          后续请求在 Header 中携带 Token
```

### 1.2 登录接口实现

```java
@PostMapping("/login")
public Result login(@RequestBody LoginRequest loginRequest) {
    // 1. 查询用户
    Emp emp = empMapper.findByUsername(loginRequest.getUsername());
    if (emp == null) {
        return Result.error("用户名或密码错误");
    }

    // 2. 校验密码（实际应对密码做 MD5 + 盐值加密）
    if (!emp.getPassword().equals(loginRequest.getPassword())) {
        return Result.error("用户名或密码错误");
    }

    // 3. 生成 JWT 令牌
    Map<String, Object> claims = new HashMap<>();
    claims.put("id", emp.getId());
    claims.put("username", emp.getUsername());
    String token = JwtUtils.generateJwt(claims);

    // 4. 返回令牌
    Map<String, Object> data = new HashMap<>();
    data.put("token", token);
    return Result.success(data);
}
```

> ⚠️ 注意：不要返回具体是"用户不存在"还是"密码错误"，防止撞库攻击。统一提示"用户名或密码错误"。

---

## 2. 会话技术演进

### 2.1 为什么需要会话跟踪？

**HTTP 是无状态的**：服务器不知道连续两次请求来自同一个用户。

```
请求1：GET /emps → 服务器：这是谁？
请求2：POST /emps → 服务器：还是刚才那个人吗？
```

### 2.2 Cookie + Session（传统方案）

```
① 登录 → 服务器创建 Session → Set-Cookie: JSESSIONID=abc123 → 浏览器存 Cookie
② 后续请求 → 浏览器自动带 Cookie: JSESSIONID=abc123 → 服务器查 Session → 识别用户
```

| 缺点 | 说明 |
|------|------|
| **无法跨服务器** | Session 存在一台服务器内存中，集群模式下另一台不认识 |
| **CSRF 攻击风险** | 浏览器自动带 Cookie 的特性被利用 |
| **不适合移动端** | Cookie 是浏览器概念，App 难以使用 |

### 2.3 JWT 令牌（现代方案）

```
① 登录 → 服务器生成 JWT → 返回 Token 字符串
② 前端存储 Token（localStorage）→ 后续请求手动加到 Header：Authorization: Bearer <token>
③ 服务器解析 Token → 提取用户信息（无需查 Session/DB）
```

| 优点 | 说明 |
|------|------|
| **无状态** | 服务器不存会话，Token 自带用户信息 |
| **跨服务器** | 任何服务器都能解析同一个 Token |
| **跨平台** | 浏览器/App/小程序都能用 |
| **防篡改** | 带签名校验 |

---

## 3. JWT 详解

### 3.1 JWT 结构

```
xxxxx.yyyyy.zzzzz
  ↑      ↑      ↑
Header  Payload  Signature

Header:   {"alg": "HS256", "typ": "JWT"}  → Base64编码
Payload:  {"id": 1, "username": "zhangsan", "exp": 1700000000}  → Base64编码
Signature: HMACSHA256(Header.Payload, secretKey)  → 防篡改
```

### 3.2 JWT 工具类

```java
public class JwtUtils {

    private static final String SECRET_KEY = "your-256-bit-secret-key-here-min-32-chars!!";
    private static final long EXPIRATION = 12 * 60 * 60 * 1000;  // 12小时

    // 生成 JWT
    public static String generateJwt(Map<String, Object> claims) {
        return Jwts.builder()
            .setClaims(claims)
            .setExpiration(new Date(System.currentTimeMillis() + EXPIRATION))
            .signWith(SignatureAlgorithm.HS256, SECRET_KEY)
            .compact();
    }

    // 解析 JWT
    public static Claims parseJwt(String token) {
        return Jwts.parser()
            .setSigningKey(SECRET_KEY)
            .parseClaimsJws(token)
            .getBody();
    }
}
```

---

## 4. 登录校验 — 拦截器 vs 过滤器

### 4.1 Filter（过滤器）— Servlet 原生

```java
@WebFilter(urlPatterns = "/*")
public class LoginFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;
        HttpServletResponse resp = (HttpServletResponse) response;

        // 1. 获取请求路径
        String url = req.getRequestURI();

        // 2. 放行登录接口
        if (url.contains("/login")) {
            chain.doFilter(request, response);
            return;
        }

        // 3. 校验 Token
        String token = req.getHeader("Authorization");
        if (token == null || !token.startsWith("Bearer ")) {
            resp.setStatus(401);
            resp.getWriter().write("{\"code\":0,\"msg\":\"NOT_LOGIN\"}");
            return;
        }

        try {
            // 4. 解析 Token
            Claims claims = JwtUtils.parseJwt(token.substring(7));
            // 将用户信息存入 request 供后续使用
            req.setAttribute("userId", claims.get("id"));
        } catch (Exception e) {
            resp.setStatus(401);
            resp.getWriter().write("{\"code\":0,\"msg\":\"TOKEN_INVALID\"}");
            return;
        }

        // 5. 放行
        chain.doFilter(request, response);
    }
}
```

### 4.2 Interceptor（拦截器）— Spring 框架

```java
@Component
public class LoginInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
                             Object handler) throws Exception {
        // 1. 获取 Token
        String token = request.getHeader("Authorization");

        // 2. 解析 Token（失败则返回 false）
        if (token == null || !token.startsWith("Bearer ")) {
            response.setStatus(401);
            response.getWriter().write("{\"code\":0,\"msg\":\"NOT_LOGIN\"}");
            return false;  // 拦截
        }

        try {
            Claims claims = JwtUtils.parseJwt(token.substring(7));
            // 存入 ThreadLocal 供后续使用
            UserHolder.set(claims);
        } catch (Exception e) {
            response.setStatus(401);
            return false;
        }

        return true;  // 放行
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response,
                                Object handler, Exception ex) {
        // 请求结束后清理 ThreadLocal，防止内存泄漏
        UserHolder.remove();
    }
}
```

**注册拦截器**：

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Autowired
    private LoginInterceptor loginInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(loginInterceptor)
            .addPathPatterns("/**")              // 拦截所有
            .excludePathPatterns("/login");      // 排除登录
    }
}
```

### 4.3 Filter vs Interceptor

| | Filter | Interceptor |
|------|--------|-------------|
| **归属** | Servlet 容器（Tomcat） | Spring 框架 |
| **执行顺序** | Filter 先 → Interceptor 后 | |
| **颗粒度** | 只能拦截 URL | 可拦截 Controller 方法 |
| **功能** | 只能操作 request/response | 可访问 Spring Bean、Handler |
| **常用场景** | 编码设置、CORS 处理 | 登录校验、权限校验、日志 |

### 4.4 ThreadLocal — 线程内共享数据

```java
public class UserHolder {
    private static final ThreadLocal<Claims> TL = new ThreadLocal<>();

    public static void set(Claims claims) { TL.set(claims); }
    public static Claims get() { return TL.get(); }
    public static void remove() { TL.remove(); }
}

// 在 Service 中直接获取当前登录用户
Claims claims = UserHolder.get();
Long userId = (Long) claims.get("id");
```

---

## 5. 完整登录认证流程

```
┌──────────────────────────────────────────────────────────────┐
│  ① POST /login  用户名+密码 → 校验 → 返回 JWT Token            │
│  ② 前端存 Token 到 localStorage                               │
│  ③ 后续请求 Header 带 Authorization: Bearer <token>            │
│  ④ 拦截器 → 解析 Token → 校验是否过期/签名是否有效              │
│  ⑤ 通过：提取用户信息存 ThreadLocal → Controller 可用          │
│  ⑥ 不通过：返回 401                                           │
│  ⑦ 请求结束：清理 ThreadLocal                                 │
└──────────────────────────────────────────────────────────────┘
```

---

## 6. 总结

| 知识点 | 核心要点 |
|--------|----------|
| HTTP 无状态 | 需要额外机制（Cookie/Session/Token）识别用户 |
| Cookie+Session | 传统方案，Session 存服务器内存，集群不友好 |
| JWT | Header.Payload.Signature，无状态，自带用户信息 |
| JWT 生成 | `Jwts.builder().setClaims().setExpiration().signWith().compact()` |
| JWT 解析 | `Jwts.parser().setSigningKey().parseClaimsJws().getBody()` |
| Filter | Servlet 级拦截，先于 Interceptor 执行 |
| Interceptor | Spring 级拦截，可访问 Controller 方法信息 |
| ThreadLocal | 线程隔离存储，请求结束必须 `remove()` 防内存泄漏 |
| `@Configuration` + `WebMvcConfigurer` | 注册拦截器 + 配置拦截/排除路径 |
