# 01 - 前端基础（HTML + CSS）

## 1. Web 前端概述

### 1.1 网页的组成

- **HTML**：负责网页的**结构**（页面元素和内容）
- **CSS**：负责网页的**表现**（颜色、大小、位置等样式）
- **JavaScript**：负责网页的**行为**（交互效果）

### 1.2 Web 标准

由 W3C（万维网联盟）制定，确保不同浏览器解析出相同的效果：

```
┌─────────────────────────────────────────┐
│               Web 标准                    │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ │
│  │  HTML    │ │   CSS    │ │    JS    │ │
│  │  结构     │ │   表现    │ │   行为    │ │
│  └──────────┘ └──────────┘ └──────────┘ │
└─────────────────────────────────────────┘
```

### 1.3 浏览器内核

不同浏览器有不同的内核（渲染引擎），对相同代码解析效果可能不同 — Web 标准正是为了解决这个问题。

| 浏览器 | 内核 |
|--------|------|
| Chrome / Edge | Blink |
| Firefox | Gecko |
| Safari | WebKit |

---

## 2. HTML 快速入门

### 2.1 HTML 基本结构

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>网页标题</title>
</head>
<body>
    <h1>Hello, HTML!</h1>
    <p>这是一个段落。</p>
</body>
</html>
```

| 标签 | 含义 |
|------|------|
| `<!DOCTYPE html>` | 声明文档类型为 HTML5 |
| `<html>` | 根标签 |
| `<head>` | 头部信息（编码、标题、样式引入等） |
| `<body>` | 可见内容 |

### 2.2 常见标签

**文本标签**：

```html
<h1>一级标题</h1>  <!-- h1~h6，数字越小字越大 -->
<p>段落文本</p>
<span>行内文本</span>
<br>  <!-- 换行（自闭合） -->
<hr>  <!-- 水平线 -->
<strong>加粗（语义）</strong>
<b>加粗（样式）</b>
<em>斜体（语义）</em>
```

**图片/链接/列表**：

```html
<!-- 图片 -->
<img src="image.jpg" alt="图片描述" width="200">

<!-- 超链接 -->
<a href="https://example.com" target="_blank">打开新页面</a>

<!-- 列表 -->
<ul>                          <!-- 无序列表 -->
    <li>项目一</li>
    <li>项目二</li>
</ul>
<ol>                          <!-- 有序列表 -->
    <li>第一步</li>
    <li>第二步</li>
</ol>
```

**表格**：

```html
<table border="1">
    <tr>                     <!-- 行 -->
        <th>姓名</th>         <!-- 表头 -->
        <th>年龄</th>
    </tr>
    <tr>
        <td>张三</td>         <!-- 单元格 -->
        <td>25</td>
    </tr>
</table>
```

**表单**：

```html
<form action="/submit" method="post">
    <label>用户名：</label>
    <input type="text" name="username" placeholder="请输入用户名">

    <label>密码：</label>
    <input type="password" name="password">

    <label>性别：</label>
    <input type="radio" name="gender" value="male"> 男
    <input type="radio" name="gender" value="female"> 女

    <input type="checkbox" name="hobby" value="read"> 阅读

    <select name="city">
        <option value="beijing">北京</option>
        <option value="shanghai">上海</option>
    </select>

    <button type="submit">提交</button>
</form>
```

### 2.3 块级元素 vs 行内元素

| | 块级元素 | 行内元素 |
|------|---------|---------|
| **特点** | 独占一行，可设宽高 | 不独占一行，宽高由内容决定 |
| **代表** | `div` `h1~h6` `p` `table` `ul` `ol` `form` | `span` `a` `img` `strong` `input` |
| **转换** | `display: block` | `display: inline` / `inline-block` |

---

## 3. CSS 快速入门

### 3.1 CSS 引入方式

```html
<!-- 方式一：行内样式（不推荐，无法复用） -->
<p style="color: red;">红色文字</p>

