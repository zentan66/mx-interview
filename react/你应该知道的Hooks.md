

## 什么是React Hooks

hooks之前，react存在的问题：

1. 在组件间复用状态逻辑很难
2. 复杂组件变得难以理解，高阶组件和函数组件的嵌套过深。
3. class 组件的 this 指向问题
4. 难以记忆的生命周期

## 具体

### 实现原理

Hooks基本类型：

```typescript
type Hooks = {
  memoizedState: any, // 指向当前渲染节点 Fiber
  baseState: any, // 初始化 initialState， 已经每次 dispatch 之后 newState
  baseUpdate: Update<any> | null, // 当前需要更新的 Update ，每次更新完之后，会赋值上一个 update，方便 react 在渲染错误的边缘，数据回溯
  queue: UpdateQueue<any> | null, // UpdateQueue 通过
  next: Hook | null, // link 到下一个 hooks，通过 next 串联每一 hooks
};

type Effect = {
  tag: HookEffectTag, // effectTag 标记当前 hook 作用在 life-cycles 的哪一个阶段
  create: () => mixed, // 初始化 callback
  destroy: (() => mixed) | null, // 卸载 callback
  deps: Array<mixed> | null,
  next: Effect, // 同上
};
```



## 使用原则

- 不要在循环，条件判断，函数嵌套中使用hooks
- 只能在函数组件中使用hooks



## 相关链接

[最新整理的25道前端面试真题（含答案与解析）](https://juejin.cn/post/6899291168891207688)

