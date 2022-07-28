用于记录一些前端高级知识点

## 1.Blob & ArrayBuffer & TypedArray

这些都是一些比较底层的数据类型，可以用于图片处理，数据传输，影像，声音，文件处理等资源的操作，相关文章：

1. [✨玩转前端二进制 - 阿宝哥](https://juejin.im/post/6846687590783909902)
   - 二进制、Blob、Blob URL、Base64、Data URL、ArrayBuffer、TypedArray、DataView 和图片压缩
   - 切块上传大文件 & 下载数据
2. [你不知道的 Blob - 阿宝哥](https://juejin.im/post/6844904178725158926)
3. [🚀 Blob、File、ArrayBuffer、TypedArray、DataView究竟应该如何应用 - 19组清风@掘金](https://juejin.cn/post/7093908575935807502)
   - 清晰的描述了js中的 `Blob` & `ArrayBuffer` 等类型
   - `ArrayBuffer` 二进制类型，可以通过 `TypedArray | DataView` 对二进制进行操作
   - `TypedArray` 不是具体类型，类似一种接口规范，包含 `UInt8Array | UInt16Array | Float32Array` 等等
   - 通过 `new Blob(ArrayBuffer)` 可以将 ArrayBuffer 转换为 Blob 类型；然后通过 `FileReader.readAsArrayBuffer` 将 Blob 类型转换为 ArrayBuffer 类型
   - 使用 `URL.createObjectUrl(blob)`, 可将 ArrayBuffer 类型转换为 Blob 类型



## 2. SVG相关

1. [SVG 入门指南 - 前端小智 @掘金](https://juejin.cn/post/6844904017273815048)



## 3. 跨浏览器通信

用于多个窗口之间进行通信，交互

1. [跨浏览器窗口 ，7种方式，你还知道几种呢？ - 云的世界@掘金](https://juejin.cn/post/7002012595200720927)
   - postMessage
   - StorageEvent
   - Broadcast Channel
   - Message Channel



## 4. 扫码

1. [前端实现很哇塞的浏览器端扫码功能🌟 - dragonir@掘金](https://juejin.cn/post/7018722520345870350)
   - 使用 `WebRTC` API调用相机功能
   - 使用 `video` 元素捕获画面，然后使用canvas对画面中的二维码绘制，然后转换成ImageData数据格式
   - 使用 `jsQR` lib对上面的数据进行解析，获取二维码数据
   - 该教程使用vuejs实现，但是原理是通用的


## 5. 沙箱

1. [【万字长文】一文带你打通前端沙箱的"任督二脉" - 战场小包@掘金](https://juejin.cn/post/7124969690958397471)
   - 介绍了各种沙箱方法
   - 从最简单的 `IIFE` & `eval` & `eval + with` & `with + new Function()` 讲起，介绍各种方法的限制
   - 然后介绍 `with + new Function() + Proxy` 引入代理，存在的缺点，任意使用全局变量，沙箱逃逸问题的出现
   - 快照沙箱 `SnapshotSandbox` 方案，通过 `active` & `inactive` 打开和关闭沙箱，开关的时候对沙箱或window进行还原，存在问题是需要对window进行遍历，性能较差，另外也会污染widnow对象
   - 基于Proxy的单例沙箱 `LegacySandbox`，整体思想就是用多个变量记录变量的新增和修改，然后进行还原，不通过遍历window属性方式性能较好，但是还是会污染window对象
   - 基于Proxy的多例沙箱 `ProxySandbox`，可以生成多个沙箱环境，目前为止最好的一种方式
   - iframe隔离，以及其存在的一些问题
   - EACMScript新的提案解决沙箱问题：`ShadowRealm`

