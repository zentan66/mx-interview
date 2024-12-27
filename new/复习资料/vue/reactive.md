### get

1. 调用 `Reflect.get` 获得对应数据
2. 判断不是`readonly`，调用 `track(target, TrackOpTypes.GET, key)`
3. 从 `targetMap.get(target).get(key)`获取一个`Dep`实例对象
   - `targetMap` 是一个 `WeakMap`
   - `targetMap.get(target) === depsMap` 是一个 `Map`
4. 执行 `track` 之后会将 `dep`当做依赖添加到`effect`中





### set

1. 执行`Reflect.set`
2. 继续执行 `trigger(target, TriggerOpTypes.ADD, key, value, oldValue)`
3. 
