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





实战：

1. [Service Worker：让你的 Web 应用牛逼起来 - 前端进阶之旅@wx](https://mp.weixin.qq.com/s/Vj_Uw_QKTHrwuImC2nRt0A)
   - 先简单介绍了前端性能提升的几种方式：http缓存，浏览器缓存，service worker缓存
   - 介绍 `service worker` 缓存的具体实战步骤
   - 手动配置缓存项太繁琐，[google workbox - @github](https://github.com/GoogleChrome/workbox) 对service worker进行了封装，提供了更便捷的操作方式