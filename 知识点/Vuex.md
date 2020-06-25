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

### Vuex构造函数

```javascript
class Store {
  constructor(options = {}) {
    if (!Vue && typeof window !== 'undefined' && window.Vue) {
      install(window.vue)
    }
    this._modules = new ModuleCollection(options) 
    
    const state = this._modules.root.state
    
    installModule(this, state, [], this._modules.root)
    
    resetStoreVM(this, state)
    
    plugins.forEach(plugin => plugin(this))
  }
}
```

1. `ModuleCollection`是一个用于存储Vuex数据的地方，通过Store传入的诸如state、mutations等都存在`_modules`当中
2. `resetStoreVM`方法用于对`getters`的数据进行缓存

### installModule

```javascript
function installModule(store, rootState, path, module, hot) {
  const isRoot = !path.length
  const namespace = store._modules.getNamespace(path)
  
  if (module.namespaced) {
    store._modulesNamespaceMap[namespace] = module
  }
  
  if (!Root && !hot) {
    const parentState = getNestedState(rootState, path.slice(0, -1))
    const moduleName = path[path.length - 1]
    Vue.set(parentState, moduleName, module.state) // I
  }
  
  const local = module.context = makeLocalContext(store, namespace, path) // II
  
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
  
  module.forEachChild((child, key) => {
    installModule(store, rootState, path.concat(key), child, hot)
  })
}
```

1. 在标记`I`处，可以看到使用了`Vue.set`方法，将每一个模块下的数据设置成响应式数据
2. 标记`II`处是设置一个局部上下文对象，当调用`commit`或`dispath`能自动的对每个模块的调用路径进行设置

### resetStoreVM

```javascript
function resetStoreVM(store, state, hot) {
  const oldVm = store._vm
  store.getters = {}
  store._makeLocalGettersCache = Object.create(null)
  const wrappedGetters = store._wrappedGetters
  const computed = {}
  forEachValue(wrappedGetters, (fn, key) => {
    computed[key] = partial(fn, store)
    Object.defineProperty(store.getters, key, {
      get: () => store._vm[key],
      enumerable: true
    })
  })
  
  const silent = Vue.config.silent
  Vue.config.silent = true
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
    Vue.nextTick(() => oldVm.$destroy())
  }
}
```

上述代码首先是对`getters`上的数据的`get`添加了拦截操作，而后创建一个带有`data`和`computed`数据的Vue实例，并在下一个事件循环之后将之前初始化的Vue实例销毁

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
    const options = this.$options
    if (options.store) {
      this.$store = typeof options.store === 'function' ? options.store() : options.store
    } else if (options.parent && options.parent.$store) {
      this.$store = options.parent.$store
    }
  }
}
```

从上述代码中可以看到，该方法会根据Vue版本，在每一个组件创建时执行一个`vuexInit`的方法，将store的数据注入到当前组件中

### dispatch

```javascript
export default {
  dispatch(_type, _payload) {
    // 根据type类型，找出真正的type和payload数据
    const { type, payload } = unifyObjectStyle(_type, _payload)
    const action = { type, payload }
    // 根据触发的事件类型，找到对应的事件
    const entry = this._actions[type]
    
    try {
      this._actionSubscribers
      	.slice()
        .filter(sub => sub.before)
        .forEach(sub => sub.before(action, this.state))
    } catch(e) {}
    
    const result = entry.length > 1 ? Promise.all(entry.map(handler => handler(payload))) : entry[0](payload)
  }
}
```

