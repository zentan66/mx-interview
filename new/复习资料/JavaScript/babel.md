> Babel是一个强大的js 编译器，主要用于在旧的浏览器环境中将 ECMAScript 2015+ 代码转换为向后兼容版本的 JavaScript 代码

### 模块

**@babel/core**：整个Babel 的核心，负责调度 babel 的各个组件进行代码编译

**@babel/preset-env**：预设的插件集合

1. targets：适配的环境，适配的浏览器版本
2. useBuiltIns：preset-env 如何处理 polyfills
3. corejs：维护的polyfill

**@babel/polyfill**：@babel/preset-env 提供了语法转换规则，一些浏览器缺失的新功能，比如内置的方法和对象就需要 polyfill 来做js的垫片

**babel-loader**：在 webpack 用来通知 @babel/core 去编译 js

**@babel/plugin-transform-runtime**：开发第三方模块或者组件库时，用来挂载目标浏览器缺失的功能

### 执行顺序

- 插件比预设先执行
- 插件执行顺序从前向后
- 预设执行顺序从后向前

### 流程

#### 解析（parse）

将 js 代码转为 AST

#### 转换（transform）



#### 生成（generate）