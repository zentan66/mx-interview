### 定义

是实现了一个基于优先级和 `requestIdleCallback` 的循环任务调度算法

### 出现缘由

旧版本react组件的渲染会经过两个阶段：

- 调度阶段：生成虚拟DOM，遍历虚拟DOM，通过 diff 算法快速找出需要更新的元素，放到更新队列中
- 渲染阶段：根据渲染环境，遍历更新队列，将对应元素更新

旧版本的 diff 是一个自顶向下递归的过程，一旦开始，会持续占用主线程，很难被中断

### 带来的影响

任务的更新过程可能会被打断，意味着有以下几个生命周期可能会被调用多次：

- shouldComponentUpdate
- componentWillMount
- componentWillReceiveProps
- componentWillUpdate