---
name: react-frontend-dev
description: 精通 React 前端开发的专业技能。当用户需要创建、修改、调试 React 项目，或询问函数组件、Hooks、状态管理、路由配置、性能优化等前端任务时使用此技能。支持 React 18+ 特性。
---

# React 前端开发专家

你是一名精通 React 的前端开发专家，能够帮助用户高效完成 React 项目开发任务。

## 核心能力

- **React 项目搭建** - Create React App、Vite 项目初始化
- **组件开发** - 函数组件、类组件、JSX 语法
- **Hooks** - useState、useEffect、useContext、自定义 Hooks
- **状态管理** - Redux Toolkit、Zustand、Context API
- **路由配置** - React Router v6
- **性能优化** - useMemo、useCallback、代码分割、懒加载
- **TypeScript 支持** - 类型定义与类型安全
- **调试排错** - React 常见问题诊断与修复

## 项目初始化

### 使用 Vite (推荐)

```bash
# 创建 React + TypeScript 项目
npm create vite@latest my-app -- --template react-ts

cd my-app
npm install
npm run dev
```

### 使用 Create React App

```bash
# 创建项目
npx create-react-app my-app
npx create-react-app my-app --template typescript

cd my-app
npm start
```

### 手动配置

```bash
npm init -y
npm install react react-dom react-router-dom
npm install -D vite @vitejs/plugin-react @types/react @types/react-dom
```

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    open: true
  }
})
```

## 组件开发规范

### 函数组件 (推荐)

```tsx
import React, { useState, useEffect, useCallback, useMemo } from 'react'

interface Props {
  title: string
  count?: number
  items?: Array<{ id: number; name: string }>
  onUpdate?: (value: number) => void
}

const MyComponent: React.FC<Props> = ({
  title,
  count = 0,
  items = [],
  onUpdate
}) => {
  // 状态
  const [localCount, setLocalCount] = useState<number>(count)
  const [message, setMessage] = useState<string>('')
  const [loading, setLoading] = useState<boolean>(false)

  // 计算属性
  const doubled = useMemo(() => localCount * 2, [localCount])
  
  const filteredItems = useMemo(() => {
    return items.filter(item => item.id > 0)
  }, [items])

  // 副作用
  useEffect(() => {
    console.log('组件已挂载')
    
    return () => {
      console.log('组件卸载')
    }
  }, [])

  useEffect(() => {
    console.log('count 变化:', localCount)
  }, [localCount])

  // 回调函数
  const handleClick = useCallback(() => {
    setLocalCount(prev => prev + 1)
    onUpdate?.(localCount + 1)
  }, [localCount, onUpdate])

  const handleInputChange = useCallback((e: React.ChangeEvent<HTMLInputElement>) => {
    setMessage(e.target.value)
  }, [])

  // 渲染
  return (
    <div className="component">
      <h2>{title}</h2>
      <p>Count: {localCount}</p>
      <p>Doubled: {doubled}</p>
      
      {/* 条件渲染 */}
      {loading && <div>加载中...</div>}
      
      {!loading && items.length === 0 && (
        <div>暂无数据</div>
      )}
      
      {!loading && items.length > 0 && (
        <ul>
          {filteredItems.map(item => (
            <li key={item.id}>{item.name}</li>
          ))}
        </ul>
      )}
      
      {/* 事件绑定 */}
      <button onClick={handleClick}>更新</button>
      
      {/* 表单绑定 */}
      <input
        type="text"
        value={message}
        onChange={handleInputChange}
        placeholder="输入消息"
      />
    </div>
  )
}

export default MyComponent
```

### 类组件

```tsx
import React, { Component, ErrorInfo } from 'react'

interface Props {
  title: string
  count?: number
}

interface State {
  localCount: number
  message: string
  loading: boolean
}

class MyClassComponent extends Component<Props, State> {
  state: State = {
    localCount: this.props.count ?? 0,
    message: '',
    loading: false
  }

  // 生命周期
  componentDidMount() {
    console.log('组件已挂载')
    this.fetchData()
  }

  componentDidUpdate(prevProps: Props, prevState: State) {
    if (prevProps.count !== this.props.count) {
      this.setState({ localCount: this.props.count ?? 0 })
    }
  }

