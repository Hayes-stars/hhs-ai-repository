---
name: vue3-frontend-dev
description: 精通 Vue3 前端开发的专业技能。当用户需要创建、修改、调试 Vue3 项目，或询问 Composition API、<script setup>、Pinia 状态管理、Vue Router 4 路由配置时使用此技能。
---

# Vue3 前端开发专家

你是一名精通 Vue3 的前端开发专家，能够帮助用户高效完成 Vue3 项目开发任务。

## 核心能力

- **Vue3 项目搭建** - Vite 项目初始化与配置
- **组件开发** - Composition API、`<script setup>` 语法
- **状态管理** - Pinia (推荐) / Vuex 4
- **路由配置** - Vue Router 4
- **组合式函数** - 可复用逻辑封装 (composables)
- **性能优化** - 代码分割、懒加载、响应式优化
- **TypeScript 支持** - 类型推断与类型安全
- **调试排错** - Vue3 常见问题诊断与修复

## 项目初始化

### 使用 create-vue (官方推荐)

```bash
# 创建 Vue3 项目
npm create vue@latest my-app

# 或指定模板
npm create vue@latest my-app -- --typescript --router --pinia

cd my-app
npm install
npm run dev
```

### Vite 手动配置

```bash
# 初始化项目
npm init -y
npm install vue@3 vue-router@4 pinia

# 安装 Vite
npm install -D vite @vitejs/plugin-vue

# vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  server: {
    port: 3000,
    open: true
  }
})
```

### TypeScript 配置

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "node",
    "strict": true,
    "jsx": "preserve",
    "sourceMap": true,
    "resolveJsonModule": true,
    "esModuleInterop": true,
    "lib": ["ES2020", "DOM"],
    "types": ["vite/client"],
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.vue"]
}
```

## 组件开发规范

### `<script setup>` 语法 (Vue3 首选)

```vue
<script setup lang="ts">
import { ref, computed, watch, watchEffect, onMounted, onUnmounted } from 'vue'
import type { PropType } from 'vue'

// ============ Props ============
interface Props {
  title: string
  count?: number
  items?: Array<{ id: number; name: string }>
  callback?: (data: string) => void
}

const props = withDefaults(defineProps<Props>(), {
  count: 0,
  items: () => []
})

// ============ Emits ============
interface Emits {
  (e: 'update', value: number): void
  (e: 'change', data: { id: number; name: string }): void
  (e: 'delete', id: number): void
}

const emit = defineEmits<Emits>()

// ============ 响应式状态 ============
const localCount = ref(props.count)
const message = ref('')
const loading = ref(false)
const form = ref({
  name: '',
  email: ''
})

// 响应式对象
const state = reactive({
  count: 0,
  items: [] as Array<{ id: number; name: string }>
})

// ============ 计算属性 ============
const doubled = computed(() => localCount.value * 2)

const filteredItems = computed(() => {
  return props.items.filter(item => item.id > 0)
})

// 可写的计算属性
const fullName = computed({
  get() {
    return `${form.value.firstName} ${form.value.lastName}`
  },
  set(value) {
    const [first, last] = value.split(' ')
    form.value.firstName = first
    form.value.lastName = last
  }
})

// ============ 监听器 ============
// 监听单个值
watch(localCount, (newVal, oldVal) => {
  console.log('count 变化:', newVal)
})

// 监听多个值
watch([localCount, message], ([newCount, newMessage]) => {
  console.log('多个值变化:', newCount, newMessage)
})

// 深度监听
watch(
  () => props.items,
  (newItems) => {
    console.log('items 变化:', newItems)
  },
  { deep: true }
)

// 立即执行
watch(
  localCount,
  (newVal) => {
    console.log('立即执行:', newVal)
  },
  { immediate: true }
)

// watchEffect - 自动追踪依赖
watchEffect(() => {
  console.log('doubled:', doubled.value)
})

// ============ 生命周期 ============
onBeforeMount(() => {
  console.log('挂载前')
})

onMounted(() => {
  console.log('组件已挂载')
  fetchData()
})

onBeforeUpdate(() => {
  console.log('更新前')
})

onUpdated(() => {
  console.log('组件已更新')
})

onBeforeUnmount(() => {
  console.log('卸载前，清理资源')
  cleanup()
})

onUnmounted(() => {
  console.log('组件已卸载')
})

// ============ 方法 ============
function handleClick() {
  localCount.value++
  emit('update', localCount.value)
}

function handleDelete(id: number) {
  emit('delete', id)
}

async function fetchData() {
  loading.value = true
  try {
    const res = await fetch('/api/data')
    const data = await res.json()
    // 处理数据
  } catch (e) {
    console.error(e)
  } finally {
    loading.value = false
  }
}

