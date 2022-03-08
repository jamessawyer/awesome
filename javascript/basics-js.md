记录一些关于js的基础知识，但不是简简单单的基础，旨在理解其原理



原型链：

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
   
     

