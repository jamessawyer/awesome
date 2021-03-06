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

辅助链接：

1. [React lifecycle methods diagram](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/): React生命周期图表📈



上面的链接中包含大量外部链接，都是不错的资源，对深度学习React是非常有帮助的