- React源码阅读
- 类组件与函数式组件什么区别
- 组件通信
- Hook的副作用
- react中的优化点（useMemo、useCallback）
- React Portal的理解与使用
- 无状态组件、有状态组件
- Redux工作流
- 普通函数与自定义hook的区别
- 怎么理解jsx
- React 组件是怎么渲染为 DOM 元素到页面上的？
- React 中 `setState` 调用以后会经历哪些流程？
- React Context，说一下它的原理
- `useState` 是怎么实现的？
- 页面数据，表格特别多的时候，是怎么解决的。
- 编辑表格更新是怎么实现的。
- React 的 HOC 是什么？





## 事件机制

- 在jsx上绑定的事件，根本没有注册到真实的dom上，而是绑定在document上统一管理的
- 真实dom上的click事件会被单独处理，已经被react底层替换成空函数
- react并不是一开始把所有事件都绑定在document上的，而是采用了一种按需绑定，比如发现了click事件，再去绑定document click事件

### 事件合成

在react中，我们绑定的事件onClick等，并不是原生事件，而是由原生事件合成的React事件，比如click事件合成为onClick事件。比如blur、input、change等，合成为onChange

**为什么采用这种事件合成的模式？**

- 将事件绑定到document统一管理，防止很多事件直接绑定到原生dom元素上
- React想实现一个全浏览器框架，为了实现这种目标就需要提供全浏览器一致性的事件系统

事件绑定阶段：

- 在React，diff DOM元素类型的fiber的props的时候，如果发现是react合成事件，比如onClick，会按照事件系统逻辑单独处理
- 根据React合成事件类型，找到对应的原生事件的类型，然后调用判断原生事件类型，大部分事件按照冒泡逻辑处理，少数事件会按照捕获逻辑处理
- 调用`addTrappedEventListener`进行真正的事件绑定，绑定在`document`上，`dispatchEvent`为统一的事件处理函数
- 只有几个特殊事件比如`scroll`、`focus`、`blur`等是在事件捕获阶段发生

### 事件触发处理函数

React事件注册的时候，统一的监听器`dispatchEvent`，当我们点击按钮之后，首先执行的是`dispatchEvent`

```javascript
function dispatchEvent(topLevelType, eventSystemFlags, targetContainer, nativeEvent) {
  const blockedOn = attemptToDispatchEvent(topLevelType, eventSystemFlags, targetContainer, nativeEvent)
}
```

在`attemptToDispatchEvent`中会做这几件事：

1. 首先根基真实的事件源对象，找到`e.target`真实的dom元素
2. 再根据dom元素，找到与它对应的fiber对象`targetInst`
3. 然后正式进去`legacy`模式的事件处理系统

#### 事件触发阶段总结

1. 首先通过统一的事件处理函数`dispatchEvent`，进行批量更新batchUpdate
2. 然后执行事件中对应的处理插件中的`extractEvents`，合成事件源对象，每次react会从事件源开始，从上遍历类型为hostComponent，即dom类型的fiber，判断props中是否有当前事件，比如onClick，最终形成一个事件执行队列，React就是用这个队列，来模拟事件捕获->事件源->事件冒泡这一过程
3. 最后通过`runEventsInBatch`执行事件队列，如果发现阻止冒泡，那么break跳出循环，最后重置事件源，放回到事件池中，完成整个流程

### V17的事件系统

1. 事件统一绑定到了container中，`ReactDOM.render(app, container)`，而不是document，这样的好处是利于微前端；微前端一个前端系统中可能有多个应用，如果继续采用全部绑定到document上，那么可能多应用会出现问题。
2. 对齐原生浏览器事件
3. 取消事件池

### 为什么自定义事件机制

- 抹平浏览器差异，实现更好的跨平台
- 避免垃圾回收，react引入事件池，在事件池中获取或释放事件对象，避免频繁的去创建和销毁
- 方便事件统一管理和事务机制

