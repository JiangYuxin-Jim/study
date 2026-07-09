# 14 - 前端 Tlias 案例实战

## 1. 案例概述

基于 Vue 3 + Element Plus + Axios 构建 Tlias 智能学习辅助系统的前端，与 SpringBoot 后端通过 REST API 通信。

**核心页面**：
- 部门管理（CRUD）
- 员工管理（分页查询、新增/修改/批量删除、文件上传）
- 员工信息统计（柱状图）
- 班级/学员管理

---

## 2. 部门管理页面

```vue
<!-- views/DeptView.vue -->
<template>
    <div class="dept-container">
        <!-- 搜索栏 -->
        <el-form :inline="true">
            <el-form-item>
                <el-input v-model="searchName" placeholder="部门名称" />
            </el-form-item>
            <el-form-item>
                <el-button type="primary" @click="loadData">查询</el-button>
                <el-button type="success" @click="openAddDialog">新增部门</el-button>
            </el-form-item>
        </el-form>

        <!-- 数据表格 -->
        <el-table :data="deptList" border>
            <el-table-column prop="id" label="ID" width="80" />
            <el-table-column prop="name" label="部门名称" />
            <el-table-column prop="createTime" label="创建时间" />
            <el-table-column label="操作" width="200">
                <template #default="scope">
                    <el-button size="small" @click="openEditDialog(scope.row)">编辑</el-button>
                    <el-button size="small" type="danger" @click="handleDelete(scope.row.id)">删除</el-button>
                </template>
            </el-table-column>
        </el-table>

        <!-- 新增/编辑弹窗 -->
        <el-dialog v-model="dialogVisible" :title="isEdit ? '修改部门' : '新增部门'">
            <el-form :model="form">
                <el-form-item label="部门名称">
                    <el-input v-model="form.name" />
                </el-form-item>
            </el-form>
            <template #footer>
                <el-button @click="dialogVisible = false">取消</el-button>
                <el-button type="primary" @click="handleSubmit">确定</el-button>
            </template>
        </el-dialog>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { ElMessage, ElMessageBox } from 'element-plus';
import { getDeptList, addDept, updateDept, deleteDept } from '@/api/dept';

const deptList = ref([]);
const dialogVisible = ref(false);
const isEdit = ref(false);
const form = ref({ id: null, name: '' });

// 加载数据
function loadData() {
    getDeptList().then(res => { deptList.value = res.data; });
}

// 新增
function openAddDialog() {
    isEdit.value = false;
    form.value = { id: null, name: '' };
    dialogVisible.value = true;
}

// 编辑
function openEditDialog(row) {
    isEdit.value = true;
    form.value = { ...row };
    dialogVisible.value = true;
}

// 提交
function handleSubmit() {
    if (isEdit.value) {
        updateDept(form.value).then(() => {
            ElMessage.success('修改成功');
            dialogVisible.value = false;
            loadData();
        });
    } else {
        addDept(form.value).then(() => {
            ElMessage.success('新增成功');
            dialogVisible.value = false;
            loadData();
        });
    }
}

// 删除
function handleDelete(id) {
    ElMessageBox.confirm('确定删除该部门吗？', '提示', { type: 'warning' })
        .then(() => deleteDept(id))
        .then(() => {
            ElMessage.success('删除成功');
            loadData();
        });
}

onMounted(() => { loadData(); });
</script>
```

---

## 3. 员工管理页面（核心功能点）

### 3.1 分页条件查询

```vue
<script setup>
const page = ref(1);
const pageSize = ref(10);
const total = ref(0);
const empList = ref([]);
const searchParams = ref({ name: '', gender: null, begin: '', end: '' });

function loadData() {
    getEmpPage({
        page: page.value,
        pageSize: pageSize.value,
        ...searchParams.value
    }).then(res => {
        empList.value = res.data.rows;
        total.value = res.data.total;
    });
}

function handlePageChange(newPage) {
    page.value = newPage;
    loadData();
}
</script>
```

### 3.2 文件上传

