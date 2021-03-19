React渲染行为细节，以及如何使用 `Context` 和 `React-Redux` 如何影响渲染。

我看到很多人对 **React 什么时候，为什么以及如何 渲染和重渲染组件？ 以及使用 `Context & React-Redux` 对重渲染的时机和作用于有什么影响** 有很多疑惑。下面是汇总的一些理解，希望能帮助人们理清这些问题。

内容表：

1. 什么是渲染（What's 'Rendering'?）
   - 渲染过程概况（Rendering Process Overview）
   - 渲染和提交阶段(Render and Commit Phases)
2. React是如何处理渲染的？（How Does React Handle Renders?）
   - 渲染队列（Queuing Renders）
   - 标准渲染行为（Standard Render Behavior）
   - React渲染规则（Rules of React Rendering）
   - 组件元数据和Fibers（Component Metadata and Fibers）
   - 组件类型和一致性比较（Component Types and Reconciliation）
   - 键和一致性比较（Keys and Reconciliation）
   - 渲染批处理和时机（Render Batching and Timing）
   - 渲染行为边界情况（Render Behavior Edge Cases）
3. 提升渲染性能（Imporving Rendering Performance）
   - 组件渲染优化技术（Component Render Optimization Techniques）
   - 新的属性引用如何影响渲染优化（How New Props References Affect Render Optimizations）
   - 优化属性引用（Optimizing Props References）
   - 记忆一切？（Memoize Everything？）
   - 不可变性和重渲染（Immutability and Rerendering）
   - 测量React组件渲染性能（Measuring React Component Rendering Performance）
4. 上下文和渲染行为（Context and Rendering Behavior）
   - 上下文基础（Context Basics）
   - 更新上下文的值（Updating Context Values）
   - 状态更新，上下文和重渲染（State Updates， Context and Re-Renders）
   - 上下文更新和渲染优化（Context Updates and Render Optimizations）
5. React-Redux和渲染行为（React-Redux and Rendering Behavior）
   - React-Redux订阅（React-Redux Supscriptions）
   - `connect` 和 `useSelector` 的差异
6. 总结
7. 最后补充说明
8. 进一步阅读



## 1. 什么是渲染（What's 'Rendering'?）



渲染是React要求你的组件根据当前  **属性和状态** 组合，描述他们希望自己的UI部分现在是什么样子的过程。

### 1.1渲染过程概况（Rendering Process Overview）

在渲染过程中，React会从组件树的根组件开始，然后循环向下找出所有标记了需要更新的组件。对每个标记了的组件，React会调用 **`classComponentInstance.render()`**(对class组件) 或者 **`FunctionComponent()`**(对函数组件)，然后保存渲染输出。

一个组件的渲染输出通常以 `JSX` 语法书写，JSX在JS编译或者部署时，会转换为 `React.createElement()` 调用。 `createElement()` 会返回React元素，它是一种用来描述UI结构的普通JS对象。例如：

```js
// JSX语法
return <SomeComponent a={42} b="testing">Text Here</SomeComponent>

// 编译时转换为 React.createElement() 调用
return React.createElement(SomeComponent, { a: 42, b: 'testing'}, 'Text here')

// 最终变为一个React元素对象
{type: SomeComponent, props: {a: 42, b: 'testing'}, children: ["Text Here"]}
```

在收集了整个组件树的渲染输出后，React会对新的对象树进行  `diff` （**通常也被叫做 `Virtual DOM`**）, 并收集所有需要应用的变化列表，以使真实的DOM看起来像当前所需的输出，这个diff和计算过程称之为 **一致性比对**（`reconciliation`）。

然后，React将所有计算出的变化以一个同步的顺序应用到DOM中。



### 1.2渲染和提交阶段(Render and Commit Phases)

React工作组将这个工作概念上划分为2个阶段：

1. 渲染阶段：包含所有渲染组件和计算变化的工作
2. 提交阶段：将这些变化应用到DOM的过程

在React在提交阶段更新了DOM后，它会更新所有指向请求DOM节点和组件实例的引用（**`refs`**）, 然后同步的调用 **`componentDidMount & componentDidUpdate`** 生命周期方法，和 `useLayoutEffect` 钩子。

然后React会设置一个短暂的超时（timeout），当超时结束后，运行所有的 `useEffect` 钩子，这一步也称之为 **被动Effects** 阶段（passive Effects Phase）.

可以通过 [React Lifecycle methods diagram](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/) 查看生命周期的方法图（这个图片现在没有显示effect hooks时机，后面我会加上的）。

*在React即将到来的 `Concurrent Mode` 中，React能够暂停（pause）渲染阶段的工作，从而允许浏览器处理事件。React之后可以恢复（Resume），或者离开（throw away），又或者在合适的时候重计算，一旦渲染完成，React仍旧会同步的运行提交阶段*。

其中需要理解的一个关键点是， **渲染和更新DOM不是一回事**， 一个组件可能会被渲染，而不会因此发生任何可见的变化。当React渲染一个组件时：

- 组件可能返回和上次相同的渲染输出，因此不需要做任何变化
- 在并发模式中，React可能多次渲染一个组件，但如果其他更新使当前正在进行的工作无效，则每次都会丢掉渲染输出



## 2.React是如何处理渲染的？（How Does React Handle Renders?）

### 2.1 渲染队列（Queuing Renders）

在首次渲染完成后，有以下几种方式告诉React将重渲染加入队列：

- 类组件（class Components）
  - **`this.setState()`**
  - **`this.forceUpdate()`**
- 函数组件（function Components）
  - **`useState`** setters
  - **`useReducer`** dispatches
- 其他：
  - 重新调用 `ReactDOM.render(<App>)` (这相当于对根组件调用 `forceUpdate()`)



### 2.2 标准渲染行为（Standard Render Behavior）

记住下面这段话很重要：

**React默认行为是，当父组件渲染时，React会递归渲染其包含的所有子组件**。

例如，假设有个组件树 `A -> B -> C -> D`, 当用户在 `B` 中点击按钮用来增加Counter值时：

1. 我们在 `B` 组件中，调用 `setState()`, 这将会把B组件的重渲染加入队列
2. React从组件树的顶部开始渲染传递
3. React发现 `A` 没有标记需要更新，于是跳过它
4. React发现 `B` 标记了需要更新，于是渲染它，`B` 和上次一样返回 `<C />`
5. `C` 一开始并没有标记需要更新，但是，因为它的父组件 `B` 渲染了，React向下移动并也会渲染 `C`。组件 `C` 是返回 `<D />`
6. `D` 也没有标记需要渲染，但是父组件 `C` 渲染了，React向下移动，也会渲染 `D`

