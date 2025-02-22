### 定义

新一代的前端构建工具，主要利用浏览器`ESM`特性导入组织代码，在服务端按需编译返回，完全跳过打包这个过程

特点：

- 快速冷启动
- 即时的模块热更新
- 真正的按需加载

### 热更新

热更新过程如下：

1. 创建一个`websocket`服务器和`client`文件，启动服务
2. 通过`chokidar`监听文件变更
3. 当代码变更后，服务端进行判断并推送到客户端
4. 客户端根据推送的信息执行不同操作的更新



#### 优化：浏览器的缓存策略提高响应速度

利用`HTTP`加速整个页面的重新加载

设置响应头使得依赖模块(`dependency module`)进行强缓存，而源码文件通过设置 `304 Not Modified` 而变成可依据条件而进行更新。





### 基于esbuild的依赖预编译优化

1. 为什么需要预构建
   - 支持`commonJS`依赖
   - 减少模块和请求数量
2. 为什么使用esbuild
   - 多线程
   - 编译运行