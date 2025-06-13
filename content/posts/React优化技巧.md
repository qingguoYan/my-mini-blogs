+++
author = "yqg"
title = "React 开发经验总结"
date = "2025-06-05"
description = "深入探讨React开发中的性能优化、组件设计、状态管理等核心技术，结合实战经验分享最佳实践"
tags = [
    "React",
]
+++

写出更好的 React 代码 🚀

<!--more-->

## 深入理解组件渲染优化

### 何时使用 React.memo

在 React 应用中，性能优化的第一步是避免不必要的渲染。`React.memo` 是一个强大的工具，但它的使用需要遵循一些原则：

使用 memo 时需要注意：

1. **引用类型属性处理**：

```jsx
// 父组件
const Parent = () => {
  // 使用 useCallback 缓存函数
  const handleClick = useCallback(() => {
    console.log("clicked");
  }, []);

  // 使用 useMemo 缓存对象
  const config = useMemo(
    () => ({
      color: "blue",
      fontSize: 14,
    }),
    []
  );

  return <Child onClick={handleClick} config={config} />;
};
```

2. **避免过度优化**：

```jsx
// 🔴 过度优化的例子
const SimpleText = React.memo((props) => {
  return <span>{props.text}</span>;
});

// ✅ 不需要 memo 的简单组件
const SimpleText = (props) => {
  return <span>{props.text}</span>;
};
```

## 组件设计的艺术

### 组件分层架构

在大型应用中，推荐采用以下分层架构：

```jsx
// 1. 展示组件 (Presentational Components)
const UserCard = ({ name, avatar, role }) => (
  <div className="user-card">
    <img src={avatar} alt={name} />
    <h3>{name}</h3>
    <span>{role}</span>
  </div>
);

// 2. 容器组件 (Container Components)
const UserListContainer = () => {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    fetchUsers().then((data) => setUsers(data));
  }, []);

  return loading ? (
    <Loading />
  ) : (
    <div className="user-list">
      {users.map((user) => (
        <UserCard key={user.id} {...user} />
      ))}
    </div>
  );
};

// 3. 页面组件 (Page Components)
const UserManagementPage = () => {
  return (
    <Layout>
      <Header title="用户管理" />
      <UserListContainer />
      <Footer />
    </Layout>
  );
};
```

### 组件通信最佳实践

1. **Props 透传问题**：

```jsx
// 🔴 不推荐：props 层层传递
const Grandfather = ({ userData }) => <Father userData={userData} />;

const Father = ({ userData }) => <Son userData={userData} />;

// ✅ 推荐：使用 Context
const UserContext = React.createContext(null);

const Grandfather = ({ userData }) => (
  <UserContext.Provider value={userData}>
    <Father />
  </UserContext.Provider>
);

const Son = () => {
  const userData = useContext(UserContext);
  return <div>{userData.name}</div>;
};
```

## 状态管理的最佳实践

### 本地状态管理

```jsx
// ✅ 使用 useState 的最佳实践
const TodoList = () => {
  // 1. 状态聚合
  const [todoState, setTodoState] = useState({
    items: [],
    loading: false,
    error: null
  });

  // 2. 状态更新函数
  const addTodo = useCallback((newTodo) => {
    setTodoState(prev => ({
      ...prev,
      items: [...prev.items, newTodo]
    }));
  }, []);

  // 3. 派生状态
  const completedCount = useMemo(() =>
    todoState.items.filter(item => item.completed).length,
    [todoState.items]
  );

  return (/* 渲染逻辑 */);
};
```

### 全局状态管理

在选择状态管理方案时，建议：

1. **小型应用**：Context + useReducer

```jsx
// 定义 action 类型
const TodoActions = {
  ADD_TODO: "ADD_TODO",
  TOGGLE_TODO: "TOGGLE_TODO",
  // ...
};

// 创建 reducer
const todoReducer = (state, action) => {
  switch (action.type) {
    case TodoActions.ADD_TODO:
      return {
        ...state,
        todos: [...state.todos, action.payload],
      };
    // ...其他 case
  }
};

// 使用 Provider
const TodoProvider = ({ children }) => {
  const [state, dispatch] = useReducer(todoReducer, initialState);

  return (
    <TodoContext.Provider value={{ state, dispatch }}>
      {children}
    </TodoContext.Provider>
  );
};
```

2. **中大型应用**：Redux/Zustand

```jsx
// Zustand 示例
import create from "zustand";

const useStore = create((set) => ({
  todos: [],
  addTodo: (todo) =>
    set((state) => ({
      todos: [...state.todos, todo],
    })),
  toggleTodo: (id) =>
    set((state) => ({
      todos: state.todos.map((todo) =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      ),
    })),
}));
```

