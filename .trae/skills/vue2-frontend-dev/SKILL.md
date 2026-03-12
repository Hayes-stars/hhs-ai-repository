---
name: vue2-frontend-dev
description: 精通 Vue2 前端开发的专业技能。当用户需要创建、修改、调试 Vue2 项目，或询问 Options API 组件开发、Vuex 状态管理、Vue Router 3 路由配置时使用此技能。
---

# Vue2 前端开发专家

你是一名精通 Vue2 的前端开发专家，能够帮助用户高效完成 Vue2 项目开发任务。

## 核心能力

- **Vue2 项目搭建** - Vue CLI 项目初始化与配置
- **组件开发** - Options API、单文件组件 (.vue)、混入 (mixins)
- **状态管理** - Vuex 4
- **路由配置** - Vue Router 3
- **性能优化** - 代码分割、懒加载、渲染优化
- **调试排错** - Vue2 常见问题诊断与修复

## 项目初始化

### Vue CLI 创建项目

```bash
# 安装 Vue CLI
npm install -g @vue/cli

# 创建项目
vue create my-app

# 选择预设时选择 Vue 2
# 推荐配置：Babel, Router, Vuex, CSS Pre-processors
```

### 手动配置 Vue2

```bash
npm init -y
npm install vue@2 vue-router@3 vuex@4
npm install -D @vue/cli-service webpack
```

## 组件开发规范

### Options API (Vue2 标准)

```vue
<script>
export default {
  name: 'MyComponent',
  
  // 组件选项
  components: {
    ChildComponent
  },
  
  // Props 定义
  props: {
    title: {
      type: String,
      required: true
    },
    count: {
      type: Number,
      default: 0
    },
    items: {
      type: Array,
      default: () => []
    }
  },
  
  // 响应式数据
  data() {
    return {
      localCount: this.count,
      message: '',
      loading: false
    }
  },
  
  // 计算属性 (带缓存)
  computed: {
    doubled() {
      return this.localCount * 2
    },
    
    filteredItems() {
      return this.items.filter(item => item.active)
    },
    
    // 带 setter 的计算属性
    fullName: {
      get() {
        return `${this.firstName} ${this.lastName}`
      },
      set(value) {
        const [first, last] = value.split(' ')
        this.firstName = first
        this.lastName = last
      }
    }
  },
  
  // 监听器
  watch: {
    // 监听 prop 变化
    count(newVal, oldVal) {
      this.localCount = newVal
    },
    
    // 深度监听对象
    items: {
      handler(newVal) {
        console.log('items 变化:', newVal)
      },
      deep: true,
      immediate: true
    }
  },
  
  // 方法
  methods: {
    handleClick() {
      this.localCount++
      this.$emit('update', this.localCount)
    },
    
    async fetchData() {
      this.loading = true
      try {
        const res = await fetch('/api/data')
        this.data = await res.json()
      } catch (e) {
        console.error(e)
      } finally {
        this.loading = false
      }
    }
  },
  
  // 生命周期钩子
  beforeCreate() {
    // 实例初始化之前
  },
  
  created() {
    // 实例创建完成，可访问 data/computed/methods
    this.fetchData()
  },
  
  beforeMount() {
    // 挂载开始之前
  },
  
  mounted() {
    // 组件挂载完成，可访问 DOM
    console.log('组件已挂载')
  },
  
  beforeUpdate() {
    // 数据更新、DOM 更新之前
  },
  
  updated() {
    // DOM 更新完成
  },
  
  beforeDestroy() {
    // 实例销毁之前，清理定时器、事件监听器
  },
  
  destroyed() {
    // 实例销毁完成
  }
}
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
      <div v-for="(item, index) in filteredItems" :key="item.id || index">
        {{ item.name }}
      </div>
    </div>
    
    <!-- 事件绑定 -->
    <button @click="handleClick">更新</button>
    <button @click.stop="handleClick">阻止冒泡</button>
    <button @click.prevent="handleClick">阻止默认</button>
    
    <!-- 表单绑定 -->
    <input v-model="message" placeholder="输入消息" />
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

## 混入 (Mixins)

```javascript
// mixins/formMixin.js
export default {
  data() {
    return {
      formErrors: {},
      submitting: false
    }
  },
  
  methods: {
    validateForm() {
      this.formErrors = {}
      // 验证逻辑
      return Object.keys(this.formErrors).length === 0
    },
    
    async submitForm(endpoint) {
      if (!this.validateForm()) return
      
      this.submitting = true
      try {
        await this.$axios.post(endpoint, this.form)
        this.$message.success('提交成功')
      } catch (e) {
        this.$message.error('提交失败')
      } finally {
        this.submitting = false
      }
    }
  }
}
```

```vue
<script>
import formMixin from '@/mixins/formMixin'

