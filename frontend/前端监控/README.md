埋点相关的技术：
1. [为什么大厂前端监控都在用GIF做埋点？ - @teqng](https://www.teqng.com/2022/02/11/%e4%b8%ba%e4%bb%80%e4%b9%88%e5%a4%a7%e5%8e%82%e5%89%8d%e7%ab%af%e7%9b%91%e6%8e%a7%e9%83%bd%e5%9c%a8%e7%94%a8gif%e5%81%9a%e5%9f%8b%e7%82%b9%ef%bc%9f/) 大致讲了一下前端埋点的几种方案，以及简单介绍了gif埋点的优势
2. [为什么通常在发送数据埋点请求的时候使用的是 1x1 像素的透明 gif 图片 - @github](https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/87)
   - gif埋点的优势
   - gif埋点实际上就是 `Web beacons` 技术，用于感知用户访问了某个页面或者用户的行为习惯
   - 具体实践方案
3. [为什么前端监控要用GIF打点 - @csdn](https://blog.csdn.net/huangpb123/article/details/102999371) 
   - 详细的分析对比了3种上报方式的优缺点： http请求，请求js，css文件，请求图片资源
   - 最终结论是，图片方式最优，然后分析了不同格式图片的大小，gif最小43个字节，节省更多带宽，因此gif格式最合适
4. [🚀一文彻底搞懂前端监控 - 执鸢者@掘金](https://juejin.cn/post/6903133330196299783)
   - 粗略的介绍了前端监控的整体流程
   - 各种上报指标的计算方式，FP,FCP,DCL,L等计算方式
5. [实现一个系统，统计前端页面性能、页面 JS 报错、用户操作行为、PV/UV、用户设备等消息，并进行必要的监控报警。方案如何设计，用什么技术点，什么样的系统架构，难点会在哪里 - @github](https://github.com/lgwebdream/FE-Interview/issues/1215)
   - 讲的是定制化前端监控方案的大体实现
   - 对比日志上传的几种不同方案：即时上报，批量上报，区块上报（针对异常情况），用户主动上报（用户反馈）
6. [在页面关闭时，前端上传监控数据的4个解决方案 - 高阶前端进阶@wx](https://mp.weixin.qq.com/s/sLnP6JK0Y_-KKh6XgdbiDA)
   - XMLHttpRequest 老版浏览器支持，已废弃 🚫
   - `img.src=api.xxx?yourdata`： 传输不可靠，只能 `GET` 请求，数据大小有限制
   - `navigator.sendBeacon`: 兼容性好，只能发送 `POST` 请求，大小有限制 🎉
   - `fetch keepalive`：允许页面卸载后执行，兼容性有问题

