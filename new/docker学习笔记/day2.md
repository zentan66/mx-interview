### 查询数据

在routes下创建一个文件`routes/admin/articles.js`

```javascript
const { Article } = require('./model')

const condition = { order: [['id', 'DESC']] }

// 用于倒序查询所有数据
await Article.findAll(condition)
```

#### 模糊查询

需要引入一个 Op 的东西

```javascript
const { Op } = require('sequelize')

const condition = { order: [['id', 'DESC']] }

if (query.title) {
  condition.where = {
    title: {
      [Op.like]: `%${query.title}%`
    }
  }
}

await Article.findAll(condition)
```

#### 分页查询

```javascript
const { Op } = require('sequelize')

const currentPage = Math.abs(Number(req.query.currentPage)) || 1
const pageSize = Math.abs(Number(req.query.pageSize))
const offset = (currentPage - 1) * pageSize

const condition = { order: [['id', 'DESC']], limit: currentPage, offset }

const { count, rows } = await Article.findAndCountAll(condition)
res.json({
  status: true,
  message: '',
  data: {
    articles: rows,
    pagination: {
      total: count,
      currentPage,
      pageSize
    }
  }
})
```