export default {
  mixins: [formMixin],
  
  data() {
    return {
      form: {
        name: '',
        email: ''
      }
    }
  }
}
</script>
```

## Vuex 状态管理

### Store 结构

```javascript
// store/index.js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    count: 0,
    user: null,
    items: []
  },
  
  getters: {
    // 基础 getter
    count: state => state.count,
    
    // 带参数的 getter
    getItemById: state => id => {
      return state.items.find(item => item.id === id)
    },
    
    // 使用其他 getter
    doubledCount: (state, getters) => {
      return getters.count * 2
    }
  },
  
  mutations: {
    // 同步修改 state
    increment(state) {
      state.count++
    },
    
    incrementBy(state, payload) {
      state.count += payload.amount
    },
    
    setUser(state, user) {
      state.user = user
    },
    
    // 使用常量
    [types.SET_ITEMS](state, items) {
      state.items = items
    }
  },
  
  actions: {
    // 异步操作
    async incrementAsync({ commit, state }) {
      return new Promise((resolve) => {
        setTimeout(() => {
          commit('increment')
          resolve(state.count)
        }, 1000)
      })
    },
    
    // 获取数据
    async fetchUser({ commit }, userId) {
      try {
        const res = await Vue.http.get(`/api/users/${userId}`)
        commit('setUser', res.data)
      } catch (e) {
        throw e
      }
    },
    
    // 模块化 action
    async loadData({ commit, dispatch, state, getters }) {
      await dispatch('fetchUser', 1)
      commit('increment')
    }
  },
  
  modules: {
    // 模块
    cart: {
      namespaced: true,
      state: {
        items: []
      },
      mutations: {
        add(state, item) {
          state.items.push(item)
        }
      },
      actions: {
        add({ commit }, item) {
          commit('add', item)
        }
      },
      getters: {
        total: state => {
          return state.items.reduce((sum, item) => sum + item.price, 0)
        }
      }
    }
  }
})
```

### 组件中使用 Vuex

```vue
<script>
import { mapState, mapGetters, mapMutations, mapActions } from 'vuex'

export default {
  computed: {
    // 展开 state
    ...mapState(['count', 'user']),
    
    // 展开 getters
    ...mapGetters(['doubledCount']),
    
    // 带命名空间的映射
    ...mapState('cart', ['items']),
    ...mapGetters('cart', ['total'])
  },
  
  methods: {
    // 展开 mutations
    ...mapMutations(['increment', 'setUser']),
    
    // 展开 actions
    ...mapActions(['incrementAsync', 'fetchUser']),
    
    // 带别名
    ...mapMutations({
      add: 'increment'
    })
  },
  
  // 或直接使用
  mounted() {
    this.$store.state.count
    this.$store.getters.doubledCount
    this.$store.commit('increment')
    this.$store.dispatch('incrementAsync')
  }
}
</script>
```

## Vue Router 3 路由配置

### 基础配置

```javascript
// router/index.js
import Vue from 'vue'
import VueRouter from 'vue-router'
import Home from '@/views/Home.vue'

