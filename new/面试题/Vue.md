##### Vue2 和 Vue3有哪些区别，本质区别，源码

##### Vue2 中的mixin是否了解，原理是什么？

##### Vue3 的 hook 和 React 的差别

##### 介绍一下 Vue3 的新特性，相对 Vue2 来说

```
1. 性能提升（更小的体积、更快的渲染）
2. Composition API（更灵活的代码组织方式）
3. 新的响应式系统
4. Fragment支持
5. Teleport
6. Suspense（异步组件处理）
7. 更好的TypeScript支持
8. 自定义渲染API
```

##### 前端路由的实现方式

##### Vue 的 `nextTick` 是用来做什么的？它的原理是什么？

```
用来在下次 DOM 更新循环结束之后执行延迟回调
通过浏览器的微任务队列实现，使用的是Rromise.then，并且支持降级操作，可以通过MutationObserver，甚至setTimeout实现
```



##### Vue 中 `$set` 是做什么的。写一个 `$set` 的伪代码。

##### Proxy 和 Object.defineProperty 区别

```
Proxy：可以拦截对象的各种操作
Object.defineProperty：只能拦截对象，如果对象新增属性，并不会触发更新
```



```
Vue3中对不参与更新的元素，会做静态提升，只会被创建一次，在渲染时直接复用
vue3在diff算法中相比vue2增加了静态标记
```



##### Vue2 和 Vue3 的 diff 用了什么算法库？

##### Vue 中的数据状态管理用的什么？Vuex 的作用和用法。

```
使用的是vuex
作为整个项目的状态管理工具，对数据进行存储和管理
```

##### Vue 的路由模式哪几种，实现原理。history 服务端需要做哪些配置？

```
hash模式 监听hashchange事件
history模式 popstate事件
```



##### Vue2 里面 data 为什么是一个 function ？·

##### 属性发生变化，视图会同步渲染吗？

##### 详细说一下 Vue2 和 Vue3 diff 算法的详细过程。这两个算法的复杂度。

##### 是否做过 Vue 相关的性能优化

##### 2.0 和 3.0 算法差异速度提高多少有了解过吗？

##### 数据列表渲染，动态添加一些事件有了解过吗？

##### 通过下标指定 key 的话会有什么问题？这个问题出现在 diff 算法的哪个节点？

```
当 Vue 正在更新使用 v-for 渲染的元素列表时，它默认使用“就地更新”的策略。如果数据项的顺序被改变，Vue 将不会移动 DOM 元素来匹配数据项的顺序，而是就地更新每个元素，并且确保它们在每个索引位置正确渲染
这个默认的模式是高效的，但是只适用于不依赖子组件状态或临时 DOM 状态的列表渲染输出。
key 的特殊属性主要用在 Vue 的虚拟 DOM 算法，在新旧 nodes 对比时辨识 VNodes。如果不使用 key，Vue 会使用一种最大限度减少动态元素并且尽可能的尝试修复/再利用相同类型元素的算法。使用 key，它会基于 key 的变化重新排列元素顺序，并且会移除 key 不存在的元素。
有相同父元素的子元素必须有独特的 key。重复的 key 会造成渲染错误。
```



##### Vue Router 的原理，hash 和 history 有什么区别？

```
hash
原理：通过url的hash模拟，url中#之后的内容不会被发送到服务器，因此不会导致页面的重新加载
优点：实现简单、兼容性好、不需要服务端配置
缺点：url不够美观，不利于SEO

history
原理：通过HTML5 History API（pushState和replaceState）来实现的
特点：URL更加美观，有利于SEO，需要服务器配置
```



##### KeepAlive 有用过吗？实现机制。

```
其核心原理是通过缓存和复用组件实例来避免不必要的销毁和重建操作
```



##### 如果一个组件既在 exclude 又在 include 会被缓存吗？为什么这么设计？

```
exclude的优先级会更高，include就不生效
缓存了组件之后，再次进入组件不会触发beforeCreate、created 、beforeMount、 mounted，如果你想每次进入组件都做一些事情的话，你可以放在activated进入缓存组件的钩子中
```

##### Vue3 和 Vue2 有什么区别？怎么把 Vue2 项目升级到 Vue3？

##### Vue 双向绑定原理。

```
双向数据绑定的原理是数据劫持结合发布订阅者模式实现的
对数据进行监听，当数据发生变化时，会触发setter函数，并告诉订阅者是否更新
```

##### 组件销毁的时候一般会进行什么样的操作

```
手动解绑事件监听器和观察者，以避免内存泄漏
避免在beforeDestroy钩子函数中调用副作用操作
```

vue和react源码用了哪些设计模式

vue组件通信方式



1. watch和computed区别
2. v-model本质是什么，vue2的v-model和vue3的v-model实现有什么区别，怎么实现两个v-model
3. render有哪些参数 x
4. vue的函数组件是什么，有哪些参数
5. 按需引入怎么做 install
6. 自定义指令怎么做
7. component是什么
8. 封装一个组件
9. h函数的具体用途是什么，作用是什么
10. slot具体用法，怎么把slot的数据传递出去
11. vue的父子组件通信，多层级怎么通信，父级怎么传给孙子组件，孙子组件怎么传给父级（不允许使用eventbus，provide/inject,vuex）
12. vue3项目结构怎么组织
13. hooks文件夹的hooks怎么封装的
14. useRequest怎么封装的
15. Vue2 和 Vue3 响应式的性能有区别吗？为什么？
16. Vue3 有什么其他的优化，比如静态节点的优化。
17. Vue2 和 Vue3 的 diff 用了什么算法库？
18. Vue.extend 有用过吗？什么原理？extend函数是什么
