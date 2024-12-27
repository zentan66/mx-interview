## vue2

执行流程：

1. watcher.update => patch(oldVnode, newVnode)
2. 在`patch`中判断 oldVnode 是真实节点并且是同一种类型节点后，调用`patchVnode`
3. `patchVnode`：判断新旧节点是否有子节点，均有子节点进入`updateChildren`
4. `updateChildren`：对比过程 => 旧首、新首 => 旧尾、新尾 => 旧首、新尾 => 旧尾、新首 => key值

## vue3

### diff步骤

1. 自前向后
2. 自后向前
3. 新节点多于旧节点
4. 旧节点多于新节点
5. **乱序对比（diff核心）**