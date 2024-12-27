## let和const、var

- 变量提升

  var声明的变量存在变量提升，可以在声明前调用（undefined）

  let和const不存在变量提升，在声明前使用会报错

- 暂时性死区

  在变量声明前使用会报错

- 块级作用域

- 重复声明

- 修改声明的变量



## Set、Map和weakSet、weakMap



### WeakMap

与Map的区别

- 没有遍历操作的API
- 没有clear清空方法

**只接受对象作为键名（null除外）**



## Promise

promise解决异步操作的优点：

- 链式操作降低了编码难度
- 代码可读性明显增强

### 状态

- pending
- fulfilled
- rejected

### 特点

- 对象的状态不受外界影响，只有异步操作的结果
- 一旦状态改变，就不会再变