```vue
<template>
    <el-upload
        action="http://localhost:8080/upload"
        :on-success="handleUploadSuccess"
        :before-upload="beforeUpload"
        list-type="picture"
    >
        <el-button type="primary">上传图片</el-button>
        <template #tip>
            <div style="color: #999">只能上传 jpg/png 文件，且不超过 2MB</div>
        </template>
    </el-upload>
</template>

<script setup>
import { ElMessage } from 'element-plus';

function beforeUpload(file) {
    const isImage = file.type.startsWith('image/');
    const isLt2M = file.size / 1024 / 1024 < 2;
    if (!isImage) ElMessage.error('只能上传图片文件');
    if (!isLt2M) ElMessage.error('图片大小不能超过 2MB');
    return isImage && isLt2M;
}

function handleUploadSuccess(response) {
    form.value.image = response.data;  // 把返回的 URL 存入表单
}
</script>
```

### 3.3 批量删除

```javascript
function handleBatchDelete() {
    const ids = selectedRows.value.map(row => row.id);
    if (ids.length === 0) {
        ElMessage.warning('请选择要删除的员工');
        return;
    }
    ElMessageBox.confirm(`确定删除选中的 ${ids.length} 条数据吗？`, '提示', { type: 'warning' })
        .then(() => batchDeleteEmps(ids))
        .then(() => {
            ElMessage.success('删除成功');
            loadData();
        });
}
```

---

## 4. 登录页面

```vue
<template>
    <div class="login-container">
        <el-card class="login-card">
            <h2>Tlias 智能学习辅助系统</h2>
            <el-form :model="form">
                <el-form-item>
                    <el-input v-model="form.username" placeholder="用户名" />
                </el-form-item>
                <el-form-item>
                    <el-input v-model="form.password" type="password" placeholder="密码" />
                </el-form-item>
                <el-form-item>
                    <el-button type="primary" style="width: 100%" @click="login">登录</el-button>
                </el-form-item>
            </el-form>
        </el-card>
    </div>
</template>

<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import { login as loginApi } from '@/api/auth';
import { ElMessage } from 'element-plus';

const router = useRouter();
const form = ref({ username: '', password: '' });

function login() {
    loginApi(form.value).then(res => {
        localStorage.setItem('token', res.data.token);
        ElMessage.success('登录成功');
        router.push('/');
    }).catch(() => {
        ElMessage.error('用户名或密码错误');
    });
}
</script>
```

---

## 5. 路由守卫（未登录拦截）

```javascript
// router/index.js
import { createRouter, createWebHistory } from 'vue-router';

const routes = [
    { path: '/login', component: () => import('@/views/LoginView.vue') },
    // 需要登录才能访问的页面
    {
        path: '/',
        component: () => import('@/views/LayoutView.vue'),
        children: [
            { path: 'dept', component: () => import('@/views/DeptView.vue') },
            { path: 'emp', component: () => import('@/views/EmpView.vue') },
        ],
        meta: { requiresAuth: true },
    },
];

const router = createRouter({
    history: createWebHistory(),
    routes,
});

// 全局前置守卫
router.beforeEach((to, from, next) => {
    const token = localStorage.getItem('token');
    if (to.meta.requiresAuth && !token) {
        next('/login');  // 没登录 → 跳登录页
    } else {
        next();
    }
});

export default router;
```

---

## 6. 前后端联调流程

```
① 后端启动：在 IDEA 运行 SpringBootApplication → http://localhost:8080
② 前端启动：npm run dev → http://localhost:5173
③ 配置代理（vite.config.js）解决跨域：
   server: { proxy: { '/api': { target: 'http://localhost:8080', changeOrigin: true } } }
④ 前后端联调测试 → 修复问题
⑤ 前端打包：npm run build → dist/
⑥ 将 dist/ 放到 SpringBoot 的 resources/static/ 下
   → 或者放到 Nginx 下独立部署
```

---

## 7. 总结

| 知识点 | 核心要点 |
|--------|----------|
| Vue 工程化 | Vite 创建项目，`.vue` 单文件组件开发 |
| Element Plus CRUD | `el-table` + `el-pagination` + `el-dialog` + `el-form` |
| 文件上传 | `el-upload`，`before-upload` 校验类型和大小 |
| 批量删除 | 多选表格，收集 ID 列表，一次发 DELETE |
| Axios 拦截器 | 请求拦截加 Token，响应拦截处理 401 |
| 路由守卫 | `router.beforeEach`，检查 Token，未登录跳 `/login` |
| 代理跨域 | `vite.config.js` 配 proxy，开发阶段解决跨域 |
| 部署 | 前端 `npm run build` → `dist/` → Nginx 或 SpringBoot static |
