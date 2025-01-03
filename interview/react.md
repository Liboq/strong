## React 面试题

### 场景一：类组件 vs 函数组件

text
面试官：你能比较一下类组件和函数组件的区别吗？在实际项目中如何选择？

候选人：
好的，我可以结合实际项目经验来说明。在我们最近的一个项目重构中，就经历了从类组件到函数组件的迁移过程。

主要区别体现在几个方面：

首先是代码组织方式。类组件通过生命周期方法来组织代码，而函数组件使用 Hooks。比如说，我们之前有一个数据加载的逻辑：

````javascript
// 类组件示例
class UserProfile extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      user: null
    }
    // 需要绑定this
    this.handleClick = this.handleClick.bind(this)
  }

  componentDidMount() {
    this.fetchUser()
  }

  componentDidUpdate(prevProps) {
    if (prevProps.userId !== this.props.userId) {
      this.fetchUser()
    }
  }

  componentWillUnmount() {
    // 清理工作
  }

  async fetchUser() {
    const user = await fetchUserData(this.props.userId)
    this.setState({ user })
  }

  handleClick() {
    // 处理点击事件
  }

  render() {
    return <div onClick={this.handleClick}>{this.state.user?.name}</div>
  }
}


而使用函数组件后，代码变得更加简洁和直观：

```javascript
// 函数组件示例
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    const fetchUser = async () => {
      const user = await fetchUserData(userId);
      setUser(user);
    };

    fetchUser();

    // 清理函数
    return () => setUser(null);
  }, [userId]);

  const handleClick = () => {
    // 处理点击事件
  };

  return <div onClick={handleClick}>{user?.name}</div>;
}
````

这种改变带来了几个明显的好处：

1. 代码更容易理解和维护，相关的逻辑可以放在一起
2. 不需要处理 this 的绑定问题
3. 通过自定义 Hook 更容易复用逻辑

在我们的实践中，对于新的组件，我们都优先使用函数组件，主要考虑：

1. 代码更简洁，易于维护
2. Hook 的逻辑复用机制更灵活
3. 未来 React 的优化方向主要是函数组件

### 场景二：状态管理方案

````text
面试官：在你的项目中是如何进行状态管理的？能具体谈谈 Redux 和 Context 的使用场景吗？

候选人：
好的，我们在项目中根据不同场景选择不同的状态管理方案。

对于简单的状态共享，我们使用 Context API。比如主题切换功能：
```javascript
// ThemeContext.js
const ThemeContext = React.createContext({
  theme: 'light',
  toggleTheme: () => {}
})

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light')

  const toggleTheme = useCallback(() => {
    setTheme(t => t === 'light' ? 'dark' : 'light')
  }, [])

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  )
}


而对于复杂的状态管理，特别是涉及到异步操作和多组件共享的状态，我们会使用 Redux：

```javascript
// userSlice.js
import { createSlice, createAsyncThunk } from "@reduxjs/toolkit";

export const fetchUserProfile = createAsyncThunk(
  "user/fetchProfile",
  async (userId) => {
    const response = await fetch(`/api/users/${userId}`);
    return response.json();
  }
);

const userSlice = createSlice({
  name: "user",
  initialState: {
    profile: null,
    loading: false,
    error: null,
  },
  reducers: {
    updateProfile: (state, action) => {
      state.profile = action.payload;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUserProfile.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchUserProfile.fulfilled, (state, action) => {
        state.loading = false;
        state.profile = action.payload;
      });
  },
});
````

选择状态管理方案时，我们主要考虑以下因素：

1. 状态的作用范围：

   - 组件内部状态：useState
   - 小范围共享：Context API
   - 全局状态：Redux

2. 状态的复杂度：

   - 简单状态：Context 足够
   - 复杂状态：Redux 更合适

3. 团队因素：
   - 团队规模
   - 开发习惯
   - 维护成本

### 场景三：性能优化实践

````text
面试官：你在 React 项目中做过哪些性能优化？能举一些具体的例子吗？

候选人：
好的，我可以分享一下在我们项目中实践的几个性能优化方案。

1. 针对大列表渲染的优化：
```javascript
import { FixedSizeList } from 'react-window'

