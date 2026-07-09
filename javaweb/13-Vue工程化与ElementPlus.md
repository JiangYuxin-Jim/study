# 13 - Vue 工程化与 ElementPlus

## 1. Vue 工程化

### 1.1 为什么要工程化？

之前在 HTML 里直接写 Vue 的方式只适合小练习。工程化开发才能支持大型项目：

| 工程化能力 | 工具 |
|-----------|------|
| **模块化** | ES6 import/export，组件拆分 |
| **打包构建** | Vite / Webpack |
| **开发服务器** | 热更新（改代码页面自动刷新） |
| **代码规范** | ESLint + Prettier |

### 1.2 创建 Vue 工程

```bash
# 使用 Vite 创建项目
npm create vite@latest tlias-frontend -- --template vue

cd tlias-frontend
npm install
npm run dev       # 启动开发服务器 → http://localhost:5173
```

**工程目录结构**：

```
tlias-frontend/
├── index.html              # 入口 HTML
├── package.json            # 项目依赖
├── vite.config.js          # Vite 配置
├── src/
│   ├── main.js             # 应用入口
│   ├── App.vue             # 根组件
│   ├── views/              # 页面级组件
│   ├── components/         # 可复用组件
│   ├── api/                # API 请求封装
│   ├── router/             # 路由配置
│   └── assets/             # 静态资源
└── public/                 # 公共静态文件
```

### 1.3 .vue 单文件组件 (SFC)

```vue
<template>
    <!-- HTML 模板 -->
    <div class="container">
        <h1>{{ title }}</h1>
        <button @click="handleClick">点击</button>
    </div>
</template>

<script setup>
// JavaScript 逻辑（Composition API）
import { ref } from 'vue';

const title = ref('Hello, Vue!');

function handleClick() {
    title.value = '你点击了按钮';
}
</script>

<style scoped>
/* 局部样式（scoped = 只作用于当前组件） */
.container {
    padding: 20px;
}
</style>
```

### 1.4 Vue Router

```javascript
// router/index.js
import { createRouter, createWebHistory } from 'vue-router';

const routes = [
    { path: '/', redirect: '/dept' },
    { path: '/dept', component: () => import('@/views/DeptView.vue') },
    { path: '/emp',  component: () => import('@/views/EmpView.vue') },
];

const router = createRouter({
    history: createWebHistory(),
    routes,
});

export default router;
```

```vue
<!-- App.vue -->
<template>
    <div>
        <nav>
            <router-link to="/dept">部门管理</router-link>
            <router-link to="/emp">员工管理</router-link>
        </nav>
        <router-view />  <!-- 匹配的路由组件渲染在这里 -->
    </div>
</template>
```

---

## 2. Element Plus

### 2.1 是什么？

Element Plus 是一套基于 Vue 3 的**桌面端组件库**，提供按钮、表格、表单、弹窗等常见 UI 组件。

```bash
npm install element-plus
```

```javascript
// main.js
import ElementPlus from 'element-plus';
import 'element-plus/dist/index.css';
import zhCn from 'element-plus/locale/zh-cn';

app.use(ElementPlus, { locale: zhCn });  // 中文
```

### 2.2 常用组件

**表格**：

```vue
<template>
    <el-table :data="tableData" border stripe>
        <el-table-column prop="name" label="姓名" />
        <el-table-column prop="age" label="年龄" />
        <el-table-column label="操作">
            <template #default="scope">
                <el-button size="small" @click="handleEdit(scope.row)">编辑</el-button>
                <el-button size="small" type="danger" @click="handleDelete(scope.row.id)">删除</el-button>
            </template>
        </el-table-column>
    </el-table>
</template>
```

**分页**：

```vue
<el-pagination
    v-model:current-page="page"
    v-model:page-size="pageSize"
    :total="total"
    layout="total, prev, pager, next, sizes"
    @current-change="loadData"
/>
```

**表单**：

```vue
<el-form :model="form" label-width="80px">
    <el-form-item label="用户名">
        <el-input v-model="form.username" />
    </el-form-item>
    <el-form-item label="性别">
        <el-select v-model="form.gender">
            <el-option label="男" :value="1" />
            <el-option label="女" :value="2" />
        </el-select>
    </el-form-item>
</el-form>
```

**弹窗**：

```vue
<el-dialog v-model="dialogVisible" title="新增员工" width="500px">
    <el-form :model="form"> ... </el-form>
    <template #footer>
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="handleSubmit">确定</el-button>
    </template>
</el-dialog>
```

**消息提示**：

```javascript
import { ElMessage, ElMessageBox } from 'element-plus';

ElMessage.success('操作成功');
ElMessage.error('操作失败');

ElMessageBox.confirm('确定要删除吗？', '提示', { type: 'warning' })
    .then(() => { /* 确认 */ })
    .catch(() => { /* 取消 */ });
```

---

## 3. API 请求封装

```javascript
// api/request.js
import axios from 'axios';
import { ElMessage } from 'element-plus';
import router from '@/router';

const request = axios.create({
    baseURL: 'http://localhost:8080',  // 后端地址
    timeout: 5000,
});

// 请求拦截器 → 自动加 Token
request.interceptors.request.use(config => {
    const token = localStorage.getItem('token');
    if (token) {
        config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
});

// 响应拦截器 → 统一错误处理
request.interceptors.response.use(
    response => {
        if (response.data.code === 0) {
            ElMessage.error(response.data.msg);
            return Promise.reject(response.data.msg);
        }
        return response.data;
    },
    error => {
        if (error.response?.status === 401) {
            ElMessage.error('登录已过期，请重新登录');
            router.push('/login');
        }
        return Promise.reject(error);
    }
);

export default request;
```

```javascript
// api/dept.js
import request from './request';

export const getDeptList = () => request.get('/depts');
export const addDept = (data) => request.post('/depts', data);
export const updateDept = (data) => request.put('/depts', data);
export const deleteDept = (id) => request.delete(`/depts/${id}`);
```

---

## 4. 总结

| 知识点 | 核心要点 |
|--------|----------|
| Vite | 新一代前端构建工具，开发服务器秒启动 |
| .vue 单文件组件 | `<template>` + `<script setup>` + `<style scoped>` |
| Composition API | `ref()` 响应式数据，`setup` 中写逻辑 |
| Vue Router | `createRouter` + `<router-link>` + `<router-view>` |
| Element Plus | 表格 `el-table`、分页 `el-pagination`、表单 `el-form`、弹窗 `el-dialog` |
| Axios 封装 | 请求拦截器（加 Token）+ 响应拦截器（401 跳登录） |
