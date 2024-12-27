1. WeakSet 的成员只能是对象和 Symbol 值，而不能是其他类型的值。
2. WeakSet 中的对象都是弱引用，即垃圾回收机制不考虑 WeakSet 对该对象的引用
3. ES6 规定 WeakSet 不可遍历
4. `WeakMap`只接受对象（`null`除外）和 [Symbol 值](https://github.com/tc39/proposal-symbols-as-weakmap-keys)作为键名，不接受其他类型的值作为键名。
5. `WeakMap`的键名所指向的对象，不计入垃圾回收机制