function VirtualizedList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      <ListItem data={items[index]} />
    </div>
  )

  return (
    <FixedSizeList
      height={400}
      width="100%"
      itemCount={items.length}
      itemSize={50}
    >
      {Row}
    </FixedSizeList>
  )
}


2. 使用 memo 和 useMemo 避免不必要的重渲染：

```javascript
const ExpensiveComponent = memo(function ExpensiveComponent({
  data,
  onItemClick,
}) {
  const processedData = useMemo(() => {
    return data.map((item) => ({
      ...item,
      fullName: `${item.firstName} ${item.lastName}`,
    }));
  }, [data]);

  const handleClick = useCallback(
    (id) => {
      onItemClick(id);
    },
    [onItemClick]
  );

  return (
    <ul>
      {processedData.map((item) => (
        <li key={item.id} onClick={() => handleClick(item.id)}>
          {item.fullName}
        </li>
      ))}
    </ul>
  );
});
````

3. 代码分割和懒加载：

```javascript
const AdminDashboard = lazy(() => import("./AdminDashboard"));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Switch>
        <Route path="/admin" component={AdminDashboard} />
      </Switch>
    </Suspense>
  );
}
```

4. 状态更新优化：

```javascript
function Counter() {
  const [count, setCount] = useState(0);

  // 批量更新
  const handleClick = () => {
    setCount((c) => c + 1); // 使用函数式更新
    setCount((c) => c + 1); // 这些更新会被批处理
    setCount((c) => c + 1);
  };

  return <button onClick={handleClick}>{count}</button>;
}
```

这些优化带来的效果：

1. 虚拟列表将渲染时间从 300ms 降到了 50ms
2. 代码分割将首屏 JS 体积减少了 40%
3. memo 和 useMemo 减少了 60% 的不必要渲染

当然，性能优化是一个持续的过程，我们会根据性能监控的数据来决定优化的重点。

### 场景四：Hooks 最佳实践

````text
面试官：能谈谈你在项目中是如何使用 Hooks 的？有哪些最佳实践？

候选人：
好的，我来分享一下我们团队在使用 Hooks 时总结的一些经验。

1. 自定义 Hooks 封装复杂逻辑：
```javascript
// useAsync.js
function useAsync(asyncFunction) {
  const [data, setData] = useState(null)
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState(null)

  const execute = useCallback(async (...args) => {
    try {
      setLoading(true)
      setError(null)
      const result = await asyncFunction(...args)
      setData(result)
      return result
    } catch (error) {
      setError(error)
      throw error
    } finally {
      setLoading(false)
    }
  }, [asyncFunction])

  return { data, loading, error, execute }
}

// 使用示例
function UserProfile({ userId }) {
  const {
    data: user,
    loading,
    error,
    execute: fetchUser
  } = useAsync(fetchUserProfile)

  useEffect(() => {
    fetchUser(userId)
  }, [userId, fetchUser])

  if (loading) return <Spinner />
  if (error) return <ErrorMessage error={error} />
  if (!user) return null

  return <div>{user.name}</div>
}


2. 使用 useReducer 处理复杂状态：

