### Ref

ref的get和set并不是通过proxy拦截的

对象类型会触发 `toReactive` 方法，然后调用 `reactive(value)`

```javascript
class RefImpl {
  private _value
  private _rawValue
  
  constructor(value, __v_isShallow) {
    this._rawValue = __v_isShallow ? value : toRaw
    this._value = __v_isShallow ? value : toReactive(value)
  }
  
  get value () {
    trackRefValue(this)
    return this._value
  }
  
  set value(newVal) {
    newVal = this.__v_isShallow ? newVal : toRaw(newVal)
    if (hasChanged(newVal, this._rawValue)) {
      this._rawValue = newValue
      this._value = this.__v_isShallow ? newVal : toReactive(newVal)
      triggerRefValue(this, newVal)
    }
  }
}
```



### Render



```javascript
const render = (vnode, container, isSVG) => {
    // vnode 不存在
    if (vnode == null) {
      // 存在旧节点
      if (container._vnode) {
        // 卸载旧节点
        unmount(container._vnode, null, null, true)
      }
    } else {
      // 更新节点
      patch(container._vnode || null, vnode, container, null, null, null, isSVG)
    }
    flushPostFlushCbs()
    // _vnode 赋值旧节点
    container._vnode = vnode
}
```

再看看`patch`方法

```javascript
const patch = (
	n1, // 旧节点
  n2, // 新节点
  container, // 容器
  anchor = null, // 锚点
  parentComponent = null,
  parentSuspense = null,
  isSVG = false,
  slotScopeIds = null,
  optimized
) => {}
```



#### 组件render





### reactive

1. 执行 createReactiveObject 方法
2. createReactiveObject 方法主要创建一个 proxy 实例对象
3. get 方法实际执行了 createGetter 方法，该方法中 `track` 函数进行依赖收集，而 set 方法实际执行了 createSetter 方法，该方法中 `trigger` 进行依赖触发



### ref

1. 执行 `createRef` 方法
2. createRef 实际是创建了一个 `RefImpl`实例
3. 实例的 `get value` 会触发 `trackRefValue` 进行依赖收集，将`effect`添加到实例的`dep` 中
4. 实例的 `set value` 会将 value 用`toReactive`进行包裹，然后`triggerRefValue`触发更新，遍历`dep`，挨个调用`effect.schuduler`或者`effect.run`
5. 如果用来创建的初始值是对象，就会先执行 `reactive(value)` ，否则直接返回初始值



### computed

1. 创建一个`ComputedRefImpl`实例
2. 构造函数会创建一个`ReactiveEffect`实例，赋值给实例的`effect`
3. 实例有一个`getter value`，获取是会触发`trackRefValue`进行依赖收集
4. 修改`setter value` 会调用传入的 set 方法



### watch