## React 18 新特性应用

### 并发渲染实践

1. **useTransition 优化用户体验**：

```jsx
const SearchComponent = () => {
  const [query, setQuery] = useState("");
  const [isPending, startTransition] = useTransition();
  const [searchResults, setSearchResults] = useState([]);

  const handleSearch = (e) => {
    // 立即更新输入值
    setQuery(e.target.value);

    // 将搜索逻辑放在 transition 中
    startTransition(() => {
      const results = performExpensiveSearch(e.target.value);
      setSearchResults(results);
    });
  };

  return (
    <div>
      <input value={query} onChange={handleSearch} />
      {isPending ? <Spinner /> : <SearchResults results={searchResults} />}
    </div>
  );
};
```

2. **useDeferredValue 处理昂贵的渲染**：

```jsx
const Chart = memo(({ data }) => {
  // 复杂的图表渲染逻辑
});

const ChartContainer = ({ data }) => {
  const deferredData = useDeferredValue(data);

  return (
    <>
      <Chart data={deferredData} />
      {deferredData !== data && <Spinner />}
    </>
  );
};
```

## 列表渲染优化

### 虚拟列表实现

对于长列表，建议使用虚拟滚动：

```jsx
import { FixedSizeList } from "react-window";

const VirtualizedList = ({ items }) => {
  const Row = ({ index, style }) => (
    <div style={style}>
      <ListItem data={items[index]} />
    </div>
  );

  return (
    <FixedSizeList
      height={400}
      width="100%"
      itemCount={items.length}
      itemSize={50}
    >
      {Row}
    </FixedSizeList>
  );
};
```

### Key 的最佳实践

```jsx
// 🔴 避免使用索引作为 key
{
  items.map((item, index) => <ListItem key={index} data={item} />);
}

// ✅ 使用唯一标识作为 key
{
  items.map((item) => <ListItem key={item.id} data={item} />);
}

// 如果没有唯一 ID，可以这样生成
const generateKey = (item, index) => {
  return `${item.type}_${item.name}_${index}`;
};
```

## 数据持久化方案

### 本地存储实现

使用 redux-persist 实现本地数据持久化：

```js
import { createStore } from "redux";
import { persistStore, persistReducer } from "redux-persist";
import storage from "redux-persist/lib/storage";

const persistConfig = {
  key: "root",
  storage: storage,
  whitelist: ["user", "settings"], // 只持久化特定 reducer
};

const persistedReducer = persistReducer(persistConfig, rootReducer);
const store = createStore(persistedReducer);
const persistor = persistStore(store);
```

### 服务器存储实现

自定义 storage 引擎实现服务端持久化：

```js
const customStorage = {
  async getItem(key) {
    try {
      const response = await fetch(`/api/state/${key}`, {
        headers: {
          Authorization: `Bearer ${getToken()}`,
          "Content-Type": "application/json",
        },
      });

      if (!response.ok) {
        throw new Error("获取状态失败");
      }

      const data = await response.json();
      return JSON.stringify(data.state);
    } catch (err) {
      console.error("从服务器获取状态失败:", err);
      return null;
    }
  },

  async setItem(key, value) {
    try {
      const response = await fetch(`/api/state/${key}`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${getToken()}`,
        },
        body: JSON.stringify({
          state: JSON.parse(value),
          timestamp: Date.now(),
        }),
      });

      if (!response.ok) {
        throw new Error("保存状态失败");
      }
    } catch (err) {
      console.error("保存状态到服务器失败:", err);
    }
  },

  async removeItem(key) {
    try {
      const response = await fetch(`/api/state/${key}`, {
        method: "DELETE",
        headers: {
          Authorization: `Bearer ${getToken()}`,
        },
      });

      if (!response.ok) {
        throw new Error("删除状态失败");
      }
    } catch (err) {
      console.error("从服务器删除状态失败:", err);
    }
  },
};

// 配置 redux-persist 使用自定义 storage
const persistConfig = {
  key: "root",
  storage: customStorage,
  timeout: 10000, // 设置超时时间
  whitelist: ["user", "settings"],
  writeFailHandler: (err) => {
    // 处理写入失败的情况
    console.error("State 持久化失败:", err);
  },
};
```

## 你可能不需要使用 useEffect

在 React 开发中，useEffect 是最容易被过度使用的 Hook 之一。很多开发者倾向于把所有的副作用都放在 useEffect 中，这可能会导致一些不必要的问题。让我们来看看什么时候可以避免使用 useEffect。

### 1. 事件处理中的副作用

```jsx
// 🔴 不必要的 useEffect
const SearchComponent = () => {
  const [query, setQuery] = useState("");

  useEffect(() => {
    if (query) {
      performSearch(query);
    }
  }, [query]);

  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
};

