关于websocket，socket相关的一些文章：

1. [2万字长文肝了一个实时聊天室，只为让她学会websocket - 前端胖头鱼@wx](https://mp.weixin.qq.com/s/VjrGYYFSxwJq65ofeDHHJg)
   - websocket 相比于 http 的差异点： 双向通信，实时性强，支持二进制，控制开销
   - `socket` VS `websocket` 区别： socket是应用层与TCP/IP协议族通信的中间软件抽象层， 它是一组接口
   - websocket 事件驱动 + 初次握手采用http，然后进行协议升级
   - 聊天室功能的实现，心跳检测保活