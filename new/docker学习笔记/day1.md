## 环境配置

### 准备工具

下载docker后，在docker engine中添加以下内容，设置镜像

```json
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "registry-mirrors": [
    "https://xelrug2w.mirror.aliyuncs.com"
  ]
}
```

### 项目中配置

新建一个`docker-compose.yml`文件，内容如下：

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8.3.0
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_general_ci
    environment:
      - MYSQL_ROOT_PASSWORD=clwy1234
      - MYSQL_LOWER_CASE_TABLE_NAMES=0
    ports:
      - "3306:3306"
    volumes:
      - ./data/mysql:/var/lib/mysql
```

在项目中打开控制台，输入以下内容

```bash
$ docker-compose up -d
```

会在 docker 中新建出一个数据库，名称就是项目名称

<img src="/Users/noahcengtan/Library/Application Support/typora-user-images/image-20241114121052795.png" alt="image-20241114121052795" style="zoom:50%;" />



### 管理应用

macbook可以在应用商店中下载一个 `Sequel Ace`，用于管理数据库

## 新建数据库

### 常用数据类型

| 类型                 | 含义   | 说明                                                   |
| -------------------- | ------ | ------------------------------------------------------ |
| int                  | 整数   | 需要设定长度                                           |
| decimal              | 小数   | 金额常用，需要设定长度 decimal(10,2),总数10位，小数2位 |
| char、varchar        | 字符串 | 文字类的常用，需要设定长度。例如身份证号               |
| text                 | 文本   | 存储大文本，比如文章的正文                             |
| date、time、datetime | 日期   | 记录时间                                               |
|                      |        |                                                        |