function cleanup() {
  // 清理定时器、事件监听器等
}

// ============ 模板引用 ============
const inputRef = ref<HTMLInputElement | null>(null)
const componentRef = ref<InstanceType<typeof ChildComponent> | null>(null)

onMounted(() => {
  inputRef.value?.focus()
})

// ============ 暴露给父组件 ============
defineExpose({
  localCount,
  fetchData
})
</script>

<template>
  <div class="component">
    <h2>{{ title }}</h2>
    <p>Count: {{ localCount }}</p>
    <p>Doubled: {{ doubled }}</p>
    
    <!-- 条件渲染 -->
    <div v-if="loading">加载中...</div>
    <div v-else-if="items.length === 0">暂无数据</div>
    <div v-else>
      <div v-for="item in filteredItems" :key="item.id">
        {{ item.name }}
      </div>
    </div>
    
    <!-- 事件绑定 -->
    <button @click="handleClick">更新</button>
    <button @click.stop="handleClick">阻止冒泡</button>
    <button @click.prevent="handleClick">阻止默认</button>
    
    <!-- 表单绑定 -->
    <input v-model="message" placeholder="输入消息" />
    
    <!-- 模板引用 -->
    <input ref="inputRef" type="text" />
    
    <!-- 子组件引用 -->
    <ChildComponent ref="componentRef" />
  </div>
</template>

<style scoped>
.component {
  padding: 16px;
}

h2 {
  color: #333;
}
</style>
```

### 传统 setup 函数

```vue
<script>
import { ref, computed, onMounted } from 'vue'

export default {
  props: {
    title: String,
    count: { type: Number, default: 0 }
  },
  
  emits: ['update', 'change'],
  
  setup(props, { emit }) {
    const localCount = ref(props.count)
    
    const doubled = computed(() => localCount.value * 2)
    
    function handleClick() {
      localCount.value++
      emit('update', localCount.value)
    }
    
    onMounted(() => {
      console.log('mounted')
    })
    
    return {
      localCount,
      doubled,
      handleClick
    }
  }
}
</script>
```

## 组合式函数 (Composables)

### useFetch

```typescript
// composables/useFetch.ts
import { ref, type Ref } from 'vue'

interface UseFetchReturn<T> {
  data: Ref<T | null>
  error: Ref<Error | null>
  loading: Ref<boolean>
  execute: () => Promise<void>
}

export function useFetch<T>(url: string): UseFetchReturn<T> {
  const data = ref<T | null>(null) as Ref<T | null>
  const error = ref<Error | null>(null)
  const loading = ref(false)
  
  const execute = async () => {
    loading.value = true
    try {
      const res = await fetch(url)
      if (!res.ok) throw new Error(res.statusText)
      data.value = await res.json()
    } catch (e) {
      error.value = e as Error
    } finally {
      loading.value = false
    }
  }
  
  // 自动执行
  execute()
  
  return { data, error, loading, execute }
}

// 使用
const { data, error, loading } = useFetch<User[]>('/api/users')
```

### useLocalStorage

```typescript
// composables/useLocalStorage.ts
import { ref, watch, type Ref } from 'vue'

export function useLocalStorage<T>(
  key: string,
  defaultValue: T
): Ref<T> {
  const stored = localStorage.getItem(key)
  const value = ref<T>(stored ? JSON.parse(stored) : defaultValue)
  
  watch(value, (newVal) => {
    localStorage.setItem(key, JSON.stringify(newVal))
  }, { deep: true })
  
  return value
}

// 使用
const theme = useLocalStorage<'light' | 'dark'>('theme', 'light')
```

### useDebounce

```typescript
// composables/useDebounce.ts
import { ref, watch, type Ref } from 'vue'

export function useDebounce<T>(
  source: Ref<T>,
  delay: number = 300
): Ref<T> {
  const debounced = ref<T>(source.value) as Ref<T>
  let timer: ReturnType<typeof setTimeout> | null = null
  
  watch(source, (newVal) => {
    if (timer) clearTimeout(timer)
    timer = setTimeout(() => {
      debounced.value = newVal
    }, delay)
  })
  
  return debounced
}

// 使用
const searchQuery = ref('')
const debouncedQuery = useDebounce(searchQuery, 500)
```

### useToggle

```typescript
// composables/useToggle.ts
import { ref } from 'vue'

export function useToggle(initialValue: boolean = false) {
  const value = ref(initialValue)
  
  const toggle = () => {
    value.value = !value.value
  }
  
  const set = (v: boolean) => {
    value.value = v
  }
  
  return { value, toggle, set }
}

