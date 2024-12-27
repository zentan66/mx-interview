## useState

### 执行流程

1. 第一次加载的时候会执行`mountState(initialState)`

2. 在`mountState`中首先会通过调用`mountWorkInProgressHook`创建一个`hook`对象

   ```javascript
   function mountWorkInProgressHook() {
     var hook = {
       memoizedState: null,
       baseState: null,
       baseQueue: null,
       queue: null,
       next: null
     };
   
     if (workInProgressHook === null) {
       // 当初始化第一个 hook 节点的时候
       currentlyRenderingFiber$1.memoizedState = workInProgressHook = hook;
     } else {
       // 不是第一个节点，直接放到后面
       workInProgressHook = workInProgressHook.next = hook;
     }
   
     return workInProgressHook;
   }
   ```

3. 在创建了 hook 之后，会判断`initialState`参数是否是一个函数，将执行结果赋值`hook.memoizedState = hook.baseState = initialState`

4. 创建一个`queue`对象，保存每次创建的更新（Update）

   ```javascript
   var queue = hook.queue = {
       pending: null, // 存储产生的更新，单项循环链表
       interleaved: null,
       lanes: NoLanes,
       dispatch: null,
       lastRenderedReducer: basicStateReducer,
       lastRenderedState: initialState
   }
   var dispatch = queue.dispatch = dispatchAction.bind(null, currentRenderingFiber$1, queue)
   ```

每一个 hook 节点都会对应一个 queue 对象

### 多个update更新

1

```javascript
var pending = queue.pending
if (pending === null) {
  update.next = update
} else {
  update.next = pending.next
  pending.next = update
}
queue.pending = update
```

### 更新阶段

更新阶段会创建一个新的 hook 对象

```javascript
var newHook = {
  memoizedState: currentHook.memoizedState,
  baseState: currentHook.baseState,
  baseQueue: currentHook.baseQueue,
  queue: currentHook.queue,
  next: null
};

if (workInProgressHook === null) {
      // 当前还没有 hook 节点被初始化
      currentlyRenderingFiber$1.memoizedState = workInProgressHook = newHook;
} else {
      // 加到链表的最后面
      workInProgressHook = workInProgressHook.next = newHook;
}
```

