# 02 - 前端基础（JS + Vue + Ajax）

## 1. JavaScript 核心语法

### 1.1 JS 引入方式

```html
<!-- 方式一：内部脚本 -->
<script>
    console.log("Hello, JS!");
</script>

<!-- 方式二：外部脚本（推荐） -->
<script src="app.js"></script>
```

### 1.2 变量与数据类型

```javascript
// 变量声明
var name = "张三";     // 旧式，函数作用域，有变量提升
let age = 18;          // 块级作用域，推荐
const PI = 3.14;       // 常量，不可重新赋值

// 数据类型
let num = 10;           // number
let str = "hello";      // string
let flag = true;        // boolean
let obj = null;         // null（空对象引用）
let x;                  // undefined（未赋值）
let arr = [1, 2, 3];    // object（数组）
let person = {          // object（对象）
    name: "张三",
    age: 20,
    sayHi: function() {
        console.log("Hi!");
    }
};

// 类型检测
console.log(typeof num);   // "number"
console.log(typeof arr);   // "object"
```

### 1.3 运算符（与 Java 基本一致）

```javascript
// 比较运算符 — ⚠️ 与 Java 不同！
console.log(10 == "10");   // true（值相等，类型转换）
console.log(10 === "10");  // false（全等，类型和值都相等）
// 推荐使用 === 和 !==

// 字符串模板（ES6）
let name = "张三";
console.log(`你好，${name}！`);  // "你好，张三！"
```

### 1.4 函数

```javascript
// 方式一：函数声明
function add(a, b) {
    return a + b;
}

// 方式二：函数表达式
const multiply = function(a, b) {
    return a * b;
};

// 方式三：箭头函数（ES6，常用）
const subtract = (a, b) => a - b;
const greet = name => `Hello, ${name}`;
```

### 1.5 数组操作

```javascript
let arr = [1, 2, 3, 4, 5];

// 遍历
arr.forEach((item, index) => console.log(index, item));

// 映射
let doubled = arr.map(x => x * 2);   // [2, 4, 6, 8, 10]

// 过滤
let evens = arr.filter(x => x % 2 === 0);  // [2, 4]

// 查找
let found = arr.find(x => x > 3);    // 4（第一个匹配）

// 累加
let sum = arr.reduce((acc, cur) => acc + cur, 0);  // 15

// 增删
arr.push(6);        // 末尾添加 → [1,2,3,4,5,6]
arr.pop();          // 末尾删除 → [1,2,3,4,5]
arr.splice(1, 2);   // 从索引1删2个 → [1,4,5]
```

---

## 2. DOM 操作与事件监听

### 2.1 获取 DOM 元素

```javascript
let el1 = document.getElementById("header");
let el2 = document.querySelector(".container");    // 第一个匹配
let list = document.querySelectorAll("p");           // 所有匹配（NodeList）
```

### 2.2 修改内容与样式

```javascript
let el = document.querySelector("p");
el.innerText = "新文本";
el.innerHTML = "<strong>粗体文本</strong>";
el.style.color = "red";
el.classList.add("active");
el.classList.toggle("dark");
```

### 2.3 事件监听

```javascript
// 方式一：属性绑定（不推荐）
<button onclick="handleClick()">点击</button>

// 方式二：addEventListener（推荐）
document.querySelector("button").addEventListener("click", function() {
    console.log("按钮被点击");
});

// 常见事件
click       // 鼠标点击
dblclick    // 双击
mouseover   // 鼠标移入
keydown     // 键盘按下
submit      // 表单提交
change      // 内容改变
input       // 实时输入
load        // 页面加载完成
DOMContentLoaded  // DOM 解析完成
```

---

## 3. Vue 核心语法

### 3.1 Vue 概述

Vue 是一套**渐进式**前端框架，用于构建用户界面。核心特点：**数据驱动视图**（数据变了，页面自动更新）。

### 3.2 入门程序

```html
<div id="app">
    <h1>{{ message }}</h1>           <!-- 插值表达式 -->
    <button @click="handleClick">点击</button>
</div>

<script>
    new Vue({
        el: "#app",                   // 挂载点
        data: {                       // 数据
            message: "Hello, Vue!"
        },
        methods: {                    // 方法
            handleClick() {
                this.message = "你点击了按钮";
            }
        }
    });
</script>
```

### 3.3 Vue 常用指令