## setState

- 在合成事件或者生命周期中是异步
- 在原生事件，定时器中是同步

## 函数式编程 [link](https://juejin.cn/post/6844903936378273799)

### 为什么叫做函数式编程

函数即是一种描述集合和集合之间转换关系，输入通过函数都会返回有且只有一个输出值

函数实际上是一个关系，或者说是一种映射

### 无状态和数据不可变

- 数据不可变：要求所有数据都是不可变的，如果想修改一个对象，应该创建一个新的对象用来修改
- 无状态：强调对于一个函数，不管你何时运行，都应该像第一次运行一样，给定相同的输入，给出相同的输出

保证函数没有副作用，一来可以保证数据的不可变性，二来可以避免很多因为共享状态带来的问题

### 纯函数

纯函数的概念就两点：

- 不依赖外部状态（无状态）
- 没有副作用（数据不变）：不修改

纯函数的意义：

- 便于测试和优化：健壮性更强
- 可缓存性
- 自文档化
- 更少的bug

## Hooks



### useEffect

需要注意的点：

- 每一个Effect必然在渲染之后执行，因此不会阻塞渲染，提高了性能
- 在运行每个Effect之前，运行前一次渲染的Effect Cleanup函数
- 当组件销毁，运行最后一次Effect的Cleanup函数

useEffect的第二个参数是deps，依赖数组在判断元素是否发生改变时使用了`Object.is`进行比较，因此当`deps`中某一个元素为非原始类型（例如函数、对象等），**每次渲染都会发生改变**，从而每次都会触发Effect，失去了`deps`本身的意义

为什么在`useEffect`中使用`useState`的Setter函数，不需要将之作为Effect的依赖？

因为React内部已经对setter函数做了Memoization处理，每次渲染拿到的Setter函数都是完全一样的

#### 与componentDidMount区别

在 `render` 执行之后，`componentDidMount` 会执行，如果在这个生命周期中再一次 `setState` ，会导致再次 `render`，返回了新的值，浏览器只会渲染第二次 `render` 返回的值，这样可以避免闪屏。

但是 `useEffect` 是在真实的 DOM 渲染之后才会去执行，这会造成两次 `render` ，有可能会闪屏。



## React diff



## Fiber

浏览器在一帧里可能会做执行下列任务，而且它们的执行顺序基本是固定的：

- 处理用户输入事件
- JavaScript执行
- requestAnimation调用
- 布局Layout
- 绘制Paint

上述所说的理想一帧时间是`16ms`，如果浏览器处理完上述的任务（布局和绘制之后），还有盈余的时间，浏览器就会调用`requestIdleCallback`的回调

### 双缓存

在react中最多同时存在两棵Fiber树，当前屏幕上显示内容对应的Fiber树称为current Fiber树，正在内存中构建的Fiber树称为workInProgress Fiber树。构建分为mount和update两个阶段

### mount

首次执行ReactDOM.render会创建**fiberRootNode**和**rootFiber**。其中**fiberRootNode**是整个应用的根节点，**rootFiber**是<App />所在组件树的根节点

多次调用ReactDOM.render渲染不同的组件树，会拥有不同的rootFiber，但是整个应用的根节点只有一个，那就是`fiberRootNode`

`fiberRootNode`的current会指向当前页面上已渲染内容对应fiber树，即current Fiber树

```javascript
fiberRootNode.current = rootFiber
```

由于是首屏渲染，页面中还没有挂载任何DOM，所以`fiberRootNode.current`指向的`rootFiber`没有任何`子fiber`节点

#### beginWork

beginWork的工作就是传入当前Fiber节点，创建子Fiber节点

```typescript
function beginWork(current: Fiber | null, workInProgress: Fiber, renderLanes: Lanes): Fiber | null {}
```

传参：

