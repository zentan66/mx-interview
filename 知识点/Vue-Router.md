## 目录

- [基础](##基础)
  - [动态路由匹配](###动态路由匹配)
  - [嵌套路由](###嵌套路由)
- [编程式导航](##编程式导航)
- [命名路由](##命名路由)
- [命名视图](##命名视图)
- [重定向和别名](##重定向和别名)
  - [重定向](###重定向)
  - [别名](###别名)
- [路由组件传参](##路由组件传参)
  - [布尔模式](###布尔模式)
  - [对象模式](###对象模式)
  - [函数模式](###函数模式)
- [HTML5 History模式](##HTML5 History模式)
- [导航守卫](##导航守卫)
  - [全局前置守卫](##全局前置守卫)
  - [全局解析守卫](##全局解析守卫)
  - [全局后置守卫](##全局后置守卫)
  - [路由独享守卫](##路由独享守卫)
  - [组件内守卫](##组件内守卫)
  - [完整的导航解析流程](##完整的导航解析流程)
- [路由元信息](##路由元信息)
- [过渡效果](##过渡效果)
  - [单个路由过渡](##单个路由过渡)
  - [基于路由的动态过渡](##基于路由的动态过渡)
- [滚动行为](##滚动行为)

## 基础

```javascript
import Vue from 'vue'
import VueRouter from 'vue-router'

const Foo = { template: '<div>Foo</div>'}
const Bar = { template: '<div>Bar</div>'}

const routes = [
  { path: '/foo', component: Foo },
  { path: '/bar', component: Bar }
]
const router = new VueRouter({
  routes
})
const app = new Vue({
  router
}).$mount('#app')
```

通过在Vue实例中注入路由器，可以在任何组件内通过`this.$router`访问路由器，也可以通过`this.$route`访问当前路由

### 动态路由匹配

```javascript
const User = { template: '<div>User</div>' }
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User }
  ]
})
```

一个“路径参数”使用冒号`:`标记，当匹配到一个路由时，参数值会被设置到`this.$route.params`，可以在每一个组件内使用

```javascript
const User = {
  template: '<div>User {{ this.$route.params.id }}</div>'
}
```

也可以在一个路由中设置多段“路径参数”，对应的值都会设置到`$route.params`中

```javascript
{ path: '/user/:username/post/:post_id' } // /user/evan/post/123
```

#### 响应路由参数变化

当使用路由参数时，例如从`/user/foo`导航到`/user/bar`，原来的组件实例会被复用，因为两个路由都渲染同个组件，比起销毁再创建，复用则显得更加高效。**这意味着组件的生命周期钩子不会再被调用**

复用组件时，相对路由参数的变化作出响应，又两种方法：

1. 通过wath（监测变化）`$route`对象：

   ```javascript
   export default {
     watch: {
       $route(to, from) {}
     }
   }
   ```

   

2. 使用2.2引入的`beforeRouteUpdate`导航守卫

   ```javascript
   export default {
     beforeRouteUpdate(to, from, next) {}
   }
   ```

#### 捕获所有路由或404路由

如果想匹配任意路径，可以使用通配符(`*`)

```javascript
{
  path: '*'
}
```

使用通配符路由时，需要注意路由的顺序是正确的

### 嵌套路由

路由配置文件：

```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/user/:id',
      component: User,
      children: [
        {
          path: 'profile',
          component: UserProfile
        },
        {
          path: 'posts',
          component: UserPosts
        }
      ]
    }
  ]
})
```

User组件内容：

```vue
<template>
	<div>
    <p>User</p>
    <router-view></router-view>
  </div>
</template>
```

**以`/`开头的嵌套路径会被当作根路径**

如果`children`内部的组件path路径是使用`/`开头的，那么该路由的路径就是相对于根路径的，例如父组件是User，子组件Profile的path是`profile`，那么该子组件的访问路径就应该是`/user/profile`，如果子组件Profile的path是`/profile`，那么访问路径是`/profile`

## 编程式导航

| 声明式                            | 编程式                |
| --------------------------------- | --------------------- |
| `<router-link :to="...">`         | `router.push(...)`    |
| `<router-link :to="..." replace>` | `router.replace(...)` |

## 命名路由

通过一个名称来标识一个路由显得更方便些

```javascript
const routes = [
  { path: '/user/:userId', name: 'user', component: User }
]
```

要链接到一个命名路由，可以使用下列两种方式：

```javascript
// 声明式
const Link = {
  template: '<router-link :to="{}" />'
}

// 编程式
router.push({ name: 'user', params: { userId: 123 }})
```

## 命名视图

有时候想同级展示多个视图，这个时候就需要使用命名视图了

```vue
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```

而路由配置文件如下：

```javascript
new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }
    }
  ]
})
```

## 重定向和别名

### 重定向

```javascript
// 基础调用
{ path: '/a', redirect: '/b' }

// 重定向到命名路由
{ path: '/a', redirect: { name: 'foo' }}

// 动态返回重定向目标
{ path: '/a', redirect: to => {
  // 接收目标路由作为参数
  // return 重定向的字符串路径/路径对象
}}
```

重定向的时候，原路由上的导航守卫没有任何效果，目标路由上的才有用

### 别名

设置路由的别名，当访问别名的路径时，实际访问的却是另外个路由

```javascript
{ path: '/a', component: A, alias: '/b' }
```

上述例子中，`/a`的别名是`/b`，意味着用户访问`/b`时，URL会保持`/b`，但是路由匹配的却是`/a`

## 路由组件传参

在组件中获取路由的参数会使用`$route`，这使得路由与之形成了高度耦合，从而使组件只能在特定的URL上使用，可以使用`props`将路由与组件解耦：

```javascript
const User = {
  props: ['id'],
  template: '<div>User {{ id }}</div>'
}
new VueRouter({
  routes: [
    { path: '/user/:id', component: User, props: true },
    // 对于包含命名视图的路由 需要在每个视图下设置props选项
    {
      path: '/user/:id',
      components: {
        default: User,
        sidebar: Sidebar
      },
      props: {
        default: true,
        sidebar: true
      }
    }
  ]
})
```

### 布尔模式

如果`props`设置为`true`，则`route.params`将会被设置为组件属性

### 对象模式

如果`props`是一个对象，那么将会按原样设置为组件属性

```javascript
{ path: '/a', component: A, props: { newsletterPopup: false }}
```

上述例子中的`newsletterPopup`就会被设置为组件属性

### 函数模式

可以创建一个函数返回一个`props`，能将静态值与基于路由的值结合

```javascript
{ path: '/search', component: Search, props: (route) => ({ query: route.query.q })}
```

**应该尽可能保持`props`函数为无状态的，只在路由发生变化时起作用**

## HTML5 History模式

`vue-router`默认是hash模式——使用URL的hash来模拟一个完整的URL

也可以使用路由的history模式

```javascript
new VueRouter({
  mode: 'history',
  routes: [...]
})
```

**history模式需要后台配置，如果后台没有正确的配置，用户访问就会返回404**

**服务端只需要设置URL匹配不到任何静态资源的时候，返回同一个`index.html`页面就可以了**

## 导航守卫

### 全局前置守卫

```javascript
const router = new VueRouter({})

router.beforeEach((to, from, next) => {})
```

守卫是异步解析执行，此时导航在所有守卫resolve完之前一直处于等待中

每个守卫方法接收三个参数：

- `to <Route>`：即将要进入的目标（路由对象）
- `from <Route>`：当前导航正要离开的路由
- `next <Function>`：
  - `next()`：进入管道中的下一个钩子
  - `next(false)`：中断当前导航
  - `next('/')`或者`next({ path: '/' })`：跳转到一个不同的地址
  - `next(error)`：如果参数是一个`Error`实例，则导航会被终止且该错误会被传递给`route.onError()`注册过的回调

### 全局解析守卫

在2.5.0+可以使用`router.beforeResolve`注册一个全局守卫，在导航被确认之前，同时在所有组件内守卫和异步路由组件被解析之后，解析守卫就被调用

### 全局后置守卫

```javascript
router.afterEach((to, from) => {})
```

### 路由独享守卫

```javascript
{ path: '/foo', component: Foo, beforeEnter: (to, from, next) => {}}
```

### 组件内守卫

```javascript
const Foo = {
  template: `...`,
  beforeRouteEnter(to, from, next) {
    // 在渲染该组件的对应路由被comfirm前调用
    // 不能获取组件实例
  },
  beforeRouteUpdate(to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
  },
  beforeRouteLeave(to, from, next) {}
}
```

`beforeRouteEnter`守卫不能访问`this`，因为这个时候新组件还未被创建

不过可以通过`next`传一个回调来访问组件实例：

```javascript
beforeRouteEnter(to, from, next) {
  next(vm => {
    // 通过`vm`访问组件实例
  })
}
```

### 完整的导航解析流程

1. 导航触发
2. 失活的组件内调用`beforeRouteLeave`
3. 调用全局`beforeEach`
4. 重用的组件内调用`beforeRouteUpdate`
5. 路由配置里调用`beforeEnter`
6. 解析异步路由组件
7. 被激活的组件内调用`beforeRouteEnter`
8. 调用全局`beforeResolve`
9. 导航被确认
10. 调用全局`afterEach`
11. 触发DOM更新
12. 调用创建好的实例中`beforeRouteEnter`守卫传的`next`回调函数

## 路由元信息

在定义路由的时候可以配置`meta`字段

```javascript
new VueRouter({
  routes: [
    { path: '/foo', component: Foo, meta: { auth: true }}
  ]
})
```

当一个路由匹配成功后，可能会匹配到多个路由记录（`route`配置中的每个路由对象）

一个路由匹配到的所有路由记录会暴露为`$route`对象的`$route.matched`数组

```javascript
router.beforeEach((to, from, next) => {
  if (to.matched.some(record => record.meta.auth)) {
    if (!auth.loggedIn()) {
      next({
        path: '/login',
        query: { redirect: to.fullPath }
      })
    } else {
      next()
    }
  } else {
    next()
  }
})
```

## 过渡效果

因为`router-view`是基本的动态组件，可以通过`<transition>`组件给它添加一些过渡效果：

```vue
<transition>
	<router-view></router-view>
</transition>
```

### 单个路由过渡

```javascript
const Foo = {
  template: `
    <transition name="slide">
      <div class="foo">...</div>
    </transition>
  `
}
const Bar = {
  template: `
    <transition name="fade">
      <div class="bar">...</div>
    </transition>
  `
}
```

### 基于路由的动态过渡

```vue
<template>
  <transition :name="transitionName">
    <router-view />
  </transition>
</template>
<script>
export default {
  watch: {
    '$route' (to, from) {
      const toDepth = to.path.split('/').length
      const fromDepth = from.path.split('/').length
      this.transition = toDepth < fromDepth ? 'slide-right' : 'slide-left'
    }
  }
}
</script>
```

## 滚动行为

在创建一个Router实例时，可以提供一个`scrollBehavior`方法：

```javascript
new VueRouter({
  routes: [...],
  scrollBehavior(to, from, savedPosition){
    // return 期望滚动到哪个位置
  }
})
```

## API

### \<router-link\>

`<router-link`可以通过作用域插槽暴露底层的定制能力，在使用`v-slot`API时，需要向`router-link`传入一个单独的子元素

```vue
<router-link to="/about" v-slot="{ href, route, navigate, isActive, isExactActive }">
	<li :class="[isActive && 'router-link-active', isExactActive && 'router-link-exact-active']"></li>
</router-link>
```

