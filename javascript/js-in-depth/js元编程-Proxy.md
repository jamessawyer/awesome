基础知识：

1. [🚀🚀【温故而知新】你可能不知道的 Proxy - 微医大前端技术@掘金](https://juejin.cn/post/7114845886122819621) 强烈推荐，写的是真的好

   - 从ECMAScript规范上理解Proxy，JS对象判定的标准就是是否存在某些内部方法和内部槽，比如 `[[Get]]`, `[[GetPrototypeOf]]` 等等
   - 函数对象的判定标准是否存在 `[[Call]]` 内部槽，而构造函数的判定标准是 `[[Call]] + [[Construct]]`
   - JS对象分了2种：普通对象和异质对象，满足ECMAScript规范10.2.1章节或10.2.2章节定义的是普通对象，其余的则为异质对象
   - `Reflect` 方法中的 `receiver` 参数表示 `this` 是 `proxy`, 而不是 `target`(对象)，而代理拦截只针对代理自身，而不是target，当调用的 `getter` 中存在 `this` 时，就需要注意this的指向了，如果指向的是target，不会触发proxy上的 `get` 陷阱，如果指向的是proxy自身，才会接着再次触发 `get` 陷阱
   - 代理的本质：**是对代理对象自身的拦截，而不是target的拦截**，对于没有定义的handler traps，只是proxy将操作转发到了target上🤩
   - 对操作的拦截本质上是定义proxy对象的内部方法，如果某个方法没有定义，则可能会将该操作转发到target内部方法处理

2. [为什么Proxy一定要配合Reflect使用？ - 19组清风@掘金](https://juejin.cn/post/7080916820353351688) Proxy + Reflect 如何正确将 `this` 指向想要的对象

   - 代理handler中traps中 `receiver` 对象表示的是 **代理对象本身或者继承自代理的对象**， `Object.setPrototype(obj, proxy)` 使obj继承自proxy
   - Reflect中传递 `receiver` 参数表示修改原始操作时的 `this` 指向

   