Vue.use(VueRouter)

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home,
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
    props: true, // 将 params 转为 props
    meta: {
      requiresAuth: true
    }
  },
  {
    path: '/search',
    name: 'Search',
    component: () => import('@/views/Search.vue'),
    props: route => ({ query: route.query.q })
  },
  {
    path: '*',
    redirect: '/'
  }
]

const router = new VueRouter({
  mode: 'history', // 或 'hash'
  base: process.env.BASE_URL,
  routes,
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      return savedPosition
    } else {
      return { x: 0, y: 0 }
    }
  }
})

export default router
```

### 路由守卫

```javascript
// 全局前置守卫
router.beforeEach((to, from, next) => {
  document.title = to.meta.title || '默认标题'
  
  const isAuthenticated = localStorage.getItem('token')
  
  if (to.meta.requiresAuth && !isAuthenticated) {
    next({
      path: '/login',
      query: { redirect: to.fullPath }
    })
  } else {
    next()
  }
})

// 全局解析守卫
router.beforeResolve((to, from, next) => {
  // 在所有守卫之后调用
  next()
})

// 全局后置守卫
router.afterEach((to, from) => {
  // 无需调用 next()
  console.log('路由跳转完成')
})

// 组件内守卫
export default {
  beforeRouteEnter(to, from, next) {
    // 无法访问 this
    next(vm => {
      // 可以访问组件实例
    })
  },
  
  beforeRouteUpdate(to, from, next) {
    // 路由复用组件时调用
    next()
  },
  
  beforeRouteLeave(to, from, next) {
    // 离开路由时调用
    if (this.formChanged) {
      const answer = window.confirm('有未保存的更改，确定离开？')
      next(answer)
    } else {
      next()
    }
  }
}
```

### 编程式导航

```javascript
export default {
  methods: {
    goToHome() {
      this.$router.push('/')
      this.$router.push({ name: 'Home' })
      this.$router.push({ path: '/user', query: { id: 1 } })
    },
    
    replacePage() {
      this.$router.replace('/about')
    },
    
    goBack() {
      this.$router.go(-1)
      this.$router.back()
    },
    
    goForward() {
      this.$router.forward()
    }
  }
}
```

## 常用工具函数

### 事件总线 (Event Bus)

```javascript
// utils/eventBus.js
import Vue from 'vue'
export const EventBus = new Vue()

// 使用
// 发送：EventBus.$emit('event-name', payload)
// 接收：EventBus.$on('event-name', handler)
// 移除：EventBus.$off('event-name', handler)
```

### 自定义指令

```javascript
// directives/focus.js
export default {
  inserted(el) {
    el.focus()
  }
}

// main.js
import focus from '@/directives/focus'
Vue.directive('focus', focus)

// 使用：<input v-focus />
```

### 过滤器

```javascript
// filters/currency.js
export function currency(value) {
  if (!value) return '¥0.00'
  return '¥' + parseFloat(value).toFixed(2)
}

// main.js
import { currency } from '@/filters/currency'
Vue.filter('currency', currency)

// 使用：{{ price | currency }}
```

## 性能优化

### 组件懒加载

```javascript
// 路由级别
const User = () => import('@/views/User.vue')

// 组件级别
const HeavyComponent = () => import('@/components/HeavyComponent.vue')

// 带 loading 组件
const AsyncComponent = () => ({
  component: import('@/components/Heavy.vue'),
  loading: LoadingComponent,
  error: ErrorComponent,
  delay: 200,
  timeout: 3000
})
```

### 列表优化

```vue
<template>
  <!-- 必须使用 key -->
  <div v-for="item in items" :key="item.id">
    {{ item.name }}
  </div>
  
  <!-- 使用 v-once 静态内容 -->
  <div v-once>
    静态内容，不会更新
  </div>
  
  <!-- 使用 v-if 而非 v-show 处理不常用切换 -->
  <div v-if="isVisible">内容</div>
</template>
```

### 防抖节流

```javascript
// utils/debounce.js
export function debounce(fn, delay) {
  let timer = null
  return function(...args) {
    if (timer) clearTimeout(timer)
    timer = setTimeout(() => fn.apply(this, args), delay)
  }
}

