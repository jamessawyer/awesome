前置知识：

1. [Web性能领域常见的专业术语 - 玖五@知乎](https://zhuanlan.zhihu.com/p/98880815)
   - `FP | FCP | FMP | LCP` 含义是什么
   - `TTI | TTFB | FCI | FID | DCL | L` 又表示什么含义
2. [🚀Web 性能优化-首屏和白屏时间 - lizhen](https://lz5z.com/Web%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96-%E9%A6%96%E5%B1%8F%E5%92%8C%E7%99%BD%E5%B1%8F%E6%97%B6%E9%97%B4/) 强烈推荐阅读
   - 详细的介绍了白屏时间和首屏时间的计算
   - `performance.timing` 各种时间点的具体含义
   - `DOMContentLoaded` vs `load` 事件的差异
   - 这篇文章是由这个面试题引申来的 ： [第 137 题：如何在 H5 和小程序项目中计算白屏时间和首屏时间，说说你的思路 - @github](https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/272)
3. [Web 性能优化-缓存-HTTP 缓存 - lizhen](https://lz5z.com/Web%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96-HTTP%E7%BC%93%E5%AD%98/) 
   - 缓存类型：强缓存 & 协商缓存
   - HTTP1.1协议缓存相关的headers: `Cache-Control` & `Last-Modified / If-Modified-Since` & `ETag / If-None-Match`
   - `ETag / If-None-Match` 相比  `Last-Modified / If-Modified-Since` 的优势



🎉 tricks & Tips：

1. [⭐️ Web Performance Trivia - Lydia Hallie@youtube](https://www.youtube.com/watch?v=8dB_TpSM8ak) 介绍几种常见的性能优化方式（顺便可以学一下chrome performance）
   - `<script>` & `<script async>`  & `<script defer>` 区别
     - `<script>` 阻塞式加载
     - `<script async>` parse html的同时，会去加载 script, 加载完成后，执行script的过程又会阻塞parse过程; 如果有多个 async 加载，先加载完成的会先执行，因此如果scripts执行的顺序有关系，则不能使用这种方式
     - `<script defer>` 在DOM创建完成之后执行，不会阻塞页面的parse
   - `<link rel="prefetch">` & `<link rel="preload">` 
     - `<link rel="prefetch">` 在后台获取资源，并将资源缓存起来
     - `<link rel="preload">` 资源优先级比较高，需要提前获取，资源加载时间最后控制在 `3ms` 内
   - `<img loading="lazy">` 懒加载图片
   - 利用GPU渲染动画，使用 `Composite` Layer，避免不必要Layout shift；浏览器绘制过程： `Parsing` -> `Recalculating styles` -> `Layout` -> `Painting` -> `Composite`;利用chrome layer & Rendering 工具查看
   - 使用 `Service Worker` & `Shared/Dedicated Worker` & `Worklet`







:gun: 实战：

1. [Service Worker：让你的 Web 应用牛逼起来 - 前端进阶之旅@wx](https://mp.weixin.qq.com/s/Vj_Uw_QKTHrwuImC2nRt0A)
   - 先简单介绍了前端性能提升的几种方式：http缓存，浏览器缓存，service worker缓存
   - 介绍 `service worker` 缓存的具体实战步骤
   - 手动配置缓存项太繁琐，[google workbox - @github](https://github.com/GoogleChrome/workbox) 对service worker进行了封装，提供了更便捷的操作方式
2. [如何将 Lighthouse Performance 评分从 20 提高到 96 - tyfy@掘金](https://juejin.cn/post/7012567366198362120)
   - 本文介绍了如何测试页面性能得分：使用lighthouse工具
   - 分析性能指标：`FP | FCP | LCP | TTI | TBT`, 并以实际项目作为依据去提升这些指标的得分
   - 亮点：以vue项目为例使用异步加载组件的方式，利用 **分片加载（使用 `requestIdleCallback`）** 去加载和注册首屏不需要立马使用到的组件
3. [🚀 得物App H5秒开优化实战 - 得物技术@掘金](https://juejin.cn/post/7086284339364757517)
   - 多方面介绍了如何优化h5页面的加载速度，异常监控分析等，很有深度的一篇文章，有些地方实现细节还不是很明白，需要多看几遍
   - 客户端（iOS & Android）优化： HTML预加载，HTML预请求，离线包的设计，接口预请求
   - H5优化：SSR服务端渲染，预渲染HTML & CDN，css优化，图片优化
   - 监控：使用 performance 进行性能指标上报，白屏监控，网络监控，异常监控