再次强调一下这个过程： **默认情况下，渲染一个组件会导致其所有的子组件也会被渲染**。

另一个关键点是：**在普通的渲染中，React不关心是否 “属性变化了”， 它会因为父组件的渲染，而无条件的渲染子组件。**

这也意味着，在根组件 **`<App>`** 调用 `setState()`,即使不改变任何行为，都将导致React重新渲染组件树中的每一个组件。毕竟，React最初的宣传之一就是 **就像我们每次更新都要重新绘制整个应用一样** 。

现在，组件树中的大多数组件很可能都会返回和上次一样的渲染输出，因此React不需要对DOM做出任何改变。但是，React还是需要做询问组件的工作，让组件渲染自身以及比对渲染输出。这两点都是需要花费时间和精力。

记住，**渲染不是一件坏事情，这是React知道是否需要对DOM做出改变的方式。**



### 2.3 React渲染规则（Rules of React Rendering）

React渲染最重要的规则之一就是 **渲染必须要纯(`pure`)，没有副作用（`side effects`）**。这令人很棘手和困惑，因为很多副作用是不显著的，并且也不会导致任何崩溃。例如，严格上来讲 `console.log()` 语句就是一种副作用，但是它不会破坏什么。在渲染过程中调用AJAX无疑也是一种副作用，它会根据请求的类型导致应用的异常行为。

