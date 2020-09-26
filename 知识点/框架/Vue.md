



### 1.请说一下Vue响应式数据的原理

`Vue`底层对于响应式数据的核心是`object.defineProperty`，`Vue`在初始化数据时，会给`data`中的属性使用`object.defineProperty`重新定义属性（劫持属性的`getter`和`setter`），当页面使用对应属性时，会通过Dep类进行**依赖收集**（收集当前组件的`watcher`）,如果属性发生变化，会通知相关依赖调用其update方法进行更新操作

- 总结： `Vue`通过数据劫持配合发布者-订阅者的设计模式，内部通过调用`object.defineProperty()`来劫持各个属性的`getter`和`setter`，在数据变化的时候通知订阅者，并触发相应的回调



### 2.Vue如何实现响应式数据的？

实现一个监听器「Observer」：对数据对象进行遍历，包括子属性对象的属性，利用`Object.defineProperty()`在属性上都加上`getter`和`setter`，这样后，给对象的某个值赋值，就会触发`setter`，那么就能监听到数据变化

实现一个解析器「Compile」：解析`Vue`模板指令，将模板中的变量都替换成数据，然后初始化渲染页面视图，并将每个指令对应的节点绑定更新函数，添加监听数据的订阅者，一旦数据有变动，收到通知，调用更新函数进行数据更新

实现一个订阅者「Watcher」：`Watcher`订阅者是`Observer`和`Compile`之间通信的桥梁，主要任务是订阅`Observer`中的属性值变化的消息，当收到属性值变化的消息时，触发解析器`Compile`中对应的更新函数

实现一个订阅器「Dep」：订阅器采用发布-订阅设计模式，用来收集订阅者`Watcher`，对监听器`Observer`和订阅者`Watcher`进行统一管理



### 3.Vue如何实现对象和数组的监听

由于`Object.defineProperty()`只能对属性进行数据劫持，而不能对整个对象（数组）进行数据劫持，因此Vue框架通过遍历数组和对象，对每一个属性进行劫持



### 4.直接给一个数组项赋值，Vue能检测到变化吗？

由于`JavaScript`的限制，Vue不能检测以下数组的变动：

- 利用索引直接修改一个数组项时
- 修改数组的长度时

### 5.Vue如何检测数组的变化？

核心思想：使用了函数劫持的方式，重写了数组的方法（`push,pop,unshift,shift···`）

`Vue`将`data`中的数组，进行了原型链的重写，指向了自己所定义的数组原型方法，当调用数组的`API`时，可以通知依赖更新，如果数组中包含着引用类型，会对数组中的引用类型再次进行监控



### 6.Vue如何通过`vm.$set`来解决对象新增/删除属性不能响应的问题

由于`JavaScript`的限制，Vue无法检测到对象属性的添加或删除。这是由于Vue会在初始化实例时对属性的`getter`和`setter`进行劫持，所以属性必须在data对象上存在才能让Vue将它们转换为响应式数据。

`Vue`提供了`Vue.set(object, propertyName, value) / vm.$set(object, propertyName, value)`来实现为对象添加响应式属性，其原理如下：

- 如果目标是数组，直接使用数组的`splice`方法触发响应式
- 如果目标是对象，会先判断对象属性是否存在，对象是否是响应式，最终如果要对属性进行响应式处理，则是通过调用`defineReactive()`方法进行响应式处理（`defineReactive()`是Vue对`Object.defineProperty()`的二次封装）