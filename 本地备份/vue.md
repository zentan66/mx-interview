## 面试题

1. vue组件间的哪些通信方式？
2. 一个父组件嵌套了子组件，他的生命周期函数顺序是怎么执行的？
3. vue的权限管理应该怎么做？路由级和按钮级分别怎么处理？
4. 说一下你对虚拟DOM的理解
5. 了解diff算法吗？vue的diff算法是怎样的一个过程
6. 能说一下v-**for**中key的作用吗？
7. 做过vue项目哪些性能方面的优化？
8. vue组件为什么只能有一个根元素？
9. 如何实现路由懒加载呢？
10. 客户端渲染和服务端渲染有什么区别呢？在之前的工作中有做过服务端渲染吗？
11. Vue长列表的优化方式怎么做？
12. Vue3相比Vue2有哪些优化？
13. 为什么在模板中绑定事件的时候要加.native?
14. 能说一下响应式原理的过程吗？
15. 数组的响应式怎么实现的？
16. Vue是数据改变后页面也会重新改变嘛；**this**.a = 1; **this**.a = 2; 他是怎么实现异步更新优化整个渲染过程的？
17. render函数封装有什么特别的，或者用到比较巧妙的东西吗？
18. nextTick, setTimeout 以及 setImmediate 三者有什么区别? 
19. vuex跟redux的区别有哪些？
20. computed和watch的区别？
21. vue的history模式和hash模式的区别是什么？
22. history模式下会出现404，怎么处理？
23. 懒加载的实现原理是怎样的？
24. Vue 中 `$set` 是做什么的。写一个 `$set` 的伪代码。
25. 在vue项目中使用pinia解决了什么问题
    - 简化状态管理：Pinia 提供了一个简洁的 API，使得我们可以更容易地定义和管理状态，并在整个应用程序中共享它们。
    - 更好的类型支持：Pinia 提供了一个类型安全的 API，可以让我们更容易地编写类型安全的代码，并减少错误。
    - 更好的可测试性：Pinia 的状态管理使得我们可以更容易地对 Vue 3 组件进行单元测试，从而提高代码的可测试性。
    - 更好的性能：Pinia 的状态管理实现了基于 Proxy 的响应式系统，从而提高了性能并减少了不必要的重渲染
26. pinia的store如何进行设计的
    - 状态的划分：我们需要考虑应用程序中需要管理的状态，并根据不同的功能和需求进行划分。通常情况下，我们会将状态划分为多个 store，每个 store 管理一部分相关的状态。
    - Store 的命名：我们需要为每个 store 提供一个唯一的名称，以便在整个应用程序中引用它们
    - Store 的定义：我们需要定义每个 store 的结构，包括 store 的状态、getter、mutation 和 action 等。
    - Store 的注册：我们需要将定义好的 store 注册到应用程序中，以便在应用程序的其他地方使用。
    - Store 的使用：我们需要在组件中使用 store，通过 getter 获取 store 的状态，并在需要时通过 mutation 和 action 来修改 store 的状态。





## diff

### Vue2

diff算法是框架中核心内容之一，如果没有diff算法找出新旧节点树之间的差异，就没办法更新界面内容。阅读vue源码之后会发现核心的方法是`updateChildren`

```javascript
function updateChildren(parentElm, oldCh, newCh, insertedVnodeQueue, removeOnly) {
  // 声明变量
  while (oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) {
    // something
  }
}
```

从上述代码可以看出，比较的过程是使用双指针对新旧节点集合进行比较的

在比较的过程中会对节点进行判断，代码如下：

```javascript
if (isUndef(oldStartVnode)) {
  oldStartVnode = oldCh[++oldStartIdx]
} else if (isUndef(oldEndVnode)) {
  oldEndVnode = oldCh[--oldEndIdx]
} else if (sameVnode(oldStartVnode, newStartVnode)) {
  patchVnode(...参数)
  oldStartVnode = oldCh[++oldStartIdx]
  newStartVnode = newCh[++newStartIdx]
} else if (sameVnode(oldEndVnode, newEndVnode)) {
  //
} else if (sameVnode(oldStartVnode, newEndVnode)) {
  // 节点向右移动
} else if (sameVnode(oldEndVnode, newStartVnode)) {
  // 节点向左移动
} else {
  // 通过key来进行节点查找
}
```

在diff过程中，会用新前旧前，新后旧后，新前旧后，新后旧前四种方式来比较节点，如果发现没有匹配到的节点，就会使用key值来进行比较

key比较会先通过key来创建一个键值对的映射关系，然后判断新节点集合中是否能找到；

如果没有找到，则会直接创建一个新的节点；

如果找到了就会判断是否是同一类型节点；

sameVnode校验通过就会执行`patchVnode`方法，并进行一个插入操作；

否则就会判断为同key，但是非同类型节点，直接进行节点创建；



在双指针的逻辑执行结束后，可能会有剩余节点的情况

```javascript
if (oldStartIdx > oldEndIdx) {
  refElm = isUndef(newCh[newEndIdx + 1]) ? null : newCh[newEndIdx + 1].elm
  addVnodes(parentElm, refElm, newCh, newStartIdx, newEndIdx, insertedVnodeQueue)
} else if (newStartIdx > newEndIdx) {
  removeVnodes(oldCh, oldStartIdx, oldEndIdx)
}
```

### vue3

快速diff算法：借鉴了文本diff算法的预处理思路，先处理新旧节点中相同的前置节点和后置节点。当前置节点和后置节点全部处理完毕后，如果无法通过简单的挂载新节点或者卸载已经不存在的节点来更新，则需要根据节点间的索引关系，构造出一个最长递增子序列，最长递增子序列所指向的节点即为不需要移动的节点