// 使用
export default {
  methods: {
    handleInput: debounce(function(e) {
      this.search(e.target.value)
    }, 300)
  }
}
```

## 常见模式

### 父子组件通信

```vue
<!-- 父组件 -->
<template>
  <ChildComponent
    ref="child"
    :title="title"
    @custom-event="handleEvent"
  />
  <button @click="$refs.child.doSomething()">调用子组件方法</button>
</template>

<script>
export default {
  data() {
    return {
      title: '标题'
    }
  },
  methods: {
    handleEvent(data) {
      console.log('子组件事件:', data)
    }
  }
}
</script>
```

```vue
<!-- 子组件 -->
<script>
export default {
  props: ['title'],
  methods: {
    doSomething() {
      this.$emit('custom-event', { data: 'from child' })
    }
  }
}
</script>
```

### Provide / Inject

```javascript
// 祖先组件
export default {
  provide() {
    return {
      theme: this.theme,
      updateTheme: this.updateTheme
    }
  },
  data() {
    return {
      theme: 'dark'
    }
  },
  methods: {
    updateTheme(newTheme) {
      this.theme = newTheme
    }
  }
}

// 后代组件
export default {
  inject: ['theme', 'updateTheme'],
  mounted() {
    console.log(this.theme)
  }
}
```

### 作用域插槽

```vue
<!-- 子组件 -->
<template>
  <div>
    <slot :item="item" :index="index"></slot>
  </div>
</template>

<!-- 父组件 -->
<template>
  <ChildComponent v-slot="{ item, index }">
    <div>{{ index }} - {{ item.name }}</div>
  </ChildComponent>
</template>
```

## 调试技巧

### Vue DevTools

- 安装 Vue Devtools 浏览器扩展
- 检查组件树、状态、事件
- 使用 Time Travel 调试状态变化

### 常见错误排查

1. **响应式问题**
   - Vue2 无法检测对象属性的添加/删除
   - 使用 `Vue.set(obj, key, value)` 或 `this.$set`
   - 数组变更使用变异方法 (push, pop, splice 等)

2. **this 指向问题**
   - 回调函数中使用箭头函数或 `bind(this)`
   - 方法中保持 this 指向

3. **内存泄漏**
   - beforeDestroy 中清理定时器、事件监听器
   - 移除 EventBus 监听

```javascript
export default {
  data() {
    return {
      timer: null
    }
  },
  mounted() {
    this.timer = setInterval(() => {}, 1000)
    EventBus.$on('event', this.handler)
  },
  beforeDestroy() {
    clearInterval(this.timer)
    EventBus.$off('event', this.handler)
  }
}
```

## 最佳实践

1. **组件命名** - 使用 PascalCase 或多词 kebab-case
2. **Props 验证** - 始终定义 props 类型和默认值
3. **Key 属性** - v-for 必须使用唯一 key
4. **避免 v-if + v-for** - 使用计算属性过滤
5. **CSS 作用域** - 使用 scoped 避免样式污染
6. **代码组织** - 按选项顺序组织组件代码
7. **错误处理** - 使用 errorCaptured 钩子

## 项目结构建议

```
src/
├── assets/          # 静态资源
├── components/      # 通用组件
│   ├── base/       # 基础组件 (Button, Input)
│   └── business/   # 业务组件
├── views/          # 页面组件
├── router/         # 路由配置
├── store/          # Vuex 状态管理
├── mixins/         # 混入
├── directives/     # 自定义指令
├── filters/        # 过滤器
├── utils/          # 工具函数
├── api/            # API 请求
├── styles/         # 全局样式
└── App.vue
```

## 触发此技能

当用户需要以下帮助时主动使用此技能：

- 创建或修改 Vue2 项目
- 编写 Options API 组件
- 配置 Vuex 状态管理
- 设置 Vue Router 3 路由
- 解决 Vue2 相关错误或警告
- 询问 Vue2 最佳实践或设计模式
- 混入、过滤器、自定义指令相关开发
