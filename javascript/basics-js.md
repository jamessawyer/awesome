记录一些关于js的基础知识，但不是简简单单的基础，旨在理解其原理



⛓ 原型链：

1. [图文并茂🌈聊聊原型与原型链 - Rockky@掘金](https://juejin.cn/post/7053331458101887007)
   - 个人感觉，这篇文件对理解原型最为直白
   
1. [原型链 - @bilibili](https://www.bilibili.com/video/BV16q4y1o7EG?p=72)
   
   - 这是 `JavaScript: The Advanced Concepts` 的课程的一节: `86 Prototypal Inhertance 6`
   
   - 其中提到了 `Only functions has 'prototype' property`，即只有函数才有 `prototype` 属性。
   
     ```js
     typeof Object
     // 打印结果
     'function'
     
     // 本质上，像Object, String, Boolean等这些包装类型都是函数
     // 因此它们拥有 prototype 属性
     ```
   
1. [这也许是你会遇到的Google Chrome Bug - 19组清风@掘金](https://juejin.cn/post/7074935443355074567) 

   - 这个讲解了继承中，对child添加属性，如果child自身不存在该属性，原型上存在该属性 `setter & getter` 情况下，可能会出现 **屏蔽效果** ，算是一个小的知识点





🕸 Proxy & Reflect:

1. [为什么Proxy一定要配合Reflect使用？ - 19组清风@掘金](https://juejin.cn/post/7080916820353351688)
   - 代理 `handler` 函数中，对 `get(target, key, reciver)` trap的第3个参数 `receiver` 的作用
   - 使用 `Reflect.get(target, key, receiver)` 可以改变target的指向，即类似于 `target[key].call(receiver)` 的作用，这在继承关系中可能有用

 
