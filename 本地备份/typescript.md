- 泛型是什么



#### #type和interface的区别

1. interface可以重复声明，type不行
2. type使用交叉类型方式实现继承，interface使用extends实现
3. 建议interface描述对象对外暴露的接口，使用type对类型进行重命名

#### # any、unknown、never

any和unknown在ts类型中属于最顶层的top type，所有类型都是这个的子类型，never是所有类型的子类型



