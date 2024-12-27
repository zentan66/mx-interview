### 执行过程

1. 调用 `this.updater.enqueueSetState`

2. 获取当前组件对应的 `react fiber`

3. 计算本次调度更新的优先级lane

4. 通过`lane`和`eventTime`创建一个`Update`对象

5. 调用`enqueueUpdate`，将创建的update对象链接到`updateQueue.shared.pending`中

6. **调度阶段**

7. 调用`scheduleUpdateOnFiber`函数

8. 接着执行`ensureRootIsScheduled`，在其会以一个优先级调度`render阶段`的开始函数`performSyncWorkOnRoot/performConcurrentWorkOnRoot`

9. **Render阶段** => `renderRootSync`

10. 构建下一次需要展示的 current 树，为每个节点调用 `beginWork` 和 `completeWork`

11. **beginWork开始（自顶向下）**

12. `beginWork`从`rootFiber`开始深度优先遍历，根据不同组件类型调用处理函数

13. 构建本轮更新的`updateQueue`，更新`workInProgress`节点中的`updateQueue`、`memoizedState`属性

14. 执行`getStateFromUpdate`计算新的 State 

15. 接着调用 `reconciliChildren`，**diff算法核心**，会尝试复用`current`中的`fiber`节点

16. 设置进行的DOM操作类型保存在`fiber.flags`上

17. **beginWork结束**

18. **completeWork开始**：创建当前节点的 DOM 节点，对子节点的 DOM 节点和 EffectList进行收拢，给节点打上`Update`的`EffectTag`

19. 每个执行完 completeWork 且存在 effectTag 的 Fiber 节点会被保存在一条被称为 effectList 的单向链表中；effectList 中第一个 Fiber 节点保存在 fiber.firstEffect ，最后一个元素保存在 fiber.lastEffect

20. **completeWork结束**

21. **commit阶段** => `commitRoot`

22. 在`rootFiber.firstEffect`上保存了一条需要执行`副作用`的`Fiber节点`的单向链表`effectList`，这些`Fiber节点`的`updateQueue`中保存了变化的`props`

23. 一些生命周期钩子（比如`componentDidXXX`）、`hook`（比如`useEffect`）需要在`commit`阶段执行

    `commit`阶段的主要工作（即`Renderer`的工作流程）分为三部分：

    - before mutation阶段（执行`DOM`操作前）
    - mutation阶段（执行`DOM`操作）
    - layout阶段（执行`DOM`操作后）

24. **beforeMutation阶段**

    - 处理DOM节点渲染/删除后的 autoFocus、blur逻辑
    - 调用 getSnapshotBeforeUpdate 生命周期钩子
    - 调度 useEffect

25. **mutation阶段**

    遍历effectList，对每一个Fiber节点执行以下操作：

    - 重置文字节点
    - 更新ref
    - 根据 effectTag 分别处理（Placement | Update）

26. **layout阶段**

    - 调用生命周期钩子和hook相关操作
    - 赋值ref





### QAQ

1. setState是异步还是同步？

   在组件生命周期或者react事件中·是异步

   在定时器或者原生 DOM 事件中是同步

   本质是同步的，只是执行的时候会等生命周期、事件处理器执行完毕，再批量执行，所以可以视为异步的

2. 为什么设计成异步的（批处理）？

   - 保证state和props的一致性
   - 提高性能，避免不必要的重新渲染
   - 更多的可能性