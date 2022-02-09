服务端渲染相关文章：

1. [React SSR 详解【近 1W 字】+ 2个项目实战 - 秋天不落叶@掘金](https://juejin.cn/post/6844904017487724557) 这篇文章比较通俗易懂的解释了什么是ssr，ssr渲染流程，同构应用，注水（hydrate）等概念，配合对应的示例项目去理解
   - 对应的demo地址 [react-ssr-demo - @github](https://github.com/yjdjiayou/react-ssr-demo)
2. [⭐️⭐️⭐️New Suspense SSR Architecture in React 18 - gaearon@github](https://github.com/reactwg/react-18/discussions/37)
   - 介绍了什么是SSR，现有React中存在的问题
   - 服务端 `Suspense` 如何解决这些问题，选择性水合方案
   - streaming HTML 优先发送需要的内容
3. [vivo 商城架构升级-SSR 实战篇Nuxt.js - vivo@掘金](https://juejin.cn/post/6908883342034632712) 实战项目，介绍了ssr性能优化的一些手段，降级，ci/cd，监控，日志等等
4. [理解Vue SSR原理，搭建项目框架 - Alaso@掘金](https://juejin.cn/post/6950802238524620837) 这篇是使用vue的ssr，其基本原理和React类似





## 视频

1. [🚀server side rendering with react and redux - @bilibili](https://www.bilibili.com/video/BV194411t7aq?p=1) 这个是搬运的udemy上的课程，强烈推荐学习，从根本上讲解了SSR的原理
   - 同构应用路由配置
   - Redux服务端和客户端配置的差异
   - 服务端渲染状态的同步，异步数据的获取
   - 使用 `serialize-javascript` 防止xss攻击
   - 权限验证处理

