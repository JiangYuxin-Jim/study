# 07 - 部门管理 CRUD 实战

## 1. 开发规范

### 1.1 前后端分离开发

```
┌─────────┐    HTTP/JSON    ┌─────────┐
│  前端    │ ←─────────────→ │  后端    │
│ (Vue)   │    REST API     │(Spring)  │
└─────────┘                 └─────────┘

前端和后端是两个独立的项目，通过 HTTP 接口通信。
```

**接口文档（YApi / Swagger）**：前后端开发的**共同约定**

| 项目 | 说明 |
|------|------|
| 请求路径 | `/depts` |
| 请求方式 | GET / POST / PUT / DELETE |
| 请求参数 | 类型、是否必填、示例 |
| 响应格式 | JSON 结构、状态码 |

### 1.2 统一响应格式

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Result {
    private Integer code;    // 1 成功, 0 失败
    private String msg;      // 提示信息
    private Object data;     // 返回数据

    public static Result success() {
        return new Result(1, "success", null);
    }
    public static Result success(Object data) {
        return new Result(1, "success", data);
    }
    public static Result error(String msg) {
        return new Result(0, msg, null);
    }
}
```

---

## 2. 查询部门

### 2.1 接口

```
GET /depts → [ {id, name, createTime}, ... ]
```

### 2.2 实现

```java
// Controller
@RestController
@RequestMapping("/depts")
public class DeptController {

    @Autowired
    private DeptService deptService;

    @GetMapping
    public Result list() {
        List<Dept> list = deptService.list();
        return Result.success(list);
    }
}

// Service
@Service
public class DeptServiceImpl implements DeptService {

    @Autowired
    private DeptMapper deptMapper;

    @Override
    public List<Dept> list() {
        return deptMapper.list();
    }
}

// Mapper
@Mapper
public interface DeptMapper {

    @Select("SELECT * FROM tb_dept ORDER BY id")
    List<Dept> list();
}
```

### 2.3 前后端联调

后端启动 SpringBoot（默认 8080），前端启动开发服务器（如 5173），通过代理或 CORS 解决跨域。

---

## 3. 删除部门

```java
// Controller
@DeleteMapping("/{id}")
public Result delete(@PathVariable Integer id) {
    deptService.delete(id);
    return Result.success();
}

// Service
@Override
public void delete(Integer id) {
    deptMapper.deleteById(id);
}

// Mapper
@Delete("DELETE FROM tb_dept WHERE id = #{id}")
void deleteById(Integer id);
```

---

## 4. 新增部门

```java
// Controller
@PostMapping
public Result add(@RequestBody Dept dept) {
    deptService.add(dept);
    return Result.success();
}

// Service
@Override
public void add(Dept dept) {
    dept.setCreateTime(LocalDateTime.now());
    deptMapper.insert(dept);
}

// Mapper
@Insert("INSERT INTO tb_dept(name, create_time) VALUES(#{name}, #{createTime})")
void insert(Dept dept);
```

---

## 5. 修改部门

```java
// ① 查询回显（打开修改弹窗时）
@GetMapping("/{id}")
public Result getById(@PathVariable Integer id) {
    Dept dept = deptService.getById(id);
    return Result.success(dept);
}

// ② 提交修改
@PutMapping
public Result update(@RequestBody Dept dept) {
    deptService.update(dept);
    return Result.success();
}

// Mapper
@Select("SELECT * FROM tb_dept WHERE id = #{id}")
Dept getById(Integer id);

@Update("UPDATE tb_dept SET name=#{name} WHERE id=#{id}")
void update(Dept dept);
```

---

## 6. 日志技术 — Logback

### 6.1 为什么需要日志？

- `System.out.println()` 只在控制台输出，不能持久化
- 无法按级别过滤（开发时想看 DEBUG，线上只看 ERROR）
- 无法按日期、大小自动切割日志文件

### 6.2 Logback 使用

```java
// 1. 创建 Logger
private static final Logger log = LoggerFactory.getLogger(DeptController.class);

// 2. 使用（不同级别）
@GetMapping
public Result list() {
    log.info("查询所有部门");
    log.debug("调试信息：{}", someVar);
    log.error("查询部门出错", exception);
    return Result.success(deptService.list());
}
```

### 6.3 日志级别（从低到高）

```
TRACE < DEBUG < INFO < WARN < ERROR

配置 INFO → 只输出 INFO、WARN、ERROR
配置 DEBUG → 输出 DEBUG、INFO、WARN、ERROR
```

### 6.4 配置文件

```xml
<!-- logback-spring.xml -->
<configuration>
    <!-- 控制台输出 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 文件输出（按日期滚动） -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/tlias.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/tlias.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>  <!-- 保留30天 -->
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>

    <!-- 特定包设置不同级别 -->
    <logger name="com.heima.mapper" level="DEBUG"/>
</configuration>
```

---

## 7. 总结

| 知识点 | 核心要点 |
|--------|----------|
| 前后端分离 | 两个独立工程，通过 REST API + JSON 通信 |
| 统一响应 | `Result {code, msg, data}`，成功 code=1，失败 code=0 |
| CRUD 四步 | Controller 接收请求 → Service 处理逻辑 → Mapper 执行 SQL → 返回 Result |
| 接口规范 | `GET /depts` `GET /depts/{id}` `POST /depts` `PUT /depts` `DELETE /depts/{id}` |
| Logback | 日志框架，替代 `System.out`，支持级别过滤 + 文件滚动 |
| 日志级别 | TRACE < DEBUG < INFO < WARN < ERROR |
