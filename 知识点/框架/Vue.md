



### 1.请说一下Vue响应式数据的原理

`Vue`底层对于响应式数据的核心是`object.defineProperty`，`Vue`在初始化数据时，会给`data`中的属性使用`object.defineProperty`重新定义属性（劫持属性的`getter`和`setter`），当页面使用对应属性时，会通过Dep类进行**依赖收集**（收集当前组件的`watcher`）,如果属性发生变化，会通知相关依赖调用其update方法进行更新操作

- 总结： `Vue`通过数据劫持配合发布者-订阅者的设计模式，内部通过调用`object.defineProperty()`来劫持各个属性的`getter`和`setter`，在数据变化的时候通知订阅者，并触发相应的回调



### 2.Vue如何实现响应式数据的？

实现一个监听器「Observer」：对数据对象进行遍历，包括子属性对象的属性，利用`Object.defineProperty()`在属性上都加上`getter`和`setter`，这样后，给对象的某个值赋值，就会触发`setter`，那么就能监听到数据变化

实现一个解析器「Compile」：解析`Vue`模板指令，将模板中的变量都替换成数据，然后初始化渲染页面视图，并将每个指令对应的节点绑定更新函数，添加监听数据的订阅者，一旦数据有变动，收到通知，调用更新函数进行数据更新

实现一个订阅者「Watcher」：`Watcher`订阅者是`Observer`和`Compile`之间通信的桥梁，主要任务是订阅`Observer`中的属性值变化的消息，当收到属性值变化的消息时，触发解析器`Compile`中对应的更新函数

实现一个订阅器「Dep」：订阅器采用发布-订阅设计模式，用来收集订阅者`Watcher`，对监听器`Observer`和订阅者`Watcher`进行统一管理



### 3.Vue如何实现对象和数组的监听

由于`Object.defineProperty()`只能对属性进行数据劫持，而不能对整个对象（数组）进行数据劫持，因此Vue框架通过遍历数组和对象，对每一个属性进行劫持



### 4.直接给一个数组项赋值，Vue能检测到变化吗？

由于`JavaScript`的限制，Vue不能检测以下数组的变动：

- 利用索引直接修改一个数组项时
- 修改数组的长度时

### 5.Vue如何检测数组的变化？

核心思想：使用了函数劫持的方式，重写了数组的方法（`push,pop,unshift,shift···`）

`Vue`将`data`中的数组，进行了原型链的重写，指向了自己所定义的数组原型方法，当调用数组的`API`时，可以通知依赖更新，如果数组中包含着引用类型，会对数组中的引用类型再次进行监控



### 6.Vue如何通过`vm.$set`来解决对象新增/删除属性不能响应的问题

由于`JavaScript`的限制，Vue无法检测到对象属性的添加或删除。这是由于Vue会在初始化实例时对属性的`getter`和`setter`进行劫持，所以属性必须在data对象上存在才能让Vue将它们转换为响应式数据。

`Vue`提供了`Vue.set(object, propertyName, value) / vm.$set(object, propertyName, value)`来实现为对象添加响应式属性，其原理如下：

- 如果目标是数组，直接使用数组的`splice`方法触发响应式
- 如果目标是对象，会先判断对象属性是否存在，对象是否是响应式，最终如果要对属性进行响应式处理，则是通过调用`defineReactive()`方法进行响应式处理（`defineReactive()`是Vue对`Object.defineProperty()`的二次封装）

### 7.为什么Vue采用异步渲染

如果不采用异步渲染，每次更新数据都会进行重新渲染，为了提高性能，Vue通过异步渲染的方式，在本轮数据更新后，再去异步更新数据

### 8.Object.defineProperty有什么缺点

- 只能劫持对象的属性
- 对新增属性需要手动进行观察

### 9.虚拟Dom的优缺点

优点：

- 保证性能下限：框架的虚拟DOM需要适配任何上层API可能产生的操作，性能不是最优的，但是必须是最普遍适用的
- 无需手动操作DOM：框架会根据虚拟DOM和数据的双向绑定，更新视图，提高开发效率
- 跨平台：虚拟DOM本质上是JavaScript对象，而真实DOM与平台相关

缺点：

- 无法进行极致优化

### 10.虚拟DOM的实现原理

1. 用JavaScript对象模拟真实DOM树，对真实DOM进行抽象
2. 通过diff算法对比两个虚拟DOM对象进行比较
3. 通过patch算法将两个虚拟DOM对象的差异应用到真实DOM树上

