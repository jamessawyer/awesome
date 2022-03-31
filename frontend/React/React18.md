## 学习资料
1. [what's new in React 18 - evening kid@youtube](https://www.youtube.com/watch?v=bpVRWrrfM1M&t=1s&ab_channel=eveningkid)
   - 简明的讲解了React18批量更新以及 `flushSync`
   - `Suspense` 的变化 & `startTransition` 手动定义任务优先级，避免动画卡顿等问题
   - react18对SSR 并行模式，可以使用部分hydrate，让需要交互部分组件优先hydrate
2. [Concurrent Rendering Adventures in React 18 - mauricedb@bilibili](https://www.bilibili.com/video/BV1FR4y1b7Qx?p=1)
   - 先讲了react17中的 `Suspense` & `ErrorBoundary` 的使用，以及结合 `swr` 一起使用
   - 嵌套Suspense的用法
   - 然后就是React18相关内容：`SuspenseList` & `useTransition` & `startTransition` & `unstable_useOpaqueIdentifier` & `useDeferredValue`
     - `SuspenseList` 用于编排多个 `Suspense`
     - `startTransition` & `useTransition` 用于定义优先级比较低的任务，避免对主任务造成卡顿的现象
     - `unstable_useOpaqueIdentifier` 用于生成不同的id
     - `useDeferredValue` 类似debouce效果
3. [🀄️React18面试 - bucuocuo@bilibili](https://www.bilibili.com/video/BV1rK4y137D3?p=1) 这个是一个中文视频，基本和上面的2的内容差不多
   - [React18正式版源码级剖析](https://juejin.cn/post/7080854114141208612) 其实就是React18的一些新特性的简单介绍，但是文章中有一些特性类比还是不错的