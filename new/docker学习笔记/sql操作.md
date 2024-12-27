### 插入数据

插入语句格式：

```sql
INSERT INTO 表名 (列1，...) VALUES (值1，...)

INSERT INTO 表名 (列1，...) VALUES (值1，...),(值2，...),(值3，...),(值4，...)

INSERT INTO Articles (title, content) VALUES ('行路难·其一', '长风破浪会有时，直挂云帆济沧海');
```

### 修改数据

```sql
UPDATE 表名 SET 列1=值1, 列2=值2, ... WHERE 条件

UPDATE Articles SET title='黄鹤楼送孟浩然之广陵', content='故人西辞黄鹤楼，烟花三月下扬州' WHERE id=2
```

### 删除数据

```sql
DELETE FROM 表名 WHERE 条件

DELETE FROM Articles WHERE id=2
```

### 查询条件

```sql
SELECT * FROM 表名 WHERE 条件;

# 排序
SELECT * FROM 表名 WHERE id>2 ORDER BY id DESC; # 倒序

# 模糊查询
SELECT * FROM 表名 WHERE title like '%内容%'

# 限制条数
SELECT * FROM 表名 LIMIT 0,10
```

