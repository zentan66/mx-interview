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

### Module

