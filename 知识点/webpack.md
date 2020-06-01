## webpack

### 原理

#### 流程概述

1. 初始化参数：从配置文件和shell语句中读取与合并参数
2. 开始编译：用上一步得到的参数初始化Compiler对象，加载所有配置的插件，执行对象的run方法开始执行编译
3. 确定入口：根据配置中的entry找出所有入口文件
4. 编译模块：从入口文件出发，调用所有配置的Loader对模块进行翻译，再找出该模块依赖的模块，递归本步骤知道所有入口依赖的文件都经过了本步骤的处理
5. 完成模块编译：经过上一步使用Loader翻译完所有模块后，得到了每个模块被翻译后的最终内容以及之间的依赖关系
6. 输出资源：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的chunk，再把每个chunk转换成一个单独的文件加入输出列表中
7. 输出完成

在以上过程中，webpack会在特定的时间点广播出特定的事件，插件在监听到感兴趣的事件后会执行特定的逻辑，并且插件可以调用webpack提供的api改变webpack的运行结果

### 优化

**定位体积大的模块**

使用`webpack-bundle-analyzer`包展现项目的体积结构

通过`npm install webpack-bundle-analyzer --save-dev`安装，而后在webpack配置信息中的plugins中添加`new BundleAnalyzerPlugin()`即可

**提取公共模块**

通过webpack自带的`CommonsChunkPlugin`可以对项目中多次引用的模块进行提取

在最新版的webpack中，该配置被转移了

```javascript
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'vendor',
      minSize: 30000,
      minChunks: 3,
      maxAsyncRequests: 5,
      maxInitialRequests: 3,
      automaticNameDelimiter: '~',
      name: true,
      cacheGroups: {}
    }
  }
}
```



**移除不必要的文件**

在项目中可能会引入一些体积比较大的库，可以通过webpack自带的两个库实现这个功能：

- IgnorePlugin
- ContextReplacementPlugin

**模块化引入**

在使用`lodash`这类库时，直接引用了整个库的内容，简写可以这样：

```javascript
import chain from 'lodash/chain';
```

**通过CDN引用**

对于一些必要的库，又没办法进行体积优化的，可以通过外部引入的方式来减小打包文件体积

```html
<script src="https://xxx/jquery.min.js"></script>
```

引入方式

```javascript
{
  externals: {
    jquery: 'jquery'
  }
}
```



### 文章推荐

- [【webpack进阶】前端运行时的模块化设计与实现](https://www.jianshu.com/p/b52b6996d612)
- [Webpack打包优化](https://juejin.im/post/5b1e303b6fb9a01e605fd0b3)
- [webpack优化解决项目体积大、打包时间长、刷新时间长问题！](https://juejin.im/post/5ed1c85df265da77190bbab4)

