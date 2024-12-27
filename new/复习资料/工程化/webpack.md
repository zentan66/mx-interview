### 定义

> webpack是一个静态模块化打包工具，为现代的JavaScript应用程序



### 处理图片

1. 在webpack5之前是通过`raw-laoder`，`url-loader`和`file-loader`实现
2. 在webpack5之后，使用的是**资源模块类型**替换上述这些loader

资源模块类型添加了4中新的模块类型：

- `asset/resource`替换 file-loader
- `asset/inline` 替换 url-loader
- `asset/source` 替换 raw-loader
- `asset` 替换 使用url-loader，并配置资源体积限制的实现



### 常见loader

```
css-loader：解析.css文件
style-loader：将css嵌入到文档中
```



### 过程

1. 初始化参数：从配置文件，配置对象、shell参数中读取
2. 创建编译器对象
3. 初始化编译环境：注入内置插件、注册各种模块工厂，初始化RuleSet集合
4. 开始编译：执行 compiler 对象的 run 方法
5. 确定入口：根据配置中的 entry 找出所有的入口文件，调用 `compilition.addEntry` 将入口文件转换为 `dependence` 对象
6. 编译模块：根据 `entry` 对应的 `dependence` 创建`module`对象，调用`loader` 将模块转译为标准`JS`内容，调用 `JS` 解释器将内容转换为 `AST` 对象，从中找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过本步骤的处理
7. 完成模块编译
8. 输出资源
9. 写入文件系统





## HMR

### 过程

1. webpack 监听文件系统，某一个文件发生修改就对模块重新编译打包，保存在内存中
2. 通过 sockjs 在浏览器端和服务端之间建立一个 socket 长连接，将 webpack 编译打包的各个阶段的状态信息告诉浏览器端，以及服务端传递的新模块的 hash 值
3. 通过配置文件，webpack 决定是刷新浏览器还是进行模块热更新
4. HotModuleReplacement.runtime 是客户端 HMR 的中枢，接收到新模块的 hash 值，通过 JsonpMainTemplate.runtime 向服务端发送 Ajax 请求，服务端返回一个 json（包含了所有要更新的模块的 hash 值），获取到更新列表后，该模块再次通过 jsonp 请求，获取到最新的模块代码
5. HotModulePlugin 将会对新旧模块进行对比，决定是否更新模块，在决定更新模块后，检查模块之间的依赖关系，更新模块的同时更新模块间的依赖引用
6. HMR 失败后，回退到 live reload 操作，进行浏览器刷新



## 优化

### 打包阶段

1. js需要使用babel编译，有些脚本需要跳过编译（打包速度能提升几十到几百毫秒）

   ```js
   // vue.config.js
   function resolve(dir) {
     return path.join(__dirname, dir)
   }
   module.exports = {
     chainWebpack: config => {
       if (process.env.NODE_ENV === 'development') {
         config.module.rule('js').exclude
         	.add(resolve('/node_modules'))
           .add(resolve('./src'))
       }
     }
   }
   ```

2. **dll 优化**：优先打包第三库，后续打包就可以跳过这些包（大概提升几秒钟）

   ```js
   // webpack.dll.js
   // webpack-cli 3.3.12版本
   const path = require('path')
   const webpack = require('webpack')
   
   const dllPath = 'public/vendor'
   
   module.exports = {
     mode: 'production',
     entry: {
       // 需要提取的库文件
       vendor: ['vue', 'vuex', 'vue-router', 'element-ui', 'echarts']
     },
     output: {
       path: path.join(__dirname, dllPath),
       filename: '[name].dll.js',
       // vendor.dll.js 中暴露的全局变量名
       // 保持与 webpack.DllPlugin 中名称一致
       library: '[name]_[hash]'
     },
     plugins: [
       new webpack.DllPlugin({
         path: path.join(__dirname, dllPath, '[name]-manifest.json'),
         // 保持与 output.library 中名称一致
         name: '[name]_[hash]',
         context: process.cwd()
       })
     ]
   }
   
   // vue.config.js
   const path = require('path')
   const webpack = require('webpack')
   function resolve(dir) {
     return path.join(__dirname, dir)
   }
   module.exports = {
     publicPath: process.env.NODE_ENV === 'production' ? './' : '/',
     configureWebpack: {
       plugins: [
         new webpack.DllReferencePlugin({
           context: process.cwd(),
           manifest: require('./public/vendor/vendor-manifest.json')
         })
       ]
     }
   }
   ```

   在配置完成之后还需要在 html 文件中手动引入

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
     <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
     <title>Document</title>
     <link rel="shortcut icon" href="<% BASE_URL %>favicon.ico">
     <script src="<%= BASE_URL %>vendor/vendor.dll.js"></script>
   </head>
   <body>
     <div id="app"></div>
   </body>
   </html>
   ```

3. sourcemap （关闭后能提升100,200毫秒）

4. 压缩代码

5. 利用 CDN 加速

6. 删除死代码（tree shaking）

7. 提取公共代码



### 提高处理效率

1. 多线程
2. 多入口情况下，使用 CommonsChunkPlugin 来提取公共代码
3. 使用 externals 配置来提取公共库
4. 利用 DllPlugin 和 DllReferencePlugin 预编译资源模块
5. 使用 Happypack 实现多线程加速编译
6. 使用 webpack-uglify-parallel 来提升 uglifyPlugin 的压缩速度
7. 使用 Tree-shaking 和 Scope Hoisting 来剔除多余代码



### 减少操作的数量

1. 移除不必要的插件



### webpack5

1. 新增 `type/asset` 类型，用于处理图片文件
2. 更好的 treeshaking
3. 



## FAQs

1. webpack 与grunt、gulp的不同？

   grunt和gulp是基于任务和流的，找到一个（或一类）文件，对其做一系列链式操作，更新流上的数据，整条链式操作构成了一个任务，多个任务就构成了整个web的构建流程

   webpack是基于入口的，会自动地递归解析入口所需要加载的所有资源文件，然后用不同的loader来处理不同的文件，用plugin拓展webpack的功能

   **构建思路**

   gulp和grunt需要开发者将整个前端构建过程拆分为多个Task，并合理控制所有Task的调用关系

   webpack需要开发者找到入口，并需要对于不同的资源应该使用什么loader做何种解析和加工
