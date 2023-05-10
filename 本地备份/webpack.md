## 概览

### webpack的作用

- 模块打包
- 编译兼容
- 能力拓展

### 模块打包运行原理

webpack的整个打包流程：

1. 读取webpack的配置参数
2. 启动webpack，创建`Compiler`对象并开始解析项目
3. 从入口文件开始解析，并且找到其导入的依赖模块，递归遍历分析，形成依赖关系树
4. 对不同文件类型的依赖模块文件使用对应的Loader进行编译，最终转为JavaScript文件
5. 整个过程中webpack会通过发布订阅模式，向外抛出一些hooks，而webpack的插件即可通过监听这些关键的事件节点，执行插件任务进而打到干预输出结果的目的

其中文件的解析与构建是一个比较复杂的过程，在webpack源码中主要依赖于`compiler`和`compilation`两个核心对象实现

`compiler`对象是一个全局单例，他负责把控整个`webpack`打包的构建流程。 `compilation`对象是每一次构建的上下文对象，它包含了当次构建所需要的所有信息，每次热更新和重新构建，`compiler`都会重新生成一个新的`compilation`对象，负责此次更新的构建过程。

而每个模块间的依赖关系，则依赖于`AST`语法树。每个模块文件在通过`Loader`解析完成之后，会通过`acorn`库生成模块代码的`AST`语法树，通过语法树就可以分析这个模块是否还有依赖的模块，进而继续循环执行下一个模块的编译解析。

### sourceMap

sourceMap是一项将编译、打包、压缩后的代码映射回源代码的技术。由于打包压缩后的代码并没有阅读性可言，一旦在开发中报错或者遇到问题，直接在混淆代码中debug问题会带来非常糟糕的体验，sourceMap可以帮助我们快速定位到源代码的位置，提高我们的开发效率



## 优化策略

### 缩小文件的搜索范围

1. 优化Loader配置
   - 优化正则表达式
   - 通过cacheDirectory选项开启缓存
   - 通过include、exclude来减少被处理的文件
2. 优化resolve.modules配置
3. 优化resolve.alias配置
4. 优化resolve.extensions配置
   - 后缀尝试列表尽可能少，不要将项目中不可能存在的情况写到后缀尝试列表中
   - 频率出现最高的文件后缀要优先放在最前面，以做到尽快退出寻找过程
   - 在源码中导入语句时，要尽可能带上后缀，从而避免寻找过程
5. 优化resolve.noParse配置

### 减少冗余代码

### 使用HappyPack多进程解析和处理文件

### tree-shaking

tree-shaking的实现就是先标记出模块导出值中哪些没有用过，然后使用Terser删掉这些没被用到的导出语句，标记过程大致可划分为三个步骤：

- Make 阶段，收集模块导出变量并记录到模块依赖关系图 ModuleGraph 变量中
- Seal 阶段，遍历 ModuleGraph 标记模块导出变量有没有被使用
- 生成产物时，若变量没有被其它模块使用则删除对应的导出语句

## HMR

HMR就是模块热更新

应用开发的模块热更新流程如下：