  componentWillUnmount() {
    console.log('组件将卸载')
  }

  // 错误处理
  static getDerivedStateFromError(error: Error) {
    return { loading: false }
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('错误:', error, errorInfo)
  }

  // 方法
  handleClick = () => {
    this.setState(prevState => ({
      localCount: prevState.localCount + 1
    }))
  }

  handleInputChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    this.setState({ message: e.target.value })
  }

  async fetchData() {
    this.setState({ loading: true })
    try {
      const res = await fetch('/api/data')
      const data = await res.json()
      // 处理数据
    } catch (e) {
      console.error(e)
    } finally {
      this.setState({ loading: false })
    }
  }

  render() {
    const { title } = this.props
    const { localCount, message, loading } = this.state

    return (
      <div className="component">
        <h2>{title}</h2>
        <p>Count: {localCount}</p>
        <input
          type="text"
          value={message}
          onChange={this.handleInputChange}
        />
        <button onClick={this.handleClick}>更新</button>
      </div>
    )
  }
}

export default MyClassComponent
```

## Hooks 详解

### useState

```tsx
import { useState } from 'react'

// 基础用法
const [count, setCount] = useState<number>(0)

// 对象状态
const [user, setUser] = useState<{ name: string; age: number }>({
  name: '',
  age: 0
})

// 函数式更新
setCount(prev => prev + 1)

// 更新对象
setUser(prev => ({ ...prev, name: 'New Name' }))
```

### useEffect

```tsx
import { useEffect } from 'react'

// 挂载时执行一次
useEffect(() => {
  console.log('挂载')
}, [])

// 依赖变化时执行
useEffect(() => {
  console.log('count 变化:', count)
}, [count])

// 清理副作用
useEffect(() => {
  const timer = setInterval(() => {}, 1000)
  
  return () => {
    clearInterval(timer)
  }
}, [])

// 异步操作
useEffect(() => {
  let cancelled = false
  
  const fetchData = async () => {
    const res = await fetch('/api/data')
    if (!cancelled) {
      const data = await res.json()
    }
  }
  
  fetchData()
  
  return () => {
    cancelled = true
  }
}, [])
```

### useContext

```tsx
import { createContext, useContext, useState, ReactNode } from 'react'

// 创建 Context
interface ThemeContextType {
  theme: 'light' | 'dark'
  toggleTheme: () => void
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined)

// Provider
export function ThemeProvider({ children }: { children: ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light')
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light')
  }
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  )
}

// 使用
export function useTheme() {
  const context = useContext(ThemeContext)
  if (!context) {
    throw new Error('useTheme 必须在 ThemeProvider 内使用')
  }
  return context
}

// 组件中使用
function MyComponent() {
  const { theme, toggleTheme } = useTheme()
  
  return (
    <div className={theme}>
      <button onClick={toggleTheme}>切换主题</button>
    </div>
  )
}
```

### useReducer

```tsx
import { useReducer } from 'react'

interface State {
  count: number
  loading: boolean
}

type Action =
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' }
  | { type: 'SET_COUNT'; payload: number }
  | { type: 'SET_LOADING'; payload: boolean }

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 }
    case 'DECREMENT':
      return { ...state, count: state.count - 1 }
    case 'SET_COUNT':
      return { ...state, count: action.payload }
    case 'SET_LOADING':
      return { ...state, loading: action.payload }
    default:
      return state
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0, loading: false })
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+1</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-1</button>
    </div>
  )
}
```

### useMemo 和 useCallback

```tsx
import { useMemo, useCallback } from 'react'

function ExpensiveComponent({ items, onSelect }: Props) {
  // 缓存计算结果
  const filteredItems = useMemo(() => {
    console.log('过滤计算')
    return items.filter(item => item.active)
  }, [items])

  // 缓存函数引用
  const handleClick = useCallback((id: number) => {
    onSelect(id)
  }, [onSelect])

  return (
    <ul>
      {filteredItems.map(item => (
        <li key={item.id} onClick={() => handleClick(item.id)}>
          {item.name}
        </li>
      ))}
    </ul>
  )
}
```

### 自定义 Hooks

```tsx
// hooks/useFetch.ts
import { useState, useEffect } from 'react'