```javascript
function formReducer(state, action) {
  switch (action.type) {
    case "SET_FIELD":
      return {
        ...state,
        [action.field]: action.value,
        errors: {
          ...state.errors,
          [action.field]: null,
        },
      };
    case "SET_ERROR":
      return {
        ...state,
        errors: {
          ...state.errors,
          [action.field]: action.error,
        },
      };
    case "RESET":
      return initialState;
    default:
      return state;
  }
}

function RegistrationForm() {
  const [state, dispatch] = useReducer(formReducer, {
    username: "",
    password: "",
    errors: {},
  });

  const handleChange = (field) => (event) => {
    dispatch({
      type: "SET_FIELD",
      field,
      value: event.target.value,
    });
  };
}
````

3. 合理使用 useEffect 的依赖：

```javascript
function SearchResults({ query }) {
  const [results, setResults] = useState([]);

  // 使用 useCallback 缓存函数
  const fetchResults = useCallback(async (searchQuery) => {
    const data = await searchApi(searchQuery);
    setResults(data);
  }, []); // 空依赖数组，因为函数不依赖任何外部变量

  // 使用防抖优化请求
  const debouncedFetch = useMemo(
    () => debounce(fetchResults, 300),
    [fetchResults]
  );

  useEffect(() => {
    if (query) {
      debouncedFetch(query);
    }
    // 清理函数
    return () => debouncedFetch.cancel();
  }, [query, debouncedFetch]);
}
```

我们总结的一些最佳实践：

1. 命名规范：

   - 自定义 Hook 必须以 use 开头
   - 语义化命名，表明 Hook 的用途

2. 依赖处理：

   - 合理使用依赖数组
   - 使用 useCallback/useMemo 优化依赖
   - 注意清理副作用

3. 状态设计：

   - 合理拆分状态
   - 复杂状态用 useReducer
   - 共享状态用 Context

4. 性能考虑：
   - 避免过度使用 useMemo/useCallback
   - 大型列表考虑虚拟滚动
   - 合理使用 Context 避免重渲染

### 场景五：组件设计模式

````text
面试官：在实际项目中，你是如何设计和组织 React 组件的？能分享一些组件设计模式吗？

候选人：
好的，我来分享一下我们在项目中常用的几种组件设计模式。

1. 复合组件模式（Compound Components）：
```javascript
// 实现一个可复用的 Select 组件
const SelectContext = createContext()

function Select({ children, onValueChange, value }) {
  return (
    <SelectContext.Provider value={{ onValueChange, value }}>
      <div className="select">{children}</div>
    </SelectContext.Provider>
  )
}

Select.Option = function Option({ value, children }) {
  const { onValueChange, value: selectedValue } = useContext(SelectContext)
  const isSelected = value === selectedValue

  return (
    <div
      className={`option ${isSelected ? 'selected' : ''}`}
      onClick={() => onValueChange(value)}
    >
      {children}
    </div>
  )
}

// 使用示例
function UserSelect() {
  const [value, setValue] = useState('')

  return (
    <Select value={value} onValueChange={setValue}>
      <Select.Option value="admin">管理员</Select.Option>
      <Select.Option value="user">普通用户</Select.Option>
    </Select>
  )
}


2. 容器组件模式（Container/Presentational）：

```javascript
// 容器组件
function UserListContainer() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function fetchUsers() {
      try {
        const data = await fetchUserList();
        setUsers(data);
      } finally {
        setLoading(false);
      }
    }
    fetchUsers();
  }, []);

  return <UserList users={users} loading={loading} />;
}

// 展示组件
function UserList({ users, loading }) {
  if (loading) return <Spinner />;

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
````

3. 高阶组件模式（HOC）：

```javascript
// 权限控制 HOC
function withPermission(WrappedComponent, requiredPermission) {
  return function PermissionWrapper(props) {
    const { permissions } = useAuth();

    if (!permissions.includes(requiredPermission)) {
      return <NoAccess />;
    }

    return <WrappedComponent {...props} />;
  };
}

