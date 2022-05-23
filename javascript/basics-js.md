记录一些关于js的基础知识，但不是简简单单的基础，旨在理解其原理



🔥 [Lydia Hallie](https://www.lydiahallie.io/) JS 可视化系列：如果dev.io访问不了，可以访问 [这个中文翻译的blog](https://zhuangyin8.github.io/tags/Visualized/) (必看系列🚀)

1. [EventLoop](https://dev.to/lydiahallie/javascript-visualized-event-loop-3dif) 特别要注意是 `async-await` 在事件循环时的执行机制；另外讲解了事件循环和浏览器HTML渲染的过程
2. [Hoisting](https://dev.to/lydiahallie/javascript-visualized-hoisting-478h) Hoisting的原理：在执行代码之前,将函数和变量存储在内存中以用于执行上下文.这称为 *变量提升*. 函数以对整个函数的引用存储,带有 `var` 关键字的变量值为 `undefined` ,而带有 `let` 和 `const` 关键字的变量 *未初始化* .（这和传统错误理解的将变量和函数物理上拿到文件最顶部的想法是不同的）
3. [Scope Chain](https://dev.to/lydiahallie/javascript-visualized-scope-chain-13pd) 可视化的方式展现作用域链，对理解闭包的概念有所帮助
4. [the JavaScript Engine](https://dev.to/lydiahallie/javascript-visualized-the-javascript-engine-4cdf) 可视化的方式演示 **HTML -> 浏览器编译** 的过程
   - [Lydia Hallie: JavaScript Visualized: A Script's Life - @youtube](https://www.youtube.com/watch?v=4ynZURtf7kU) 对应的youtube视频
     - HTML 解析
     - byte stream decoder字节流解码
     - Tokens 分词 -> Parser 解析 -> Nodes 创建节点 -> AST 抽象语法树
     - 字节码标志，寄存器，accumulators
     - 字节码生成，字节码优化，优化器，字节码解释器
     - Shape的概念（js中的对象），inline caching优化
5. [Prototypal Inheritance](https://dev.to/lydiahallie/javascript-visualized-prototypal-inheritance-47co) 理解 实例包含不可枚举的 `__proto__` 属性，指向构造函数的 `prototype` 属性，即 `dog.__proto__ === Dog.prototype`，而 `Dog.prototype` 又是 `Object` 的实例，即 `Dog.prototype.__proto__ === Object.prototype`
6. [Generators and Iterators](https://dev.to/lydiahallie/javascript-visualized-generators-and-iterators-e36) 理解Generator和可迭代协议；`yield` 的作用：暂停函数的执行； `yield*` 后面可以接一个 **可迭代对象 或者 另一个 generator**: 表示将当前任务委托给另一个generator；以及 `yield` 返回值，可以传入 `it.next('something here')`; `[Symbol.iterator]` 自定义可迭代对象
7. [Promises & Async/Await](https://dev.to/lydiahallie/javascript-visualized-promises-async-await-5gke) Promise语法，以及对事件循环过程的补充
8. [CS Visualized: CORS](https://dev.to/lydiahallie/cs-visualized-cors-5b8h)： 对跨域最直白的一篇可视化理解，以及避免跨域的方法：CORS（跨域资源共享）的原理。
   - 跨域：浏览器同源策略的限制
   - CORS：浏览器简单请求自动添加 `Origin`，表示请求的原地址；服务端Response时，设置Headers，可设置的头有 `Access-Control-Allow-Origin` & `Access-Control-Allow-Methods` 等等；浏览器对比 `Origin` 和 服务端返回的Headers，如果能匹配，则表示浏览器允许跨域资源请求；
   - 复杂请求则一般会采用 **预检请求**（`Preflight Request`），先查看服务端是否允许请求，如果不允许则直接失败，如果允许，则浏览器再发起真正的请求；可以设置 `Access-Control-Max-Age` 缓存预检结果，避免重复的预检开销
   - 如果需要携带Cookies，则服务端需要设置 `Access-Control-Allow-Credentials`, 前端则需要设置 `{credentials: true}`





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



🤡 tricks:

1. [you don't need void 0](https://p42.ai/blog/2022-05-10/you-dont-need-void-0) 
   - 以前经常在代码中看到 `void 0`， 不明所以，它其实就是 `undefined`
   - 为什么写成这种形式，是因为历史原因， `undefined` 在 ES5前不是 **只读的**, 可以被篡改，导致问题的出现
   - 现在通过ESLint等工具的校验，完全可以避免这种问题的出现，因此不再需要 `void 0` 这种令人难以理解的写法了，直接写为 `undefined` 即可

 
