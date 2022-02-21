关于浏览器渲染相关的文章：

1. [🚀【7000字】一晚上爆肝浏览器从输入到渲染完毕原理 - 道道里@掘金](https://juejin.cn/post/7039036362653171742) 这个是翻译外网的一篇文章
   - 介绍了浏览器的基本架构和组成: 进程，线程，chrome多进程架构
   - 从输入到跳转整个渲染流程：**跳转 -> 渲染 -> 事件处理**，不同进程以及线程之间的通信
   - [浏览器默认样式列表](https://source.chromium.org/chromium/chromium/src/+/master:third_party/blink/renderer/core/html/resources/html.css)