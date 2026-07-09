# 03 - Maven 基础与高级

## 1. Maven 概述

### 1.1 为什么需要 Maven？

在没有 Maven 的项目中：
- **jar 包管理混乱**：需要手动下载、复制 jar 包，版本冲突难处理
- **项目结构不统一**：每个项目目录结构各异
- **构建流程不标准**：编译、测试、打包各搞各的

Maven 是一个**项目构建与依赖管理**工具，解决了以上所有问题。

### 1.2 Maven 核心概念

```
┌─────────────────────────────────────────────────────┐
│                  Maven 核心模型                        │
│                                                      │
│  pom.xml (Project Object Model)                      │
│  ┌──────────────────────────────────────────────┐    │
│  │  项目坐标 (groupId + artifactId + version)    │    │
│  │  依赖管理 (dependencies)                      │    │
│  │  构建配置 (plugins)                           │    │
│  └──────────────────────────────────────────────┘    │
│                                                      │
│  仓库体系：本地仓库 → 私服 → 中央仓库（阿里云镜像）       │
│  生命周期：clean → compile → test → package → install  │
└─────────────────────────────────────────────────────┘
```

### 1.3 Maven 坐标

```xml
<groupId>com.heima</groupId>        <!-- 组织/公司标识 -->
<artifactId>tlias-web</artifactId>   <!-- 项目名 -->
<version>1.0-SNAPSHOT</version>     <!-- 版本号 -->
```

> 坐标（GAV）在仓库中**唯一标识一个项目**。

### 1.4 依赖配置

```xml
<dependencies>
    <!-- Spring Boot Web Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.7.5</version>
    </dependency>

    <!-- MySQL 驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.30</version>
        <scope>runtime</scope>  <!-- 依赖范围 -->
    </dependency>
</dependencies>
```

**依赖范围 scope**：

| scope | 编译 | 测试 | 运行 | 示例 |
|-------|:--:|:--:|:--:|------|
| `compile`（默认） | ✅ | ✅ | ✅ | spring-web |
| `provided` | ✅ | ✅ | ❌ | servlet-api（Tomcat 已提供） |
| `runtime` | ❌ | ✅ | ✅ | mysql-connector |
| `test` | ❌ | ✅ | ❌ | junit |

### 1.5 依赖传递

```
A → B → C

A 依赖 B，B 依赖 C → A 自动获得 C（传递性依赖）
但 scope 为 provided/test 的依赖不传递
```

**排除传递性依赖**：

```xml
<dependency>
    <groupId>com.heima</groupId>
    <artifactId>module-a</artifactId>
    <exclusions>
        <exclusion>
            <groupId>com.unwanted</groupId>
            <artifactId>bad-lib</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

---

## 2. Maven 生命周期

```
clean → compile → test → package → install → deploy
 清空     编译     测试    打包     安装到    部署到
                                    本地仓库   远程仓库
```

| 命令 | 作用 |
|------|------|
| `mvn clean` | 删除 `target` 目录 |
| `mvn compile` | 编译源码 → `target/classes` |
| `mvn test` | 运行单元测试 |
| `mvn package` | 编译 + 测试 + 打包（jar/war） |
| `mvn install` | package + 安装到本地仓库 |
| `mvn deploy` | install + 部署到远程仓库（私服） |

> 执行后面的命令会自动执行前面的。如 `mvn package` 会先 `compile` 再 `test` 再打包。

---

## 3. 单元测试 — JUnit

```java
@Test                    // 标记为测试方法
@BeforeEach              // 每个测试方法前执行
@AfterEach               // 每个测试方法后执行
@BeforeAll               // 所有测试前执行一次（static）
@AfterAll                // 所有测试后执行一次（static）
@DisplayName("测试名称")   // 自定义显示名称
@Disabled                // 跳过此测试

// 断言
Assertions.assertEquals(expected, actual);
Assertions.assertNotNull(object);
Assertions.assertTrue(condition);
Assertions.assertThrows(XxxException.class, () -> { ... });
```

```java
class UserServiceTest {

    @Autowired
    private UserService userService;

    @Test
    @DisplayName("根据ID查询用户")
    void testFindById() {
        User user = userService.findById(1L);
        assertNotNull(user);
        assertEquals("张三", user.getName());
    }

    @Test
    @DisplayName("用户不存在时抛异常")
    void testFindByIdNotFound() {
        assertThrows(ResourceNotFoundException.class,
            () -> userService.findById(999L));
    }
}
```

---

## 4. Maven 高级

### 4.1 分模块设计与开发

```
传统单体项目                   分模块项目
┌──────────┐                 ┌──────────┐
│  tlias   │                 │ tlias-parent (父工程，pom)
│  所有代码 │     ──→         │  ├─ tlias-pojo (实体类)
│  混在一起 │                 │  ├─ tlias-mapper (数据访问)
└──────────┘                 │  ├─ tlias-service (业务逻辑)
                             │  └─ tlias-web (Controller)
                             └──────────┘
```

**好处**：
- 模块可独立开发、测试、复用
- 修改某模块只需重新编译该模块
- 团队协作：不同组负责不同模块

### 4.2 继承（父工程统一管理）

**父工程 pom.xml**：

```xml
<packaging>pom</packaging>  <!-- 父工程打包方式必须是 pom -->

<!-- 统一版本号 -->
<properties>
    <java.version>17</java.version>
    <spring-boot.version>2.7.5</spring-boot.version>
</properties>

<!-- 依赖管理：只声明版本，子模块引入时不写 version -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>${spring-boot.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

**子模块**：

```xml
<parent>
    <groupId>com.heima</groupId>
    <artifactId>tlias-parent</artifactId>
    <version>1.0-SNAPSHOT</version>
</parent>

<!-- 子模块引入父工程管理的依赖，无需写 version -->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

### 4.3 聚合（一键构建所有模块）

```xml
<!-- 父工程中配置聚合 -->
<modules>
    <module>tlias-pojo</module>
    <module>tlias-mapper</module>
    <module>tlias-service</module>
    <module>tlias-web</module>
</modules>
```

> **继承** = 版本统一管理，**聚合** = 一键构建所有模块。两者通常配合使用。

### 4.4 私服（Nexus）

```
开发者 → 上传 jar ──→ [私服 Nexus] ──→ 下载 jar ←── 其他开发者
                          │
                          ▼
                     中央仓库 (Maven Central)
```

| 操作 | 配置位置 | 用途 |
|------|----------|------|
| 从私服下载 | `settings.xml` → `<mirrors>` | 加速下载 |
| 上传到私服 | `pom.xml` → `<distributionManagement>` | 共享内部 jar |

---

## 5. 总结

| 知识点 | 核心要点 |
|--------|----------|
| Maven 是什么 | 项目构建 + 依赖管理工具 |
| 坐标（GAV） | `groupId` + `artifactId` + `version` 唯一标识项目 |
| 依赖范围 | `compile`/`provided`/`runtime`/`test` |
| 生命周期 | `clean` → `compile` → `test` → `package` → `install` → `deploy` |
| 依赖传递 | A→B→C，A 自动获得 C（排除用 `<exclusions>`） |
| JUnit | `@Test` + `assertEquals`/`assertThrows` |
| 分模块设计 | 按 pojo/mapper/service/web 拆分，职责清晰 |
| 继承 | `<parent>` + `<dependencyManagement>` 统一版本 |
| 聚合 | `<modules>` 一键构建所有子模块 |
| 私服 | Nexus，加速下载 + 内部 jar 共享 |
