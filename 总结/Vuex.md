## Vuex是什么

Vuex是一个专为Vue.js应用程序开发的状态管理模式。采用集中式存储管理应用的所有组件的状态。

### 单向数据流

<img src="../assets/flow.png" alt="单向数据流理念图" style="zoom:50%;" />

- `state`：驱动应用的数据源
- `view`：以声明方式将state映射到视图
- `actions`：响应view上用户输入导致的状态变化

**Vuex是专门为Vue.js设计的状态管理库，以利用Vue.js的细粒度数据响应机制来进行高效的状态变更**

<img src="../assets/vuex.png" alt="vuex数据流图" style="zoom:70%;" />

### 概述

每一个Vuex应用的核心就是store，“store”就是一个容器，包含着应用中大部分的状态

## 核心概念

### State

Vue使用单一状态树，用一个对象就包含了全部的应用层级状态。

#### 如何在组件中获取vuex中的状态

由于Vuex的状态存储是响应式的，从store实例中读取状态的最简单方法就是在计算属性中返回某个状态

在初始化应用实例时，通过将store传入，可以将状态注入到每一个子组件中

```javascript
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return this.$store.state.count
    }
  }
}
```

当一个组件需要多个状态时，将这些状态都声明为计算属性有些重复和冗余，可以使用`mapState`辅助函数帮助生成计算属性：

```javascript
import { mapState } from 'vuex'

export default {
  computed: mapState({
    count: state => state.count,
    
    countAlias: 'count' // 等价于 state => state.count
  })
}
```

### Getter

类似于Vue中的computed，在vuex中也有一个计算属性“getter”，可以缓存数据，只有当数据更改时才会重新计算。

#### 使用方式

```javascript
// store.js
const store = new Vuex.Store({
  state: {
    todos: []
  },
  getters: {
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    }
  }
})

// TodoList.vue
export default {
  computed: {
    doneTodosCount() {
      return this.$store.getters.doneTodos.length
    }
  }
}
```

#### 通过方法访问

```javascript
new Vue.Store({
  // ...
  getters: {
    getTodoById: state => id => {
      return state.todos.find(todo => todo.id === id)
    }
  }
})
```



### Mutation

在Vuex中更改store的状态的唯一方法就是提交mutation。

- 提交方式一：

  ```javascript
  store.commit('increment', { amount: 10 })
  ```

- 对象风格提交

  ```javascript
  store.commit({ type: 'increment', amount: 10 })
  ```

#### 遵守Vue响应规则

- 最好提前在store中初始化好所需属性

- 当在对象上添加新属性时，应该使用：

  - `Vue.set(obj, 'newProp', 123)`

  - 以新对象替换老对象，例如对象展开符：

    ```javascript
    state.obj = { ...state.obj, newProp: 123 }
    ```

#### 使用常量替代Mutation事件类型

```javascript
// mutation-types.js
export const SOME_MUTATION = 'SOME_MUTATION'

// store.js
import Vuex from 'vuex'
import { SOME_MUTATION } from './mutation-types'

const store = new Vuex.Store({
  state: { ... },
  mutations: {
    // 我们可以使用 ES2015 风格的计算属性命名功能来使用一个常量作为函数名
    [SOME_MUTATION] (state) {
      // mutate state
    }
  }
})
```

#### 组件中提交Mutation

```javascript
import { mapMutations } from 'vuex'
export default {
  methods: {
    ...mapMutations([
      'increment'
    ]),
    // 或者
    ...mapMutations({
      add: 'increment'
    })
  }
}
```



### Action

#### 特点

- 提交的是mutation，而不是直接变更状态
- 可以包含任意异步操作

Action的第一个参数是一个与store实例具有相同方法和属性的context对象

#### 分发方式

- 载荷形式分发

  ```javascript
  store.dispatch('incrementAsync', { amount: 10 })
  ```

- 对象形式

  ```javascript
  store.dispatch({ type: 'incrementAsync', amount: 10 })
  ```

  

