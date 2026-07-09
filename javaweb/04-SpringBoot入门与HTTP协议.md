# 04 - SpringBoot 入门与 HTTP 协议

## 1. SpringBootWeb 入门

### 1.1 Spring 家族简介

Spring 是目前 Java 开发的事实标准，官网口号：**"Spring makes Java simple."**

```
Spring Framework (核心：IOC/DI + AOP)
├── Spring Boot (快速开发框架，自动配置)
├── Spring MVC (Web 层框架)
├── Spring Cloud (微服务)
├── Spring Data (数据访问，含 JPA)
├── Spring Security (安全框架)
└── Spring Cloud Alibaba (阿里微服务生态)
```

### 1.2 SpringBoot 入门程序

```java
// 1. 创建 SpringBoot 工程（IDEA → Spring Initializr）
// 2. 编写 Controller
@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String hello() {
        return "Hello, SpringBoot!";
    }
}

// 3. 启动类（自动生成）
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// 4. 浏览器访问 http://localhost:8080/hello → Hello, SpringBoot!
```

### 1.3 @SpringBootApplication 做了什么事？

```java
@SpringBootApplication  // 组合注解，等价于：
    @SpringBootConfiguration  // 标记为配置类
    @EnableAutoConfiguration  // 自动配置（核心！根据依赖自动配 Bean）
    @ComponentScan            // 组件扫描（扫描当前包及子包）
```

### 1.4 内嵌 Tomcat

SpringBoot 内嵌了 Tomcat，不需要单独安装和部署：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <!-- 内部包含 spring-boot-starter-tomcat -->
</dependency>
```

> 启动 `main` 方法 → 自动启动 Tomcat → 默认端口 8080。
> 修改端口：`application.properties` 中 `server.port=9090`

---

## 2. HTTP 协议

### 2.1 概述

HTTP（HyperText Transfer Protocol）是浏览器和服务器之间的**通信协议**，基于 TCP。

```
浏览器 ──→ [HTTP 请求] ──→ Tomcat 服务器
        ←── [HTTP 响应] ──←
```

**特点**：
- **无状态**：每次请求独立，服务器不记得上次是谁请求的（需要 Session/Token 解决）
- **请求-响应模型**：一次请求对应一次响应

### 2.2 HTTP 请求格式

```
POST /api/users HTTP/1.1                     ← 请求行（方法 + URI + 版本）
Host: localhost:8080                         ← 请求头（Key: Value）
Content-Type: application/json
Authorization: Bearer eyJhbGciOi...
User-Agent: Mozilla/5.0 ...
                                             ← 空行（头与体分隔）
{                                            ← 请求体（JSON / 表单数据）
    "name": "张三",
    "age": 18
}
```

**常见请求头**：

| 请求头 | 说明 |
|--------|------|
| `Host` | 请求的主机名和端口 |
| `Content-Type` | 请求体格式：`application/json`、`application/x-www-form-urlencoded` |
| `Authorization` | 认证信息（如 JWT Token） |
| `User-Agent` | 浏览器标识 |
| `Cookie` | 客户端存储的 Cookie |

### 2.3 HTTP 响应格式

```
HTTP/1.1 200 OK                               ← 状态行（版本 + 状态码 + 描述）
Content-Type: application/json                ← 响应头
Content-Length: 35
                                              ← 空行
{                                             ← 响应体
    "code": 200,
    "message": "success",
    "data": { "id": 1, "name": "张三" }
}
```

### 2.4 常见状态码

| 状态码 | 含义 | 典型场景 |
|--------|------|----------|
| **200 OK** | 成功 | 正常返回数据 |
| **201 Created** | 创建成功 | POST 新增资源 |
| **204 No Content** | 成功但无返回体 | DELETE 删除成功 |
| **301** | 永久重定向 | URL 变更 |
| **302** | 临时重定向 | 登录后跳转 |
| **400 Bad Request** | 请求参数错误 | 参数校验失败 |
| **401 Unauthorized** | 未认证 | 未登录 |
| **403 Forbidden** | 无权限 | 已登录但权限不足 |
| **404 Not Found** | 资源不存在 | URL 错误 |
| **405 Method Not Allowed** | 方法不允许 | GET 访问了 POST 接口 |
| **500 Internal Server Error** | 服务器内部错误 | 代码抛异常 |

### 2.5 GET vs POST

| | GET | POST |
|------|-----|------|
| **参数位置** | URL 的 `?` 后面（Query String） | 请求体（Body） |
| **参数长度** | 受限（浏览器/服务器 URL 长度限制） | 无限制 |
| **安全性** | 参数暴露在 URL | 参数在 Body 中 |
| **幂等性** | ✅ 幂等 | ❌ 不幂等 |
| **缓存** | 可缓存 | 不可缓存 |
| **适用场景** | 查询数据 | 新增/修改数据 |

---

## 3. 请求参数接收

### 3.1 SpringBoot 中接收参数的几种方式

```java
@RestController
public class UserController {

    // 1. 简单参数：URL ?name=张三&age=18
    @GetMapping("/users")
    public Result list(@RequestParam(defaultValue = "1") Integer page,
                       @RequestParam(defaultValue = "10") Integer size) {
        // ...
    }