interface UseFetchReturn<T> {
  data: T | null
  error: Error | null
  loading: boolean
  refetch: () => void
}

export function useFetch<T>(url: string): UseFetchReturn<T> {
  const [data, setData] = useState<T | null>(null)
  const [error, setError] = useState<Error | null>(null)
  const [loading, setLoading] = useState(true)

  const fetchData = async () => {
    setLoading(true)
    try {
      const res = await fetch(url)
      if (!res.ok) throw new Error(res.statusText)
      setData(await res.json())
    } catch (e) {
      setError(e as Error)
    } finally {
      setLoading(false)
    }
  }

  useEffect(() => {
    fetchData()
  }, [url])

  return { data, error, loading, refetch: fetchData }
}

// hooks/useLocalStorage.ts
import { useState, useEffect } from 'react'

export function useLocalStorage<T>(key: string, defaultValue: T) {
  const [value, setValue] = useState<T>(() => {
    const stored = localStorage.getItem(key)
    return stored ? JSON.parse(stored) : defaultValue
  })

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value))
  }, [key, value])

  return [value, setValue] as const
}

// hooks/useDebounce.ts
import { useState, useEffect } from 'react'

export function useDebounce<T>(value: T, delay: number = 500): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value)
    }, delay)

    return () => {
      clearTimeout(timer)
    }
  }, [value, delay])

  return debouncedValue
}

// hooks/useEvent.ts
import { useEffect } from 'react'

export function useEvent<K extends keyof WindowEventMap>(
  event: K,
  handler: (e: WindowEventMap[K]) => void,
  target: Window | Document | HTMLElement = window
) {
  useEffect(() => {
    target.addEventListener(event, handler as EventListener)
    
    return () => {
      target.removeEventListener(event, handler as EventListener)
    }
  }, [event, handler, target])
}
```

## 状态管理

### Redux Toolkit (推荐)

```typescript
// store/index.ts
import { configureStore, createSlice, createAsyncThunk } from '@reduxjs/toolkit'
import type { PayloadAction } from '@reduxjs/toolkit'

// 异步 Thunk
export const fetchUser = createAsyncThunk(
  'user/fetch',
  async (userId: number) => {
    const res = await fetch(`/api/users/${userId}`)
    return res.json()
  }
)

// Slice
const userSlice = createSlice({
  name: 'user',
  initialState: {
    data: null as User | null,
    loading: false,
    error: null as string | null
  },
  reducers: {
    setName(state, action: PayloadAction<string>) {
      if (state.data) {
        state.data.name = action.payload
      }
    },
    clearUser(state) {
      state.data = null
      state.error = null
    }
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.loading = true
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.loading = false
        state.data = action.payload
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.loading = false
        state.error = action.error.message ?? '请求失败'
      })
  }
})

export const { setName, clearUser } = userSlice.actions

export const store = configureStore({
  reducer: {
    user: userSlice.reducer
  }
})

export type RootState = ReturnType<typeof store.getState>
export type AppDispatch = typeof store.dispatch
```

```tsx
// 组件中使用
import { useSelector, useDispatch } from 'react-redux'
import type { RootState, AppDispatch } from './store'

function MyComponent() {
  const dispatch = useDispatch<AppDispatch>()
  const user = useSelector((state: RootState) => state.user.data)
  const loading = useSelector((state: RootState) => state.user.loading)

  return (
    <div>
      {loading && <div>加载中...</div>}
      {user && <div>{user.name}</div>}
      <button onClick={() => dispatch(fetchUser(1))}>获取用户</button>
      <button onClick={() => dispatch(setName('New Name'))}>更新名字</button>
    </div>
  )
}
```

### Zustand (轻量级)

```typescript
// store/useUserStore.ts
import { create } from 'zustand'

interface User {
  id: number
  name: string
  email: string
}

interface UserStore {
  user: User | null
  loading: boolean
  setUser: (user: User) => void
  clearUser: () => void
  fetchUser: (id: number) => Promise<void>
}

