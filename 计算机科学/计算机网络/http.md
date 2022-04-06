http协议相关的一些文章：

1. [http/2 究竟有多快？| http2 http1.1 http1.0 对比评测 - Adam小哥哥@bilibili](https://www.bilibili.com/video/BV1p541147LD)
   - 通过实际实例对比 `http 1.0 & 1.1 & 2.0` 网站加载速度
   - 从原理上讲解协议升级带来速度升级的原因
   - http1.0是短连接，每次请求都要握手，然后断开
   - http1.1通过 `keep-alive` 技术，实现一次握手，然后请求结束后，再断开tcp连接
   - http2.0通过二进制分帧层压缩请求头，一次请求多个资源，节省贷款；多路复用等技术；并且解决了1.1中的 `队头阻塞` （即前面请求没有完成，后面请求会被阻塞）的问题
2. [⭐️⭐️ 深入淺出 HTTP/2 通訊協定 wuli保哥@youtube](https://www.youtube.com/watch?v=O62ufq-orfY)
   - [b站地址](https://www.bilibili.com/video/BV1n44y1K7Mv)
   - `http 1.0` 每次只能处理一个连接，处理完成后会断开连接，再次请求，需要再次建立连接
   - `http 1.1` 在1.0基础上增加了 `Pipeline`(管线) & `Persist Connection` (持久化连接 `Keep-Alive`) & `Cache-Control`
     - Pipeline: 一次连接可以发送多个请求，但是会在应用层（七层架构）存在 `Head-of-line blocking` 阻塞问题，必须要等第一个请求返回之后，再去返回第2个请求
     - Persist Connection: 每次请求后不会断开连接，而是根据 `Keep-Alive Timeout` 时间去断开连接
     - Cache-Control: 对请求资源进行缓存协商
   - `http 2.0` 在1.1基础上，将请求头明文传输改为了 binary 传输，并通过 `SID` (Stream ID) 实现 `Mutliplexing` 多工传输，可以不按顺序传输多个frame，在应用层上解决了Head-of-Line blocking的问题（但是这个问题在TCP层，仍旧存在，因为TCP是可靠传输，因此HTTP3.0采用了UDP的方式）另外还通过 `Header Compression` (`HPack` 压缩算法) 将请求头进行了压缩，并支持 `Server Push` 服务端主动给客户端推送资源
     - Binary Protocol: 所有请求以 Binary Frame方式进行传输
     - MultiPlexing: 多工复用，使用stream id
     - Header Compression: 每次请求头基本上都差不多会重复，进行压缩可以减小开销
     - Server Push: 服务端主动将资源