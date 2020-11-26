



## 基础

1. typeof是怎样判断类型的？更加好的类型判断方式？

   ```txt
   typeof一般用于判断一个变量的类型，可以用来判断number、string、object、boolean、undefined、function、symbol这七种类型。js在底层存储变量的时候会在变量的机器码的低位1-3位存储其类型信息
   000：对象
   010：浮点数
   100：字符串
   110：布尔
   1：整数
   null所有机器码都是0 => 所以typeof null === 'object'是true
   ```

2. 自定义事件的使用和触发

   ```javascript
   const event = new Event('build')
   elem.addEventListener('build', e => {}, false)
   elem.dispatchEvent(event)
   ```

   

## QA

- 为什么js是单线程的，而不是多线程
- 什么是事件委托？有什么好处？
- 实现一个js的持续动画（requestAnimationFrame）
- 同步和异步的区别是什么？