// 使用示例
const AdminPanel = withPermission(BaseAdminPanel, "admin");
```

4. 自定义 Hook 模式：

```javascript
// 表单处理 Hook
function useForm(initialValues) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});

  const handleChange = useCallback((name, value) => {
    setValues((prev) => ({
      ...prev,
      [name]: value,
    }));
  }, []);

  const handleBlur = useCallback((name) => {
    setTouched((prev) => ({
      ...prev,
      [name]: true,
    }));
  }, []);

  return {
    values,
    errors,
    touched,
    handleChange,
    handleBlur,
  };
}
```

在选择设计模式时，我们主要考虑以下因素：

1. 组件的复用性：

   - 是否需要在多处使用
   - 是否需要共享逻辑

2. 组件的可维护性：

   - 职责是否单一
   - 是否容易测试

3. 组件的灵活性：

   - 是否支持自定义
   - 是否易于扩展

4. 团队协作：
   - 是否符合团队规范
   - 是否易于理解

### 场景六：生命周期和副作用处理

````text
面试官：能详细说说 React 中的生命周期和副作用处理吗？特别是在 Hooks 中如何处理？

候选人：
好的，我来分享一下在实际项目中处理生命周期和副作用的经验。

1. 基本的副作用处理：
```javascript
function UserProfile({ userId }) {
  // 数据加载
  useEffect(() => {
    let isMounted = true

    async function loadUser() {
      try {
        const data = await fetchUser(userId)
        // 防止组件卸载后设置状态
        if (isMounted) {
          setUser(data)
        }
      } catch (error) {
        if (isMounted) {
          setError(error)
        }
      }
    }

    loadUser()

    // 清理函数
    return () => {
      isMounted = false
    }
  }, [userId])
}


2. 事件监听的处理：

```javascript
function WindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    // 使用防抖优化
    const handleResize = debounce(() => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    }, 250);

    window.addEventListener("resize", handleResize);

    // 清理函数
    return () => {
      handleResize.cancel(); // 取消防抖
      window.removeEventListener("resize", handleResize);
    };
  }, []); // 空依赖数组，只在挂载时执行

  return (
    <div>
      Window size: {size.width} x {size.height}
    </div>
  );
}
````

3. 订阅模式的处理：

```javascript
function NotificationCenter() {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    // 订阅消息
    const subscription = messageService.subscribe((message) => {
      setMessages((prev) => [...prev, message]);
    });

    // 清理订阅
    return () => subscription.unsubscribe();
  }, []);
}
```

4. 多个副作用的组织：

```javascript
function Dashboard() {
  // 数据加载副作用
  useEffect(() => {
    loadDashboardData();
  }, []);

  // WebSocket 连接副作用
  useEffect(() => {
    const ws = new WebSocket(WS_URL);
    ws.onmessage = handleMessage;

    return () => ws.close();
  }, []);

  // 页面标题副作用
  useEffect(() => {
    const originalTitle = document.title;
    document.title = `Dashboard - ${user.name}`;

    return () => {
      document.title = originalTitle;
    };
  }, [user.name]);
}
```

在处理副作用时，我们需要注意以下几点：

1. 清理工作：

   - 取消订阅和监听
   - 清理定时器
   - 取消未完成的请求

2. 依赖管理：

   - 正确设置依赖数组
   - 避免遗漏依赖
   - 合理使用 useCallback/useMemo

3. 竞态条件：

   - 处理异步操作
   - 检查组件是否已卸载
   - 使用取消令牌

4. 性能优化：
   - 避免不必要的副作用
   - 合理使用防抖和节流
   - 优化依赖项变化频率

### 场景七：React Router 实践

````text
面试官：能谈谈你在项目中是如何使用 React Router 的？包括路由设计、权限控制等。

候选人：
好的，我来分享一下我们在项目中使用 React Router 的经验。

1. 基本的路由配置：
```javascript
// 路由配置
const routes = [
  {
    path: '/dashboard',
    component: Dashboard,
    auth: true,
    routes: [
      {
        path: '/dashboard/overview',
        component: Overview
      },
      {
        path: '/dashboard/users',
        component: UserList,
        permission: 'user:list'
      }
    ]
  }
]

// 路由渲染
function renderRoutes(routes) {
  return routes.map(route => (
    <Route
      key={route.path}
      path={route.path}
      element={
        <RouteGuard route={route}>
          <route.component routes={route.routes} />
        </RouteGuard>
      }
    />
  ))
}