| 指令 | 作用 | 示例 |
|------|------|------|
| `v-bind` / `:属性` | 动态绑定属性 | `:src="imgUrl"` `:class="{active: isActive}"` |
| `v-model` | 双向数据绑定 | `<input v-model="username">` |
| `v-if` / `v-else` | 条件渲染（DOM 增删） | `v-if="score >= 60"` |
| `v-show` | 条件显示（display 切换） | `v-show="isVisible"` |
| `v-for` | 列表渲染 | `v-for="item in list" :key="item.id"` |
| `v-on` / `@事件` | 事件绑定 | `@click="handleClick"` |
| `v-html` | 输出 HTML（防 XSS） | `v-html="rawHtml"` |

```html
<div id="app">
    <!-- v-model 双向绑定 -->
    <input v-model="name" placeholder="输入姓名">
    <p>你输入的是：{{ name }}</p>

    <!-- v-for 列表循环 -->
    <ul>
        <li v-for="(item, index) in todos" :key="item.id">
            {{ index + 1 }}. {{ item.text }}
        </li>
    </ul>

    <!-- v-if 条件渲染 -->
    <p v-if="score >= 90">优秀</p>
    <p v-else-if="score >= 60">及格</p>
    <p v-else>不及格</p>

    <!-- @click 事件绑定 -->
    <button @click="addTodo">添加</button>
</div>

<script>
    new Vue({
        el: "#app",
        data: {
            name: "",
            score: 85,
            todos: [
                { id: 1, text: "学习 HTML" },
                { id: 2, text: "学习 CSS" }
            ]
        },
        methods: {
            addTodo() {
                this.todos.push({
                    id: Date.now(),
                    text: this.name
                });
                this.name = "";
            }
        }
    });
</script>
```

---

## 4. Ajax 与 Axios

### 4.1 同步 vs 异步

```javascript
// 同步：代码一行行执行，等前一行结束才执行下一行
// 异步：发起请求后不等结果，继续执行，结果回来时回调

// JS 是单线程 + 异步非阻塞模型（Event Loop）
```

### 4.2 Axios 基本用法

Axios 是对 Ajax 的封装，基于 Promise，语法更简洁：

```javascript
// GET 请求
axios.get("/api/users", {
    params: { page: 1, size: 10 }
}).then(response => {
    console.log(response.data);  // 服务器返回的数据
}).catch(error => {
    console.error("请求失败:", error);
});

// POST 请求
axios.post("/api/users", {
    name: "张三",
    age: 18
}).then(response => {
    console.log("添加成功:", response.data);
});

// DELETE 请求
axios.delete(`/api/users/${userId}`);

// PUT 请求
axios.put(`/api/users/${userId}`, { name: "新名字" });
```

### 4.3 Vue + Axios 案例

```html
<div id="app">
    <button @click="loadUsers">加载用户</button>
    <ul>
        <li v-for="user in users" :key="user.id">
            {{ user.name }} — {{ user.age }}岁
            <button @click="deleteUser(user.id)">删除</button>
        </li>
    </ul>
</div>

<script>
    new Vue({
        el: "#app",
        data: {
            users: []
        },
        methods: {
            loadUsers() {
                axios.get("/api/users").then(res => {
                    this.users = res.data;
                });
            },
            deleteUser(id) {
                axios.delete(`/api/users/${id}`).then(() => {
                    this.users = this.users.filter(u => u.id !== id);
                });
            }
        }
    });
</script>
```

---

## 5. Vue 生命周期

```
创建阶段：
beforeCreate → created（数据初始化完成，可以发 Ajax）
                    │
挂载阶段：           ▼
beforeMount  → mounted（DOM 渲染完成，可以操作 DOM）
                    │
更新阶段：           ▼
beforeUpdate → updated（数据变化，DOM 重新渲染完成）
                    │
销毁阶段：           ▼
beforeDestroy → destroyed（清理定时器、事件监听等）
```

```javascript
new Vue({
    el: "#app",
    data: { ... },
    mounted() {
        // 页面加载后自动获取数据
        this.loadUsers();
    },
    methods: { ... }
});
```

---

## 6. 总结

| 知识点 | 核心要点 |
|--------|----------|
| JS 变量 | `let`(可变) `const`(常量) 替代 `var`，块级作用域 |
| `==` vs `===` | `==` 比较值（类型转换），`===` 比较值和类型，推荐 `===` |
| 箭头函数 | `(a, b) => a + b`，简洁，不绑定自己的 `this` |
| 数组方法 | `forEach` `map` `filter` `find` `reduce` |
| DOM 操作 | `querySelector` + `addEventListener` |
| Vue 核心 | 数据驱动视图，`data` + `methods` + 指令 |
| 常用指令 | `v-model` `v-bind` `v-if` `v-show` `v-for` `v-on`/@ |
| Axios | Promise 风格：`axios.get/post/put/delete().then().catch()` |
| Vue 生命周期 | `mounted`（发送 Ajax 的最佳时机） |
