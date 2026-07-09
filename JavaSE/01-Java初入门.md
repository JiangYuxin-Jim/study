# 01 - Java 初入门

## 1. Java 语言概述

### 1.1 Java 是什么？

Java 是由 Sun Microsystems（后被 Oracle 收购）于 1995 年推出的面向对象编程语言。

| 特性 | 说明 |
|------|------|
| **跨平台** | "Write Once, Run Anywhere" — 编译成字节码，由 JVM 执行 |
| **面向对象** | 封装、继承、多态 |
| **自动内存管理** | JVM 垃圾回收（GC），无需手动 free |
| **健壮** | 强类型 + 异常处理 + 编译期检查 |
| **多线程** | 内置多线程支持 |

### 1.2 Java 三大体系

```
Java 平台
├── Java SE (Standard Edition)    ← 基础，桌面应用
├── Java EE (Enterprise Edition)  ← 企业级，Web 应用（现称 Jakarta EE）
└── Java ME (Micro Edition)       ← 嵌入式，移动设备（已边缘化）
```

---

## 2. JDK / JRE / JVM

### 2.1 三者关系

```
┌─────────────── JDK (Java Development Kit) ───────────────┐
│  ┌─────────── JRE (Java Runtime Environment) ──────────┐ │
│  │  ┌─────────── JVM (Java Virtual Machine) ────────┐  │ │
│  │  │                                                │  │ │
│  │  │  类加载器 → 字节码校验 → 解释执行 / JIT 编译    │  │ │
│  │  │                                                │  │ │
│  │  └────────────────────────────────────────────────┘  │ │
│  │  核心类库 (rt.jar / java.base)                        │ │
│  └──────────────────────────────────────────────────────┘ │
│  开发工具 (javac, java, jar, javadoc, jdb...)             │
└──────────────────────────────────────────────────────────┘
```

| 缩写 | 全称 | 作用 | 谁需要 |
|------|------|------|--------|
| **JDK** | Java Development Kit | 开发工具包，包含 JRE + 编译器等工具 | 开发者 |
| **JRE** | Java Runtime Environment | 运行环境，包含 JVM + 核心类库 | 用户 |
| **JVM** | Java Virtual Machine | 执行字节码，屏蔽底层操作系统差异 | 所有人 |

> JDK 包含 JRE，JRE 包含 JVM。开发装 JDK，用户装 JRE 即可。

### 2.2 跨平台原理

```
Hello.java ──→ [javac 编译] ──→ Hello.class (字节码，与平台无关)
                                       │
                    ┌──────────────────┼──────────────────┐
                    │                  │                  │
                    ▼                  ▼                  ▼
              ┌──────────┐     ┌──────────┐      ┌──────────┐
              │ JVM(Win) │     │ JVM(Linux)│     │ JVM(Mac) │
              └──────────┘     └──────────┘      └──────────┘
              Windows            Linux              macOS
```

**核心**：不同平台安装对应的 JVM 即可执行同一份 `.class` 文件，Java 程序不直接和 OS 打交道。

---

## 3. 环境搭建

### 3.1 JDK 安装

1. 官网下载 JDK（课程使用 JDK 8 / 11 / 17）
2. 安装到指定目录，如 `C:\Program Files\Java\jdk-17`

### 3.2 环境变量配置

```bash
# JAVA_HOME — JDK 安装目录
JAVA_HOME = C:\Program Files\Java\jdk-17

# Path — 追加 JDK bin 目录
Path += %JAVA_HOME%\bin

# 验证安装
java -version
javac -version
```

### 3.3 JAVA_HOME 的作用

- `javac`、`java` 等命令存放在 `%JAVA_HOME%\bin`
- Tomcat、Maven、Gradle 等工具依赖 `JAVA_HOME` 查找 JDK
- 升级 JDK 时只需改 `JAVA_HOME` 变量值

---

## 4. HelloWorld — 第一个 Java 程序

### 4.1 编写源码

```java
// HelloWorld.java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

### 4.2 编译与运行

```bash
# 编译：生成 HelloWorld.class
javac HelloWorld.java

# 运行：JVM 执行字节码
java HelloWorld

# 输出：
# Hello, World!
```

### 4.3 代码解读

| 要素 | 说明 |
|------|------|
| `public class HelloWorld` | 定义一个类，**类名 = 文件名** |
| `public static void main(String[] args)` | 程序入口，**固定格式** |
| `System.out.println(...)` | 控制台输出并换行 |

### 4.4 常见错误

| 错误 | 原因 |
|------|------|
| `类名与文件名不一致` | 编译报错，必须一致（public class） |
| `找不到 main 方法` | 格式写错或遗漏，JVM 找不到入口 |
| `javac 不是内部命令` | 环境变量 Path 未配置 |
| `大小写错误` | `String` ≠ `string`，`System` ≠ `system`，Java 区分大小写 |

---

## 5. 注释

```java
// 1. 单行注释
// 这是一条注释

// 2. 多行注释
/*
 * 这是多行注释
 * 可以跨越多行
 */

// 3. 文档注释（可被 javadoc 提取生成 API 文档）
/**
 * 计算两个整数的和
 * @param a 第一个整数
 * @param b 第二个整数
 * @return  两数之和
 */
public int add(int a, int b) {
    return a + b;
}
```

---

## 6. 关键字与标识符

### 6.1 关键字

Java 中具有特殊含义的单词，约 50+ 个，**不能作为标识符**：

```
数据类型：byte short int long float double char boolean
流程控制：if else switch case for while do break continue return
访问权限：public protected private
修饰符：  static final abstract synchronized volatile
异常：    try catch finally throw throws
类相关：  class interface extends implements package import new
```

### 6.2 标识符命名规则

| 规则 | 示例 |
|------|------|
| 由字母、数字、下划线 `_`、美元符 `$` 组成 | ✅ `userName` ✅ `_count` ✅ `$value` |
| 不能以数字开头 | ❌ `1name` |
| 不能是关键字 | ❌ `class` ❌ `public` |
| 区分大小写 | `name` ≠ `Name` |

### 6.3 命名规范（约定俗成）

| 类型 | 规范 | 示例 |
|------|------|------|
| 类名 | 大驼峰（PascalCase） | `HelloWorld`、`StudentManager` |
| 方法/变量名 | 小驼峰（camelCase） | `userName`、`getAge()` |
| 常量 | 全大写 + 下划线 | `MAX_VALUE`、`DEFAULT_SIZE` |
| 包名 | 全小写，域名倒写 | `com.heima.study` |

---

## 7. 总结

| 知识点 | 核心要点 |
|--------|----------|
| Java 跨平台 | 编译成 `.class` 字节码，由各平台 JVM 执行 |
| JDK vs JRE vs JVM | JDK ⊃ JRE ⊃ JVM；开发装 JDK，运行装 JRE |
| 环境变量 | `JAVA_HOME` 指向 JDK 目录，`Path` 追加 `bin` |
| HelloWorld | `public class` + `main` 方法 + `System.out.println` |
| 编译运行 | `javac` 编译 → `.class` 文件，`java` 命令运行 |
| 注释 | 单行 `//`、多行 `/* */`、文档 `/** */` |
| 关键字 | 50+ 保留词，不能作为标识符 |
| 命名规范 | 类用大驼峰，方法/变量用小驼峰，常量全大写 |