### Module

vuex允许我们将store分割成模块，每个模块有自己的state、mutation、action、getter。

```javascript
const moduleA = {
  state: () => ({ ... }),
  mutations: {},
  actions: {},
  getters: {}
}
```

对于模块内部的action，局部状态通过`context.state`暴露出来，根节点状态则为`context.rootState`

```javascript
const moduleA = {
  actions: {
    increment({ state, commit, rootState }) {}
  }
}
```

对于模块内部的getter，根节点状态会作为第三个参数暴露出来

```javascript
const moduleA = {
  getters: {
    sumWithRootCount(state, getters, rootState) {}
  }
}
```

#### 命名空间

如果希望模块具有更高的封装性和复用性，可以通过添加`namespaced: true`的方式使其成为带命名空间的模块

```javascript
new Vue.Store({
  modules: {
    foo: {
      namespaced: true,
      state: () => ({}),
      getters: {},
      actions: {},
      mutations: {}
    }
  }
})
```

#### 在带命名空间的模块内访问全局内容

`rootState`和`rootGetters`会作为第三和第四个参数传入getter，也可以通过`context`对象的属性传入action

#### 在带命名空间的模块中注册全局action

若需要在带命名空间的模块注册全局action，可以添加`root: true`

```javascript
export default {
  namespaced: true,
  actions: {
    someAction: {
      root: true,
      handler(namespacedContext, payload) {}
    }
  }
}
```

#### 带命名空间的绑定函数

```javascript
export default {
  computed: {
    ...mapState('module', {
      a: state => state.a
    })
  },
  methods: {
    ...mapActions('module', ['foo', 'bar'])
  }
}
```

也可以通过使用`createNamespacedHelpers`创建基于某个命名空间的辅助函数，这会返回一个对象

```javascript
import { createNamespacedHelpers } from 'vuex'
const { mapState, mapActions } = createNamespacedHelpers('some/nested/module')
```

这样使用可以不添加命名空间，直接对store的内容进行分发

## 源码

### 目录

```
vuex
└── src
    ├── module
    │   ├── module-collection.js
    │   └── module.js
    ├── plugins
    │   ├── devtool.js
    │   └── logger.js
    ├── helpers.js # 包含一些诸如mapState、mapMutations等辅助函数
    ├── index.js # 入口文件，导出vuex的函数
    ├── mixin.js # 将store注入到每个组件中
    ├── store.js # 主要功能文件
    ├── util.js # 工具函数
```

### store.js

#### 构造函数constructor

```javascript
class Store {
  constructor(options = {}) {
    if (!Vue && typeof window !== 'undefined' && window.Vue) {
      install(window.vue)
    }
    
    // 声明实例的数据
    this._modules = new ModuleCollection(options)
    this._watcherVM = new Vue()
    // ... 其他变量
    
    
    const store = this
    const { dispatch, commit } = this
    this.dispatch = function boundDispatch(type, payload) {
      return dispatch.call(store, type, payload)
    }
    this.commit = function boundCommit(type, payload, options) {
      return commit.call(store, type, payload, options)
    }
    
    const state = this._modules.root.state
    
    // 初始化根模块
    // 递归地注册所有子模块
    // 搜集所有模块的getters到this._wrappedGetters中
    installModule(this, state, [], this._modules.root)
    
    // 初始化负责响应式的存储对象vm
    // 并且将 _wrappedGetters注册为计算属性
    resetStoreVM(this, state)
    
    // 应用插件
    plugins.forEach(plugin => plugin(this))
  }
}
```

从构造函数可以看到其主要功能就是：

1. 将store注入到每个组件中
2. 初始化一些必要的变量，存储状态数据，监听的函数
3. 缓存getters

#### commit和dispatch

