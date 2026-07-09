# 05 - MySQL 数据库与多表查询

## 1. 数据库概述

### 1.1 基本概念

| 概念 | 说明 |
|------|------|
| **数据库（DB）** | 存储数据的仓库 |
| **数据库管理系统（DBMS）** | 操作和管理数据库的软件（MySQL、Oracle、PostgreSQL） |
| **SQL** | 结构化查询语言，操作关系型数据库的统一标准 |

### 1.2 MySQL 数据模型

```
MySQL 服务器
└── 数据库 (Database) — 一个项目对应一个库
    └── 表 (Table) — 一个实体对应一张表
        ├── 列 (Column) / 字段 — 实体的属性
        └── 行 (Row) / 记录 — 一条数据
```

---

## 2. SQL 语句

### 2.1 SQL 分类

| 分类 | 全称 | 作用 | 核心命令 |
|------|------|------|----------|
| **DDL** | Data Definition Language | 定义数据库/表结构 | `CREATE` `ALTER` `DROP` |
| **DML** | Data Manipulation Language | 操作数据 | `INSERT` `UPDATE` `DELETE` |
| **DQL** | Data Query Language | 查询数据 | `SELECT` |
| **DCL** | Data Control Language | 权限控制 | `GRANT` `REVOKE` |

### 2.2 DDL — 表操作

```sql
-- 创建表
CREATE TABLE tb_emp (
    id        INT PRIMARY KEY AUTO_INCREMENT COMMENT '主键ID',
    username  VARCHAR(20) NOT NULL UNIQUE COMMENT '用户名',
    name      VARCHAR(10) NOT NULL COMMENT '姓名',
    gender    TINYINT(1)  DEFAULT 1 COMMENT '性别 1:男 2:女',
    salary    DECIMAL(10,2) COMMENT '薪资',
    entrydate DATE COMMENT '入职日期',
    dept_id   INT COMMENT '所属部门ID',
    create_time DATETIME DEFAULT CURRENT_TIMESTAMP,
    update_time DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) COMMENT '员工表';

-- 修改表
ALTER TABLE tb_emp ADD COLUMN email VARCHAR(50);
ALTER TABLE tb_emp MODIFY COLUMN username VARCHAR(50);
ALTER TABLE tb_emp DROP COLUMN email;

-- 删除表
DROP TABLE IF EXISTS tb_emp;
```

**常用数据类型**：

| 类型 | 说明 | 示例 |
|------|------|------|
| `INT` | 整数 | `age INT` |
| `BIGINT` | 长整数 | `id BIGINT` |
| `DECIMAL(M,D)` | 定点小数，M=总位数，D=小数位 | `salary DECIMAL(10,2)` |
| `VARCHAR(N)` | 可变长字符串 | `name VARCHAR(20)` |
| `TEXT` | 长文本 | `content TEXT` |
| `DATE` | 日期 | `birthday DATE` |
| `DATETIME` | 日期+时间 | `create_time DATETIME` |
| `TINYINT` | 小整数，常做布尔/枚举 | `gender TINYINT(1)` |

### 2.3 DML — 增删改

```sql
-- 插入
INSERT INTO tb_emp (username, name, gender, dept_id)
VALUES ('zhangsan', '张三', 1, 2);

-- 批量插入
INSERT INTO tb_emp (username, name, gender) VALUES
('lisi', '李四', 2),
('wangwu', '王五', 1);

-- 更新
UPDATE tb_emp SET salary = 8000, dept_id = 3 WHERE id = 1;
-- ⚠️ 不加 WHERE 会更新全表！

-- 删除
DELETE FROM tb_emp WHERE id = 5;
-- ⚠️ DELETE 不加 WHERE 会删除全表！
```

### 2.4 DQL — 查询（核心）

