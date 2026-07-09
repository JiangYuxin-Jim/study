# 06 - JDBC 与 MyBatis

## 1. JDBC 基础

### 1.1 什么是 JDBC？

JDBC（Java Database Connectivity）是 Java 操作数据库的一套**标准接口**，各数据库厂商提供对应的**驱动实现**。

```
Java 程序 ──→ JDBC API ──→ MySQL Driver ──→ MySQL 数据库
                            (mysql-connector-java)
```

### 1.2 JDBC 查询流程

```java
// 1. 注册驱动（JDBC 4.0+ 自动注册，可省略）
// Class.forName("com.mysql.cj.jdbc.Driver");

// 2. 获取连接
String url = "jdbc:mysql://localhost:3306/tlias?useSSL=false&serverTimezone=Asia/Shanghai";
Connection conn = DriverManager.getConnection(url, "root", "password");

// 3. 创建 Statement
PreparedStatement pstmt = conn.prepareStatement("SELECT * FROM tb_emp WHERE id = ?");
pstmt.setLong(1, 1L);

// 4. 执行查询
ResultSet rs = pstmt.executeQuery();

// 5. 解析结果
while (rs.next()) {
    Long id = rs.getLong("id");
    String name = rs.getString("name");
    // 封装成对象...
}

// 6. 释放资源（逆序关闭）
rs.close();
pstmt.close();
conn.close();
```

### 1.3 PreparedStatement vs Statement

| | Statement | PreparedStatement |
|------|-----------|-------------------|
| **SQL 注入** | ❌ 拼接字符串，有注入风险 | ✅ 预编译，`?` 占位，防注入 |
| **性能** | 每次编译 SQL | 预编译一次，可重复使用 |
| **推荐** | ❌ | ✅ 始终使用 |

### 1.4 JDBC 的痛点

```java
// 大量重复的模板代码：获取连接 → 预编译 → 设参数 → 执行 → 解析 → 关资源
// 每个查询都要写一遍，枯燥且易出错
// → MyBatis 解决了这些问题
```

---

## 2. MyBatis 入门

### 2.1 MyBatis 是什么？

MyBatis 是一个**持久层框架**，封装了 JDBC 的重复操作，让开发者只需关注 SQL 本身。

```
SpringBoot ──→ MyBatis ──→ JDBC ──→ MySQL

开发者只需要写：
1. Mapper 接口（Java 方法）
2. SQL 语句（注解 / XML）
MyBatis 负责中间的映射转换。
```

### 2.2 快速入门

**1. 引入依赖**：

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.3.0</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

**2. 配置数据源**（application.properties）：

```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/tlias
spring.datasource.username=root
spring.datasource.password=123456
```

**3. 定义实体类**：

```java
@Data
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

**4. 编写 Mapper 接口**：

```java
@Mapper  // MyBatis 会自动生成代理实现类
public interface UserMapper {

    @Select("SELECT * FROM tb_user WHERE id = #{id}")
    User findById(Long id);

    @Select("SELECT * FROM tb_user")
    List<User> findAll();

    @Insert("INSERT INTO tb_user(name, age, email) VALUES(#{name}, #{age}, #{email})")
    @Options(useGeneratedKeys = true, keyProperty = "id")  // 回填自增ID
    void insert(User user);

    @Update("UPDATE tb_user SET name=#{name}, age=#{age}, email=#{email} WHERE id=#{id}")
    void update(User user);

    @Delete("DELETE FROM tb_user WHERE id=#{id}")
    void deleteById(Long id);
}
```

**5. 使用**：

```java
@Service
public class UserServiceImpl implements UserService {

    @Autowired
    private UserMapper userMapper;

    public User getById(Long id) {
        return userMapper.findById(id);
    }
}
```

### 2.3 参数占位符

| 占位符 | 说明 | SQL 注入 |
|--------|------|:--:|
| `#{参数名}` | **预编译**占位符，参数会变成 `?`，安全 | ✅ 安全 |
| `${参数名}` | 直接拼接字符串（有注入风险） | ❌ 危险 |

```java
// #{name} → SELECT * FROM tb_user WHERE name = ?  → 安全
// ${name} → SELECT * FROM tb_user WHERE name = 张三  → 危险，且字符串要加引号

// ${} 唯一合法用途：动态表名/列名/排序字段
@Select("SELECT * FROM ${tableName} ORDER BY ${orderBy}")
// 此时需要确保 tableName 和 orderBy 的值不被外部直接传入
```

### 2.4 驼峰命名映射

```properties
# 数据库字段：dept_id, create_time
# Java 属性：deptId, createTime
# 开启驼峰映射，MyBatis 自动转换
mybatis.configuration.map-underscore-to-camel-case=true
```

### 2.5 XML 映射文件（复杂 SQL 场景）

当 SQL 比较复杂（多表关联、动态条件），不适合用注解时，使用 XML 映射：

```xml
<!-- resources/mapper/UserMapper.xml -->
<mapper namespace="com.heima.mapper.UserMapper">

    <select id="findByCondition" resultType="com.heima.pojo.User">
        SELECT * FROM tb_user
        <where>
            <if test="name != null and name != ''">
                AND name LIKE CONCAT('%', #{name}, '%')
            </if>
            <if test="age != null">
                AND age = #{age}
            </if>
        </where>
        ORDER BY id DESC
    </select>

</mapper>
```

**动态 SQL 标签**：

| 标签 | 用途 |
|------|------|
| `<if>` | 条件判断 |
| `<where>` | 自动处理 `WHERE` 和多余的 `AND`/`OR` |
| `<set>` | UPDATE 时自动处理 `SET` 和多余的逗号 |
| `<foreach>` | 遍历集合，常见于 `IN (?,?,?)` |
| `<sql>` + `<include>` | SQL 片段复用 |

```xml
<!-- foreach 示例 -->
<delete id="deleteByIds">
    DELETE FROM tb_emp WHERE id IN
    <foreach collection="ids" item="id" open="(" separator="," close=")">
        #{id}
    </foreach>
</delete>

<!-- sql 复用 -->
<sql id="baseColumns">id, name, age, email</sql>
<select id="findById" resultType="User">
    SELECT <include refid="baseColumns"/> FROM tb_user WHERE id = #{id}
</select>
```

---

## 3. SpringBoot 多环境配置

```properties
# application.properties — 公共配置
spring.profiles.active=dev
```

```properties
# application-dev.properties — 开发环境
server.port=8080
spring.datasource.url=jdbc:mysql://localhost:3306/tlias_dev

# application-prod.properties — 生产环境
server.port=80
spring.datasource.url=jdbc:mysql://prod-server:3306/tlias
```

> 通过 `spring.profiles.active` 切换环境，打包部署时无需改代码。

---

## 4. 总结

| 知识点 | 核心要点 |
|--------|----------|
| JDBC | Java 操作数据库的标准 API，但模板代码多 |
| MyBatis | 持久层框架，封装 JDBC，写 SQL 即可 |
| `@Mapper` | MyBatis 自动生成接口的代理实现类 |
| `#{ }` vs `${ }` | `#{}` 预编译防注入，`${}` 字符串拼接（仅用于动态表名） |
| 驼峰映射 | `map-underscore-to-camel-case=true` |
| 动态 SQL | `<if>` `<where>` `<set>` `<foreach>` |
| 多环境 | `application-{profile}.properties` + `spring.profiles.active` |