- current：当前组件对应的Fiber节点在上一次更新时的Fiber节点，即`workInProgress.alternate`
- workInProgress：当前组件对应的Fiber节点

在beginWork中可以分为两个阶段

- Update：这个时候current是存在的，满足一定条件时可以复用current节点
- mount时：除fiberRootNode以外，current === null，会根据fiber.tag不同，创建不同类型的子fiber节点

#### completeWork

和`beginWork`一样，根据`current === null`判断是mount还是update

针对`HostComponent`类型，判断`update`时还需要考虑`workInProgress.state !== null`

##### udpate时

在这个阶段，`Fiber节点`已经存在对应的`DOM节点`，不需要生成`DOM节点`，需要做的就是处理`props`：

- onClick、onChange等回调函数的注册
- 处理style prop
- 处理`DANGEROUSLY_SET_INNER_HTML prop`
- 处理`children prop`

##### mount

这个阶段的逻辑主要包括三个：

- 为`Fiber节点`生成对应的`DOM节点`
- 将子孙DOM节点插入刚生成的DOM节点中
- 与update逻辑中updateHostComponent类似的处理props的过程

`mount`时只会在`rootFiber`存在`Placement effectTag`。那么`commit阶段`是如何通过一次插入`DOM`操作（对应一个`Placement effectTag`）将整棵`DOM树`插入页面的呢？

原因就在于`completeWork`中的`appendAllChildren`方法。

由于`completeWork`属于“归”阶段调用的函数，每次调用`appendAllChildren`时都会将已生成的子孙`DOM节点`插入当前生成的`DOM节点`下。那么当“归”到`rootFiber`时，我们已经有一个构建好的离屏`DOM树`

#### effectList

作为`DOM`操作的依据，`commit阶段`需要找到所有有`effectTag`的`Fiber节点`并依次执行`effectTag`对应操作。难道需要在`commit阶段`再遍历一次`Fiber树`寻找`effectTag !== null`的`Fiber节点`么？

为了解决这个问题，在`completeWork`的上层函数`completeUnitOfWork`中，每个执行完`completeWork`且存在`effectTag`的`Fiber节点`会被保存在一条被称为`effectList`的单向链表中。

`effectList`中第一个`Fiber节点`保存在`fiber.firstEffect`，最后一个元素保存在`fiber.lastEffect`。

在“归”阶段，所有有`effectTag`的`Fiber节点`都会追加在`effectList`中，最终形成一条以`rootFiber.firstEffect`为起点的单向链表

```javascript
rootFiber.firstEffect.nextEffect = fiberA
fiberA.nextEffect = fiberB
```

### commit阶段

在`rootFiber.firstEffect`上保存了一条需要执行`副作用`的`Fiber节点`的单向链表`effectList`，这些`Fiber节点`的`updateQueue`中保存了变化的`props`。

除此之外，一些生命周期钩子（比如`componentDidXXX`）、`hook`（比如`useEffect`）需要在`commit`阶段执行。

`commit`阶段的主要工作（即`Renderer`的工作流程）分为三部分：

- before mutation阶段（执行`DOM`操作前）
- mutation阶段（执行`DOM`操作）
- layout阶段（执行`DOM`操作后）

在`before mutation阶段`之前和`layout阶段`之后还有一些额外工作，涉及到比如`useEffect`的触发、`优先级相关`的重置、`ref`的绑定/解绑。

下面逻辑是执行在`before Mutation`之前的：

