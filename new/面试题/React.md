> 111



##### React的diff算法

1. Tree Diff：同一层级比较
2. Component Diff：组件同类型比较
3. Element Diff：通过key值比较

##### 说一下react fiber的原理

1. 任务分割
2. 增量渲染
3. 优先级调度
4. 双缓冲技术
5. 链表遍历
6. 并发模式

##### 怎么理解受控组件和非受控组件

```
受控就是组件是受用户的输入来更新的

非受控组件并不是为每个状态更新都编写新的数据处理函数。
```

##### 说一下 Fiber 架构

1. 平时怎么写一个组件的

```
1 确定组件的功能
2 设计组件的内部状态（有哪些state数据）
3 暴露到外部的API
整个过程会尽量避免组件与外
部组件有高耦合存在，其次是考虑组件的拓展性、通用性、健壮性
```

2. hooks 可以放在函数的内部吗？为什么？

3. HOC是什么

4. Redux的原理

5. React中`setState` 调用以后会经历哪些流程

6. setState是同步还是异步，为什么

7. pureComponent

8. React 里面有一个 A 组件，里面包裹了 B 子组件，A 组件是 10 个属性，B 只引用了其中两三个，如何提升性能？ShouldComponentUpdate。

9. React Router 如何实现权限拦截？

10. React hooks 和 Vue3 的 composition api 有什么区别？

11. useMemo，useCallback，useRef 的作用。

12. 说一下 useLayoutEffect 和 useEffect 的区别是什么？

13. useEffect 的返回值有什么作用，执行时机。

14. react 函数式组件，hooks 有一定的写法规范，是出于什么样的考虑，会有这样的限制？

15. 函数式组件，在一个应用周期里，什么时机会被调用到呢，函数会被调用多少次

16. 什么情况下会触发组件更新？

17. React 组件是怎么渲染为 DOM 元素到页面上的

18. 如何进行数据管理

19. useState 是怎么实现的

20. react keep-alive是怎么做的

21. useCallback 和 useMemo 有什么区别？适合什么场景下使用？是不是所有变量或者函数都需要用着两个hook包裹？包裹后性能一定会好吗？底层理解

22. react页面白屏检测 [喂，你的页面白了！阿里解决「前端白屏」的方案](https://zhuanlan.zhihu.com/p/399348866)

23. 对 React Hooks 的理解

24. 子组件没有任何的 props，父组件在渲染的时候，子组件会跟着渲染吗

    回答：不会跟着渲染，只有子组件传入的属性被修改，即便没有使用仍旧会触发渲染操作

25. react fiber 是在什么情况下诞生的，是为了解决什么问题

26. React 有哪些优化方式

27. react 优化需要手动优化、有没有一些方案可以自动处理这个问题

28. redux用过么，conenct具体做了什么

29. React Context的原理

30. 对 React 的 Fiber 架构有什么了解吗？中间的节点怎么通过链表连接起来。（Fiber是一个优先级任务循环架构，）

31. React 没有响应式，是怎么实现的？（单向数据流）

32. Vue 或 React 中我们连续三次赋值，比如 `this.a`，会怎么实现？（vue2多次设置state，会进行一个合并操作，通过nextTick在下一次dom更新更新数据；react是多次修改放入一个队列，最后合并成一次状态修改)

33. react生产问题怎么定位到具体错误代码行数

34. 组件 return JSX，这个需要在编译的时候转化才能运行，在编译阶段会被转义成什么 JS 代码？（React.createElement(tag, attrs, ...element) ）

35. Hooks 怎么做到在组件的多次渲染之间保持状态（存储在memoizedState中，双缓存结束workInProgress）

36. Hooks 链表的节点插入顺序是什么样的

37. 介绍一下 `React.lazy` 和 `Suspense`的用途和实现原理

38. 从class重构成 function 组件需要注意哪些问题

39. 类组件和 function 组件最大的区别

40. hooks 大概的原理

41. 一个hooks的更新机制

42. setState是同步还是异步

43. react 有什么优化的方式

44. 路由懒加载的原理

45. react fiber如何中断的

