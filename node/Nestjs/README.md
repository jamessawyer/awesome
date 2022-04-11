:anguished: 基础知识：

1. [Nest.js 的 AOP 架构的好处，你感受到了么？ - zxg_神说要有光@掘金](https://juejin.cn/post/7076431946834214925)
   - 很简单的介绍了几个名词含义： `MVC` & `IOC` & `AOP`
   - 重点介绍了Nest.js中 `AOP`, 面向切面编程的作用
   - AOP实际应用场景：中间件，Guard, Interceptor, Pipe, ExceptionFilter的基本使用和注册方式，全局注册，单个路由注册
   - [整个请求链路的执行顺序](https://docs.nestjs.com/faq/request-lifecycle#summary)



:sunglasses: 进阶：

1. [学习nestjs，这可能是原理讲的最清晰的一篇文章了 - 孟祥_成都@掘金](https://juejin.cn/post/7077372768378945573)
   - 介绍了nestjs IOC的实现原理，如何使用 `reflect-metadata` 库实现一个简易的依赖注入
   - 对装饰器模式使用理解
   - express库的简易实现，模拟实现express库各种APIs
2. [Nestjs模块机制的概念和实现原理 - 子慕大诗人@掘金](https://juejin.cn/post/7083294481327325192)
   - Nestjs中最常见的几个基本概念：依赖反转，依赖注入，装饰器，@Module 中的 `providers & imports & exports` 原理是啥
   - 依赖反转：就是将人为控制的事情交给系统去完成，减轻人为压力和出错的概率。就好比如，自己开车去上班，需要规划路径，小心驾驶等，而乘坐地铁去上班，就不用考虑这些，只需要知道乘坐路线即可，其余的交给公共交通系统去帮你完成
   - 依赖注入：通过传值的方式注入，声明一个全局的对象，将需要的类挂载到这个全局的对象上
   - 装饰器：涉及到元数据编程，描述数据的数据，主要是描述数据属性的信息，也可以理解为描述程序的数据
   - @Module中的属性：用于管理梳理依赖关系，和注入实例



:gun: 实战：

1. [How to perform NestJS Config Validation? - @progressivecoder](https://progressivecoder.com/how-to-perform-nestjs-config-validation/)
   - 如何使用 `Joi` 对 `@nestjs/config` 配置文件进行验证
   - 如何结合 `class-transformer & class-validator` 自定义验证逻辑



:book: 资源：

1. [⭐️⭐️ 周末整理了这些 Nest.js 学习文章，先收藏着，空了再看！ - @wx](https://mp.weixin.qq.com/s/3Qy8CIlamr_GZhzBNb90hg)
