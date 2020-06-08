## 基本内容

> Vue是一款用于构建用户界面的渐进式JavaScript框架，其核心库只关注图层

### 条件与循环

#### 条件v-if和v-show

Vue中会尽可能高效地渲染元素，所以会复用已有元素而不是从头开始渲染，所以需要使用`key`管理可复用的元素

```html
<template v-if="loginType === 'username'">
	<input placeholder="Enter your username" key="username" />
</template>
<template v-else>
	<input placeholder="Enter your email address" key="email" />
</template>
```

**不推荐v-for和v-if同时使用，因为v-for具有比v-if更高的优先级**

#### 解析DOM模板

在`<ul>`、`<ol>`、`<table>`和`<select>`等元素下，内容是有一定限制的，所以可以在内部标签上加`is`

```html
<ul>
  <li is="todo-item" v-for="todo in todos" :key="todo.id"></li>
</ul>
```





### 指令

## 生命周期

### beforeCreate

### created

### beforeMount

###  Mounted

### beforeUpdate

### updated

### beforeDestroy

### destroyed



## 数据传递

### 单向数据流

所有的prop都使得其父子prop之间形成了一个单向下行绑定

### 父子组件

**通过prop向子组件传递数据**

```html
<blog-item :post="post" ></blog-item>
```

**通过事件向父组件传递数据**

```html
<!-- 父组件 -->
<blog-item @handle-click="handleClick"></blog-item>

<!-- 子组件 -->
<button @click="$emit('handle-click', 1)">
  Click here
</button>
```



## 组件

### 组件注册

组件的类型有两种，一种全局的，另一种就是局部的

**全局组件**

```javascript
import Vue from 'vue'
import BaseButton from './components/BaseButton'

Vue.component('base-button', BaseButton)
```

**局部组件**

```vue
<template>
	<div>
    <BaseButton></BaseButton>
  </div>
</template>
<script>
import BaseButton from './components/BaseButton';
export default {
  components: {
    'base-button': BaseButton
  }
}
</script>
```

如果在webpack环境下，可以通过`require.context`自动全局注册非常通用的基础组件

```javascript
import Vue from 'vue'
import upperFirst from 'lodash/upperFirst'
import camelCase from 'lodash/camelCase'

const requireComponent = require.context(
  // 组件目录相对路径
  './components',
  // 是否查询其子目录
  false,
  // 匹配基础组件文件名的正则表达式
  /Base[A-Z]\w+\.(vue|js)$/
)

requireComponent.keys().forEach(fileName => {
  const componentConfig = requireComponent(fileName)

  const componentName = upperFirst(camelCase(fileName.split('/').pop().replace(/\.\w+$/, '')))
  
  Vue.component(componentName, componentConfig.default || componentConfig)
})
```

### 自定义事件

**自定义组件的v-model**

在自定义组件上使用`v-model`时，组件内部默认会利用`value`和`input`事件，而在一些特殊的情况下则可以用`model`选项避免一些冲突：

```vue
<template>
	<input type="checkbox" :checked="checked" @change="$emit('change', $event.target.checked)" />
</template>
<script>
export default {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    checked: Boolean
  }
}
</script>
```

**快速将绑定到组件上的事件传递给内部元素**

如果想在一个组件上直接监听一个原生事件，可以使用`native`修饰符：

```vue
<base-input @focus.native="onFocus" />
```

但是如果内部元素的根元素是一个`div`或者其他元素，那么`native`将静默失败，也不会产生任何报错，为了解决这个问题，vue提供了一个`$listener`property，这是一个对象，包含了作用在这个组件上的所有监听器。

因为子组件可以这样写：

```vue
<template>
	<label>
  	<input v-bind="$attrs" :value="value" v-on="inputListeners" />
  </label>
</template>
<script>
export default {
  inheritAttrs: false,
  props: ['label', 'value'],
  computed: {
    inputListeners: function() {
      var vm = this
      return Object.assign({}, this.$listeners, {
        input: function(event) {
          vm.$emit('input', event.target.value)
        }
      })
    }
  }
}
</script>
```

**.sync修饰符**

在某些情况下，可能需要对一个prop进行“双向绑定”，不幸的是真正的双向绑定会带来维护问题，所以官方推荐使用`update:myPropName`的模式触发事件取而代之。例如：

```vue
<!-- 父组件 -->
<Child :title.sync="docTitle" />

<!-- 上述写法等于 -->
<Child :title="docTitle" @update:title="docTitle = $event" />
```

**有一点儿需要注意的是带了.sync修饰符的v-bind不能和表达式一起使用(v-bind:title.sync="docTitle + '!'")，这将是无效的**

但一个对象设置多个prop的时候，可以将`sync`和`v-bind`结合

```vue
<text-doc v-bind.sync="doc" />
```

### 插槽

##### 具名插槽

在2.6之后的版本中，子组件内的多个插槽可以通过设置一个name值，使之可以具体定位到某个元素

```vue
<template>
	<div>
    <slot name="header" />
    <slot /> <!-- 默认插槽 相当于name="default" -->
    <slot name="footer" />
  </div>
</template>
```

而我们在父组件里面就可以通过`v-slot`来设置具名插槽

```vue
<base-layout>
	<template v-slot:header>
  	<h1>
      Here might be a page title
    </h1>
  </template>
  <p>
    And another one.
  </p>
  <template v-slot:footer>
  	<p>
      Here's some contact info
    </p>
  </template>
</base-layout>
```

##### 在父组件插槽中访问子组件数据

有时候会想着在父组件中访问到子组件内的数据，那么我们可以这样写：

```vue
<!-- 子组件 -->
<span>
	<slot v-bind:user="user">{{ user.lastName }}</slot>
</span>
<!-- 父组件 -->
<current-user>
	<template v-slot:default="slotProps">
  	{{ slotProps.user.firstName }}
  </template>
</current-user>
```

绑定在`<slot>`元素上的attribute被称为`插槽prop`。

##### 独占默认插槽的缩写语法

当提供的内容只有默认插槽时，组件的标签才可以被当作插槽的模板来使用：

```vue
<current-user v-slot:default="slotProps">
	{{ slotProps.user.firstName }}
</current-user>
```

在2.6.0新增了`v-slot`的缩写形式`#`，例如`v-slot:header`就可以被写作`#header`

但是这种语法必须在有参数的时候才可用，下面这种写法就会触发警告：

```vue
<current-user #="{user}">
	{{ user.firstName }}
</current-user>
```

### 动态组件&异步组件