<!-- 方式二：内部样式（写在 <style> 标签中） -->
<style>
    p { color: blue; font-size: 16px; }
</style>

<!-- 方式三：外部样式（推荐，HTML/CSS 分离） -->
<link rel="stylesheet" href="style.css">
```

### 3.2 选择器

```css
/* 1. 标签选择器 — 选中所有同类标签 */
p { color: red; }

/* 2. 类选择器 — 最常用，可复用 */
.title { font-size: 24px; font-weight: bold; }

/* 3. ID 选择器 — 唯一，一个页面中 ID 不重复 */
#header { background: #333; }

/* 4. 后代选择器 — 选中某元素下的所有子孙 */
.container p { line-height: 1.5; }

/* 5. 子代选择器 — 只选直接子元素 */
.container > p { margin: 0; }

/* 6. 并集选择器 — 同时选中多个 */
h1, h2, h3 { font-family: sans-serif; }

/* 7. 伪类选择器 — 特殊状态 */
a:hover { color: orange; }        /* 鼠标悬停 */
li:nth-child(odd) { background: #f0f0f0; }  /* 奇数行 */
```

### 3.3 常用样式属性

```css
/* 文字样式 */
p {
    color: #333;           /* 颜色 */
    font-size: 16px;       /* 字号 */
    font-weight: bold;     /* 粗细 */
    font-family: "微软雅黑", sans-serif;  /* 字体 */
    text-align: center;    /* 对齐：left/center/right */
    text-decoration: none; /* 下划线：none/underline */
    line-height: 1.8;      /* 行高 */
}

/* 背景 */
.box {
    background-color: #f5f5f5;
    background-image: url("bg.jpg");
    background-size: cover;    /* 铺满 / contain */
    background-position: center;
}

/* 尺寸 */
div {
    width: 200px;
    height: 100px;
    max-width: 100%;     /* 响应式 */
}
```

### 3.4 盒子模型

```
┌───────────────────────────────┐
│          margin (外边距)        │
│   ┌─────────────────────────┐ │
│   │      border (边框)       │ │
│   │   ┌─────────────────┐   │ │
│   │   │  padding (内边距) │   │ │
│   │   │   ┌─────────┐   │   │ │
│   │   │   │ content  │   │   │ │
│   │   │   │  (内容)   │   │   │ │
│   │   │   └─────────┘   │   │ │
│   │   └─────────────────┘   │ │
│   └─────────────────────────┘ │
└───────────────────────────────┘
```

```css
.box {
    width: 200px;           /* 内容宽度 */
    padding: 10px;          /* 内边距 */
    border: 2px solid #333; /* 边框 */
    margin: 20px;           /* 外边距 */
    /* 实际占用宽度 = 200 + 10×2 + 2×2 + 20×2 = 264px */
}

/* 简化：使用 box-sizing */
.box2 {
    box-sizing: border-box; /* width 包含 padding 和 border */
    width: 200px;           /* 实际占用 = 200 + 20×2 = 240px */
}
```

### 3.5 布局 — Flexbox

```css
.container {
    display: flex;
    justify-content: space-between;  /* 主轴对齐：center/flex-start/space-around */
    align-items: center;             /* 交叉轴对齐 */
    flex-wrap: wrap;                 /* 允许换行 */
}

.item {
    flex: 1;  /* 等分剩余空间 */
}
```

---

## 4. 总结

| 知识点 | 核心要点 |
|--------|----------|
| HTML 结构 | `html > head + body`，标签成对出现 |
| 常用标签 | `h1~h6` `p` `div` `span` `a` `img` `table` `form` `input` |
| 块级 vs 行内 | 块独占一行可设宽高（`div`），行内不独占（`span`） |
| CSS 引入 | 外部样式表（`link`）最佳，行内样式最差 |
| 选择器 | 标签/类/ID/后代/子代/并集/伪类，类选择器最常用 |
| 盒子模型 | content + padding + border + margin；`box-sizing: border-box` |
| Flexbox | `display: flex` + `justify-content` + `align-items` |
