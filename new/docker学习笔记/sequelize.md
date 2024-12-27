## 作用

简化sql操作，不用自己写sql语句

**官网地址**: sequelize.org

### 命令行安装

```bash
npm i -g sequelize-cli
```

### 生成配置文件

```bash
sequelize init
```

自动在项目中生成sequelize的配置文件

### 建立模型和迁移文件

```bash
sequelize model:generate --name Article --attributes title:string,content:text
```

这个语句生成的内容会出现在项目中，生成以下文件夹：

- `config`：配置文件，数据库不同环境的参数数据，用户名密码，数据库名称，时区等

- `migrations`：迁移文件，用于创建真实数据库
- `models`：模型文件，数据库对应的模型
- `seeders`：执行语句

### 运行迁移（在mysql中建立数据库）

```bash
sequelize db:migrate
```

这个语句的作用是创建真实数据库



### 新建种子（创建执行文件）

```bash
sequelize seed:generate --name article
```

执行该语句，会在项目的`seeders`文件夹下生成一个文件，可以执行插入删除等语句

### 运行种子（执行文件，操作数据库数据）

在seeders下的文件中写好代码后，执行该命令：

```sh
sequelize db:seed --seed xxx-article # xxx-article是seeders文件夹下的文件名称
```

#### 回滚迁移（删除数据表）

```sh
sequelize db:migrate:undo
```

