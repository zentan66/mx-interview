## 目录

- [生命周期](##生命周期)
- [数据传递](##数据传递)
- [组件](##组件)
- [进入/离开&列表过渡](##进入/离开&列表过渡)
- [可复用性](##可复用性)
- [自定义指令](##自定义指令)
- [渲染函数&JSX](##渲染函数&JSX)

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

#### 具名插槽

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

#### 在父组件插槽中访问子组件数据

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

#### 独占默认插槽的缩写语法

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

#### keep-alive

在vue中可以通过`is`切换不同的组件

```vue
<component :is="componentName"></component>
```

但是切换不同的组件就会重新创建组件，有时候我们会想将组件第一次创建后的状态缓存下来，为了解决这个问题我们可以使用一个`<keep-alive>`标签将动态组件包裹：

```vue
<keep-alive>
	<component :is="componentName"></component>
</keep-alive>
```

**\<keep-alive\>要求被切换到的组件都有自己的名字，不论是通过组件的name选项还是局部/全局注册**

#### 异步组件

在大型应用中，可能需要将应用分割成小一些的代码块，并且只在需要的时候才从服务器加载一个模块

```javascript
Vue.component('async-webpack-example', function(resolve) {
  require(['./my-async-component'], resolve)
})
```

也可以把webpack与ES2015语法加在一起：

```javascript
Vue.component('async-webpack-example', () => import('./my-async-component'))
```

在2.3.0+新增了加载状态：

```javascript
const AsyncComponent = () => ({
  component: import('./MyComponent.vue'),
  // 异步组件加载时使用的组件
  loading: LoadingComponent,
  // 加载失败时使用的组件
  error: ErrorComponent,
  // 展示加载时组件的延时时间
  delay: 200,
  // 如果提供了超时时间且组件加载也超时
  timeout: 3000
})
```

### 处理边界情况

#### 访问根组件

> this.$root

#### 访问父组件实例

> this.$parent

#### 访问子组件实例或子元素

尽管存在prop和事件，有时候需要在JavaScript里面访问一个子组件，这时候就可以使用`ref`为子组件添加一个ID引用

```vue
<base-input ref="myInput"></base-input>
```

在定义了这个`ref`的组件里就可以使用：

```javascript
this.$refs.myInput
```

访问这个组件实例

**$refs只会在组件渲染完成后生效，并且不是响应式的。应该避免在模板或计算属性中访问$refs**

#### 依赖注入

vue中提供了`provide`以及`inject`选项，允许我们将数据跨层级传递

比如在一个组件内部定义了一个`getMap`方法：

```javascript
export default {
  provide: function() {
    getMap: this.getMap
  }
}
```

然后在这个组件的任何后代组件里，我们都可以使用`inject`选项来接收我们想要添加在这个实例上的property：

```javascript
export default {
  inject: ['getMap']
}
```

**好处：**

- 祖先组件不需要知道哪些后代组件使用它提供的property
- 后代组件不需要知道被注入的property来自哪里

**负面影响：**

- 应用程序组件和当前组织方式耦合起来，重构更加困难
- 提供的property是非响应式的，共享的这个property是应用特有的，而不是通用的

### 程序化的事件侦听器

Vue实例在其事件接口中提供了其他的方法：

- 通过`$on(eventName, eventHandler)`侦听一个事件
- 通过`$once(eventName, eventHandler)`一次性侦听一个事件
- 通过`$off(eventName, eventHandler)`停止侦听一个事件

有一种模式在第三方库中经常可见：

```javascript
export default {
  mounted() {
    this.picker = new Pikaday({})
  },
  beforeDestroy() {
    this.picker.destroy()
  }
}
```

上述例子有两个潜在问题：

- 组件实例中保存这个`picker`，如果可以的话最好只有在生命周期钩子可以访问到
- 建立代码独立于清理代码，比较难于程序化地清理我们建立的所有东西

可以通过一个程序化的侦听器解决这两个问题：

```javascript
export default {
  mounted() {
    var picker = new Pikaday({})
    this.$once('hook:beforeDestroy', function() {
      picker.destroy()
    })
  },
}
```

通过这个策略，甚至可以让多个输入框元素同时使用不同的Pikaday，每个新的实例都程序化地在后期清理它自己：

```javascript
export default {
  mounted() {
    this.attachDatepicker('startDateInput')
    this.attachDatepicker('endDateInput')
  },
  methods: {
    attachDatepicker: function(refName) {
      var picker = new Pikaday({
        field: this.$refs[refName]
      })
      this.$once('hook:beforeDestroy', function() {
        picker.destroy()
      })
    }
  }
}
```

### 循环引用

#### 递归组件

组件可以再自己的模板中调用自身的，不同必须要有`name`选项：

```javascript
name: 'unique-name-of-component'
```

但是这种用法很可能就会导致无限循环，所以请保证递归调用是条件性的

#### 组件之间的循环引用

假设构建一个文件目录树，就可能需要一个`tree-folder`组件，模板中有一个`tree-folder-contents`：

```vue
<template>
	<p>
    <span>{{folder.name}}</span>
    <tree-folder-contents :children="folder.children" />
  </p>
</template>
```

还有一个`<tree-folder-contents>`组件，模板是这样的：

```vue
<template>
  <ul>
    <li v-for="child in children">
      <tree-folder v-if="child.children" :folder="child.children" />
    </li>
  </ul>
</template>
```

仔细观察会发现这些组件在渲染树中互为对方的后代和祖先——悖论，当然可以通过`Vue.component`全局注册组件

如果使用一个模块系统依赖/导入组件，就会遇见一个错误：

```
Failed to mount component: template or render function not defined.
```

我们可以通过在生命周期钩子`beforeCreate`时注册内部组件：

```javascript
export default {
  beforeCreate() {
    this.$options.components.TreeFolderContents = require('./tree-foler-contents')
  }
}
```

或者本地注册组件的时候，使用webpack的异步`import`：

```javascript
export default {
  components: {
    TreeFolderContents: () => import('./tree-folder-contents')
  }
}
```

### 控制更新

#### 强制更新

如果在已经注意到数组和对象的变更检测注意事项，或者可能依赖了一个未被Vue响应系统追踪的状态，但是还是在某些情况下需要手动强制更新，那么可以使用：

> $forceUpdate更新状态

#### 通过v-once创建低开销的静态组件

渲染普通HTML元素在vue中是非常快速的，但是有时候组件中包含大量的静态内容，这个时候可以在根元素上添加`v-once`，确保内容只计算一次然后缓存起来：

```vue
<template>
	<div v-once>
    <h1>
      Terms Of Service
  </h1>
  </div>
</template>
```

**不能过度使用这个模式，当需要渲染大量静态内容时，极少数情况下会给你带来便利**

## 进入/离开&列表过渡

### 单元素/组件过渡

Vue提供了`transition`的封装组件，在以下情形中，可以给任何元素和组件添加进入/离开过渡：

- 条件渲染(`v-if`)
- 条件展示(`v-show`)
- 动态组件
- 组件根节点

下列就是一个经典例子：

```vue
<div>
  <button @click="show = !show">
    Toggle
  </button>
  <tranition name="fade">
  	<p v-if="show">
      hello
    </p>
  </tranition>
</div>
```

```css
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
```

#### 过渡类名

- `v-enter`：进入过渡的开始状态
- `v-enter-active`：定义进入过渡生效时的状态
- `v-enter-to`：定义进入过渡的结束状态。在元素被插入之后下一帧生效，在过渡/动画完成之后移除
- `v-leave`：定义离开过渡的开始状态
- `v-leave-active`：定义离开过渡生效时的状态
- `v-leave-to`：定义离开过渡的结束状态

<img src="https://github.com/zentan66/peggy-interview-groups/raw/master/assets/transition.png" style="zoom:50%;" />

#### 自定义过渡的类名

可以通过attribute来自定义过渡类名：

- `enter-class`
- `enter-active-class`
- `enter-to-class`
- `leave-class`
- `leave-active-class`
- `leave-to-class`

优先级高于普通类名

```vue
<link href="animate.css">

<transition name="custom-classes-transition" enter-active-class="animated tada" leave-active-class="animated bounceOutRight">
	<p v-if="show">
    hello
  </p>
</transition>
```

#### javascript钩子

可以在attribute中声明JavaScript钩子

```vue
<transition
  @before-enter="beforeEnter"
  @enter="enter"
  @after-enter="afterEnter"
  @enter-cancelled="enterCancelled"
  @before-leave="beforeLeave"
  @leave="leave"
  @after-leave="afterLeave"
  @leave-cancelled="leaveCancelled"
></transition>
```

```javascript
export default {
  beforeEnter(el) {},
  enter(el, done) {},
  afterEnter(el) {},
  beforeLeave(el) {}
}
```

这些钩子函数可以结合CSS`transition/animations`使用，也可以单独使用

**当只用JavaScript过渡的时候，在enter和leave中必须使用done回调，否则会被同步调用，过渡立即完成**

#### 初始渲染的过渡

可以通过`appear`attribute设置节点在初始渲染的过渡：

```vue
<transition appear></transition>
```

#### 过渡模式

transition的默认行为-进入和离开同时发生

在同时生效的进入和离开的过渡不能满足所有要求，Vue提供了**过渡模式**

- `in-out`：新元素先进行过渡，完成之后当前元素过渡离开
- `out-in`：当前元素先进行过渡，完成之后新元素过渡进入

### 列表过渡

单个节点过渡就用`transition`，但是多个节点同时渲染的时候，就可以使用`transition-group`组件，这个组件的几个特点：

- 不同于`<transition>`，该元素会以一个真实元素呈现：默认是`<span>`，可以通过`tag`attribute更换为其他元素
- 过渡模式不可用
- 内部元素总需要提供一个唯一的`key`attribute值
- CSS过渡的类将会应用在内部的元素中，而不是这个组/容器本身

##### 列表的排序过渡

`<transition-group>`组件有一个特殊之处。不仅可以进入和离开动画，还可以改变定位。要使用这个新功能只需了解新增的`v-move`class。

## 可复用性

### 混入

混入提供了一种非常灵活的方式，来分发Vue组件中的可复用功能。

```javascript
var myMixin = {
  create() {},
  methods: {
    hello() {}
  }
}

var Component = Vue.extend({
  mixins: [myMixin]
})
```

当组件和混入对象含有同名选项时，这些选项将以恰当的方式进行"合并"。当选项发生冲突时以组件数据优先。

同名钩子函数将合并为一个数组，都将被调用。混入对象的钩子将在组件自身钩子之前调用。

值为对象的选项，例如`methods`、`components`和`directives`将被合并为同一个对象。两个对象键名冲突时，取组件对象的键值对。

#### 全局混入

混入可以全局注册，使用时格外小心！一旦使用全局混入，将影响每一个之后创建的Vue实例。

```javascript
Vue.mixin({
  created() {}
})
```

#### 自定义选项合并策略

自定义选项使用默认策略，即简单地覆盖已有值。如果想让自定义选项以自定义逻辑合并，可以向`Vue.config.optionMergeStrategies`添加一个函数：

```javascript
Vue.config.optionMergeStrategies.myOption = function(toVal, fromVal) {}
```

## 自定义指令

如果需要对普通DOM元素进行底层操作，这个时候就会用到自定义指令：

```javascript
Vue.directive('focus', {
  // 当被绑定元素插入到DOM中时……
  inserted(el) {
    el.focus();
  }
})
```

也可以注册局部指令：

```javascript
export default {
  directives: {
    focus: {
      inserted(el) {
        el.focus();
      }
    }
  }
}
```

### 钩子函数

一个指令定义对象可以提供以下几个钩子函数：

- `bind`：只调用一次，指令第一次绑定到元素时调用
- `inserted`：被绑定元素插入父节点时调用
- `update`：所有组件的VNode更新时调用，可能发生在其子节点VNode更新之前
- `componentUpdated`：指令所在组件的VNode及其子VNode全部更新后调用
- `unbind`：只调用一次，指令与元素解绑时调用

#### 钩子函数参数

- `el`：指令所绑定的元素，可直接操作DOM
- `binding`：一个对象，包含以下property
  - `name`：指令名
  - `value`：指令绑定值
  - `oldValue`：指令绑定的前一个值
  - `expression`：字符串形式的指令表达式
  - `arg`：传给指令的参数
  - `modifiers`：一个包含修饰符的对象
- `vnode`：vue编译生成的虚拟节点
- `oldVnode`：上一个虚拟节点

### 动态指令参数

指令的参数可以是动态的。例如：在`vue-mydirective:[argument]="value"`中，`argument`参数可以根据组件实例数据进行更新。

## 渲染函数&JSX

### 基础内容

通过jsx可以更加容易创建出需要的组件模板：

```javascript
export default {
  props: {
    level: {
      type: Number,
      required: true
    }
  },
  render(h) {
    return h(
      'h' + this.level, // 标签名
      this.$slots.default // 子节点数组
    )
  }
}
```

向组件中传递不带`v-slot`指令的子节点时，这些节点就存在组件实例的`$slots.default`中

### 虚拟DOM

Vue通过建立一个虚拟DOM来追踪自己要如何改变真实DOM

```javascript
createElement('h1', this.blogTitle);
```

`createElement`其实返回的其实不是一个实际的DOM元素，其名字实际应该是叫做`createNodeDescription`，包含的信息会告诉Vue页面渲染什么样的节点，包括其子节点的描述信息

#### createElement参数

```javascript
// @return {VNode}
createElement(
  // {String|Object|Function}
  // 一个HTML标签名、组件选项对象
  'div',
  // {Object}
  // 一个与模板中attribute对应的数据对象，可选
  {},
  // {String|Array}
  // 子级虚拟节点
  [
    '文字文字文字',
    createElement('h1', '一则头条'),
    createElement(myComponent, {
      props: {
        someProp: 'foobar'
      }
    })
  ]
)
```

#### 数据对象

```javascript
{
  // 接受一个字符串、对象或字符串和对象组成的数组
  'class': {
    foo: true
  },
  // 接受一个字符串、对象或对象组成的数组
  style: {
    color: 'red'
  },
  // 普通HTML attribute
  attrs: {
    id: 'foo'
  },
  // 组件prop
  props: {
    myProp: 'bar'
  },
  // DOM property
  domProps: {
    innerHTML: 'baz'
  },
  // 事件监听器在“on“内
  // 不再支持如`v-on:keyup.enter`这样的修饰器
  on: {
    click: this.clickHandler
  },
  // 仅用于组件，用于监听原生事件，而不是组件内部使用
  nativeOn: {
    click: this.nativeClickHandler
  },
  // 自定义指令
  directives: [
    {
      name: 'my-custom-directive',
      value: '2',
      expression: '1 + 1',
      arg: 'foo',
      modifiers:{
        bar: true
      }
    }
  ],
  // 作用域插槽格式为
  // { name: props => VNode | Array<VNode> }
  scopedSlots: {
    default: props => createElement('span', props.text)
  },
  // 如果组件是其他组件的子组件，需为插槽指定名称
  slot: 'name-of-slot',
  key: 'myKey',
  ref: 'myRef',
  // 如果渲染函数中给多个元素都应用了相同的ref名
  // 那么`$refs.myRef`会变成一个数组
  refInFor: true
}
```

#### 约束

**VNode必须是唯一的**

组件树种的VNode必须是唯一的。以下渲染函数是不合法的：

```javascript
export default {
  render(h) {
    var myParagraphVNode = h('p', 'hi')
    return h('div', [
      myParagraphVNode, myParagraphVNode
    ])
  }
}
```

#### v-model

渲染函数中没有对应的`v-model`，自己实现的逻辑如下：

```javascript
export default {
  props: ['value'],
  render(h) {
    var self = this
    return h('input', {
      domProps: {
        value: self.value
      },
      on: {
        input(event) {
          self.$emit('input', event.target.value)
        }
      }
    })
  }
}
```

#### 事件&按键修饰符

对于诸如`.passive`、`.capture`和`.once`这些修饰符，Vue都提供了相应的前缀可用于`on`：

| 修饰符                           | 前缀 |
| -------------------------------- | ---- |
| `.passive`                       | `&`  |
| `.capture`                       | `!`  |
| `.once`                          | `~`  |
| `.capture.once`或`.once.capture` | `~!` |

实际应用如下：

```javascript
{
  on: {
    '!click': this.clickHandler
  }
}
```

而其他修饰都不是必须的，可以自己实现

#### 插槽

可以通过`this.$slots`访问静态插槽内容，每一个插槽都是一个VNode数组

#### 函数式组件

对于没有任何管理状态，也不用监听传递给它的状态，也没有生命周期的组件，且只是一个接受一些prop的函数，这种情况可以将该组件标记为`functional`，意味着这是无状态，没有实例的

```javascript
export default {
  functional: true,
  props: {},
  render(h, context) {}
}
```

组件需要的一切都是通过`context`参数传递的，是一个包含以下字段的对象：

- `props`：提供所有prop的对象
- `children`：VNode子节点的数组
- `slots`：一个函数，返回了包含所有插槽的对象
- `scopedSlots`：一个暴露传入的作用域插槽的对象
- `data`：传递给组件的整个数据对象
- `parent`：对父组件的引用
- `listeners`：一个包含了所有父组件为当前组件注册的事件监听器的对象
- `injections`：如果使用了`inject`选项，则该对象包含了应当被注入的property

**函数式组件只是函数，所以渲染开销低很多**

## 插件

### 功能

- 添加全局方法或者property
- 添加全局资源：指令/过滤器/过渡等
- 通过全局混入来添加一些组件选项
- 添加Vue实例方法

### 如何使用

通过全局方法`Vue.use()`使用插件，需要在`new Vue()`启动应用之前完成

```javascript
Vue.use(MyPlugin, { someOption: true })
```

`Vue.use`会自动阻止多次注册相同插件，即使多次调用也只会注册一次该插件

### 开发插件

Vue.js的插件应该暴露一个`install`方法，第一个参数是`Vue`构造器，第二个参数是一个可选的选项对象

```javascript
MyPlugin.install = function (Vue, options) {
  // 1. 添加全局方法或 property
  Vue.myGlobalMethod = function () {
    // 逻辑...
  }

  // 2. 添加全局资源
  Vue.directive('my-directive', {
    bind (el, binding, vnode, oldVnode) {
      // 逻辑...
    }
    ...
  })

  // 3. 注入组件选项
  Vue.mixin({
    created: function () {
      // 逻辑...
    }
    ...
  })

  // 4. 添加实例方法
  Vue.prototype.$myMethod = function (methodOptions) {
    // 逻辑...
  }
}
```

## 过滤器

Vue.js允许自定义过滤器，可用于一些常用文本格式化。可用在两个地方：**双花括号**和**v-bind表达式**

```vue
{{ message | capitlize }}

<div v-bind:id="rawId | formatId"></div>
```

### 定义方式

**局部**

```javascript
export default {
  filters: {
    capitalize(value) {
      return 'something'
    }
  }
}
```

**全局定义**

```javascript
Vue.filter('capitalize', function(value){
  return 'something'
})
```

### 过滤器可多个调用

```vue
{{ message | filterA |filterB }}
```
