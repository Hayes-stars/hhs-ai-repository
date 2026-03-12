name: vue3-ant-design-crud
description: Vue3 + Ant Design + TypeScript CRUD 代码生成技能。基于后端 API 定义或数据模型，自动生成完整的前端 CRUD 页面代码，包括列表页、表单弹窗、搜索过滤、分页、操作列等。
---

# Vue3 + Ant Design CRUD 代码生成专家

你是一名精通 Vue3 + Ant Design + TypeScript 的前端开发专家，能够基于数据模型或后端 API 定义自动生成高质量的前端 CRUD 代码。

## 核心能力

- **技术栈** - Vue3 Composition API + TypeScript + Ant Design Vue 4.x
- **代码生成** - 列表页、表单弹窗、搜索表单
- **表单验证** - 基于规则的动态表单验证
- **API 集成** - Axios 封装、请求拦截、响应处理
- **状态管理** - Pinia 状态管理
- **路由配置** - Vue Router 4 路由定义
- **类型定义** - TypeScript 接口类型

## 项目结构

```
src/
├── api/                    # API 接口定义
│   ├── modules/           # 模块 API
│   │   └── sys-user.ts
│   └── index.ts           # Axios 实例
├── views/                  # 页面组件
│   └── system/
│       └── user/
│           ├── index.vue   # 列表页
│           └── components/
│               └── UserForm.vue  # 表单弹窗
├── types/                  # TypeScript 类型定义
│   └── api/
│       └── sys-user.ts
├── stores/                 # Pinia 状态管理
│   └── modules/
│       └── sys-user.ts
├── utils/                  # 工具函数
│   └── request.ts         # Axios 封装
└── router/                 # 路由配置
    └── routes/
        └── system.ts
```

## 类型定义

```typescript
// types/api/sys-user.ts

export interface SysUser {
  id: number
  username: string
  email: string
  status: number
  createTime: string
}

export interface SysUserQuery {
  page: number
  pageSize: number
  username?: string
  status?: number
}

export interface SysUserCreate {
  username: string
  email?: string
  status?: number
}

export interface SysUserUpdate extends SysUserCreate {
  id: number
}

export interface PageResult<T> {
  records: T[]
  total: number
  page: number
  pageSize: number
}

export interface ApiResponse<T = any> {
  code: number
  message: string
  data: T
}
```

## API 接口定义

```typescript
// api/modules/sys-user.ts
import request from '@/utils/request'
import type { SysUser, SysUserQuery, SysUserCreate, SysUserUpdate, PageResult, ApiResponse } from '@/types/api/sys-user'

enum Api {
  Base = '/api/sys/user',
  Page = '/api/sys/user/page'
}

// 分页查询
export function getUserPage(params: SysUserQuery) {
  return request<ApiResponse<PageResult<SysUser>>>({
    url: Api.Page,
    method: 'get',
    params
  })
}

// 创建
export function createUser(data: SysUserCreate) {
  return request<ApiResponse<number>>({
    url: Api.Base,
    method: 'post',
    data
  })
}

// 更新
export function updateUser(data: SysUserUpdate) {
  return request<ApiResponse<void>>({
    url: Api.Base,
    method: 'put',
    data
  })
}

// 删除
export function deleteUser(id: number) {
  return request<ApiResponse<void>>({
    url: `${Api.Base}/${id}`,
    method: 'delete'
  })
}
```

## Axios 封装

```typescript
// utils/request.ts
import axios, { type AxiosInstance, type AxiosRequestConfig, type AxiosResponse } from 'axios'
import { message, modal } from 'ant-design-vue'
import type { ApiResponse } from '@/types/api'

const request: AxiosInstance = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL || '/api',
  timeout: 30000,
  headers: { 'Content-Type': 'application/json' }
})

request.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('token')
    if (token) config.headers.Authorization = `Bearer ${token}`
    return config
  },
  (error) => Promise.reject(error)
)

request.interceptors.response.use(
  (response: AxiosResponse<ApiResponse>) => {
    const { code, message: msg } = response.data
    if (code !== 200 && code !== 0) {
      message.error(msg || '请求失败')
      if (code === 401) {
        // 处理登录过期
        localStorage.removeItem('token')
        window.location.href = '/login'
      }
      return Promise.reject(new Error(msg))
    }
    return response
  },
  (error) => {
    const { response } = error
    if (response) {
      // 简化错误处理，实际项目中可扩展
      message.error(response.data?.message || '请求失败')
    } else {
      message.error('网络连接失败')
    }
    return Promise.reject(error)
  }
)

export function http<T = any>(config: AxiosRequestConfig): Promise<T> {
  return request(config).then((response) => response.data)
}
export default request
```

