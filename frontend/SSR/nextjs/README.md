Next.js 相关的资料或视频



## 👩‍🎓‍ 基础

1. [🔥🔥 rendering patterns - @lydiahallie.io](https://www.lydiahallie.io/talks/rendering-patterns) 以 `Next.js` 作为示例介绍了多种渲染模式，很好的一篇文章
   1. 对应 [Reactathon 2022 - @youtube](https://www.youtube.com/watch?v=V5hPAl1q7vo&t=3563s)
   2. 介绍最常见的几种渲染模式： `SSG`, `Static with Client-Side fetch`, `ISR & On-Demand ISR` , `SSR`, `Streaming SSR`， 结合视频演示，每个过程都很清晰
   3. 不同渲染模式对应的不同 `Core Web Vitals`: `TTFB & FCP & LCP & TTI & CLS & FID` 等指标（通过 `Static with Client-Side Fetch` 演示，很明显的区分 `FCP & LCP` 的区别）
   4. 选取渲染模式时，考虑的其他因素： `Fast Build Times`（快的构建时间） & `Low Server Costs`（低服务器成本） & `Easy Rollbacks`（方便的回滚） & `Reliable uptime` （可靠的更新，避免数据滞后）& `Dynamic Content`（是否支持动态内容） & `Scalable infrastructure`（可扩展的基础设施）
   5. 选取渲染模式策略：
      - `SSG`: 不需要请求外部数据；缺点：只能针对静态内容
      - `Static with Client-Side fetch`: 客户端加载后请求数据 & 稳定的占位组件；缺点：LCP, CLS等指标不是很好
      - `ISR`(Incremental Static Regeneration): 对部分页面进行预渲染，其它页面动态生成，按照一定时间间隔对数据进行生成 或者 按需生成；缺点：如果以一定间隔进行revalidate，可能导致服务端资源的浪费
      - `On-Demand ISR`: 基于特定的事件生成（比如 webhooks）；最理想的一种方案，缺点：无法生成个性化内容
      - `SSR`: 动态数据，个性化数据，比如用户设置，dashboard，需要根据session或者token返回的数据；缺点：TTFB, FCP, LCP, TTI， FID等指标都很高，另外需要更好的服务器资源
      - `Streaming SSR`: 使用 React18 提出的 `React Server Component` 的概念，选择性水合，按需水合，优先级水合，服务端渲染组件等





## :books: 资料

1. [🚀 Next.js相关内容 - 基础知识 - 进阶知识 - 最佳实践 - 前端周公子@掘金](https://juejin.cn/column/6968345399929077790) 合集包含项目实战中碰到的问题
2. [🔥 手把手教你koa2 + nextjs搭建自己的博客（带源码） - 雪王@掘金](https://juejin.cn/post/6931633501183836167) 项目实战，包含 `i18` 国际化， 可预览效果，包含 [源代码](https://github.com/fantasticit/wipi)
3. [next.js官方示例仓库 - @Github](https://github.com/vercel/next.js/tree/canary/examples)
4. [beta.reactjs.org 官方网站也是nextjs写的 - @Github](https://github.com/reactjs/reactjs.org/tree/main/beta)



## :tv: 视频

1. [How to remove loading spinners in Next.js - Sam Selikoff@youtube](https://www.youtube.com/watch?v=JHGoXMvzTY4&ab_channel=SamSelikoff) 介绍在next.js去掉spinner提升用户体验的2种方式，都利用到 `swr` 这个库进行数据获取
   - 1.鼠标悬停时 `prefetch`
   - 2.点击时不跳转，而是等数据完成后再跳转的方式，将spinner放置到标题处显示