    // 2. 路径参数：/users/5
    @GetMapping("/users/{id}")
    public Result getById(@PathVariable Long id) {
        // ...
    }

    // 3. JSON 参数：请求体是 JSON
    @PostMapping("/users")
    public Result add(@RequestBody User user) {
        // ...
    }

    // 4. 复合：/users/5 + JSON body
    @PutMapping("/users/{id}")
    public Result update(@PathVariable Long id, @RequestBody User user) {
        user.setId(id);
        // ...
    }
}
```

### 3.2 @RestController = @Controller + @ResponseBody

```java
@Controller          // 标记为 MVC Controller，方法返回视图名
@ResponseBody        // 方法返回值直接写入 HTTP 响应体（JSON）

// 等价于
@RestController      // REST 风格 Controller，默认将返回值序列化为 JSON
```

---

## 4. 分层解耦与 IOC/DI

### 4.1 三层架构

```
┌─────────────────────────────────────────────┐
│                  Controller 层                │
│      接收请求、参数校验、返回响应               │
│              (不写业务逻辑)                    │
└──────────────────┬──────────────────────────┘
                   │ 调用
                   ▼
┌─────────────────────────────────────────────┐
│                  Service 层                   │
│      业务逻辑、事务管理                        │
│              (不写 SQL)                       │
└──────────────────┬──────────────────────────┘
                   │ 调用
                   ▼
┌─────────────────────────────────────────────┐
│                  Mapper 层                    │
│      数据访问、SQL 操作                       │
│              (MyBatis / MyBatis-Plus)        │
└─────────────────────────────────────────────┘
```

**为什么要分层？**

| 好处 | 说明 |
|------|------|
| 职责清晰 | Controller 只管请求响应，Service 只管业务，Mapper 只管数据 |
| 易于维护 | 改 SQL 只改 Mapper，改业务只改 Service |
| 便于复用 | Service 可以同时被 Controller 和定时任务调用 |
| 利于测试 | 每层可以独立单元测试 |

### 4.2 解耦 — IOC 与 DI

**问题**：Controller 直接 `new Service()` → 紧耦合，不利于替换和测试。

**IOC（控制反转）**：把对象的创建权交给 Spring 容器

```java
@Service      // 将 UserServiceImpl 交给 IOC 容器管理（创建对象）
public class UserServiceImpl implements UserService { ... }

@Repository   // 将 UserMapper 交给 IOC 容器管理
public interface UserMapper extends BaseMapper<User> { ... }
```

**DI（依赖注入）**：Spring 容器自动为需要的 Bean 注入依赖

```java
@RestController
public class UserController {

    @Autowired   // 从 IOC 容器获取 UserService 实例并注入
    private UserService userService;
}
```

**IOC/DI 的好处**：
- 对象创建由 Spring 管理，不用手动 `new`
- 面向接口编程，实现类可灵活替换
- 默认**单例**，内存中只有一个实例

### 4.3 Bean 的声明注解

| 注解 | 作用 | 所在层 |
|------|------|--------|
| `@Component` | 通用组件 | 工具类 |
| `@Controller` | 控制器组件 | Controller 层 |
| `@Service` | 业务逻辑组件 | Service 层 |
| `@Repository` | 数据访问组件 | Mapper 层 |

### 4.4 DI 注入方式

```java
// 方式一：@Autowired（按类型注入，Spring 提供）
@Autowired
private UserService userService;

// 方式二：构造器注入（推荐，Spring 官方推荐）
private final UserService userService;
public UserController(UserService userService) {
    this.userService = userService;
}

// 方式三：@Resource（按名称注入，JDK 提供）
@Resource(name = "userServiceImpl")
private UserService userService;
```

| | @Autowired | @Resource |
|------|-----------|-----------|
| **来源** | Spring 框架 | JDK（javax.annotation） |
| **默认注入方式** | 按类型（byType） | 按名称（byName） |
| **配合 @Qualifier** | 可以指定 Bean 名称 | 不需要（自带 name 属性） |

---

## 5. 总结

| 知识点 | 核心要点 |
|--------|----------|
| SpringBoot 入门 | `@SpringBootApplication` = 配置 + 自动配置 + 组件扫描 |
| 内嵌 Tomcat | 无需外部部署，`main` 方法启动，默认端口 8080 |
| HTTP 请求 | 请求行 + 请求头 + 空行 + 请求体 |
| HTTP 响应 | 状态行 + 响应头 + 空行 + 响应体 |
| 常见状态码 | 200(成功) 400(参数错) 401(未登录) 403(无权限) 404(不存在) 500(服务器错) |
| GET vs POST | GET 参数在 URL，POST 在 Body；GET 幂等，POST 不幂等 |
| `@RestController` | `@Controller` + `@ResponseBody`，返回值 → JSON |
| 参数接收 | `@RequestParam`(查询参数) `@PathVariable`(路径) `@RequestBody`(JSON) |
| 三层架构 | Controller → Service → Mapper，职责分层清晰 |
| IOC | 对象创建权交给 Spring 容器，`@Component`/`@Service`/`@Repository` |
| DI | 容器自动注入依赖，`@Autowired`(按类型) `@Resource`(按名称) |
