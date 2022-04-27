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