// 使用
const { value: isVisible, toggle, set } = useToggle(false)
```

## Pinia 状态管理

### Store 定义

```typescript
// stores/counter.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

// 使用 setup 函数
export const useCounterStore = defineStore('counter', () => {
  // State
  const count = ref(0)
  const name = ref('counter')
  
  // Getters
  const doubled = computed(() => count.value * 2)
  
  // Actions
  function increment() {
    count.value++
  }
  
  function decrement() {
    count.value--
  }
  
  function incrementBy(amount: number) {
    count.value += amount
  }
  
  async function incrementAsync() {
    return new Promise<void>((resolve) => {
      setTimeout(() => {
        increment()
        resolve()
      }, 1000)
    })
  }
  
  return { count, name, doubled, increment, decrement, incrementBy, incrementAsync }
})

// 或使用 options API
export const useUserStore = defineStore('user', {
  state: () => ({
    id: null as number | null,
    name: null as string | null,
    email: null as string | null
  }),
  
  getters: {
    isLoggedIn: (state) => !!state.id,
    displayName: (state) => state.name || '匿名用户'
  },
  
  actions: {
    login(user: { id: number; name: string; email: string }) {
      this.id = user.id
      this.name = user.name
      this.email = user.email
    },
    
    logout() {
      this.$reset()
    },
    
    async fetchUser(userId: number) {
      const res = await fetch(`/api/users/${userId}`)
      const user = await res.json()
      this.login(user)
    }
  }
})
```

### 组件中使用 Store

```vue
<script setup lang="ts">
import { useCounterStore } from '@/stores/counter'
import { useUserStore } from '@/stores/user'
import { storeToRefs } from 'pinia'

// 直接使用 store
const counterStore = useCounterStore()
const userStore = useUserStore()

// 解构 state (保持响应式)
const { count, doubled } = storeToRefs(counterStore)

// 直接使用 actions
const { increment, incrementAsync } = counterStore
const { login, logout } = userStore

// 使用
increment()
incrementAsync()
</script>

<template>
  <div>
    <p>Count: {{ count }}</p>
    <p>Doubled: {{ doubled }}</p>
    <button @click="increment">+1</button>
  </div>
</template>
```

### Store 插件

```typescript
// stores/plugins/persist.ts
import type { PiniaPluginContext } from 'pinia'

export function persistPlugin({ store }: PiniaPluginContext) {
  const saved = localStorage.getItem(`store:${store.$id}`)
  if (saved) {
    store.$patch(JSON.parse(saved))
  }
  
  store.$subscribe((mutation, state) => {
    localStorage.setItem(`store:${store.$id}`, JSON.stringify(state))
  })
}

// main.ts
import { createPinia } from 'pinia'
import { persistPlugin } from './stores/plugins/persist'

const pinia = createPinia()
pinia.use(persistPlugin)
app.use(pinia)
```

## Vue Router 4 路由配置

### 基础配置

```typescript
// router/index.ts
import { createRouter, createWebHistory, type RouteRecordRaw } from 'vue-router'

const routes: RouteRecordRaw[] = [
  {
    path: '/',
    name: 'Home',
    component: () => import('@/views/Home.vue'),
    meta: {
      title: '首页',
      requiresAuth: false
    }
  },
  {
    path: '/about',
    name: 'About',
    component: () => import('@/views/About.vue'),
    meta: {
      title: '关于'
    }
  },
  {
    path: '/user/:id',
    name: 'User',
    component: () => import('@/views/User.vue'),
    props: true,
    meta: {
      requiresAuth: true
    }
  },
  {
    path: '/search',
    name: 'Search',
    component: () => import('@/views/Search.vue'),
    props: (route) => ({ query: route.query.q })
  },
  {
    path: '/:pathMatch(.*)*',
    redirect: '/'
  }
]

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes,
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      return savedPosition
    }
    return { top: 0 }
  }
})

export default router
```

### 路由守卫

```typescript
// router/index.ts
import router from './index'
import { useUserStore } from '@/stores/user'

// 全局前置守卫
router.beforeEach(async (to, from) => {
  document.title = (to.meta.title as string) || '默认标题'
  
  const userStore = useUserStore()
  const isAuthenticated = userStore.isLoggedIn
  
  if (to.meta.requiresAuth && !isAuthenticated) {
    return {
      path: '/login',
      query: { redirect: to.fullPath }
    }
  }
})

// 全局后置守卫
router.afterEach((to, from) => {
  console.log('路由跳转完成:', to.path)
})

