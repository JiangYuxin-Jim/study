# 11 - AOP 面向切面编程

## 1. AOP 基础

### 1.1 什么是 AOP？

AOP（Aspect-Oriented Programming）面向切面编程，是一种**在不修改原代码的情况下，为方法动态添加额外功能**的编程范式。

```
没有 AOP：
每个方法里都写同样的日志/事务/权限代码 → 代码重复、难以维护

有了 AOP：
将日志/事务/权限代码提取成一个"切面"，通过配置"织入"到目标方法
→ 原代码不变，额外功能自动生效
```

### 1.2 核心概念

```
┌─────────────────────────────────────────────────────────┐
│                    AOP 核心概念                            │
│                                                          │
│  JoinPoint（连接点）    → 所有可能被增强的方法              │
│  Pointcut（切入点）     → 实际被增强的方法（表达式匹配）     │
│  Advice（通知）         → 增强的逻辑（何时执行、做什么）     │
│  Aspect（切面）         = Pointcut + Advice                │
│                                                          │
│  例：记录所有 Service 方法的执行时间                         │
│  Pointcut：com.heima.service 包下所有方法                  │
│  Advice：记录方法开始时间和结束时间，计算差值               │
│  Aspect：Pointcut + Advice                                │
└─────────────────────────────────────────────────────────┘
```

### 1.3 AOP 入门示例

```java
// 1. 引入依赖
// spring-boot-starter-aop（已包含在 spring-boot-starter-web 中）

// 2. 编写切面类
@Component
@Aspect  // 声明这是一个切面
public class TimeAspect {

    @Around("execution(* com.heima.service.*.*(..))")  // 切入点表达式
    public Object recordTime(ProceedingJoinPoint joinPoint) throws Throwable {
        // 前置：记录开始时间
        long begin = System.currentTimeMillis();

        // 调用原方法
        Object result = joinPoint.proceed();

        // 后置：记录结束时间
        long end = System.currentTimeMillis();
        log.info("{} 耗时：{}ms",
            joinPoint.getSignature().toString(), (end - begin));

        return result;
    }
}
```

> 就这样，所有 Service 层方法自动被计时，**原方法代码一行没改**。

---

## 2. AOP 通知类型

| 通知类型 | 注解 | 执行时机 |
|----------|------|----------|
| **前置通知** | `@Before` | 目标方法执行前 |
| **后置通知** | `@After` | 目标方法执行后（无论成功/异常） |
| **返回后通知** | `@AfterReturning` | 目标方法**成功返回后** |
| **异常后通知** | `@AfterThrowing` | 目标方法**抛异常后** |
| **环绕通知** | `@Around` | 包裹目标方法（前后都可写逻辑），**最强大** |

```java
@Component
@Aspect
public class LogAspect {

    // 前置：记录操作日志
    @Before("execution(* com.heima.service.DeptService.*(..))")
    public void before(JoinPoint joinPoint) {
        log.info("调用方法：{}", joinPoint.getSignature().getName());
    }

    // 返回后：记录返回值
    @AfterReturning(value = "execution(* com.heima.service.*.*(..))",
                    returning = "result")
    public void afterReturn(JoinPoint joinPoint, Object result) {
        log.info("{} 返回值：{}", joinPoint.getSignature(), result);
    }

    // 异常后：记录异常
    @AfterThrowing(value = "execution(* com.heima.service.*.*(..))",
                   throwing = "ex")
    public void afterThrow(JoinPoint joinPoint, Exception ex) {
        log.error("{} 异常：{}", joinPoint.getSignature(), ex.getMessage());
    }

    // 环绕：功能最强，可替代以上所有
    @Around("execution(* com.heima.service.*.*(..))")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        log.info("开始执行：{}", pjp.getSignature());
        try {
            Object result = pjp.proceed();   // 执行原方法
            log.info("执行成功：{}", pjp.getSignature());
            return result;
        } catch (Exception e) {
            log.error("执行异常：{}", e.getMessage());
            throw e;
        }
    }
}
```

### 2.1 通知执行顺序

```
正常执行：
  @Around（前）→ @Before → 目标方法 → @AfterReturning → @After → @Around（后）

异常执行：
  @Around（前）→ @Before → 目标方法抛异常 → @AfterThrowing → @After → @Around（后）
```

---

## 3. 切入点表达式

### 3.1 execution 表达式（最常用）

```
execution([修饰符] 返回值类型 [包名.类名.]方法名(参数类型) [throws 异常])

execution(* com.heima.service.*.*(..))
         ↑            ↑           ↑  ↑
       任意返回值   包下所有类  所有方法 任意参数

示例：
execution(* com.heima.service.DeptService.delete(..))
execution(* com.heima.service.*.get*(..))            // get 开头的方法
execution(* com.heima..*(..))                        // com.heima 及所有子包
```

### 3.2 @annotation 表达式

```java
// 自定义注解
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogOperation {
    String value() default "";
}

// 切面：对有 @LogOperation 的方法记录日志
@Around("@annotation(logOp)")
public Object log(ProceedingJoinPoint pjp, LogOperation logOp) throws Throwable {
    log.info("操作：{}，参数：{}", logOp.value(), Arrays.toString(pjp.getArgs()));
    return pjp.proceed();
}

// 使用
@Service
public class DeptService {
    @LogOperation("删除部门")
    public void delete(Integer id) { ... }
}
```

---

## 4. AOP 实战案例 — 操作日志

```java
@Component
@Aspect
public class OperationLogAspect {

    @Autowired
    private OperateLogMapper operateLogMapper;  // 日志写入数据库

    @Around("@annotation(com.heima.anno.Log)")  // 拦截有 @Log 注解的方法
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        // 1. 记录开始时间
        long begin = System.currentTimeMillis();

        // 2. 执行原方法
        Object result = pjp.proceed();

        // 3. 记录结束时间
        long cost = System.currentTimeMillis() - begin;

        // 4. 获取当前登录用户（从 ThreadLocal 取）
        Claims claims = UserHolder.get();
        Long userId = claims != null ? (Long) claims.get("id") : null;

        // 5. 记录操作日志到数据库
        OperateLog log = new OperateLog();
        log.setOperateEmpId(userId);
        log.setOperateTime(LocalDateTime.now());
        log.setClassName(pjp.getTarget().getClass().getName());
        log.setMethodName(pjp.getSignature().getName());
        log.setMethodParams(Arrays.toString(pjp.getArgs()));
        log.setReturnValue(result != null ? result.toString() : "void");
        log.setCostTime(cost);
        operateLogMapper.insert(log);

        return result;
    }
}
```

---

## 5. 总结

| 知识点 | 核心要点 |
|--------|----------|
| AOP 是什么 | 不修改原代码，动态增强方法功能 |
| 核心概念 | JoinPoint → Pointcut + Advice = Aspect |
| `@Around` | 环绕通知最强大，包围整个方法执行 |
| `@Before` `@After` | 前置/后置通知 |
| execution 表达式 | `execution(* 包名.类名.方法名(..))` |
| `@annotation` | 按自定义注解匹配切入点 |
| ProceedingJoinPoint | 代表被增强的方法，`.proceed()` 执行原方法 |
| 典型应用 | 日志记录、事务管理、权限校验、性能统计 |