```sql
-- 基本语法顺序
SELECT   [DISTINCT] 字段列表
FROM     表名
WHERE    条件
GROUP BY 分组字段
HAVING   分组后条件
ORDER BY 排序字段 [ASC|DESC]
LIMIT    分页参数;

-- 条件查询
SELECT * FROM tb_emp WHERE salary > 5000 AND dept_id = 2;
SELECT * FROM tb_emp WHERE name LIKE '张%';      -- 姓张的
SELECT * FROM tb_emp WHERE name LIKE '%三';       -- 名三的
SELECT * FROM tb_emp WHERE salary BETWEEN 5000 AND 10000;
SELECT * FROM tb_emp WHERE dept_id IN (1, 2, 3);

-- 聚合函数
SELECT COUNT(*) FROM tb_emp;            -- 总记录数
SELECT AVG(salary) FROM tb_emp;         -- 平均薪资
SELECT MAX(salary), MIN(salary) FROM tb_emp;
SELECT SUM(salary) FROM tb_emp;         -- 薪资总和

-- 分组查询
SELECT dept_id, AVG(salary) FROM tb_emp
GROUP BY dept_id
HAVING AVG(salary) > 6000;

-- 排序：ORDER BY 默认 ASC（升序），DESC 降序
SELECT * FROM tb_emp ORDER BY salary DESC, entrydate ASC;

-- 分页查询 LIMIT 起始索引, 每页条数
-- 第1页，每页10条
SELECT * FROM tb_emp LIMIT 0, 10;
-- 第2页：LIMIT (page-1)*size, size
SELECT * FROM tb_emp LIMIT 10, 10;
```

---

## 3. 多表关系

### 3.1 三种关系

```
一对多：  部门 1 ──→ N 员工
         (dept_id 做外键，多方持有一方的主键)

一对一：  用户 1 ──→ 1 用户详情
         (任意一方存另一方的唯一外键 UNIQUE)

多对多：  学生 M ──→ N 课程
         (需要中间表：student_course，存两个外键)
```

### 3.2 外键约束

```sql
-- 创建表时加外键
CREATE TABLE tb_emp (
    ...
    dept_id INT,
    CONSTRAINT fk_emp_dept FOREIGN KEY (dept_id) REFERENCES tb_dept(id)
);
```

> 外键保证数据完整性：员工不能属于不存在的部门，有员工的部门不能直接删。

---

## 4. 多表查询

### 4.1 内连接（取交集）

```sql
-- 隐式内连接
SELECT e.name, d.name AS dept_name
FROM tb_emp e, tb_dept d
WHERE e.dept_id = d.id;

-- 显式内连接（推荐）
SELECT e.name, d.name AS dept_name
FROM tb_emp e
INNER JOIN tb_dept d ON e.dept_id = d.id;
```

### 4.2 外连接

```sql
-- 左外连接：左边的表全显示，右边匹配不上的填 NULL
SELECT e.name, d.name
FROM tb_emp e
LEFT JOIN tb_dept d ON e.dept_id = d.id;

-- 右外连接：右边的表全显示
SELECT e.name, d.name
FROM tb_emp e
RIGHT JOIN tb_dept d ON e.dept_id = d.id;
```

### 4.3 子查询

```sql
-- 标量子查询（结果：单个值）
SELECT * FROM tb_emp WHERE salary > (SELECT AVG(salary) FROM tb_emp);

-- 列子查询（结果：一列）配合 IN/ANY/ALL
SELECT * FROM tb_emp WHERE dept_id IN (
    SELECT id FROM tb_dept WHERE name IN ('技术部', '市场部')
);

-- 行子查询（结果：一行多列）
SELECT * FROM tb_emp WHERE (salary, dept_id) = (
    SELECT MAX(salary), dept_id FROM tb_emp WHERE dept_id = 2
);

-- 表子查询（结果：多行多列）通常作为临时表再 JOIN
SELECT e.name, t.avg_salary
FROM tb_emp e
JOIN (SELECT dept_id, AVG(salary) AS avg_salary FROM tb_emp GROUP BY dept_id) t
  ON e.dept_id = t.dept_id;
```

---

## 5. 总结

| 知识点 | 核心要点 |
|--------|----------|
| DDL | `CREATE TABLE` `ALTER TABLE` `DROP TABLE` |
| DML | `INSERT INTO` `UPDATE ... SET ... WHERE` `DELETE FROM ... WHERE` |
| DQL | `SELECT ... FROM ... WHERE ... GROUP BY ... ORDER BY ... LIMIT` |
| 聚合函数 | `COUNT` `SUM` `AVG` `MAX` `MIN`，不能跟在 WHERE 后（用 HAVING） |
| 分页 | `LIMIT (page-1)*size, size` |
| 一对多 | 多方加外键指向一方主键 |
| 多对多 | 中间表存两个外键 |
| 内连接 | `INNER JOIN ... ON`，取交集 |
| 外连接 | `LEFT JOIN`（左全显示）`RIGHT JOIN`（右全显示） |
| 子查询 | 标量/列/行/表子查询，结果可嵌套在 WHERE/FROM/SELECT 中 |