```javascript
class Store {
  commit(_type, _payload, _options) {
    const { type, payload, options } = unifyObjectStyle(_type, _payload, options)
    const mutation = { type, payload }
    
    const entry = this._mutations[type]
    // 将需要执行的mutation函数包裹在一个函数中
    // mutation函数执行时需要将 this._commiting打开
    // 执行结束后再重置为执行前的状态
    this._withCommit(() => {
      entry.forEach(function commitIterator(handler) {
        handler(payload)
      })
    })
    
    // mutation事件订阅
    this._subscribers
    	.slice() // 浅拷贝，防止订阅者同步取消订阅时，遍历失效
      .forEach(sub => sub(mutation, this.state))
  }
  
  dispatch(_type, _payload) {
    const { type, payload } = unifyObjectStyle(_type, _payload)
    const action = { type, payload }
    const entry = this._actions[type]
    
    // 首先将事件订阅器中设置了before的执行完
    try {
      this._actionSubscribers
      	.slice()
        .forEach(sub => sub.before(action, this.state))
    } catch(e) {}
    
    // 由于action是异步调用，多个的时候就用Promise.all包裹住
    const result = entry.length > 1
    	? Promise.all(entry.map(handler => handler(payload)))
    	: entry[0](payload)
    
    // 返回一个Promise
    return new Promise((resolve, reject) => {
      // Promise.all执行后，
      // 如果执行成功，执行action事件监听器中的after钩子
      // 将结果暴露到外层
      // 如果执行失败，就执行error钩子
      // 并将错误暴露到外层
      result.then(res => {
        try {
          this._actionSubscribers
          	.filter(sub => sub.after)
          	.forEach(sub => sub.after(action, this.state))
        } catch(e) {}
        resolve(res)
      }, error => {
        try {
          this._actionSubscribers
          	.filter(sub => sub.error)
          	.forEach(sub => sub.error(action, this.state, error))
        } catch(e) {}
        reject(error)
      })
    })
  }
}
```

从commit和dispatch的源码中可以看出两种方式的实现其实有些相似，都是讲同一种`type`的事件存到一个数组中，当使用`commit`和`dispatch`调用时，取出该事件数组，依次执行。在执行的过程中，也会对手动订阅的某个事件进行调用。比如通过commit的`subscribe`订阅添加一个事件，将每次commit存为一个快照。

#### resetStoreVM (todo)

```javascript
function resetStoreVM(store, state, hot) {
  const oldVm = store._vm
  
  // 绑定一个公共的getters
  store.getters = {}
  // 重置局部getter缓存
  store._makeLocalGettersCache = Object.create(null)
  const wrappedGetters = store._wrappedGetters
  const computed = {}
  forEachValue(wrappedGetters, (fn, key) => {
    // 设置getters的键为一个参数仅为store的方法
    computed[key] = partial(fn, store)
    // 设置访问器属性，当访问 store.getters[key]时
    // 对应的数值从store._vm中获取
    Object.defineProperty(store.getters, key, {
      get: () => store._vm[key],
      enumerable: true
    })
  })
  
  const silent = Vue.config.silent
  Vue.config.silent = true
  // 设置 .vm为一个存储了store所有state
  // 且getters对应为computed的vue实例
  store._vm = new Vue({
    data: {
      $$state: state
    },
    computed
  })
  Vue.config.silent = silent
  if (store.strict) {
    enableStrictMode(store)
  }
  if (oldVm) {
    if (hot) {
      store._withCommit(() => {
        oldVm._data.$$state = null
      })
    }
    // 下一个循环结束后，销毁旧的.vm实例
    Vue.nextTick(() => oldVm.$destroy())
  }
}
```

该方法的主要功能就是将getters数据的getter关联到一个Vue实例的computed上，通过computed实现数据计算以及缓存功能，并清除上一个Vue实例

### installModule

