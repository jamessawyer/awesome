深入理解React，主要从以下几个方面：

1. React组件和React元素的关系和区别
2. 元素的数据结构
3. React生命周期
4. React Fiber架构
5. 元素是如何转换成Fiber节点的，Fiber数据结构
6. Fiber树：current树 和 workInProgress 树
7. 首次渲染和更新
8. 渲染阶段和提交阶段的具体流程
9. `effectTag` 类型和副作用链表(`effects list`)
10. 属性和状态更新的具体流程
11. 调度器的基本原理
12. React源码解读



有以下比较好的资源：（🀄️ 表示中文学习资料）

1. 来自 [indepth.dev](https://indepth.dev/react) 有关React深度学习的几篇好文章
   1. [Inside Fiber: in-depth overview of the new reconciliation algorithm in React - Max Koretskyi@In-depth-dev](https://indepth.dev/posts/1008/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react) 已翻译✅
   2. [The how and why on React’s usage of linked list in Fiber to walk the component’s tree - Max Koretskyi @in-depth-dev ](https://indepth.dev/posts/1007/the-how-and-why-on-reacts-usage-of-linked-list-in-fiber-to-walk-the-components-tree) 已翻译✅
   3. [In-depth explanation of state and props update in React - Max Koretskyi@in-depth-dev](https://indepth.dev/posts/1009/in-depth-explanation-of-state-and-props-update-in-react) 已翻译✅

2. 来自 [Mark's Dev Blog](https://blog.isquaredsoftware.com/) 的有关React渲染的文章
   1. [Mark's Dev Blog - A Mostly Complete Guide to React Rendering Behavior](https://blog.isquaredsoftware.com/2020/05/blogged-answers-a-mostly-complete-guide-to-react-rendering-behavior/#improving-rendering-performance) 已翻译✅ React渲染指南

3. 🀄️ [魔术师卡颂](https://space.bilibili.com/453618117) 的系列文章，推荐学习 
   1. [React技术解密](https://react.iamkasong.com/)
   2. [魔术师卡颂 - b站视频](https://space.bilibili.com/453618117) 

4. 🀄️ [React-guidebook 关于Fiber和调度器部分](https://tsejx.github.io/react-guidebook/architect/internal/fiber)

5. [pomb - build your own React](https://pomb.us/build-your-own-react/)：如果自己创建一个React库

6. 🀄️ [剖析 React 内部运行机制 - 上古鹏@慕课网](https://www.imooc.com/read/86)：这个是收费的，写得比较好，由浅入深

7. [Fiber Principles- sebmarkbage@github](https://github.com/facebook/react/issues/7942?source=post_page---------------------------#issue-182373497)

8. [React Fiber Architecture - @github](https://github.com/acdlite/react-fiber-architecture?source=post_page---------------------------)

9. [Lin Clark - A Cartoon Intro to Fiber @youtube](https://www.youtube.com/watch?v=ZCuYPiUIONs&ab_channel=FacebookDevelopers)

10. 🀄️ [一文读懂 React 组件渲染核心原理 - Tecvan](https://mp.weixin.qq.com/s/M6orAXsSXDSKouIyuC9XUg)

11. [🚀🀄️ react进阶时间指南 - 我不是外星人@掘金小册](https://juejin.cn/book/6945998773818490884/section/6951186955321376775) 很棒的进阶文章

12. [🚀🚀🀄️ 全网最简单的React Hooks源码解析，看不懂，打我 - React中文社区@wx](https://mp.weixin.qq.com/s/4-JYjizitK-VbRk5CQqlKA) 很详细的介绍了 `useState & useReducer & useEffect` mount & update的处理逻辑
    - React如何管理区分Hooks: 每个hook都会创建一个 `Hook` 对象，React通过单链表依次将 `Hook` 添加到链表中，这也是为什么不能使用条件语句的原因
    - useState & useReducer 更新时，将 `dispatch` 函数添加到 `queue` 中，这个 `queue` 是一个 **循环链表**
    - `useEffect` 分为 `mountEffect & updateEffect`, mount阶段也是通过 **循环链表** 挂载所有的effect；更新阶段，通过对比依赖，如果没有更新，则打上 `NoHookEffect` flag, commit阶段会跳过该Effect
    - useState & useReducer 的数据将挂载在 `FiberNode` 的 `memoizedState` 属性上；useEffect则挂载在 `FiberNode` 上的 `updateQueue` 上

13. [🚀🀄️ React合成事件 - React中文社区@wx](https://mp.weixin.qq.com/s/A1oV--p5NpiKjtdFv9ODBQ)

    - 合成事件的原因：抹平浏览器差异；对事件进行封装，对不同时间提供不同 **优先级**，便于调度阶段，执行时机的差异

    - 合成事件的时间点：`commit` 阶段将包含事件的prop的对应的fiber节点绑定到 `root` 上

    - 事件优先级：事件根据是否是离散事件（比如点击），持续事件（比如touch,mousemove）来判断优先级，可参考 [getEventPriority](https://github.com/facebook/react/blob/main/packages/react-dom/src/events/ReactDOMEventListener.js#L409)

    - 事件对象的合成和事件的收集：

      - 合成：通过插件的形式，使用 `SyntheticEvent` 构造函数，封装原生事件，

      - 收集：因为事件有 **冒泡和捕获** 2种形式，收集会从当前节点开始，向上对 **相同类型事件** 进行收集（比如所有的 `onClick` 事件），收集到 `dispatchQueue` 的 `listeners` 中，最终得到的dispatchQueue结构如下：

        ```js
        [
          {
            // 合成事件对象 这个对象对于listeners是共享的
            // listeners中的listener可以改变event中的 currentTarget
            // 而event中的 stopPropagation 可阻止事件的冒泡
            event: SyntheticEvent,
            // listeners 是从 current -> root 向上收集 同类事件
            listeners: [listener1, listener2], // 事件收集路径
          }
        ]
        ```

    - 事件的执行：根据合成事件对象中 `IS_CAPTURE_PHASE` 来判断事件是捕获还是冒泡，

      - 如果是事件捕获，则会从 `listeners` 后面开始执行，每次执行时都会判断是有存在 `isPropagationStopped()` 来终止事件的继续捕获
      - 如果是事件冒泡，则会从 `listeners` 最左边开始执行

14. [🀄️ 阿里三面：灵魂拷问——有react fiber，为什么不需要vue fiber？ - 小李的前端小屋@wx](https://mp.weixin.qq.com/s/iM2RKUNHPZPPSwsYR6WkUA) 对React遍历和遍历完成顺序解释的很清楚

    - 对比React & Vue响应式的差异：React依赖Fiber树结构；Vue则使用Proxy对数据劫持
    - React & Vue重渲染的差异：React必须从Fiber树头到尾进行遍历，找出更新的Fiber节点，并形成Effect List;Vue则利用数据劫持，给组件添加监视器的形式
    - React 新的Fiber数据结构，能中断渲染的原理：Fiber节点持有 `return & child & sibling` 链表数据结构，能够从中断的位置恢复
    - React使用polyfill的 `requestIdleCallback` 函数，使用调度的方式，每次时间到了之后，就会让出渲染的控制权，将时间片交给更紧急的任务，这也是React能够中断渲染的原因
    - **React遍历的顺序**：深度优先算法。遍历顺序是从顶至下，遍历完成顺序则是最底下没有子节点的，然后兄弟节点，没有兄弟节点则返回完成父节点，然后再兄弟节点的形式。

15. [👍 Why React Re-Renders - Josh Comeau](https://www.joshwcomeau.com/react/why-react-re-renders/) 

    - 很基础，并通俗易懂的一篇文章
    - React重新渲染的唯一触发条件：`状态改变`
    - 组件状态更改会导致组件和后代组件（如果存在的话）重新渲染，因为单向数据流，父组件是不会受到影响的
    - 后代组件的重渲染，和属性没有关系，因为组件默认对整颗子树进行重渲染
    - `React.memo` 和 `React.PureComponent` 的作用在于，告诉React该组件是一个纯组件，即使父组件的状态变化，而传入的属性(`props`)没有发生变化的话，就不需要重渲染该组件了
    - 可以把 `Context` 理解为一种 `不可见` | `内部的` 依赖，但状态发生变化时，也会触发使用到了Context的组件的重渲染
    - 😎如何使用 React DevTools 的 `Profiler` 工具定位组件重渲染的原因，以及高亮重渲染组件

16. [👍 Understanding useMemo and useCallback](https://www.joshwcomeau.com/react/usememo-and-usecallback/) 终于有一篇文章能将 `useMemo` & `useCallback` 的作用已经使用场景写得这么直白了🎉

    - `useMemo` & `useCallback` 都可用于 `记忆` 对象，`useMemo` 用于记忆 `arrays & objects`; 而 `useCallback` 用于记忆 `functions`（函数也是对象）

    - `useCallback` 是 `useMemo` 的语法糖，底层实际也是使用的 `useMemo`

    - `React.memo` vs `useMemo` : `React.memo` 是对整个组件进行包裹，对传入的属性进行比较；`useMemo` 对组件内局部对象进行记忆

    - React性能优化的2个方式

      - 减少每次渲染的工作量
      - 减少组件渲染的次数

    - `useMemo` 用于记忆对象，可以用于减少每次渲染的工作量。比如，组件内存在2个状态：`A` & `B`，其中 `B` 比较耗时，这时就可能 `B` 进行记忆，这样`A`发生变化时，不用重新去计算 `B`

      - 但是我们不一定需要使用 `useMoemo` ，我们可以通过 **状态分离** 的方式，将包含 `A` 状态部分写成一个独立的组件 `A`，包含 `B` 状态部分也单独写成一个独立的组件 `B`，使用 `React.memo` 对整个组件进行包裹；这样，`A` 组件的变化，就不会再影响到 `B` 组件了

    - `useMemo` 的另一个作用就是，对对象进行记忆，例如：每次 `App` 组件因为 `name`状态发生变化重新渲染时，都会生成一个全新的对象 `boxes`，因为对象属性通过引用比较，这导致每次传入 `Boxes` 组件的属性都是一个新的对象，因此 `Boxes` 组件每次都会重新渲染🚀

      ```jsx
      import React from 'react';
      import Boxes from './Boxes';
      
      function App() {
        const [name, setName] = React.useState('');
        const [boxWidth, setBoxWidth] = React.useState(1);
        
        const id = React.useId();
        
        // 🚨 改变 `name` 状态，App都会重渲染，每次都生成一个新的boxes对象
        // 从而导致 Boxes 组件每次接收到的对象属性都是不同的属性，继而导致 Boxes 组件的重新渲染
        const boxes = [
          { flex: boxWidth, background: 'hsl(345deg 100% 50%)' },
          { flex: 3, background: 'hsl(260deg 100% 40%)' },
          { flex: 1, background: 'hsl(50deg 100% 60%)' },
        ];
        
        return (
          <>
            <Boxes boxes={boxes} />
            
            <section>
              <label htmlFor={`${id}-name`}>
                Name:
              </label>
              <input
                id={`${id}-name`}
                type="text"
                value={name}
                onChange={(event) => {
                  setName(event.target.value);
                }}
              />
              <label htmlFor={`${id}-box-width`}>
                First box width:
              </label>
              <input
                id={`${id}-box-width`}
                type="range"
                min={1}
                max={5}
                step={0.01}
                value={boxWidth}
                onChange={(event) => {
                  setBoxWidth(Number(event.target.value));
                }}
              />
            </section>
          </>
        );
      }
      
      ```

      改进版本：

      ```jsx
      import React from 'react';
      
      import Boxes from './Boxes';
      
      function App() {
        const [name, setName] = React.useState('');
        const [boxWidth, setBoxWidth] = React.useState(1);
        
        const id = React.useId();
        
        // 🎉 使用 useMemo 对对象进行缓存，这样App每次重渲染时，都使用缓存的对象
        const boxes = useMemo(() => ([
          { flex: boxWidth, background: 'hsl(345deg 100% 50%)' },
          { flex: 3, background: 'hsl(260deg 100% 40%)' },
          { flex: 1, background: 'hsl(50deg 100% 60%)' },
        ]), [boxWidth]);
        
        return (
          <>
            <Boxes boxes={boxes} />
            
            <section>
              <label htmlFor={`${id}-name`}>
                Name:
              </label>
              <input
                id={`${id}-name`}
                type="text"
                value={name}
                onChange={(event) => {
                  setName(event.target.value);
                }}
              />
              <label htmlFor={`${id}-box-width`}>
                First box width:
              </label>
              <input
                id={`${id}-box-width`}
                type="range"
                min={1}
                max={5}
                step={0.01}
                value={boxWidth}
                onChange={(event) => {
                  setBoxWidth(Number(event.target.value));
                }}
              />
            </section>
          </>
        );
      }
      ```

    - `useCallback` 则是对 `函数对象` 进行缓存，它是 `useMemo` 的语法糖

      ```jsx
      // This: 直接使用函数
      React.useCallback(function helloWorld(){}, []);
      
      // 等价于 返回一个函数对象
      // ...Is functionally equivalent to this:
      React.useMemo(() => function helloWorld(){}, []);
      ```

    - 👩‍🏫 但是要记住的是，不要对性能过度的优化，因为React已经做的很好了，因此这2个hooks的使用场景

      - 通用的自定义hooks中，因为自定义的hooks可能被使用很多次，进行记忆缓存有可能带来性能的提升
      - `Context Provider` 中，因为可能有很多纯函数消费该context，使用 `useMemo` | `useCallback` 会减少纯组件的渲染可能性

    

    

    

    

    

    

    

辅助链接：

1. [React lifecycle methods diagram](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/): React生命周期图表📈



react内部实现：

1. [react ref 为什么更新前要先设置为null，然后再赋值 - @github issues](https://github.com/facebook/react/issues/9328#issuecomment-298438237)
2. [⭐️ reactjs/rfcs - @github](https://github.com/reactjs/rfcs) react功能讨论区，对很多react功能设计的缘由以及用法





视频资源：

1. [⭐️⭐️⭐️⭐️⭐️How Does React Actually Work - Philip Fabianek@youtube](https://www.youtube.com/watch?v=7YhdqIR2Yzo&list=PLxRVWC-K96b0ktvhd16l3xA6gncuGP7gJ&index=1) 强烈推荐这一系列视频
   - React组件，元素和实例区别；
   - 简要的介绍Fiber架构，如何理解Fiber，React渲染的不同阶段
   - `useState` & `useEffect` 的基本原理，解释为什么不能将其放在条件语句和for循环中
   - 解释React的更新过程，生命周期的调用时机和作用



上面的链接中包含大量外部链接，都是不错的资源，对深度学习React是非常有帮助的
