# 12 - SpringBoot 原理

## 1. 自动配置原理

### 1.1 回顾 @SpringBootApplication

```java
@SpringBootApplication
    @SpringBootConfiguration    // 标记为配置类
    @EnableAutoConfiguration    // 自动配置（核心）
    @ComponentScan              // 组件扫描
```

### 1.2 自动配置是如何工作的？

```
1. @EnableAutoConfiguration
       │
2. @Import(AutoConfigurationImportSelector.class)
       │
3. 读取 META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
       │
4. 加载所有自动配置类（如 DataSourceAutoConfiguration、WebMvcAutoConfiguration...）
       │
5. @Conditional 条件判断 → 满足条件就创建对应的 Bean
```

### 1.3 @Conditional 条件注解

**只有满足条件时，配置才生效**：

```java
@Bean
@ConditionalOnClass(name = "com.mysql.cj.jdbc.Driver")  // 类路径有 MySQL 驱动
public DataSource dataSource() { ... }

@Bean
@ConditionalOnMissingBean  // 用户自己没定义 DataSource 时才创建
public DataSource dataSource() { ... }
```

| 条件注解 | 含义 |
|----------|------|
| `@ConditionalOnClass` | 类路径存在指定类 |
| `@ConditionalOnMissingClass` | 类路径不存在指定类 |
| `@ConditionalOnBean` | 容器中存在指定 Bean |
| `@ConditionalOnMissingBean` | 容器中不存在指定 Bean |
| `@ConditionalOnProperty` | 配置文件中有指定属性 |
| `@ConditionalOnWebApplication` | 当前是 Web 应用 |

**示例**：为什么引入了 `mybatis-spring-boot-starter`，MyBatis 就自动配置好了？

```
1. starter 依赖中包含 MybatisAutoConfiguration
2. @ConditionalOnClass(SqlSessionFactory.class) → 有 MyBatis jar → 条件通过
3. @ConditionalOnBean(DataSource.class) → 有数据源 → 条件通过
4. 自动创建 SqlSessionFactory、SqlSessionTemplate 等 Bean
→ 用户只需写 Mapper 接口即可
```

---

## 2. 起步依赖（Starter）

### 2.1 什么是 Starter？

Starter 是一组依赖的**聚合打包**，让你只需引入一个 starter 就能获得全套依赖。

```xml
<!-- 只需引入这一个 → 自动带上 spring-webmvc、spring-boot-starter、
     spring-boot-starter-tomcat、jackson 等所有所需依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### 2.2 自定义 Starter

```java
// 自定义 starter 的标准结构：
my-starter/
├── my-starter-autoconfigure/    // 自动配置模块
│   ├── MyAutoConfiguration.java // 配置类
│   └── META-INF/spring/xxx.imports  // 自动配置注册
└── my-starter/                  // 空壳模块，只负责引入 autoconfigure
```

---

## 3. Bean 的获取与作用域

### 3.1 获取容器中的 Bean

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        ApplicationContext ctx = SpringApplication.run(Application.class, args);

        // 按类型获取
        DeptService deptService = ctx.getBean(DeptService.class);

        // 按名称获取
        DeptService ds = (DeptService) ctx.getBean("deptServiceImpl");

        // 获取所有该类型的 Bean
        Map<String, DeptService> beans = ctx.getBeansOfType(DeptService.class);
    }
}
```

### 3.2 Bean 作用域

```java
@Scope("singleton")  // 单例（默认）：整个容器只有一个实例
@Scope("prototype")  // 多例：每次获取都创建新实例
@Scope("request")    // Web 环境：每个请求一个实例
@Scope("session")    // Web 环境：每个会话一个实例
```

| 作用域 | 说明 | 何时销毁 |
|--------|------|----------|
| `singleton` | 默认，整个容器唯一实例 | 容器关闭 |
| `prototype` | 每次获取新实例 | 不再被引用时 GC |

### 3.3 第三方 Bean 注册

```java
@Configuration
public class BeanConfig {

    // 方法返回值注册为 Bean，Bean 名称 = 方法名
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }

    // 自定义 Bean 名称
    @Bean("myMapper")
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}
```

> 注：`@Configuration` 类中 `@Bean` 方法默认也是单例（Spring 通过 CGLIB 代理保证）。

---

## 4. SpringBoot 启动流程

```
SpringApplication.run(Application.class, args)
        │
        ▼
① 创建 SpringApplication 实例
   └── 推断应用类型（Servlet / Reactive）
   └── 加载 ApplicationContextInitializer
   └── 加载 ApplicationListener
        │
        ▼
② 执行 run()
   ├── 创建 Environment（加载 application.properties/yml）
   ├── 打印 Banner
   ├── 创建 ApplicationContext（AnnotationConfigServletWebServerApplicationContext）
   ├── 执行 ApplicationContextInitializer
   │       │
   │       ▼
   ├── refreshContext（核心！）
   │   ├── 解析 @Configuration 类
   │   ├── 组件扫描 @ComponentScan
   │   ├── 自动配置 @EnableAutoConfiguration
   │   ├── 启动内嵌 Tomcat（ServletWebServerApplicationContext.onRefresh）
   │   └── 注册所有 Bean
   │       │
   │       ▼
   ├── 执行 ApplicationRunner / CommandLineRunner
   └── 应用启动完成，等待请求
```

---

## 5. 总结

| 知识点 | 核心要点 |
|--------|----------|
| `@SpringBootApplication` | 配置 + 自动配置 + 组件扫描，三位一体 |
| 自动配置原理 | `spring.factories` → 加载自动配置类 → `@Conditional` 条件判断 → 创建 Bean |
| `@Conditional` | `OnClass` `OnMissingBean` `OnProperty` 等，条件满足才配置 |
| Starter | 依赖聚合包，一次引入全套依赖 |
| Bean 作用域 | `singleton`（默认） `prototype`（多例） |
| `@Bean` | 将第三方类的实例注册到 IOC 容器 |
| `@Configuration` | 配置类，`@Bean` 方法默认也是单例（CGLIB 代理） |
| 启动流程 | 创建 Application → 加载配置 → refresh → 启动 Tomcat → 运行 |
