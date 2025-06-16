+++
author = "yqg"
title = "Zustand学习总结"
date = "2025-06-16"
description = ""
tags = [
    "Zustand",
]
+++

> Zustand 是一个小巧精美的状态管理库，可以很好的与 React 进行集成，并且提供中间件支持。
> useSyncExternalStore 是 React 与 Zustand 沟通的桥梁。

## 关于 useSyncExternalStore

useSyncExternalStore 是一个订阅外部状态 Store 的 React hook，使用者需要提供一个订阅函数和一个获取 Store 数据的函数。
详细介绍可以查看官网

https://zh-hans.react.dev/reference/react/useSyncExternalStore

## 实现 Zustand

创建一个 Create State Store 函数，设置 Store 的初始 state 值并提供修改和获取 state 的函数。
为 useSyncExternalStore 提供 subscribe，subscribe 获取 listener。当组件状态更新，触发所有订阅 store 的组件渲染

```js
// 创建一个Store
const createStore = (creatState) => {
  let state; // store 存储的值

  const listeners = new Set(); // 存储所有订阅 store 的 listener

  // 更新 state 的值
  function setState(partial) {
    const nextState = typeof partial === "function" ? partial(state) : partial;
    // 如果更新的值是一样的，不发生渲染
    if (Object.is(state, nextState)) {
      return;
    }
    state = Object.assign({}, state, nextState);
    // 触发listeners
    listeners.forEach((listener) => listeners());
  }

  // 获取状态
  function getState() {
    return this.state;
  }

  // 订阅 react 组件, react 会提供给 store一个listener
  // 当listener执行时，组件更新
  function subscribe(listener) {
    listeners.add(listener);

    return () => {
      listeners.delete(listener);
    };
  }

  state = creatState(); // 设置 state的值

  const api = { setState, getState, subscribe };

  return api;
};
```

创建 store 后，与 useSyncExternalStore 结合

```js
const useStore = (api, selecter) => {
  return React.useSyncExternalStore(api.subscribe, selecter(api.getState()));
};
```

暴漏出 create 函数

```js
const createImple = (createState) => {
  // 创建 store
  const api = createStore(createState);
  // selector获取执行状态
  const useBoundStore = (selector) => useStore(api, selector);

  return useBoundStore;
};

const create = (createState) => {
  createImple(createState);
};

export default create;
```

接下来就可以使用了

```js
const createState = (set) => ({
  bears: 0,
  increasePopulation: () => set((state) => ({ bears: state.bears + 1 })),
  removeAllBears: () => set({ bears: 0 }),
  updateBears: (newBears) => set({ bears: newBears }),
});

const useStore = create(createState);

function BearCounter() {
  const bears = useStore((state) => state.bears);
  return <h1>{bears} bears around here...</h1>;
}

function Controls() {
  const increasePopulation = useStore((state) => state.increasePopulation);
  return <button onClick={increasePopulation}>one up</button>;
}
```