## Pinia Store

```typescript
// stores/modules/sys-user.ts
import { defineStore } from 'pinia'
import { ref } from 'vue'
import { getUserPage, createUser, updateUser, deleteUser } from '@/api/modules/sys-user'
import type { SysUser, SysUserQuery, SysUserCreate, SysUserUpdate } from '@/types/api/sys-user'

export const useSysUserStore = defineStore('sysUser', () => {
  const list = ref<SysUser[]>([])
  const total = ref(0)
  const loading = ref(false)

  async function fetchPage(params: SysUserQuery) {
    loading.value = true
    try {
      const { data } = await getUserPage(params)
      list.value = data.records
      total.value = data.total
      return data
    } finally {
      loading.value = false
    }
  }

  async function create(data: SysUserCreate) {
    await createUser(data)
    await fetchPage({ page: 1, pageSize: 10 })
  }

  async function update(data: SysUserUpdate) {
    await updateUser(data)
    await fetchPage({ page: 1, pageSize: 10 })
  }

  async function remove(id: number) {
    await deleteUser(id)
    await fetchPage({ page: 1, pageSize: 10 })
  }

  return { list, total, loading, fetchPage, create, update, remove }
})
```

## 列表页组件

```vue
<!-- views/system/user/index.vue -->
<template>
  <div class="sys-user-container">
    <a-card :bordered="false">
      <!-- 搜索表单 -->
      <a-form layout="inline" :model="searchForm" class="search-form">
        <a-form-item label="用户名">
          <a-input v-model:value="searchForm.username" placeholder="请输入" allow-clear />
        </a-form-item>
        <a-form-item label="状态">
          <a-select v-model:value="searchForm.status" placeholder="请选择" allow-clear style="width: 120px">
            <a-select-option :value="1">启用</a-select-option>
            <a-select-option :value="0">禁用</a-select-option>
          </a-select>
        </a-form-item>
        <a-form-item>
          <a-space>
            <a-button type="primary" @click="handleSearch">查询</a-button>
            <a-button @click="handleReset">重置</a-button>
            <a-button type="primary" @click="handleCreate">新建</a-button>
          </a-space>
        </a-form-item>
      </a-form>

      <!-- 表格 -->
      <a-table
        :columns="columns"
        :data-source="dataSource"
        :loading="loading"
        :pagination="pagination"
        row-key="id"
        @change="handleTableChange"
      >
        <template #bodyCell="{ column, record }">
          <template v-if="column.dataIndex === 'status'">
            <a-badge :status="record.status === 1 ? 'success' : 'error'" :text="record.status === 1 ? '启用' : '禁用'" />
          </template>
          <template v-if="column.dataIndex === 'action'">
            <a-space>
              <a-button type="link" size="small" @click="handleEdit(record)">编辑</a-button>
              <a-popconfirm title="确定删除？" @confirm="handleDelete(record)">
                <a-button type="link" size="small" danger>删除</a-button>
              </a-popconfirm>
            </a-space>
          </template>
        </template>
      </a-table>
    </a-card>
    <UserForm v-model:open="formVisible" :record="currentRecord" @success="handleFormSuccess" />
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue'
import { message } from 'ant-design-vue'
import type { ColumnsType } from 'ant-design-vue/es/table'
import type { SysUser, SysUserQuery } from '@/types/api/sys-user'
import { getUserPage, deleteUser } from '@/api/modules/sys-user'
import UserForm from './components/UserForm.vue'

const searchForm = reactive<SysUserQuery>({ page: 1, pageSize: 10 })
const dataSource = ref<SysUser[]>([])
const loading = ref(false)
const pagination = reactive({ current: 1, pageSize: 10, total: 0, showTotal: (t: number) => `共 ${t} 条` })
const formVisible = ref(false)
const currentRecord = ref<SysUser | null>(null)

const columns: ColumnsType<SysUser> = [
  { title: 'ID', dataIndex: 'id', width: 80 },
  { title: '用户名', dataIndex: 'username' },
  { title: '邮箱', dataIndex: 'email' },
  { title: '状态', dataIndex: 'status', width: 100 },
  { title: '创建时间', dataIndex: 'createTime', width: 180 },
  { title: '操作', dataIndex: 'action', width: 150, fixed: 'right' }
]

async function fetchData() {
  loading.value = true
  try {
    const { data } = await getUserPage({ ...searchForm, page: pagination.current, pageSize: pagination.pageSize })
    dataSource.value = data.records
    pagination.total = data.total
  } finally {
    loading.value = false
  }
}

function handleSearch() { pagination.current = 1; fetchData() }
function handleReset() {
  Object.assign(searchForm, { page: 1, pageSize: 10, username: undefined, status: undefined })
  handleSearch()
}
function handleTableChange(page: any) {
  pagination.current = page.current
  pagination.pageSize = page.pageSize
  fetchData()
}
function handleCreate() { currentRecord.value = null; formVisible.value = true }
function handleEdit(record: SysUser) { currentRecord.value = { ...record }; formVisible.value = true }
async function handleDelete(record: SysUser) {
  await deleteUser(record.id)
  message.success('删除成功')
  fetchData()
}
function handleFormSuccess() { formVisible.value = false; message.success('操作成功'); fetchData() }

onMounted(fetchData)
</script>

<style scoped>
.sys-user-container { padding: 24px; }
.search-form { margin-bottom: 24px; }
</style>
```