export const useUserStore = create<UserStore>((set) => ({
  user: null,
  loading: false,
  
  setUser: (user) => set({ user }),
  
  clearUser: () => set({ user: null }),
  
  fetchUser: async (id) => {
    set({ loading: true })
    try {
      const res = await fetch(`/api/users/${id}`)
      const user = await res.json()
      set({ user, loading: false })
    } catch (e) {
      set({ loading: false })
    }
  }
}))

// 组件中使用
function MyComponent() {
  const { user, loading, fetchUser, setUser } = useUserStore()
  
  return (
    <div>
      {user?.name}
      <button onClick={() => fetchUser(1)}>获取</button>
    </div>
  )
}
```

## React Router v6

### 基础配置

```tsx
// App.tsx
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom'

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/user/:id" element={<User />} />
        <Route path="/search" element={<Search />} />
        <Route path="*" element={<Navigate to="/" replace />} />
      </Routes>
    </BrowserRouter>
  )
}
```

### 路由参数

```tsx
import { useParams, useSearchParams, useNavigate } from 'react-router-dom'

function User() {
  const { id } = useParams<{ id: string }>()
  const [searchParams, setSearchParams] = useSearchParams()
  const navigate = useNavigate()
  
  const query = searchParams.get('q')
  
  return (
    <div>
      <p>User ID: {id}</p>
      <p>Query: {query}</p>
      <button onClick={() => navigate('/about')}>去关于页</button>
      <button onClick={() => navigate(-1)}>返回</button>
    </div>
  )
}
```

### 嵌套路由

```tsx
// App.tsx
<Route path="/dashboard" element={<Dashboard />}>
  <Route index element={<DashboardHome />} />
  <Route path="stats" element={<Stats />} />
  <Route path="settings" element={<Settings />} />
</Route>

// Dashboard.tsx
import { Outlet } from 'react-router-dom'

function Dashboard() {
  return (
    <div>
      <nav>
        <Link to="stats">统计</Link>
        <Link to="settings">设置</Link>
      </nav>
      <Outlet />
    </div>
  )
}
```

### 路由守卫

```tsx
// components/ProtectedRoute.tsx
import { Navigate, useLocation } from 'react-router-dom'

interface Props {
  children: React.ReactNode
  requireAuth?: boolean
}

function ProtectedRoute({ children, requireAuth = true }: Props) {
  const isAuthenticated = localStorage.getItem('token')
  const location = useLocation()
  
  if (requireAuth && !isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />
  }
  
  return <>{children}</>
}

// 使用
<Route path="/profile" element={
  <ProtectedRoute>
    <Profile />
  </ProtectedRoute>
} />
```

## 性能优化

### React.memo

```tsx
import React, { memo } from 'react'

const ExpensiveComponent = memo(({ data }: Props) => {
  return <div>{data}</div>
}, (prevProps, nextProps) => {
  // 自定义比较
  return prevProps.data === nextProps.data
})
```

### 代码分割

```tsx
// 懒加载组件
import { lazy, Suspense } from 'react'

const HeavyComponent = lazy(() => import('./HeavyComponent'))

function App() {
  return (
    <Suspense fallback={<div>加载中...</div>}>
      <HeavyComponent />
    </Suspense>
  )
}

// 路由级别代码分割
const About = lazy(() => import('./pages/About'))

<Route path="/about" element={
  <Suspense fallback={<div>加载中...</div>}>
    <About />
  </Suspense>
} />
```

### 虚拟列表

```tsx
// 使用 react-window
import { FixedSizeList as List } from 'react-window'

function VirtualList({ items }: Props) {
  const Row = ({ index, style }: { index: number; style: React.CSSProperties }) => (
    <div style={style}>
      {items[index].name}
    </div>
  )
  
  return (
    <List
      height={600}
      itemCount={items.length}
      itemSize={50}
    >
      {Row}
    </List>
  )
}
```

## 高级模式

### 受控组件

```tsx
function Form() {
  const [formData, setFormData] = useState({
    name: '',
    email: ''
  })
  
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setFormData(prev => ({
      ...prev,
      [e.target.name]: e.target.value
    }))
  }
  
  return (
    <form>
      <input
        name="name"
        value={formData.name}
        onChange={handleChange}
      />
      <input
        name="email"
        value={formData.email}
        onChange={handleChange}
      />
    </form>
  )
}
```

### 非受控组件

```tsx
import { useRef } from 'react'

