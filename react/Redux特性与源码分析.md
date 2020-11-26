## 特性

### 定义

Redux是JavaScript状态容器，提供可预测化的状态管理。



### 原则

#### 单一数据源

整个应用的state被储存到一棵object tree中，这个state只存在唯一一个store中。

#### State只读

唯一改变state的方法就是触发action，而action是一个用于描述已发生事件的普通对象

#### 使用纯函数执行修改



### 基础

#### Reducer

reducer 就是一个纯函数，接收旧的 state 和 action，返回新的 state。

但是有几个操作是不可以做的：

- 修改传入参数
- 执行副作用操作
- 调用非纯函数

## 源码解读

### createStore

```javascript
export function createStore(reducer, preloadedState) {
  let currentReducer = reducer
  let currentState = preloadedState
  
  function dispatch(action) {
    currentState = currentReducer(currentState, action)
  }
  
  return {
    dispatch
  }
}
```

从dispatch中可以看到，当前状态是通过reducer传入当前state后更新的。如果执行了副作用的操作，比如异步请求，就没办法同步将state返回。