Sebastian MArkbage 写过一片很好的文章叫做 [The Rules Of React](https://gist.github.com/sebmarkbage/75f0838967cd003cd7f9ab938eb1958f)。在这篇文章中，他定义了不同React生命周期（也包括 `render`）方法中的异常行为，以及什么样的操作能被安全的称之为 **纯**， 以及哪些操作不安全。这篇文章值得全文通读，下面我总结其关键点：

1. 渲染逻辑必须不能做的：
   - 不能改变存在的变量和对象
   - 不能创建随机值，比如 `Math.random() | Date.now()`
2. 渲染逻辑可以做的：
   - 在渲染时，可变对象是最新创建出来的
   - 抛出错误
   - 懒加载未创建的数据，比如缓存值



### 2.4 组件元数据和Fibers（Component Metadata and Fibers）

React存储了一个内部数据结构，用于追踪应用中所有的当前组件实例。数据结构的核心是一个叫 `fiber` 的对象，它包含如下元数据字段：

- 此时组件树中应当被渲染的组件类型是什么
- 该组件关联的属性和状态
- 指向父组件，兄弟组件和子组件的指针
- 其它用于React追踪渲染过程的元数据

可以 [查看React 17中 Fiber的定义](https://github.com/facebook/react/blob/v17.0.0/packages/react-reconciler/src/ReactFiber.new.js#L47-L174)。

在渲染的过程中，React会迭代 `fiber` 对象，并在计算新的渲染结果时构建一颗更新后的树。

注意，这些fiber对象存储真实的组件属性和状态值。当在组件中使用 `props & state` 时，React实际上是让你访问的fiber对象上存储的值。实际上，具体拿类组件来讲，[React在渲染组件前显式的拷贝了 componentInstance.props = newProps](https://github.com/facebook/react/blob/v17.0.0/packages/react-reconciler/src/ReactFiberClassComponent.new.js#L1038-L1042)。 因此， `this.props` 只是因为React从内部数据结构中拷贝了一份才存在的。某种程度上讲，React组件其实是fiber对象的一种门面（facade pattern）。

同理，React Hooks能运行，是因为 [React将一个组件的所有Hooks以链表的形式存储在该组件的fiber对象上](https://www.swyx.io/getting-closure-on-hooks/)。当React渲染函数组件时，它从fiber中获取所有关于hook的链表数据，每次你调用另一个hook时，[它将返回存在在hook描述对象上合适的值给你（例如 `useReducer` 返回 `state & dispatch` 值）](https://github.com/facebook/react/blob/v17.0.0/packages/react-reconciler/src/ReactFiberHooks.new.js#L795)。

当父组件第一次渲染给定的子组件时，React创建一个fiber对象，用来追踪组件实例。对类组件，[它字面上调用 `const instance = new YourComponentType(props)`](https://github.com/facebook/react/blob/v17.0.0/packages/react-reconciler/src/ReactFiberClassComponent.new.js#L653) ,然后保存实际的组件实例到fiber对象上。对函数组件，React [将 `YourComponentType(props)` 作为函数调用](https://github.com/facebook/react/blob/v17.0.0/packages/react-reconciler/src/ReactFiberHooks.new.js#L405)。



### 2.5 组件类型和一致性比较（Component Types and Reconciliation）

[正如 'Reconciliation'文档](https://reactjs.org/docs/reconciliation.html#elements-of-different-types) 中描述的一样，React试着尽可能的复用已存在的组件树和DOM结构，使重渲染更加高效。如果你让React在组件树中相同的位置渲染相同类型的组件树或者HTML节点，React将复用它，并且只在合适的时候进行更新，而不是完全重新渲染。这意味着，只要你一直要求React在同一个地方渲染该组件类型，React就会保持组件实例的活力。对类组件，它实际上时用的相同的组件实例。对函数组件，它没有像类组件那样真正的实例，但是我们可以把 `<MyFunctionComponent />` 看作是一种 "实例"， 即 “这个类型的组件在这里显示并保持活力”。

那么，React是如何知道输出的时间和方式到底发生了什么变化呢？

React渲染首先会通过 `type` 属性来比较元素， 使用 `===` 进行引用比较。如果树中给定的节点的元素类型发生变化，比如从 `<div>` 变为了 `<span>` 或者 `<ComponentA>` 变为了 `<ComponentB>`, React会通过假设整棵树都发生了变化来加速比较过程。结果就是，React将销毁整个存在的组件树区域，包括DOM节点，然后重建新的组件树实例。

这意味着，你千万不要在渲染时渲染新的组件类型！每当你创建一个新的组件类型时，它都是一个不同的引用，这将导致React重复摧毁和重新创建子组件树。

换句话说，不要做以下操作：

```jsx
// 🙅
function ParentComponent() {
  // 这将导致很次都创建一个新的 `ChildComponent` 引用
  function ChildComponent() {}
  
  return <ChildComponent />
}
```

下面是正确的做法，就是分开定义组件：

```jsx
// 这只创建一个组件类型引用👌
function ChildComponent() {}

function ParentComponent() { 
  return <ChildComponent />
}
```



### 2.6 键和一致性比较（Keys and Reconciliation）

React识别组件实例的另一种方式就是通过 `key` 这个伪属性。 `key` 对React来讲是一种指令，它实际上不会传递给实际的组件。React把 `key` 当作唯一标识符，用于区分特定的组件类型实例。

使用key的主要场景就是渲染列表。如果你的数据会改变，比如排序，添加，删除列表，key就显得特别的重要了。**这里的 key应该是唯一的标识符，尽可能来自你的数据，实在没有别的key了，才去使用数组索引当作key。**

下面例子告诉你为什么这样做很重要：假设渲染一个10条数据的 `<TodoListItem>` 组件，使用数组索引作为keys。React发现10条列表项，keys是 `0-9`。假设我们删除了 `6 & 7`, 然后添加了3条新的数据在末端，现在的渲染keys是 `0-10`。在React看来，只加入了一个列表项到尾部，总数据条数从 `10` 变成了 `11`, React会很高兴的复用已经存在的组件实例和DOM节点。但是，这意味着，我们可能现在渲染 `<TodoListItem key={6}>` 可能先前是第 `8` 号元素。因此，组件实例仍然是活跃的，但是得到的是一个不同的数据对象（以prop形式）。这可能没什么问题，但是也可能会产生异常行为。另外，React现在不得不去应用更新几个列表项来改变文本和其它DOM内容，因为现有的列表项现在不得不显示和之前不同的数据，这些更新在这里真的没什么必要，因为这些列表项其实没有发生变化。

如果给列表项使用 `key={todo.id}`，React就能够正确的看到我们删除了2项，然后添加了3项到尾部。React将销毁2个被删除的组件实例和其关联的DOM，并创建3个新的组件实例和其关联的DOM。这比不必要地更新那些实际上没有变化的组件要好得多。

键对于列表之外的组件实例标识也很有用。你可以给任何React组件添加一个 `key` 用于表示其标识，改变该key，将会导致React销毁旧的组件实例和DOM，并创建新的组件实例和DOM。一个常见的用法就是 **`list + detail form`** 组合，表单用于显示当前选中项的数据。改变选中项时，渲染 `<DetailForm key={selectedItem.id}>` 将导致React销毁并重建该form，因此可以避免表单数据滞后的问题。



### 2.7 渲染批处理和时机（Render Batching and Timing）

默认情况下，每次调用 `setState()` 都会导致React开始新的渲染过程，同步的执行并返回。然而，React也自动的使用了一些优化，以 **批量渲染的形式**。渲染批处理是指多个对 `setState()` 的调用导致单个渲染通道被添加到队列中执行，通常会有轻微的延迟。

React文档提到 [状态更新可能是异步的](https://reactjs.org/docs/state-and-lifecycle.html#state-updates-may-be-asynchronous)。这指的就是渲染批处理行为。特别是，React会自动对React事件程序中发生的状态更新进行批处理。由于React事件处理程序在一个典型的React应用中占了很大的一部分代码，这意味着在一个给定的应用中，大部分的状态更新实际上都是批量化的。

React对 `event handlers`的渲染批处理， 是通过将其包裹到一个内部称之为 `unstable_batchedUpdates` 的函数中实现的。在 `unstable_batchedUpdates`运行时， React将追踪所有的在队列中的状态更新，然后在之后的一次渲染中应用它们。对 `event handlers` 而言，这一切工作正常，因为React已经确切的知道了对于特定的事件，哪一个handlers需要被调用。

理论上，下面伪代码可以大致描述React内部的工作原理：

```js
// 伪代码
function inernalHandleEvent(e) {
  const userProvidedEventHandler = findEventHandler(e)
  
  let batchedUpdates = []
  
  unstable_batchedUpdates(() => {
    // 在这里的任何更新队列 都将被放置在 batchedUpdates 中
    userProvidedEventHandler(e)
  })
  
  renderWithQUeuedStateUpdates(batchedUpdates)
}
```

**然而，这意味着在实际立即调用栈之外的状态更新队列将不会被批量在一起**。例如：

```js
const [counter, setCounter] = useState(0)

const onClick = async () => {
  // a
	setCounter(0)
  // b
  setCounter(1)
  
  // c 在这里碰到了事件处理器 因此上面的 a 和 b 都添加到了 batchedUpdates中
  const data = await fetchSomeData()
  
  // d
  setCounter(2)
  // e
  setCounter(3)
}
```

这将执行 `3` 次渲染过程：

1. `a-b-c`: 第一次将批处理 `	setCounter(0) & setCounter(1)`, 因为它俩都是在原来的事件处理调用栈期间出现，因此它们同时出现在 `unstable_batchedUpdates()` 调用中。
2. `d`: 然后 `setCounter(2)` 调用 在 `await`之后发生，这意味着原来的同步调用栈已经完成了，而后半部分的函数是在一个完全独立的事件循环调用栈中更晚的运行。正因如此，React会作为 `setCounter(2)` 调用里面的最后一步，同步执行整个渲染过程，完成渲染过程，并从 `setCounter(2)` 返回。
3. `e`: 同样的事情发生在 `setCounter(3)` 上，因为它也运行在原来的事件处理之外，因此也在批处理之外。

在提交阶段的生命周期方法 `componentDidMount & componentDidUpdate & useLayoutEffect` 中存在一些额外的边际情况（edge cases），这些主要是为了让你在渲染后，但在浏览器有机会绘制之前，执行额外的逻辑而存在的，特别是，一个常见的用例是：

- 第一次渲染组件，使用了部分数据而不是完整的数据
- 在提交阶段的生命周期，使用 `refs` 来测量页面中实际的DOM节点的尺寸
- 根据测量结果来设置组件中的某些状态
- 使用更新的数据立即重新渲染

这种情况，我们不希望 **部分** 渲染的UI对用户可见，我们只希望最后完成后的UI对用户可见。浏览器在UI被修改后，将重新计算DOM结构，但当JS脚本在执行时，它会阻塞event loop，因此屏幕上实际上不会出现绘制。因此，你可以执行多次DOM改变，比如 `div.innerHTML = '1'; div.innerHTML='b';`, `a` 将永远不会显示。

正因为如此，React将始终同步运行提交阶段生命周期中的渲染。这样一来，如果你真的尝试执行 `partial -> final` 这样的更新切换，那么只有 "最终 "的内容才会在屏幕上可见。

最后，据我所知，`useEffect` 回调中的状态更新是排队的，当所有的 `useEffect` 回调完成后，在 “被动效果” 阶段结束时进行刷新。

导出 `unstable_batchedUpdates` API没有什么价值，但是：

- 看名字就知道，`unstable` 表示不稳定，就意味着它不是React官方推荐的API
- 另一方面，React项目组说过， “它是所有不稳定API中最稳定的一个，facebook中的一半代码都依赖它”
- 不像其他核心的React APIs（通过 `react` 包导出的apis），`unstable_batchedUpdates` 是一个一致性比较相关的接口，它并不是React包的一部分。相反，它实际上是在 `react-dom & react-native` 中被导出的。这意味着其他的一些 `reconcilers`， 比如 `react-three-fiber | ink` 有可能不会导出 `unstable_batchedUpdates` 这个函数

对 React-Redux V7， [在内部开始使用了 `unstable_batchedUpdates`](https://blog.isquaredsoftware.com/2018/11/react-redux-history-implementation/#use-of-react-s-batched-updates-api), 需要一些小的技巧才能同时在react-dom 和 react-native中使用。

**在React即将到来的并发模型中，React将始终采用批量更新**。



### 2.8 渲染行为边界情况（Render Behavior Edge Cases）

React将在开发模式下，在 `<StrickMode>` 标签下2次渲染组件。这意味着，很多时候你渲染逻辑运行和提交渲染过程不一样，在计算渲染出现次数的时候，你不能依赖 `console.log()` 语句。

相反，要么使用React开发工具Profiler去捕获和计算总的提交渲染次数或者在 `useEffect` hook内部或者 `componentDidMount/Update` 生命周期中添加log，这样，日志只有在React实际完成一次渲染过程和提交之后才打印。

在正常情况下，你不应该在实际渲染逻辑中排队更新状态。换句话说，在点击事件发生时创建一个将调用 `setSomeState()` 的点击回调是可以的，但是你不能将 `setSomeState()` 作为实际渲染行为的一部分调用。

但是有一种例外情况就是，函数组件可能在渲染的时候直接调用 `setSomeState()`, 只要它条件性的完成并且当组件渲染时不要每次都执行就可以了。这种行为类似于 [类组件中的 `getDerivedStateFromProps`](https://reactjs.org/docs/hooks-faq.html#how-do-i-implement-getderivedstatefromprops)。如果一个函数组件在渲染时排队进行状态更新，React会立即应用状态更新，并同步重新渲染这一个组件，然后再继续渲染。如果组件无限的持续排队状态更新并且强制React重新渲染它，React将在尝试一定次数之后（当前是50次）打破这个循环，并且抛出一个错误。这个技术用于在状态值或者属性值发生改变之后强制更新，而不需要重新渲染和在 `useEffect` 中调用 `setSomeState()`。



## 3. 提升渲染性能（Imporving Rendering Performance）

尽管渲染是React运作的正常部分，但不可否认的是，有时渲染是无用功。如果一个组件的渲染输出结果没有发生改变，其对应的DOM也不需要更新，则渲染该组件的工作量本质上是在浪费时间。

React组件渲染输出结果应该完全基于当前属性和组件当前状态。因此，如果我们能够提前知道组件的属性和状态没有发生改变，我们就能知道其渲染输出结果是一样的，就没有必要对组件进行更新，因而可以安全的跳过这次渲染，

提升软件性能的常见的2个基本方式是：

1. 同样的工作量，做得更快些
2. 做更少的工作量

而优化组件渲染是第2种方式，通过在合适的时候跳过渲染组件从而减少工作量。



### 3.1 组件渲染优化技术（Component Render Optimization Techniques）

React提供了3个主要的APIs用来跳过渲染组件的可能：

1. **`React.Component.shouldComponentUpdate`**: 一个可选的类组件生命周期，它会在渲染过程的前期被调用。如果返回的是 `false`，则React跳过该组件的渲染。该生命周期可以包含任何用于计算，从而返回布尔值逻辑的代码，**但最常见的方式是，检测组件的属性和状态是否发生了变化。**
2. **`React.PureComponent`**: 因为组件属性和状态的比较是最常见的一种手段来实现 `shouldComponentUpdate`, `PureComponent` 默认的实现了该行为，用于简化 `Component + shouldComponentUpdate` 的方式
3. **`React.useMemo()`**: 一个内置的 [高阶组件](https://reactjs.org/docs/higher-order-components.html) 类型。它将你当前组件作为参数，返回一个包装后的组件。这个包装组件的默认行为就是去检测是否有属性发生了变化，如果没有，则会阻止该组件的重渲染，函数组件和类组件都可以使用 `React.useMemo()` 进行包装。（该方法还可以穿入一个自定义的比较回调函数，但是只能比较旧的和新的属性，因此传入的回调函数，一般用于比较特定的属性，而不是组件中全部的属性）

上面的3种方式的比较技术都是 **浅比较(`shallow equality`)**，表示比较对象中的每个字段，查看是否有不同的。例如：`obj1.a === obj2.a && obj1.b === obj2.b && ...`。这个比较过程一般很快，因为对JS引擎来讲， `===` 比较是很简单的。因此，这3种方式都等价于 **`const shouldRender = !shallowEqual(newProps, prevProps)`**。

还有一种鲜为人知的技术：**如果一个React组件在渲染输出结果中返回和上次完全一样的元素引用，React将跳过重新渲染该特定的子元素**。 有以下几种方式实现这种比较技术：

1. 如果在渲染输出结果中包含 `props.children`, 如果该组件只是状态发生了变化，则 `props.children` 子组件则不会发生变化
2. 如果使用 `useMemo()` 包装了某些元素，在依赖发生改变前，它们会保持为相同元素

例如：

```js
// 如果只更新状态 `props.children` 内容不会重渲染
function SomeProvider({children}) {
  const [counter, setCounter] = useState(0)
  
  return (
  	<div>
    	<button onClick={() => setCounter(counter + 1)}>Count: {counter1}</button>
			<OtherChildComponent />
    	{children}
    </div>
  )
}

function OptimizedParent() {
  const [counter1, setCounter1] = useState(0)
  const [counter2, setCounter2] = useState(0)
  
  const memoizedElement = useMemo(() => {
    // 只有counter1 依赖发生变化时，该组件才会重渲染
    // 状态 counter2 变化 不会触发该组件的重渲染
    return <ExpensiveChildComponent />
  }, [counter1])
  
  return (
    <div>
    	<button onClick={() => setCounter1(counter1 + 1)}>Count: {counter1}</button>
			<button onClick={() => setCounter2(counter2 + 1)}>Count: {counter2}</button>
			{memoizedElement}
    </div>
  )
}
```

上面提到的这些技术，如果跳过对组件的渲染， 同时也会跳过该子组件树的渲染，因为这会默认阻止 **递归渲染子元素** 的行为。



### 3.2 新的属性引用如何影响渲染优化（How New Props References Affect Render Optimizations）

之前我们已将看到了，**默认情况下，嵌套的子组件不管其属性有没有发生变化，React都会对其进行重渲染**。这也意味着，以属性的形式向子组件传递一个新的引用是没有关系的，因为你即使传入相同的属性，子组件也会被重渲染。因此，下面这种方式是完全没有问题的：

```js
function ParentComponent() {
  const onClick = () => {
    console.log(`Button Clicked`)
  }
  
  const data = { a: 1, b: 2 }
  
  return <NormalChildComponent onClick={onClick} data={data} />
}
```

每次 `ParentComponent` 渲染时，都会产生一个新的 `onClick` 函数引用和一个新的 `data` 对象引用，然后以属性的方式传递给 `NormalChildComponent`。（无论使用 `function` 还是 箭头函数的方式定义 `onClick` 都没关系，因为它始终是一个新的函数引用）。

**这也意味着，尝试通过 `React.useMemo()` 对宿主组件(hosting components),比如`<div> | <button>`等，进行渲染优化是没有意义的。因为这些基本元素，底层是没有子元素的，因此渲染过程到此，始终是停止的。**

然而，如果子组件是通过检测属性是否发生改变来优化渲染，而你此时传入一个新的引用作为属性，将势必导致子组件的渲染。如果新的属性引用是新的数据，而没什么问题，但是，如果父组件只是传入一个回调函数呢？

```js
const MemoizedChildComponent = React.memo(ChildComponent)

function ParentComponent() {
  const onClick = () => {
    console.log(`Button Clicked`)
  }
  
  const data = { a: 1, b: 2 }
  
  return <MemoizedChildComponent onClick={onClick} data={data} />
}
```

现在 `ParentComponent` 的每次渲染，这些新的引用都将导致 `MemoizedChildComponent` 发现属性值变成了新的引用值，从而导致其重渲染，即使 `onClick & data` 每次都是一样的。

这意味着：

1. 即使我们大多数时间想跳过重渲染 `MemoizedChildComponent` ，但是它还是会进行重渲染
2. `React.useMemo()`  中比较新旧属性的方式完全是徒劳

同样的，渲染 `<MemoizedChild><OtherComponent /></MemoizedChild>` 也会因 `props.children` 每次都是新的引用，导致强使子组件总是重渲染。



### 3.3 优化属性引用（Optimizing Props References）

类组件不用过多的担心意外的场景新的回调函数引用，因为类组件的实例方法总是返回相同引用。然而，类组件中可能需要对列表项生产为独一无二的回调，或者在匿名函数中捕获一个值，然后再传给列表项子组件，**这同样会产生新的引用**，渲染时也会创建新的子组件属性对象。React内部没有提供任何优化这种情况的方案。

对函数组件而言，React提供了2个hooks来复用相同的引用:

1. `useMemo()`：用于任何种类的一般数据，比如创建对象或者做复杂的计算工作
2. **`useCallback()`**: 专门用于创建回调函数



### 3.4 记忆一切？（Memoize Everything？）

正如上面提到的，你没有必要对每一个以属性形式向下传递的函数或对象使用 `useMemo & useCallback` - **只有当它会对子组件的行为产生影响时才这样做**。（也就是说， `useEffect` 的依赖数组比较确实增加了另一个用例，即子组件可能希望接收到相同的属性引用，这确实让事情变得更复杂了）

那么另一个问题总是被提起： **React为什么不默认将所有的组件都是用 `React.useMemo()` 进行包装呢？**

Dan Abramov 多次指出 [记忆会产生比较属性的开销](https://twitter.com/dan_abramov/status/1095661142477811717)， 并且很多时候，记忆一切检测也阻止不了重渲染，因为组件总是接收新的属性。例如：[Dan Twitter](https://twitter.com/dan_abramov/status/1083897065263034368)

> 为什么React不默认给每个组件都用 memo() 包装一下呢？这样不是会更快吗？
>
> 回答：
>
> 为什么不对每个函数都是用Lodash的 memoize() ？ 这样不会使函数更快吗？

另外，如果默认所有的组件都是用 memo() 可能会因为大家改变数据而不是不可变的更新数据导致bug。

我个人认为，大面积的使用 `React.memo()` 很可能会再整体应用的渲染性能中获得净收益：

>React社区作为一个整体似乎对 "perf "过于痴迷，然而很多讨论都是围绕着通过Medium帖子和Twitter评论流传下来的过时的 "部落智慧"，而不是基于具体的用法。
>
>对于 "render "的概念和 perf 的影响，肯定存在集体误解。是的，React完全是基于渲染--必须渲染才能做任何事情。不，大多数渲染并不太贵。
>
>"浪费 "的重渲染当然不是世界末日。从根部重新渲染整个应用也不是。也就是说，在没有DOM更新的情况下，"浪费 "的重渲染也确实是不需要消耗的CPU周期。这对大多数应用来说是个问题吗？可能不是。它是可以改进的吗？可能吧。
>
>是否有一些应用，默认的 "reerender it all "方法并不充分？当然，这就是sCU、PureComponent和memo()存在的原因。
>
>用户是否应该默认将所有的东西都包在memo()中？也许不应该，如果只是因为你应该考虑一下你的应用的 perf 需求。如果你这样做，实际上会有伤害吗？不会，而且现实中我预计它确实有净收益（尽管Dan提出了浪费比较的观点）。
>
>基准是否存在缺陷，结果是否会因场景和应用的不同而变化很大？当然是的。话虽如此，但如果人们能够开始指出这些讨论的硬性数字，而不是玩 "我曾经看到过一个评论...... "的电话游戏，那将是非常非常有帮助的。
>
>我很想看到React团队和更大的社区提供一堆基准套件，来衡量一堆方案，这样我们就可以一劳永逸地停止对这些东西的争论。功能创建、渲染成本、优化......。请提供具体的证据!



### 3.5 不可变性和重渲染（Immutability and Rerendering）

React中的状态更新必须是不可变的，有以下2个主要原因：

1. 根据你改变了什么和在哪里改变了，可能导致当你期望改变时，组件没有重渲染
2. 可能导致数据什么时候更新和为什么数据更新了的困惑

看具体的事例：

正如之前提到的，`React.shouldComponentUpdate & React.PureComponent & React.memo()` 都依赖当前属性和先前属性的浅比较。我们可以通过 `props.someValue !== prevProps.someValue` 知道一个属性是否是一个新的值，

如果你直接改变，`someValue` 是同一个引用，则组件会认为没有发生变化。

请注意，这是专门针对我们试图通过避免不必要的重新渲染来优化性能的时候。如果属性没有改变，渲染就是 "不必要的 "或 "浪费的"。如果你突变了，组件可能会错误地认为什么都没变，然后你就会想为什么组件没有重新渲染。

另一个问题就是 `useState & useReducer` hooks。每次调用 `setCounter() | dispatch()` 时，React会

排队重新渲染。然而，React要求任何通过hook更新状态必须传入或者返回一个新的引用作为新的状态值，它要么是一个新的对象或数组引用，又或者是一个新的基本类型（string｜number等）。

React在渲染阶段应用所有的状态更新。当React尝试应用一个来自hook的状态更新时，它会查看新的值是否是同一个引用。React总是会完成渲染排队更新的组件。但是，如果这个值和之前的引用是一样的，而且没有其他理由继续渲染（比如父组件已经渲染了），React就会扔掉组件的渲染结果，完全退出渲染传递。所以，如果我对一个数组进行这样的突变：

```js
const [todos, setTodos] = useState(someTodosArray)

const onClick = () => {
  todos[3].completed = true
  setTodos(todos)
}
```

则会导致组件重渲染失败。

技术上讲，只有最外层的引用必须是不可变更新，如果我们将上面的例子改为：

```js
const [todos, setTodos] = useState(someTodosArray)

const onClick = () => {
  const newTodos = todos.slice()
  newTodos[3].completed = true
  setTodos(newTodos)
}
```

则我们曾经了一个新的数组引用，组件将重渲染。

在突变和重渲染上，类组件的 `this.setState()` 和函数组件的 `setState & useReducer` hooks在行为上有个显著的区别： `this.setState()` 不在乎你是否进行了突变，它总是会完成重渲染工作。因此，下面的代码将重渲染：

```js
const { todos } = this.state
todos[3].completed = true
this.setState({todos})
```

事实上，我们传入一个空的对象也会导致重渲染，像这样 `this.setState({})`。

除了实际的渲染行为，突变对React单项数据流也会产生困惑。当预期什么都没有改变时，突变能够导致其他代码看到不同的值。这使得我们更难知道一个给定的状态究竟应该在什么时候和为什么更新，或者一个变化来自哪里。

底线： **React和其它React生态会假设不可变更新。任何时候你进行突变，都有产生bug的风险，因此不要这样做！**



### 3.6 测量React组件渲染性能（Measuring React Component Rendering Performance）

使用React DevTools Profiler来查看每次提交中哪些组件在渲染。找出那些渲染出乎意料的组件，使用DevTools找出它们渲染的原因，并进行修复（也许可以通过将它们包装在 `React.memo()` 中，或者让父组件对其传递下来的属性进行记忆）。

另外，请记住，React在开发构建中的运行速度要慢得多。你可以在开发模式下对你的应用进行剖析，看看哪些组件在渲染，以及为什么渲染，并做一些组件之间渲染所需相对时间的对比（"组件B在这次提交中的渲染时间是组件A的3倍"）。但是，千万不要用React开发构建来测量绝对的渲染时间--只能用生产构建来测量绝对时间! (否则Dan Abramov会因为你使用了不准确的数字而来骂你)。需要注意的是，如果你想真正使用剖析器从类似prod的构建中捕获时序数据，你需要使用React的特殊 "剖析 "构建。



## 4. 上下文和渲染行为（Context and Rendering Behavior）

**React的 `Context` API是一种使单个用户提供的值在其子树中也能获取的机制，** 任何在给定的 `<MyContext.Provider>` 内的组件，都能从上下文实例中读取值，不用显式的通过属性的形式进行值传递。

**Context不是一种状态管理工具！** 你必须自己管理传入到context中的值。这通常是通过在React组件状态下保存数据，并根据这些数据构建上下文值来实现的。



### 4.1 上下文基础（Context Basics）

一个 context provider 接收一个单一的 `value` 属性，例如 `<MyContext.Provider value={42}>`. 子组件通过渲染context consumer组件和提供渲染属性来使用context，比如：

```jsx
<MyContext.Consumer>{(value) => <div>{value}</div>}</MyContext.Consumer>
```

或者在函数组件中使用 `useContext`：

```js
const value = useContext(MyContext)
```

### 4.2 更新上下文的值（Updating Context Values）

当周围组件渲染provider时，React会检查一个context provider是否提供了一个新的值。如果provider值是一个新的引用，则React知道值发生了变化，则使用了该context的组件需要进行更新。

注意传入一个对象给context provider会导致其更新：

```js
function GrandchildComponent() {
  const value = useContext(MyContext)
  return <div>{value.a}</div>
}

function ChildComponent() {
  return <GrandchildComponent />
}
  
function ParentComponent() {
  const [a, setA] = useState(0)
  const [b, setB] = useState('text')
  
  const contextValue = {a, b}
  
  return (
  	<MyContext.Provider value={contextValue}>
    	<ChildComponent />
    </MyContext.Provider>
  )
}
```

在这个例子中， 每次 `ParentComponent` 渲染时，React会注意到每次 `MyContext.Provider` 都会给一个新的值，并在继续向下循环时寻找使用了 `MyContext` 的组件。当一个上下文提供者有了新的值，每一个使用了该上下文的嵌套组件都会被强制重新渲染。

注意，从React的角度来看，每个context provider都有一个单一值 - 不管它是一个对象，数组或是一个基本值，它都仅仅是一个context值。当前，没有办法使一个使用了context的组件因为新的context值的到来而不进行更新，即使该组件只使用了部分context值。



### 4.3 状态更新，上下文和重渲染（State Updates， Context and Re-Renders）

现在该将重点汇总一下了，我们已经知道：

1. 调用 `setState()` 会将渲染该组件加入队列
2. React默认的会递归渲染嵌套的组件
3. Context Provider被渲染它们的组件赋予一个value
4. 该value通常来自父组件的状态

这意味着：**默认情况下，渲染context provider的父组件中任何状态更新都会导致后代组件重新渲染，不管它是否读取了上下文的值**。

再看上面 `Parent -> Child -> Grandchild` 的例子，我们会发现 `GrandchildComponent` 都会重新渲染，不是因为context更新了，只是因为 `ChildComponent` 被重新渲染了。在这个例子中，并没有试图优化掉 "不必要 "的渲染，因此React在 `ParentComponent` 渲染时，都会默认的去渲染 `ChildComponent & GrandchildComponent`。如果父组件放入一个新的context value到 `MyContext.Provider` 中， `GrandchildComponent` 当它重渲染时可以看到到新的值并使用它， 但上下文更新并没有导致GrandchildComponent渲染--无论如何，它是会发生的。



### 4.4 上下文更新和渲染优化（Context Updates and Render Optimizations）

让我们修改一下这个例子，这样它确实会尝试优化，但我们会增加一个其他的变化，在底部放一个GreatGrandchildComponent：

```jsx
function GrandGrandchildComponent() {
	return <div>Hi</div>  
}

function GrandchildComponent() {
  const value = useContext(MyContext)
  return <div>
    {value.a}
    <GrandGrandchildComponent />
  </div>
}

function ChildComponent() {
  return <GrandchildComponent />
}

const MemoizedChildComponent = React.memo(ChildComponent)
  
function ParentComponent() {
  const [a, setA] = useState(0)
  const [b, setB] = useState('text')
  
  const contextValue = {a, b}
  
  return (
  	<MyContext.Provider value={contextValue}>
    	<MemoizedChildComponent />
    </MyContext.Provider>
  )
}
```

现在，如果我们调用 `setA(42)`:

1. `ParentComponent` 会渲染
2. 一个新的 `contextValue` 会被创建
3. React发现 `MyContext.Provider` 有一个新的值，因此任何使用了 `MyContext` 的使用者都需要进行更新
4. React将尝试渲染 `MemoizedChildComponent`, 但是发现其被 `React.memo()` 包裹。并且没有任何属性传入，因此属性没有实际上发生变化，React会完全的跳过 `ChildComponent` 的渲染
5. 然而，`MyContext.Provider` 时存在更新的，因此地下的组件可能需要知道这一点
6. React继续向下，到达 `GrandchildComponent`. React发现 `GrandchildComponent` 读取了 `MyContext` 中的值，因此它需要重新渲染。因为context发生了变化，因此React继续向前并重渲染 `GrandchildComponent`
7. 因为 `GrandchildComponent` 渲染了，React继续向前，并渲染其子组件，因此React也会重渲染 `GrandGrandchildComponent`

换句话说，正如 [Sophie Alpert](https://twitter.com/sophiebits/status/1228942768543686656) 所说:

> 在你的 Context.Provider下面的组件可能应该使用 React.memo

这样的话，父组件中的状态更新将不再强使每个组件进行重新渲染，只有那些读取了context的组件需要。（将 `ParentComponent` 渲染 `<MyContext.Provider>{props.children}</MyContext.Provider>` ,这样利用同样的元素引用技术避免子组件的重新渲染，然后再从上一层渲染 `<ParentComponent><ChildComponent /></ParentComponent>`， 可以获得同样的结果）

注意，一旦 `GrandchildComponent` 基于下一个context value渲染后，React将继续恢复到默认的递归重渲染方式，因此 `GrandGrandchildComponent` 被渲染了，任何嵌套的子组件也会被渲染。





## 5. React-Redux和渲染行为（React-Redux and Rendering Behavior）

有很多关于 **Context vs Redux** 的问题的讨论。

话说回来，人们在提到这个问题时经常指出的一个问题是 "React-Redux只重新渲染实际需要渲染的组件，所以这比上下文更好"。

这话有点道理，但答案要比这细微得多。



### 5.1 React-Redux订阅（React-Redux Supscriptions）

我看到很多人说 "React-Redux内部使用的是Context"。技术上说没错，但是 [React-Redux使用context传递Redux store实例，而不是当前状态值](https://blog.isquaredsoftware.com/2020/01/blogged-answers-react-redux-and-context-behavior/)。这意味着我们传入到 `<ReactReduxContext.Provider>` 中的值是相同的context value。

**记住，当一个action被dispatched后，Redux store将运行其所有的订阅通知回调函数**。 [需要使用Redux的UI层总是订阅Redux store， 在它们的订阅回调函数中读取最新的状态，比对数据，当数据发生变化时，强制重新渲染](https://blog.isquaredsoftware.com/2018/11/react-redux-history-implementation/)。 订阅回调处理完全脱离在React之外，只有当React-Redux知道某个React组件需要的数据发生了变化时（基于 `mapState | useSelector` 返回值），React才参与到其中。

这也导致了和context不同的性能特征组。是的，很可能会有更少的组件一直在渲染，但React-Redux每次更新存储状态时，总是要为整个组件树运行 `mapState/useSelector` 函数。大多数时候，运行这些selectors的花销要低于React重新渲染的成本，因此这通常是一种净收益，但这是必须要做的工作。然而，如果这些selectors正在进行昂贵的转换，或者在不该返回新值的时候意外地返回新值，那就会降低速度。





### 5.2 `connect` 和 `useSelector` 的差异

`connect` 是一个高阶组件。你可以传入自己的组件，`connect` 会返回一个包装组件，同时帮你订阅store，运行 `mapState & mapDispatch`，并将组合的属性传递到你的组件中去。

`connect` 组件总是表现的类似 `PureComponent | React.memo()`, 但是有轻微的不同重点：`connect` 会在传入你的组件中的组合发生变化时重新渲染。

一般来讲，最终组合的属性的形式是 `{...ownProps, ...stateProps, ...dispatchProps}`, 因此，任何来自父组件的新的属性引用都会导致组件渲染，这一点和 `React.PureComponent & React.memo()` 是一致的。除了父组件属性，[任何 `mapState` 返回的新的引用都会导致组件重渲染](https://react-redux.js.org/using-react-redux/connect-mapstate#mapstatetoprops-and-performance) （因为你可以自定义 `ownProps | stateProps | dispatchProps` 如何进行合并，所以有可能改变这一行为。）

`useSelector` 是一个在你的函数组件中被调用的hook。正因如此，在父组件重渲染时， `useSelector` 没有办法阻止你的组件渲染。

[这也是 `connect` 和 `useSelector` 性能上的差异点](https://react-redux.js.org/api/hooks#performance)。使用 `connect` 的组件类似于 `PureComponent`,  并充当阻止React默认级联向下渲染整棵组件树的防火墙。因为一般的应用有很多的 connected组件，这意味着重渲染被限定在了一个相当小的组件树区域。React-Redux会根据数据变化强制一个连接的组件进行渲染，它下面的2-3个组件可能也会进行渲染，然后React会碰到另一个不需要更新的connect组件，这样就会停止渲染级联。

此外，拥有更多connect的组件意味着每个组件都可能从store中读取较小的数据片段，因此在任何给定的操作后都不太可能需要重新渲染。

如果你只使用函数组件和 `useSelector`，那么你的组件树中很可能有更大的部分会根据Redux store的更新而重新渲染，而不同于使用connect，因为没有其他connect的组件来阻止这些渲染级联继续向下。

如果这成为性能问题，那么答案就是自己根据需要用 `React.memo()`包裹组件，以防止父组件造成不必要的重渲染。



## 6. 总结

1. React在父组件渲染后，总是默认的递归的渲染子组件
2. 渲染本身没有什么问题，这是React知晓哪些DOM发生改变必须的
3. 但是当UI根本没有发生改变时，渲染花费的时间将是一种浪费
4. 大多数时候，传递新引用，比如回调函数和对象是没有问题的
5. 如果属性没有发生变化，使用 `React.memo()` 可以跳过不必要的渲染
6. 但是如果你总是将新的引用作为属性传递，`React.memo()` 可能永远不会跳过渲染，因此你可能需要记忆这些值
7. Context使得任意层级的组件都可以轻松的访问value
8. Context Provider通过引用比较值来判断是否发生了改变
9. 一个新的context value会强制所有后代组件进行重渲染
10. 但是，由于正常的 `parent -> child` 级联过程，很多时候自带还是会重新渲染
11. 因此你可能想将context provider的子组件使用 `React.memo()` 进行包装，或使用 `{props.children}`， 从而当context value发生变化时，不必渲染整棵组件树
12. 当一个子组件因为新的context value重渲染时，React将继续渲染子组件内的嵌套组件
13. React-Redux使用订阅Redux store来检测更新，而不是通过context传递store value
14. 这些订阅者在每次Redux store更新后都会运行，因此它们必须尽可能的快
15. React-Redux做了很多工作，用来确保只有数据发生了变化的组件才进行强制重渲染
16. `connect` 类似于 `React.memo()`, 因此更多的connected组件，意味着更小的渲染次数
17. `useSelector` 是一个hook，因此它不能阻止因父组件重渲染导致的重渲染。应用中到处使用 `useSelector` ,可能需要给某些组件添加 `React.memo()` 来避免每次都要渲染



## 7. 最后补充说明

显然，整个情况要比单纯的 **"上下文让一切都渲染，Redux没有，使用Redux "** 复杂得多。不要误会我的意思，我希望人们使用Redux，但我也希望人们清楚地了解不同工具所涉及的行为和权衡，以便他们能够做出明智的决定，什么是最适合自己的用例。

由于每个人似乎总是问 "什么时候应该使用Context，什么时候应该使用(React-)Redux？"，让我继续总结一些标准的经验法则：

1. 使用Context：
   1. 当你需要传递一些不需要经常变化的简单值时
   2. 当应用中某些函数或属性需要到处使用，你又不想使用一路向下传递属性传递的方式时
   3. 你希望只使用React，不依赖别的库时
2. 使用(React-)Redux：
   1. 当应用中存在大量的应用状态时
   2. 当应用状态需要频繁的更新时
   3. 更新状态的逻辑很复杂时
   4. 当应用的代码量比较大，多人协作时



请注意，这些并不是硬性的、排他性的规则--它们只是一些建议的指导方针，说明这些工具在什么情况下可能是有意义的。一如既往，请花些时间自己决定什么是你所处理的任何情况下的最佳工具。

总的来说，希望这个解释能帮助大家了解在各种情况下React的渲染行为的实际情况。



## 8. 进一步阅读

通用：

- [Dava Ceddia: A Visual Guide to References in Javascript](https://daveceddia.com/javascript-references/)

React渲染行为：

- [React Docs - Reconciliation](https://reactjs.org/docs/reconciliation.html)
- [React lifecycle methods diagram](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/)
- [React hooks lifecycle diagram](https://github.com/donavon/hook-flow)
- [Provide more ways to bail out inside Hooks ](https://github.com/facebook/react/issues/14110)
- [React issues: why setState is async](https://github.com/facebook/react/issues/11527#issuecomment-360199710)
- [Context is good for low-frequency updates, not Flux-like state propagation](https://github.com/facebook/react/issues/14110#issuecomment-448074060)
- [React, Inline Functions, and Performance](https://cdb.reacttraining.com/react-inline-functions-and-performance-bdff784f5578)
- [Avoiding unnecessary renders with React context](https://frontarm.com/james-k-nelson/react-context-performance/)

优化渲染性能：

- [React Docs : Optimizing Performance](https://reactjs.org/docs/optimizing-performance.html)
- [Kent C.Dodds: Fix the slow render before you fix the re-render](https://kentcdodds.com/blog/fix-the-slow-render-before-you-fix-the-re-render/)
- [Kent C.Dodds: When to useMemo and useCallback](https://kentcdodds.com/blog/usememo-and-usecallback)
- [Kent C.Dodds: One Simple trick to optimize React re-renders](https://kentcdodds.com/blog/optimize-react-re-renders)
- [React issues: When should yout Not use React memo?](https://github.com/facebook/react/issues/14463)

分析React组件：

- [React Docs: Introducing the React Profiler](https://reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html)
- [React DevTools profiler interactive tutorial](https://react-devtools-tutorial.now.sh/)
- [Kent C.Dodds: Profile a React App for Performance](https://kentcdodds.com/blog/profile-a-react-app-for-performance)
- [Using the React DevTools Profiler to Diagnose React App Performance Issues](https://www.netlify.com/blog/2018/08/29/using-the-react-devtools-profiler-to-diagnose-react-app-performance-issues/)
- [Use the React Profiler for Performance](https://scotch.io/tutorials/use-the-react-profiler-for-performance)

React-Redux 性能：

- [Practical Redux, Part 6: Connected Lists, Forms, and Performance](https://blog.isquaredsoftware.com/2017/01/practical-redux-part-6-connected-lists-forms-and-performance/)
- [Idiomatic Redux: The History and Implementation of React-Redux](https://blog.isquaredsoftware.com/2018/11/react-redux-history-implementation/)
- [React-Redux docs: mapState Usage Guide - Performance](https://react-redux.js.org/using-react-redux/connect-mapstate#mapstatetoprops-and-performance)
- [High Performance Redux](http://somebody32.github.io/high-performance-redux/)
- [react-redux-links - github](https://github.com/markerikson/react-redux-links/blob/master/react-performance.md)



原文链接：

- [Mark's Dev Blog - A Mostly Complete Guide to React Rendering Behavior](https://blog.isquaredsoftware.com/2020/05/blogged-answers-a-mostly-complete-guide-to-react-rendering-behavior/#improving-rendering-performance)

2021-03-10 09:15:05