function Form() {
  const inputRef = useRef<HTMLInputElement>(null)
  
  const handleSubmit = () => {
    console.log(inputRef.current?.value)
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <input ref={inputRef} defaultValue="" />
      <button type="submit">提交</button>
    </form>
  )
}
```

### 错误边界

```tsx
import { Component, ErrorInfo, ReactNode } from 'react'

interface Props {
  children: ReactNode
  fallback?: ReactNode
}

interface State {
  hasError: boolean
}

class ErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false }
  
  static getDerivedStateFromError(): State {
    return { hasError: true }
  }
  
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('错误:', error, errorInfo)
  }
  
  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? <div>出错了</div>
    }
    
    return this.props.children
  }
}

// 使用
<ErrorBoundary fallback={<ErrorFallback />}>
  <Component />
</ErrorBoundary>
```

### Portals

```tsx
import { createPortal } from 'react-dom'

function Modal({ children }: Props) {
  return createPortal(
    <div className="modal">{children}</div>,
    document.getElementById('modal-root')!
  )
}
```

## TypeScript 支持

### 组件类型定义

```tsx
import React, { FC, ReactNode } from 'react'

interface Props {
  title: string
  count?: number
  children?: ReactNode
  onClick?: () => void
}

const MyComponent: FC<Props> = ({ title, count = 0, children, onClick }) => {
  return <div onClick={onClick}>{title}</div>
}
```

### 事件类型

```tsx
function Form() {
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault()
  }
  
  const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
    console.log(e.clientX, e.clientY)
  }
  
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    console.log(e.target.value)
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <input onChange={handleChange} />
      <button onClick={handleClick}>点击</button>
    </form>
  )
}
```

## 调试技巧

### React DevTools

- 安装 React Developer Tools 浏览器扩展
- 检查组件树、Props、State
- 使用 Profiler 分析性能

### 常见错误排查

1. **Hooks 规则违反**
   - Hooks 只能在函数组件顶层调用
   - 不能在条件语句或循环中调用 Hooks

2. **状态更新问题**
   - 状态更新是异步的
   - 使用函数式更新处理依赖前一个状态的情况

3. **内存泄漏**
   - useEffect 中清理副作用
   - 移除事件监听器、取消请求

```tsx
useEffect(() => {
  const controller = new AbortController()
  
  fetch('/api/data', { signal: controller.signal })
    .then(res => res.json())
    .catch(e => {
      if (e.name !== 'AbortError') {
        console.error(e)
      }
    })
  
  return () => {
    controller.abort()
  }
}, [])
```

## 最佳实践

1. **使用函数组件 + Hooks** - React 18 推荐方式
2. **自定义 Hooks 复用逻辑** - 抽取可复用逻辑
3. **TypeScript 类型安全** - 使用 TS 获得更好的开发体验
4. **状态管理选择** - 简单用 Context，复杂用 Redux Toolkit 或 Zustand
5. **性能优化** - 使用 React.memo、useMemo、useCallback
6. **代码分割** - 使用 lazy + Suspense
7. **ESLint + Prettier** - 保持代码风格一致
8. **单元测试** - 使用 Jest + React Testing Library

## 项目结构建议

```
src/
├── assets/          # 静态资源
├── components/      # 通用组件
│   ├── ui/         # UI 基础组件
│   └── business/   # 业务组件
├── hooks/          # 自定义 Hooks
├── pages/          # 页面组件
├── store/          # 状态管理
├── router/         # 路由配置
├── types/          # TypeScript 类型定义
├── utils/          # 工具函数
├── styles/         # 全局样式
├── App.tsx
└── main.tsx
```

## 触发此技能

当用户需要以下帮助时主动使用此技能：

- 创建或修改 React 项目
- 编写函数组件或类组件
- 使用 Hooks (useState、useEffect、useContext 等)
- 配置 Redux Toolkit 或 Zustand 状态管理
- 设置 React Router v6 路由
- 编写自定义 Hooks
- TypeScript 类型定义
- 解决 React 相关错误或警告
- 询问 React 最佳实践或设计模式
- 性能优化建议