1. 在webpack的watch模式下，文件系统中某一个文件发生修改，webpack监听到文件变化，根据配置文件对模块重新编译打包，并将打包后的代码通过简单的JavaScript对象保存在内存中
2. webpack-dev-server和webpack之间的接口交互。在这一步，主要是dev-server的中间件webpack-dev-middleware和webpack之间的交互，webpack-dev-middleware调用webpack暴露的API对代码变化进行监控，并且告知webpack，将代码打包到内存中
3. webpack-dev-middleware 对文件变化的一个监控，这一步不同于第一步，并不是监控代码变化重新打包。当我们在配置文件中配置了devServer.watchContentBase为true的时候，Server会监听这些配置文件夹中静态文件的变化，变化后会通知浏览器端对应用进行live reload，这里是浏览器刷新
4. 这步骤主要是通过sockjs在浏览器端和服务端之间简历一个websocket长连接，将webpack编译打包的各个阶段的状态信息告知浏览器端，同时也包括第三步中server监听静态文件变化的信息。浏览器端根据这些socket消息进行不同的操作，当然服务器端传递的最主要信息还是新模块的hash值，后端的步骤根据这一hash值进行模块热替换
5. webpack-dev-server/client端并不能请求更新的代码，也不会执行热更模块操作，而把这些工作又交回给了webpack，webpack/hot/dev-server的工作就是根据webpack-dev-server/client传给它的信息以及dev-server的配置决定是刷新浏览器呢还是进行模块热更新
6. HotModuleReplacement.runtime是客户端HMR的中枢，接收到上一步传递给他的新模块的hash值，通过JsonpMainTemplate.runtime向server发送ajax请求，服务端返回一个json，该json包含了所有要更新的模块的hash值，获取到更新列表后，该模块再次通过jsonp请求，获取到最新的模块代码
7. HotModulePlugin将会对新旧模块进行对比，决定是否更新模块，在决定更新模块后，检查模块之间的依赖关系，更新模块的同时更新模块间的依赖引用
8. 当HMR失败后，回退到live reload操作，也就是进行浏览器刷新来获取最新打包代码

## QAQ

### 有哪些提高开发效率的插件

- Webpack-dashboard：更加友好的展示相关打包信息
- Webpack-merge：提取公共配置，减少重复配置代码
- Speed-measure-webpack-plugin：分析出webpack打包过程中loader和plugin的耗时，有助于找到构建过程中的性能瓶颈
- Size-plugin：监控资源体积变化，尽早发现问题
- HotModuleReplacementPlugin：模块热替换
- progress-bar-webpack-plugin：查看构建进度
- webpack-build-notifier：打包完成提示
- [webpack-dashboard](https://link.juejin.cn/?target=)：优化构建UI

### 优化webpack的构建速度

- 使用高版本的webpack和nodejs

- 多进程/多实例构建：HappyPack、thread-loader

- 压缩代码

  - 多进程并行压缩
    - Webpack-paralle-uglify-plugin
    - Uglify-webpack-plugin开启parallel
    -  terser-webpack-plugin开启parallel参数
  - 通过mini-css-extract-plugin提取chunk中的css代码到单独文件，通过css-loader的minimize选项开启cssnano压缩css

- 图片压缩

  - 使用基于node库的imagemin
  - 配置image-webpack-loader

- 缩小打包作用域

  - exclude/include（确定loader规则范围）
  - resolve.modules指明第三方模块的绝对路径
  - resolve.mainFields只采用main作为入口文件描述字段
  - resolve.extensions尽可能减少后缀尝试的可能性
  - noParse对完全不需要解析的库进行忽略
  - IgnorePlugin（完全排除模块）
  - 合理使用alias

- 提取页面公共资源

  - 基础包分离

    - 使用html-webpack-externals-plugin，将基础包通过CDN引入，不打包到bundle中

    - 使用SplitChunksPlugn进行（公共脚本、基础包、页面公共文件）分离

- DLL

    - 使用DllPlugin进行分包，使用DllReferencePlugin对manifest.json引用，让一些基本不会改动的代码先打包成静态资源，避免反复编译浪费时间
    - HashedModuledIdsPlugin可以解决模块数字id问题

- 充分利用缓存提升二次构建速度
  - babel-loader开启缓存
  - terser-webpack-plugin开启缓存
  - 使用cache-loader或者hard-source-webpack-plugin

- Tree shaking
  - 打包过程中检测工程中没有引用的模块并进行标记，在资源压缩时将它们从最终的bundle中去掉，开发中尽可能使用ES6 Module的模块，提高tree shaking效率
  - 禁用babel-loader的模块依赖解析，否则webpack接收到的就都是转换过的CommonJS形式的模块，无法进行tree-shaking
  - 使用PurfyCSS或者uncss取出无用CSS代码
- Scope hoisting
  - 构建后的代码存在大量闭包，造成体积增大，运行代码时创建的函数作用域变多，然后适当的重命名一些变量以防变量名冲突
  - 必须是ES6语法
- 动态polyfill

