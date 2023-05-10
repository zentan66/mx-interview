## 面试题

1. 用过koa吗？简要阐述一下koa的洋葱模型。









1. 环境变量引入

   ```dotenv```

2. 控制器抽离

3. 服务层抽离



## 注册登录流程

### 密码加密

通过`bcryptjs`实现密码的加密，处理过程在路由的中间件中

### 颁发token





## API文档

### 1)用户模块

#### 注册

```
POST /register
```

> 请求参数

```
username, phone, password
```

> 响应参数

```json
{
  "code": 0,
  "message": "注册成功",
  "result": {}
}
```

