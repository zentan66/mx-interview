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