2. 路由守卫实现：

```javascript
function RouteGuard({ route, children }) {
  const { isAuthenticated } = useAuth();
  const { hasPermission } = usePermission();
  const navigate = useNavigate();
  const location = useLocation();

  useEffect(() => {
    // 处理认证
    if (route.auth && !isAuthenticated) {
      navigate("/login", {
        state: { from: location },
        replace: true,
      });
      return;
    }

    // 处理权限
    if (route.permission && !hasPermission(route.permission)) {
      navigate("/403");
      return;
    }
  }, [route, isAuthenticated, hasPermission, navigate, location]);

  return children;
}
````

3. 路由懒加载：

```javascript
const UserManagement = lazy(() =>
  import(/* webpackChunkName: "user" */ "./pages/UserManagement")
);

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/users/*" element={<UserManagement />} />
      </Routes>
    </Suspense>
  );
}
```

4. 路由钩子的使用：

```javascript
function usePageTracking() {
  const location = useLocation();
  const navigate = useNavigate();

  useEffect(() => {
    // 页面访问统计
    trackPageView(location.pathname);

    // 处理未授权访问
    const handleUnauthorized = () => {
      navigate("/login");
    };

    eventBus.on("unauthorized", handleUnauthorized);
    return () => {
      eventBus.off("unauthorized", handleUnauthorized);
    };
  }, [location, navigate]);
}
```

在实际项目中，我们的路由设计原则：

1. 路由结构：

   - 按业务模块组织
   - 支持嵌套路由
   - 统一的路由配置

2. 权限控制：

   - 统一的路由守卫
   - 细粒度的权限控制
   - 优雅的未授权处理

3. 性能优化：

   - 路由懒加载
   - 预加载关键路由
   - 合理的加载提示

4. 用户体验：
   - 保持导航状态
   - 处理路由切换动画
   - 优雅的错误处理

### 场景八：错误处理和边界情况

````text
面试官：在 React 项目中，你是如何处理错误和异常情况的？能谈谈错误边界的使用经验吗？

候选人：
好的，我来分享一下我们在项目中处理错误的几种方式。

1. 错误边界组件：
```javascript
class ErrorBoundary extends React.Component {
  state = { hasError: false, error: null }

  static getDerivedStateFromError(error) {
    return { hasError: true, error }
  }

  componentDidCatch(error, errorInfo) {
    // 上报错误到监控系统
    reportError(error, errorInfo)
  }

  render() {
    if (this.state.hasError) {
      return (
        <ErrorFallback
          error={this.state.error}
          onReset={() => this.setState({ hasError: false })}
        />
      )
    }

    return this.props.children
  }
}

2. 异步错误处理：

```javascript
function AsyncComponent() {
  const [error, setError] = useState(null);

  const handleAsyncOperation = async () => {
    try {
      await someAsyncOperation();
    } catch (err) {
      setError(err);
      // 根据错误类型处理
      if (err instanceof NetworkError) {
        showNetworkError();
      } else if (err instanceof ValidationError) {
        showValidationError(err.details);
      } else {
        // 未知错误上报
        reportError(err);
      }
    }
  };

  if (error) {
    return <ErrorMessage error={error} />;
  }

  return <button onClick={handleAsyncOperation}>执行操作</button>;
}
````

3. 全局错误处理：

```javascript
// 错误处理中间件
function errorMiddleware() {
  return (next) => (action) => {
    try {
      return next(action);
    } catch (err) {
      console.error("Caught an exception!", err);
      reportError(err);
      throw err;
    }
  };
}

// 全局错误处理
window.onerror = function (message, source, lineno, colno, error) {
  reportError({
    message,
    source,
    lineno,
    colno,
    error,
  });
  return true;
};

// API 错误处理
axios.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response) {
      switch (error.response.status) {
        case 401:
          redirectToLogin();
          break;
        case 403:
          showNoPermission();
          break;
        default:
          showGeneralError();
      }
    }
    return Promise.reject(error);
  }
);
```

4. 优雅降级处理：

```javascript
function FeatureComponent({ fallback }) {
  const [hasError, setHasError] = useState(false);

  if (hasError && fallback) {
    return fallback;
  }

  try {
    // 可能会出错的特性
    return <NewFeature />;
  } catch (error) {
    // 记录错误但不中断应用
    reportError(error);
    setHasError(true);
    // 返回基础功能
    return <BasicFeature />;
  }
}
```

在错误处理方面，我们遵循以下原则：

1. 错误分类：

   - UI 渲染错误
   - 网络请求错误
   - 业务逻辑错误
   - 未知系统错误

2. 错误恢复：

   - 提供重试机制
   - 支持手动恢复
   - 自动恢复策略

3. 用户体验：

   - 友好的错误提示
   - 保持部分功能可用
   - 提供问题解决建议

4. 监控和分析：
   - 错误上报系统
   - 错误追踪和分析
   - 性能影响评估

### 场景九：性能调试和监控

````text
面试官：你们在项目中是如何进行 React 应用的性能调试和监控的？

候选人：
好的，我来分享一下我们在性能调试和监控方面的实践。

1. 使用 React DevTools 进行性能分析：
```javascript
// 开发环境下的性能测量
function ExpensiveList({ items }) {
  const renderCount = useRef(0)

  // 记录组件渲染次数
  useEffect(() => {
    renderCount.current++
    console.log(`List rendered ${renderCount.current} times`)
  })

  return (
    <Profiler id="ExpensiveList" onRender={onRenderCallback}>
      {items.map(item => (
        <ListItem key={item.id} data={item} />
      ))}
    </Profiler>
  )
}

// 性能数据收集
function onRenderCallback(
  id,
  phase,
  actualDuration,
  baseDuration,
  startTime,
  commitTime
) {
  // 发送性能数据到监控系统
  trackPerformance({
    componentId: id,
    renderTime: actualDuration,
    totalTime: baseDuration
  })
}


2. 自定义性能监控 Hook：

```javascript
function usePerformanceMonitor(componentName) {
  useEffect(() => {
    const startTime = performance.now();

    return () => {
      const endTime = performance.now();
      const duration = endTime - startTime;

      if (duration > 16) {
        // 超过一帧的时间
        console.warn(`${componentName} took ${duration}ms to render`);
      }
    };
  });
}
````

3. Web Vitals 监控：

```javascript
import { getCLS, getFID, getLCP } from "web-vitals";

function reportWebVitals({ name, value }) {
  switch (name) {
    case "CLS":
      // 累积布局偏移
      trackMetric("CLS", value);
      break;
    case "FID":
      // 首次输入延迟
      trackMetric("FID", value);
      break;
    case "LCP":
      // 最大内容绘制
      trackMetric("LCP", value);
      break;
  }
}

// 初始化监控
getCLS(reportWebVitals);
getFID(reportWebVitals);
getLCP(reportWebVitals);
```

4. 性能优化工具：

```javascript
// 使用 why-did-you-render 追踪不必要的重渲染
import whyDidYouRender from "@welldone-software/why-did-you-render";

if (process.env.NODE_ENV === "development") {
  whyDidYouRender(React, {
    trackAllPureComponents: true,
    logOnDifferentValues: true,
  });
}

// 标记需要追踪的组件
ExpensiveComponent.whyDidYouRender = true;
```

我们的性能监控策略包括：

1. 关键指标监控：

   - 首屏加载时间
   - 组件渲染时间
   - 交互响应时间
   - 内存使用情况

2. 性能预警机制：

   - 设置性能阈值
   - 异常情况告警
   - 性能趋势分析

3. 优化方向：

   - 识别性能瓶颈
   - 优化渲染策略
   - 代码分割优化

4. 持续改进：
   - 性能数据分析
   - A/B 测试验证
   - 定期性能评审