// 组件内守卫
export default {
  beforeRouteEnter(to, from, next) {
    next(vm => {
      // 可以访问组件实例
    })
  },
  
  beforeRouteUpdate(to, from, next) {
    next()
  },
  
  beforeRouteLeave(to, from, next) {
    const answer = window.confirm('有未保存的更改，确定离开？')
    next(answer)
  }
}
```

### 编程式导航

```vue
<script setup lang="ts">
import { useRouter, useRoute } from 'vue-router'

const router = useRouter()
const route = useRoute()

function goToHome() {
  router.push('/')
  router.push({ name: 'Home' })
  router.push({ path: '/user', query: { id: 1 } })
}

function replacePage() {
  router.replace('/about')
}

function goBack() {
  router.back()
}

function goForward() {
  router.forward()
}
</script>
```

## 高级特性

### 异步组件

```vue
<script setup lang="ts">
import { defineAsyncComponent } from 'vue'

const AsyncComponent = defineAsyncComponent(() => import('@/components/Heavy.vue'))

// 带 loading 组件
const AsyncComponentWithOptions = defineAsyncComponent({
  loader: () => import('@/components/Heavy.vue'),
  loadingComponent: LoadingComponent,
  errorComponent: ErrorComponent,
  delay: 200,
  timeout: 3000
})
</script>
```

### Teleport

```vue
<template>
  <!-- 传送到 body -->
  <teleport to="body">
    <div class="modal">
      <slot />
    </div>
  </teleport>
  
  <!-- 传送到指定元素 -->
  <teleport to="#modal-root">
    <div class="modal">内容</div>
  </teleport>
</template>
```

### Suspense

```vue
<template>
  <Suspense>
    <template #default>
      <AsyncComponent />
    </template>
    <template #fallback>
      <div>加载中...</div>
    </template>
  </Suspense>
</template>
```

### KeepAlive

```vue
<template>
  <router-view v-slot="{ Component, route }">
    <keep-alive :include="['Home', 'About']">
      <component :is="Component" :key="route.name" />
    </keep-alive>
  </router-view>
</template>
```

## 性能优化

### 组件懒加载

```typescript
// 路由级别 - 自动代码分割
const User = () => import('@/views/User.vue')

// 组件级别
const HeavyComponent = defineAsyncComponent(() => 
  import('@/components/HeavyComponent.vue')
)
```

### 响应式优化

```typescript
// 使用 shallowRef 处理大型对象
import { shallowRef, triggerRef } from 'vue'

const largeData = shallowRef(largeObject)

// 修改时手动触发
largeData.value.someProperty = newValue
triggerRef(largeData)

// 使用 markRaw 避免不必要的响应式转换
import { markRaw, reactive } from 'vue'

const state = reactive({
  items: [],
  map: markRaw(new Map()) // Map 不需要响应式
})
```

### 列表优化

```vue
<template>
  <!-- 必须使用 key -->
  <div v-for="item in items" :key="item.id">
    {{ item.name }}
  </div>
  
  <!-- 静态内容使用 v-once -->
  <div v-once>静态内容</div>
  
  <!-- 大列表使用虚拟滚动 -->
  <RecycleScroller 
    :items="items"
    :item-size="50"
    key-field="id"
  />
</template>
```

## TypeScript 支持

### 组件类型定义

```vue
<script setup lang="ts">
import type { PropType } from 'vue'

interface User {
  id: number
  name: string
  email: string
}

const props = defineProps({
  user: {
    type: Object as PropType<User>,
    required: true
  },
  items: {
    type: Array as PropType<User[]>,
    default: () => []
  }
})

const emit = defineEmits<{
  (e: 'update', user: User): void
  (e: 'delete', id: number): void
}>()
</script>
```

### 泛型组件

```typescript
// composables/useList.ts
export function useList<T>(fetchFn: () => Promise<T[]>) {
  const items = ref<T[]>([])
  const loading = ref(false)
  const error = ref<Error | null>(null)
  
  const load = async () => {
    loading.value = true
    try {
      items.value = await fetchFn()
    } catch (e) {
      error.value = e as Error
    } finally {
      loading.value = false
    }
  }
  
  return { items, loading, error, load }
}

// 使用
const { items: users } = useList<User>(() => fetchUsers())
```

## 常见模式

### 父子组件通信

```vue
<!-- 父组件 -->
<script setup lang="ts">
import { ref } from 'vue'
import Child from './Child.vue'
import type { User } from '@/types'

const childRef = ref<InstanceType<typeof Child> | null>(null)
const message = ref('Hello')

function handleEvent(data: User) {
  console.log('子组件事件:', data)
}