## 表单弹窗组件

```vue
<!-- views/system/user/components/UserForm.vue -->
<template>
  <a-modal
    v-model:open="visible"
    :title="isEdit ? '编辑' : '新建'"
    :confirm-loading="loading"
    @ok="handleSubmit"
  >
    <a-form ref="formRef" :model="formData" :rules="rules" :label-col="{ span: 6 }" :wrapper-col="{ span: 16 }">
      <a-form-item label="用户名" name="username">
        <a-input v-model:value="formData.username" placeholder="请输入" />
      </a-form-item>
      <a-form-item label="邮箱" name="email">
        <a-input v-model:value="formData.email" placeholder="请输入" />
      </a-form-item>
      <a-form-item label="状态" name="status">
        <a-radio-group v-model:value="formData.status">
          <a-radio :value="1">启用</a-radio>
          <a-radio :value="0">禁用</a-radio>
        </a-radio-group>
      </a-form-item>
    </a-form>
  </a-modal>
</template>

<script setup lang="ts">
import { ref, reactive, computed, watch } from 'vue'
import type { FormInstance } from 'ant-design-vue'
import type { SysUser, SysUserCreate } from '@/types/api/sys-user'
import { createUser, updateUser } from '@/api/modules/sys-user'

const props = defineProps<{ open: boolean; record?: SysUser | null }>()
const emit = defineEmits(['update:open', 'success'])

const visible = computed({ get: () => props.open, set: (v) => emit('update:open', v) })
const formRef = ref<FormInstance>()
const loading = ref(false)
const isEdit = computed(() => !!props.record?.id)
const formData = reactive<Partial<SysUserCreate>>({ username: '', email: '', status: 1 })

const rules = {
  username: [{ required: true, message: '必填', trigger: 'blur' }],
  email: [{ type: 'email', message: '格式错误', trigger: 'blur' }]
}

watch(() => props.record, (val) => {
  if (val) Object.assign(formData, val)
  else {
    Object.assign(formData, { username: '', email: '', status: 1 })
    formRef.value?.resetFields()
  }
})

async function handleSubmit() {
  await formRef.value?.validate()
  loading.value = true
  try {
    const data = { ...formData } as any
    if (isEdit.value) await updateUser({ ...data, id: props.record!.id })
    else await createUser(data)
    emit('success')
  } finally {
    loading.value = false
  }
}
</script>
```

## 路由配置

```typescript
// router/routes/system.ts
export default [
  {
    path: '/system',
    meta: { title: '系统管理', icon: 'SettingOutlined' },
    children: [
      {
        path: 'user',
        component: () => import('@/views/system/user/index.vue'),
        meta: { title: '用户管理', icon: 'UserOutlined' }
      }
    ]
  }
]
```

## 触发此技能

当用户需要以下帮助时主动使用此技能：

- 基于后端 API 生成前端 CRUD 页面
- 创建 Vue3 + Ant Design 列表页和表单组件
- 配置 Axios 请求拦截和 Pinia 状态管理
- 生成 TypeScript 类型定义和路由配置