```js
do {
    // 触发useEffect回调与其他同步任务。由于这些任务可能触发新的渲染，所以这里要一直遍历执行直到没有任务
    flushPassiveEffects();
  } while (rootWithPendingPassiveEffects !== null);

  // root指 fiberRootNode
  // root.finishedWork指当前应用的rootFiber
  const finishedWork = root.finishedWork;

  // 凡是变量名带lane的都是优先级相关
  const lanes = root.finishedLanes;
  if (finishedWork === null) {
    return null;
  }
  root.finishedWork = null;
  root.finishedLanes = NoLanes;

  // 重置Scheduler绑定的回调函数
  root.callbackNode = null;
  root.callbackId = NoLanes;

  let remainingLanes = mergeLanes(finishedWork.lanes, finishedWork.childLanes);
  // 重置优先级相关变量
  markRootFinished(root, remainingLanes);

  // 清除已完成的discrete updates，例如：用户鼠标点击触发的更新。
  if (rootsWithPendingDiscreteUpdates !== null) {
    if (
      !hasDiscreteLanes(remainingLanes) &&
      rootsWithPendingDiscreteUpdates.has(root)
    ) {
      rootsWithPendingDiscreteUpdates.delete(root);
    }
  }

  // 重置全局变量
  if (root === workInProgressRoot) {
    workInProgressRoot = null;
    workInProgress = null;
    workInProgressRootRenderLanes = NoLanes;
  } else {
  }

  // 将effectList赋值给firstEffect
  // 由于每个fiber的effectList只包含他的子孙节点
  // 所以根节点如果有effectTag则不会被包含进来
  // 所以这里将有effectTag的根节点插入到effectList尾部
  // 这样才能保证有effect的fiber都在effectList中
  let firstEffect;
  if (finishedWork.effectTag > PerformedWork) {
    if (finishedWork.lastEffect !== null) {
      finishedWork.lastEffect.nextEffect = finishedWork;
      firstEffect = finishedWork.firstEffect;
    } else {
      firstEffect = finishedWork;
    }
  } else {
    // 根节点没有effectTag
    firstEffect = finishedWork.firstEffect;
  }
```



`layout`之后的逻辑会做以下事情：

- `useEffect`相关处理
- 性能追踪相关
- 在`commit`阶段会触发一些生命周期钩子（如 `componentDidXXX`）和`hook`（如`useLayoutEffect`、`useEffect`）。

#### before mutation

整个过程就是遍历`effectList`并调用`commitBeforeMutationEffects`函数处理

```javascript
// 1.保存之前的优先级，以同步优先级执行，执行完毕后恢复之前优先级
// 2.将当前上下文标记为CommitContext，作为commit阶段的标志
// 3.处理focus状态
// 4.beforeMutation阶段的主函数
commitBeforeMutationEffects(finishedWork);
```

##### commitBeforeMutationEffects

这个方法整体可以分为三个部分：

1. 处理`DOM节点`渲染/删除后的 `autoFocus`、`blur` 逻辑。
2. 调用`getSnapshotBeforeUpdate`生命周期钩子。
3. 调度`useEffect`。

**为什么需要异步调用？**

> 与 componentDidMount、componentDidUpdate 不同的是，在浏览器完成布局与绘制之后，传给 useEffect 的函数会延迟调用。这使得它适用于许多常见的副作用场景，比如设置订阅和事件处理等情况，因此不应在函数中执行阻塞浏览器更新屏幕的操作。

`useEffect`异步执行的原因主要是防止同步执行时阻塞浏览器渲染。

#### mutation

该阶段也是遍历`effectList`，执行函数

##### commitMutationEffects函数

```typescript
function commitMutationEffects(root: FiberRoot, renderPriorityLevel) {
  // 遍历effectList
  // 根据 ContentReset effectTag重置文字节点
  // 更新ref
   // 根据 effectTag 分别处理
}
```

#### Placement effect

当`Fiber节点`含有`Placement effectTag`，意味着该`Fiber节点`对应的`DOM节点`需要插入到页面中。

调用方法`commitPlacement`

1. 获取父级`DOM节点`。其中`finishedWork`为传入的`Fiber节点`。
2. 获取`Fiber节点`的`DOM`兄弟节点（**`getHostSibling`（获取兄弟`DOM节点`）的执行很耗时**）
3. 根据`DOM`兄弟节点是否存在决定调用`parentNode.insertBefore`或`parentNode.appendChild`执行`DOM`插入操作。