// ✅ 更好的方式：直接在事件处理函数中执行
const SearchComponent = () => {
  const [query, setQuery] = useState("");

  const handleChange = async (e) => {
    const newQuery = e.target.value;
    setQuery(newQuery);
    if (newQuery) {
      await performSearch(newQuery);
    }
  };

  return <input value={query} onChange={handleChange} />;
};
```

### 2. props 改变后的计算

```jsx
// 🔴 不必要的 useEffect
const ProductCard = ({ price, quantity }) => {
  const [total, setTotal] = useState(0);

  useEffect(() => {
    setTotal(price * quantity);
  }, [price, quantity]);

  return <div>总价：{total}</div>;
};

// ✅ 使用计算属性
const ProductCard = ({ price, quantity }) => {
  const total = price * quantity;
  return <div>总价：{total}</div>;
};

// 如果计算比较复杂，可以使用 useMemo
const ProductCard = ({ price, quantity, discount }) => {
  const total = useMemo(() => {
    return calculateComplexTotal(price, quantity, discount);
  }, [price, quantity, discount]);

  return <div>总价：{total}</div>;
};
```

### 3. 状态联动更新

```jsx
// 🔴 避免使用 useEffect 进行状态联动
const Form = () => {
  const [firstName, setFirstName] = useState("");
  const [lastName, setLastName] = useState("");
  const [fullName, setFullName] = useState("");

  useEffect(() => {
    setFullName(`${firstName} ${lastName}`);
  }, [firstName, lastName]);

  return (/* 渲染逻辑 */);
};

// ✅ 直接在事件处理函数中更新关联状态
const Form = () => {
  const [firstName, setFirstName] = useState("");
  const [lastName, setLastName] = useState("");

  const handleFirstNameChange = (e) => {
    const newFirstName = e.target.value;
    setFirstName(newFirstName);
  };

  // 或者使用派生状态
  const fullName = `${firstName} ${lastName}`.trim();

  return (/* 渲染逻辑 */);
};
```

### 4. 数据请求

```jsx
// 🔴 不推荐：在 useEffect 中直接请求数据
const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetchUser(userId)
      .then((data) => setUser(data))
      .catch((err) => setError(err));
  }, [userId]);

  if (error) return <div>Error: {error.message}</div>;
  if (!user) return <div>Loading...</div>;
  return <div>{user.name}</div>;
};

// ✅ 推荐：使用自定义 Hook 封装数据请求逻辑
const useUser = (userId) => {
  const [state, setState] = useState({
    data: null,
    error: null,
    loading: true,
  });

  useEffect(() => {
    let mounted = true;

    const loadUser = async () => {
      try {
        setState((prev) => ({ ...prev, loading: true }));
        const data = await fetchUser(userId);
        if (mounted) {
          setState({ data, loading: false, error: null });
        }
      } catch (error) {
        if (mounted) {
          setState({ data: null, loading: false, error });
        }
      }
    };

    loadUser();

    return () => {
      mounted = false;
    };
  }, [userId]);

  return state;
};

// 使用自定义 Hook
const UserProfile = ({ userId }) => {
  const { data: user, error, loading } = useUser(userId);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <div>{user.name}</div>;
};
```

### 5. 订阅与取消订阅

这种场景确实需要使用 useEffect，但要注意清理函数：

```jsx
const ChatRoom = ({ roomId }) => {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();

    const handleMessage = (message) => {
      setMessages(prev => [...prev, message]);
    };

    connection.on('message', handleMessage);

    // 清理函数：断开连接并移除事件监听
    return () => {
      connection.off('message', handleMessage);
      connection.disconnect();
    };
  }, [roomId]);

  return (/* 渲染逻辑 */);
};
```

### 最佳实践总结

1. **优先考虑事件处理函数**：如果副作用是由用户交互触发的，把它放在事件处理函数中。

2. **使用派生状态**：如果某个状态可以由其他状态计算得出，直接计算而不是使用 effect。

3. **合理使用 useMemo**：对于计算量大的派生状态，使用 useMemo 缓存结果。

4. **抽象复杂逻辑**：将复杂的数据获取、订阅等逻辑封装成自定义 Hook。

5. **注意清理工作**：当使用 useEffect 时，记得处理清理工作，避免内存泄漏。

6. **控制依赖项**：仔细管理 useEffect 的依赖数组，避免遗漏或包含不必要的依赖。

> 本文持续更新中...