### 11.nextTick实现原理是什么？在Vue中有什么作用

原理：`EventLoop`事件循环

- 正确流程依次是：Promise.then -> MutationObserver回调函数->setImmediate ->setTimeout
- 在nextTick中接收到一个回调函数时，不会立即调用，而是push到一个全局的queue队列，等待下一个任务队列的时候再一次性把这个queue的函数依次执行
- 这个队列可能是microTask队列，也可能是macroTask队列

作用：

在下次dom更新循环结束后执行延迟回调，当我们修改数据之后立即使用nextTick()来获取最新更新的dom

### 12.watch中的deep是如何实现的

如果当前监控的值是数类型，会对对象中的每一项进行求值，此时会将当前watcher存入到对应属性的依赖中，这样数组中的对象会发生变化也会通知数据进行更新

本质上是因为Vue内部对设置了deep的watch，会进行递归的访问（只要此属性也是响应式属性），而再此过程也会不断发生依赖收集

缺点：由于需要对每一项都进行操作，性能会降低，不建议多次使用`deep:true`

### 13.为什么v-if与v-for不建议连在一起使用

略

### 14.组件的data为什么要写成函数形式

在`Vue`中，组件都是可复用的，一个组件创建好后，可以在多个地方重复使用，而不管复用多少次，组件内的`data`都必须是相互隔离，互不影响的，如果`data`以对象的形式存在，由于`Javascript`中对象是引用类型，作用域没有隔离，当你在模板中多次声明这个组件，组件中的data会指向同一个引用，此时如果对某个组件的data进行修改，会导致其他组件里的data也被修改。因此`data`必须以函数的形式返回

- 总结：为了实现每个组件实例可以维护独立的数据拷贝，不会相互影响

**❗ 小知识：** `new Vue`根组件不需要复用，因此不需要以函数方式返回

### 15.v-for的key的作用是什么

`key`是为每个`vnode`指定唯一的`id`，在同级`vnode`的`Diff`过程中，可以根据`key`快速的进行对比，来判断是否为相同节点，并利用`key`的唯一性生成`map`来更快的获取相应的节点，另外指定`key`后，可以保证渲染的准确性。

### 16.Vue每个生命周期什么时候被调用

- `beforeCreate` → 在实例初始化之后，数据观测（`data observer`）之前被调用

- `created` → 实例已经创建完成之后被调用。在这里，实例已完成以下配置：
  - 数据观测（`data observer`)
  - 属性和方法的运算
  - `watch/event`事件回调
  - 但这里还没有`$el`

- `beforeMount` → 在挂载开始之前被调用，相关的`render`函数首次被调用

- `mounted` → `$el`被新创建的`vm.$el`替换，并挂载到实例上之后调用该钩子

- `beforeUpdate` → 数据更新时调用，发生在虚拟`DOM`重新渲染和打补丁之前

- `updated` → 由于数据更改导致的虚拟`DOM`重新渲染和打补丁，在这之后会调用该钩子（该钩子在服务器端渲染期间不被调用）

- `beforeDestroy` → 实例销毁之前调用，在这里，实例仍然完全可以使用

- `destroyed` → `Vue`实例销毁后调用。调用该钩子后，`Vue`实例指示的所有东西都会解绑，所有的事件监听器会被移除，所有的子实例也会被销毁（该钩子在服务器端渲染期间不被调用）

### 17.Vue每个生命周期内部可以做什么事

- `created` → 实例已经创建完成，由于它是最早触发的，所以可以进行一些数据，资源的请求

- `mounted` → 实例已经挂载完成，可以进行一些`DOM`操作

- `beforeUpdate` → 可以在该钩子中进一步地更改状态，这不会触发附加的渲染过程

- `updated` → 可以执行依赖于`DOM`的操作。但在大多数情况下，应避免在该钩子中更改状态，因为这可能导致更新无限循环

- `destroyed` → 可以执行一些优化操作，例如清空定时器，清理缓存，解除事件绑定等

### 18.Vue的父组件和子组件生命周期钩子执行顺序是什么

- 理解渲染过程：

> 父组件挂载完成必须是等到子组件都挂载完成之后，才算父组件挂载完，所以父组件的`mounted`肯定是在子组件`mounted`之后

So：「父」beforeCreate → 「父」created → 「父」beforeMount → 「子」beforeCreate → 「子」created → 「子」beforeMount → 「子」mounted → 「父」mounted