function callChildMethod() {
  childRef.value?.doSomething()
}
</script>

<template>
  <Child 
    ref="childRef"
    :msg="message"
    @custom-event="handleEvent"
  />
  <button @click="callChildMethod">调用子组件方法</button>
</template>
```

```vue
<!-- 子组件 -->
<script setup lang="ts">
interface Props {
  msg: string
}

const props = defineProps<Props>()

const emit = defineEmits<{
  (e: 'custom-event', data: User): void
}>()

function notifyParent() {
  emit('custom-event', { id: 1, name: 'test' })
}

defineExpose({
  doSomething: () => {
    console.log('子组件方法')
  }
})
</script>
```

### Provide / Inject

```vue
<!-- 祖先组件 -->
<script setup lang="ts">
import { provide, ref, type Ref } from 'vue'
import type { Theme } from '@/types'

const theme = ref<Theme>('dark') as Ref<Theme>

function updateTheme(newTheme: Theme) {
  theme.value = newTheme
}

provide('theme', theme)
provide('updateTheme', updateTheme)
</script>
```

```vue
<!-- 后代组件 -->
<script setup lang="ts">
import { inject, type Ref } from 'vue'
import type { Theme } from '@/types'

const theme = inject<Ref<Theme>>('theme', ref('light'))
const updateTheme = inject<(t: Theme) => void>('updateTheme')
</script>
```

### 作用域插槽

```vue
<!-- 子组件 -->
<script setup lang="ts">
interface Item {
  id: number
  name: string
}

defineProps<{
  items: Item[]
}>()
</script>

<template>
  <div>
    <slot 
      v-for="item in items" 
      :item="item"
      :index="item.id"
    ></slot>
  </div>
</template>
```

```vue
<!-- 父组件 -->
<script setup lang="ts">
import type { Item } from '@/types'
</script>

<template>
  <ChildComponent :items="items" v-slot="{ item, index }">
    <div>{{ index }} - {{ item.name }}</div>
  </ChildComponent>
</template>
```

## 调试技巧

### Vue DevTools

- 安装 Vue DevTools 浏览器扩展 (支持 Vue3)
- 检查组件树、状态、事件
- 使用 Timeline 查看性能
- 检查 Pinia/Vuex 状态

### 常见错误排查

1. **响应式丢失**
   - 确保使用 `ref`/`reactive` 创建状态
   - 解构 ref 时使用 `.value`
   - 使用 `storeToRefs` 解构 Pinia state

2. **Props 警告**
   - 检查 props 类型定义
   - 使用 `withDefaults` 设置默认值

3. **Key 警告**
   - `v-for` 必须添加唯一 `:key`
   - 避免使用 index 作为 key

4. **内存泄漏**
   - onUnmounted 中清理资源
   - 移除事件监听器、定时器

```typescript
<script setup lang="ts">
import { onMounted, onUnmounted } from 'vue'

let timer: ReturnType<typeof setInterval>

onMounted(() => {
  timer = setInterval(() => {}, 1000)
})

onUnmounted(() => {
  clearInterval(timer)
})
</script>
```

## 最佳实践

1. **使用 `<script setup>`** - Vue3 首选语法，更简洁高效
2. **组合式函数复用** - 抽取可复用逻辑到 composables
3. **TypeScript 支持** - 使用 TS 获得更好的类型推断
4. **Pinia 替代 Vuex** - Vue3 推荐的状态管理方案
5. **ESLint + Prettier** - 保持代码风格一致
6. **单元测试** - 使用 Vitest + Vue Test Utils
7. **组件命名** - 使用 PascalCase 或多词 kebab-case
8. **Props 验证** - 始终定义 props 类型和默认值

## 项目结构建议

```
src/
├── assets/          # 静态资源
├── components/      # 通用组件
│   ├── base/       # 基础组件 (Button, Input)
│   └── business/   # 业务组件
├── composables/     # 组合式函数
├── views/          # 页面组件
├── router/         # 路由配置
├── stores/         # Pinia 状态管理
├── types/          # TypeScript 类型定义
├── utils/          # 工具函数
├── styles/         # 全局样式
├── plugins/        # 插件
└── App.vue
```

## 触发此技能

当用户需要以下帮助时主动使用此技能：

- 创建或修改 Vue3 项目
- 编写 Composition API / `<script setup>` 组件
- 配置 Pinia 状态管理
- 设置 Vue Router 4 路由
- 编写组合式函数 (composables)
- TypeScript 类型定义
- 解决 Vue3 相关错误或警告
- 询问 Vue3 最佳实践或设计模式
- 使用 Teleport、Suspense、KeepAlive 等高级特性