```javascript
function installModule(store, rootState, path, module, hot) {
  // 是否是根模块
  const isRoot = !path.length
  // 获取当前模块的命名
  const namespace = store._modules.getNamespace(path)
  
  // 如果是命名模块，则将其映射到._modulesNamespaceMap上
  if (module.namespaced) {
    store._modulesNamespaceMap[namespace] = module
  }
  
  // 如果不是根模块，且不是hot（好吧，我暂时也不知道hot是什么）
  if (!isRoot && !hot) {
    // 获取当前模块的父级模块的state
    const parentState = getNestedState(rootState, path.slice(0, -1))
    // 获取当前模块对应的模块名字
    const moduleName = path[path.length - 1]
    // 将该模块设置成响应式
    Vue.set(parentState, moduleName, module.state) // I
  }
  
  // 设置一个具有局部操作的对象（包含dispatch、commit，state，getters属性或方法）
  const local = module.context = makeLocalContext(store, namespace, path) // II
  
  // 将mutation、action、getter等注册到store中
  module.forEachMutation((mutation, key) => {
    const namespacedType = namespace + key
    registerMutation(store, namespacedType, mutation, local)
  })
  
  module.forEachAction((action, key) => {
    const type = action.root ? key : namespace + key
    const handler = action.handler || action
    registerAction(store, type, handler, local)
  })
  
  module.forEachGetter((getter, key) => {
    const namespacedType = namespace + key
    registerGetter(store, namespacedType, getter, local)
  })
  
  // 递归调用该方法（作用不用我多说了吧）
  module.forEachChild((child, key) => {
    installModule(store, rootState, path.concat(key), child, hot)
  })
}
```

该方法的主要功能就是递归的将模块设置成响应式，并注册mutation、actions和getters

#### makeLocalContext

```javascript
function makeLocalContext(store, namespace, path) {
  const noNamespace = namespace === ''
  
  // 创建一个包含commit和dispatch的对象
  // 当调用两个方法时，生成一个对应的type
  // 并调用当前store对应的方法
  const local = {
    dispatch: noNamespace ? store.dispatch ? (_type, _payload, _options) {
      const args = unifyObjectStyle(_type, _payload, _options)
      const { payload, options } = args
      let { type } = args
      if (!options || !options.root) {
        type = namespace + type
      }
      return store.dispatch(type, payload)
    },
    commit: noNamespace ? store.commit ? (_type, _payload, _options) {
      // ...
    }
  }
  // 设置local的两个属性的get方法
  Object.defineProperties(local, {
    getters: {
      get: noNamespace
        ? () => store.getters
        : () => makeLocalGetters(store, namespace)
    },
    state: {
      get: () => getNestedState(store.state, path)
    }
  })
  return local
}
```

该方法作用是创建一个局部的对象，用于store中我们定义的mutations、actions和getters的参数

### 向组件中注入store

```javascript
function applyMixin(Vue) {
  const version = Number(Vue.version.split('.')[0])
  if (version >= 2) {
    Vue.mixin({ beforeCreate: vuexInit })
  } else {
    const _init = Vue.prototype._init
    Vue.prototype._init = function(options = {}) {
      options.init = options.init ? [vuexInit].concat(options.init) : vuexInit
      _init.call(this, options)
    }
  }
  
  function vuexInit() {
    const options = this.$options // Vue实例上的属性都会存到 $options
    if (options.store) {
      this.$store = typeof options.store === 'function' ? options.store() : options.store
    } else if (options.parent && options.parent.$store) {
      this.$store = options.parent.$store
    }
  }
}
```

根据Vue版本的不同，向Vue的第一个生命周期中添加一个`vueInit`方法

之所以能通过`$options.store`访问到数据仓库，就是因为在初始化应用程序的时候，我们向其中传入了一个**Vuex.Store实例**

```javascript
import Vue from 'vue'
import store from './store/index'
import App from './App.vue'

new Vue({
  store, // 在$options中访问到的store
  render: h => h(App)
}).$mount('#app')
```

只有在根组件才能通过`$options.store`访问仓库，子组件只能通过`$optoins.parent.$store`访问

