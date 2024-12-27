### 关联表模型

在models下的文件中，例如课程和分类、用户有关联

```javascript
class Course extends Model {
  static associate(models) {
    // 表示课程属于某个分类，某个用户
    models.Course.belongsTo(models.Category, { as: 'category' });
    models.Course.belongsTo(models.User, { as: 'user' });
  }
}
```

以上代码就是代表将课程关联上用户和分类表

#### 实际应用

```javascript
async function getCourse(req) {
  const { id } = req.params
  const condition = {
    // 返回的数据中去除以下两个属性值
    attributes: { exclude: ['CategoryId', 'UserId']},
    include: [
      // as 表示别名
      // attributes 表示返回的数据中只需要包含以下键
      { model: Category, as: 'category', attributes: ['id', 'name'] },
      { model: User, as: 'user', attributes: ['id', 'username', 'avatar'] }
    ]
  }
  const course = await Course.findByPk(id, condition)
  if (!course) {
    throw new NotFoundError(`ID: ${id}的课程未找到。`)
  }
  return course
}
```

#### 设置分类

```javascript
class Category extends Model {
  static associate(models) {
    // 表示分类下有多个课程
    models.Category.hasMany(models.Course, { as: 'courses' });
  }
}
```