#### Update effect

当`Fiber节点`含有`Update effectTag`，意味着该`Fiber节点`需要更新。调用的方法为`commitWork`，他会根据`Fiber.tag`分别处理。

#### HostComponent mutation

当`fiber.tag`为`HostComponent`，会调用`commitUpdate`。

最终会在[`updateDOMProperties`](https://github.com/facebook/react/blob/970fa122d8188bafa600e9b5214833487fbf1092/packages/react-dom/src/client/ReactDOMComponent.js#L378)中将`render阶段 completeWork` 中为`Fiber节点`赋值的`updateQueue`对应的内容渲染在页面上。

#### layout



### 任务优先级

为了避免任务被饿死，会设置一个超时时间。这个超时时间不是死的，低优先级的可以慢慢等待，高优先级的任务应该率先被执行。在react中定义了5个优先级：

- Immediate(-1)这个优先级的任务会同步执行，或者说马上执行且不能中断
- UserBlocking（250ms）这些任务一般是用户交互的结果，需要即时得到反馈
- Normal(5s)应对哪些不需要立即感受到的任务，例如网络请求
- Low(10s)这些任务可以放后，但是最终应该得到执行，例如分析通知
- Idle(没有超时时间)一些没有必要做的任务，可能会被饿死

React中setState执行流程图如下：

<img src="/Users/wish/Documents/Pictures/16deed1711f281b3_tplv-t2oaga2asx-zoom-in-crop-mark_1304_0_0_0.gif" style="zoom:50%;" />

### 执行过程

首先看看`performUnitOfWork`的实现，其实就是一个深度优先的遍历：

```typescript
/**
 * @param fiber 当前需要处理的节点
 * @param topWork 本次更新的根节点
 */
function performUnitOfWork(fiber: Fiber, topWork: Fiber) {
  beginWork(fiber)

  if (fiber.child) {
    return fiber.child
  }

  let temp = fiber
  while(temp) {
    completeWork(temp)

    if (temp === topWork) {
      break
    }
    if(temp.sibling) {
      return temp.sibling
    }
    temp = temp.return
  }
}
```

Fiber就是我们所说的工作单元，`performUnitOfWork`负责对`Fiber`进行操作，并按照深度遍历的顺序返回下一个Fiber

react每次渲染有两个阶段：`Reconciliation`和`Commit`阶段

- 协调阶段：可以认为是Diff阶段，**可被中断**，这个阶段会找出所有节点变更，例如节点新增、删除、属性变更等，以下生命周期钩子会在该阶段被调用：
  - constructor
  - componentWillMount
  - componentWillReceiveProps
  - static getDerivedStateFromProps
  - shouldComponentUpdate
  - componentWillUpdate
  - render
- 提交阶段：将上一个阶段计算出来的需要处理的副作用一次性执行了，这个阶段必须同步执行，不能被打断，这些生命钩子在该阶段提交：
  - getSnapshotBeforeUpdate
  - componentDidMount
  - componentDidUpdate
  - componentWillUnMount

因为协调阶段可能被中断、恢复、甚至重做。**React协调阶段的生命周期钩子可能会被调用多次**，例如`componentWillMount`可能会被调用两次

因为建议 **协调阶段的生命周期钩子不要包含副作用**，索性React就废弃了这部分可能包含副作用的生命周期方法，例如`componentWillMount`、`componentWillUpdate`

看一下`Fiber`的结构：

```typescript
interface Fiber {
  //...
  stateNode: any
  updateQueue: any
  // 新的、待处理的props
  pendingProps: any
  // 上一次渲染的props
  memoizedProps: any
  // 上一次渲染的组件状态
  memoizedState: any
  
  // 当前节点的副作用类型，例如节点更新、删除、移动
  effectTag: SideEffectTag
  // 和节点关系一样，react用链表将所有副作用的Fiber连接起来
  nextEffect: Fiber | null
  
  
 	firstEffect: Fiber
  
	lastEffect: Fiber
  
  // 指向旧树种的节点
  alternate: Fiber | null
}
```

然后再看看`beginWork`如何对Fiber进行比较的：

```typescript
function beginWork(fiber: Fiber): Fiber | undefined {
  if (fiber.tag === WorkTag.HostComponent) {
    diffHostComponent(fiber)
  } else if (fiber.tag === WorkTag.ClassComponent) {
    diffClassComponent(fiber)
  } else if (fiber.tag === WorkTag.FunctionComponent) {
    diffFunctionComponent(fiber)
  } else {
    //...
  }
}
```



#### reconcileChildren

该函数是Reconcilder模块的核心部分

- 对于`mount`的组件，会创建新的子Fiber节点
- 对于`update`的组件，会将当前组件与该组件在上一次更新时对应的Fiber节点比较（Diff），将比较的结果生成新Fiber节点

```typescript
export function reconcileChildren(
  current: Fiber | null,
  workInProgress: Fiber,
  nextChildren: any,
  renderLanes: Lanes
) {
  if (current === null) {
    // 对于mount的组件
    workInProgress.child = mountChildFibers(
      workInProgress,
      null,
      nextChildren,
      renderLanes,
    );
  } else {
    // 对于update的组件
    workInProgress.child = reconcileChildFibers(
      workInProgress,
      current.child,
      nextChildren,
      renderLanes,
    );
  }
}
```

不管走哪个逻辑，最终都会生成新的子Fiber节点并赋值给`workInProgress.child`，作为本次`beginWork`返回值，并作为下次`performUnitOfWork`执行时`workInProgress`的传参

**mountChildFibers和reconcileChildFibers两个方法逻辑基本一致，唯一的区别就是reconcileChildFibers会为生成的Fiber节点带上effectTag属性**

#### effectTag

`render`阶段的工作是在内存中进行，当工作结束后会通知`Renderer`需要执行的`DOM`操作。要执行DOM操作的具体类型就保存在`fiber.effectTag`中



## 性能优化

### 使用React.Memo缓存组件

提升应用性能的一种方法就是实现memoization。Memoization是一种优化技术，主要通过存储昂贵的函数调用结果，并在在此发生相同的输入时返回缓存的结果，以此加速程序。

父组件的每次状态更新，都会导致子组件重新渲染。可以使用React.memo来缓存组件，只有当传入组件的状态值发生变化时才会重新渲染，如果传入同样的值，则返回缓存的组件

### 使用useMemo缓存大量的计算

有时渲染是不可避免的，如果组件是功能组件，重新渲染会导致每次都调用大型计算函数，这是非常消耗性能的，我们可以使用useMemo钩子来“记忆”这个计算函数的计算结果

### 使用React.PureComponent，shouldComponentUpdate

父组件状态的每次更新，都会导致子组件的重新渲染，即时是传入相同的props。这里的重新渲染不是会说更新DOM，而是每次都会调用diff算法来判断是否需要更新DOM

### 避免使用内联对象

使用内联对象时，react会在每次渲染时重新创建此对象的引用，这回导致接收此对象的组件将其视为不同的对象，因此该组件对于prop的浅层比较始终返回false，导致组件一直重新渲染。

```react
// bad
function Component(props) {
  const aProp = { someProp: 'someValue' }
  return <AnotherComponent style={{ margin: 0 }} aProp={aProp} />
}
// good
const style = { margin: 0 }
function Component(props) {
  const aProp = { someProp: 'someValue' }
  return <AnotherComponent style={style} aProp={aProp} />
}
```

### 避免使用匿名函数

虽然匿名函数式传递函数的好方法，但它们在每次渲染上都有不同的引用。为了保持对作为prop传递给react组件的函数的相同引用，可以将其声明为类方法或使用useCallback钩子来帮助我们保持相同的引用

```react
// bad
function Component(props) {
  return <AnotherComponent onChange={() => props.callback(props.id)} />
}
```

### 延迟加载不是立即需要的组件

延迟加载实际上不可见（或不是立即需要）的组件，React加载的组件越少，加载组件的速度就越快，因此，如果初始渲染感觉相当粗糙，则可以在初始安装完成后通过在需要时加载组件来减少加载的组件数量

```react
const MUITooltip = React.lazy(() => import('antd/Tooltip'))
function Tooltip({ children, title }) {
  return (
  	<React.Suspense fallback={children}>
    	<MUITooltip title={title}>{children}</MUITooltip>
    </React.Suspense>
  )
}
```

### 尽量使用CSS而不是强制加载和卸载组件

### 使用React.Fragment避免添加额外的DOM

### QAQ

1. 为什么不使用`generator`

- generator不能在栈中间让出
- generator是有状态的，很难在中间恢复这些状态



## Hook

fiber对象属性：

```typescript
const fiber = {
  memoizedState: null, // hooks链表
}
```



hook对象属性：

```typescript
const hook: Hook = {
  memoizedState: null, // useState中 保存 state 信息 ｜ useEffect 中 保存着 effect 对象 ｜ useMemo 中 保存的是缓存的值和 deps ｜ useRef 中保存的是 ref 对象
  baseState: null, // usestate和useReducer中 保存最新的更新队列
  baseQueue: null, // usestate和useReducer中,一次更新中 ，产生的最新state值
  queue: null, // 保存待更新队列 pendingQueue ，更新函数 dispatch 等信息
  next: null // 指向下一个 hooks对象
}
```



### useEffect

学习本章需要带着以下问题

1. useEffect的使用方法
2. 执行时机
3. 回调的执行时机
4. 原理是什么



### useCallback

首先来看一下源码，初次加载的时候：

```typescript
function mountCallback<T>(callback: T, deps: Array<mixed> | void | null): T {
  const hook = mountWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  hook.memoizedState = [callback, nextDeps];
  return callback;
}
```

我们可以看到这一个hook的`memoizedState`是将函数和依赖项都存入到了`memoizedState`中

### useMemo

阅读useMemo的源码，挂载的时候：

```typescript
function mountMemo<T>(
  nextCreate: () => T,
  deps: Array<mixed> | void | null,
): T {
  const hook = mountWorkInProgressHook();
  const nextDeps = deps === undefined ? null : deps;
  const nextValue = nextCreate();
  hook.memoizedState = [nextValue, nextDeps];
  return nextValue;
}
```

前两句和其他的hook都是一样的，创建一个hook，获取依赖项deps，相比于`useCallback`，我们会发现这里会先执行参数一，并将一个值和依赖项存入`memoizedState`中



## redux

### 注意点

1. 仅使用`state`和`action`参数计算新的状态值
2. 禁止直接修改`state`，必须通过复制现有state，并对复制的值进行更改的方式来做**不可变更新**
3. 禁止任何异步逻辑、依赖随机值或导致其他“副作用”的代码



### QAQ

##### 为什么不能直接更改state？

- 会导致bug，例如UI未正确以显示最新值
- 更难理解状态更新的原因和方式
- 编写测试更加困难
- 打破了正确使用“时间旅行调试”的能力
- 违背了Redux的预期精神和使用模式

## 相关链接

[这可能是最通俗的 React Fiber(时间分片) 打开方式](https://juejin.cn/post/6844903975112671239)

[「react进阶」一文吃透react事件系统原理](https://juejin.cn/post/6955636911214067720)

[简明 JavaScript 函数式编程——入门篇](https://juejin.cn/post/6844903936378273799)

[「react进阶」一文吃透react-hooks原理](https://juejin.cn/post/6944863057000529933#heading-2)