- 子组件更新过程（取决于对父组件是否有影响）
  - 影响到父组件： 「父」beforeUpdate → 「子」beforeUpdate → 「子」updated → 「父」updated
  - 不影响父组件： 「子」beforeUpdate → 「子」updated
- 父组件更新过程（取决于对子组件是否有影响）
  - 影响到子组件： 「父」beforeUpdate → 「子」beforeUpdate → 「子」updated → 「父」updated
  - 不影响子组件： 「父」beforeUpdate → 「父」updated
- 销毁过程
  - 「父」beforeDestroy → 「子」beforeDestroy → 「子」destroyed → 「父」destroyed

### 19.父组件可以监听到子组件的生命周期嘛？

> 比如有一个父组件Parent和子组件Child，如果父组件监听到子组件挂载`mounted`就做一些逻辑处理、

- 方式一

```Vue
// Parent.vue
<Child @mounted='doSomething' />

// Child.vue
mounted(){
    this.$emit('mounted')
}
复制代码
```

- 方式二（更简单 - 通过@hook来监听）

```Vue
// Parent.vue
<Child @hook:mounted='doSomething' />

#   @hook可以监听其他生命周期
```

### 20.为什么子组件不可以修改父组件传递的prop

由于vue提倡单向数据流，即父级props的更新会流向子组件，但反过来不行，这是为了防止意外的改变父组件的状态，使得应用的数据流变得难以理解

### 21.组件间有哪些通信方式

- 父子组件通信
  - Props/event
  - $parent/$children
  - ref
  - provide/inject
  - .sync
- 非父子组件通信
  - eventBus
  - 通过根实例$root
  - vuex
  - $attr/$listeners
  - Provide/inject

### 22.computed/watch的区别是什么

- computed是依赖于其他属性的一个计算值，并且具备缓存，只有当依赖的值变化才会更新
- watch是在监听的属性发生变化的时候，触发一个回调，在回调中执行一些逻辑

### 23.对SPA单页面的理解，以及优缺点

> `SPA(singlg-page application)` 仅在Web页面初始化时加载相应的`Html`，`JavaScript`，`Css`，一旦页面加载完成，`SPA`不会因为用户的操作而进行页面的重新加载或跳转，取而代之的是利用路由机制实现`Html`内容的变换，`UI`与用户的交互，避免页面的重新加载

- 优点：
  - 用户体验好，加载速度快，内容改变不需要重新加载整个页面，避免了不必要的跳转和重复渲染
  - SPA相对对服务器压力小
  - 前后端职责分离，架构更清晰，前端进行交互逻辑，后端负责数据处理
- 缺点：
  - 首次加载耗时较多（为实现单页面Web应用功能及显示效果，需要在加载页面的时候将`JavaScript`，`Css`统一加载，部分页面按需加载）
  - 由于单页面应用在一个页面中显示所有的内容，所以不能使用浏览器的前进后退功能，所有的页面切换需要自己建立堆栈管理
  - 不利于SEO优化

### 24.vue-router中有哪些路由守卫 

### 25.vue-router中hash/history两种模式的区别

- hash模式的url会显示'#'，而history没有
- 刷新页面时，hash模式可以正常加载到hash对应的页面，而history没有处理就会返回404
- 兼容性上，hash模式可以支持低版本浏览器和IE

### 26.vue-router中hash/history是如何实现的

- hash模式
  - `#`后面的`hash`值变化，不会导致浏览器向服务器发出请求，浏览器不发出请求，就不会刷新页面，同时可以通过监听`hashchange`事件知道hash发生了哪些变化
- history模式
  - 主要是`html5`标准发布的两个api（`pushState`和`replaceState`），这两个api可以改变url，但是不会发送请求

## Vuex

### vuex有什么优缺点

**优点：**

- 解决了非父子组件的消息传递
- 减少了ajax请求次数

**缺点**

- 刷新浏览器，vuex中的`State`就会重新变回初始化状态

### vuex中的state有什么特性？

- vuex就是一个仓库，其中`state`就是数据源存放地
- `state`里面存放的数据是响应式的，`Vue`组件从`store`中读取数据，若是store中的数据改变，则依赖这个数据的组件都会更新数据
- 可以通过`mapState`把全局的`state`和`getters`映射到当前组件的`computed`计算属性中

### vuex中的getters有什么特性？



## 相关链接

- [为什么 Vue 中不要用 index 作为 key？](https://juejin.im/post/6844904113587634184)