基础知识：

1. [Nest.js 的 AOP 架构的好处，你感受到了么？ - zxg_神说要有光@掘金](https://juejin.cn/post/7076431946834214925)
   - 很简单的介绍了几个名词含义： `MVC` & `IOC` & `AOP`
   - 重点介绍了Nest.js中 `AOP`, 面向切面编程的作用
   - AOP实际应用场景：中间件，Guard, Interceptor, Pipe, ExceptionFilter的基本使用和注册方式，全局注册，单个路由注册
   - [整个请求链路的执行顺序](https://docs.nestjs.com/faq/request-lifecycle#summary)



进阶：

1. [学习nestjs，这可能是原理讲的最清晰的一篇文章了 - 孟祥_成都@掘金](https://juejin.cn/post/7077372768378945573)
   - 介绍了nestjs IOC的实现原理，如何使用 `reflect-metadata` 库实现一个简易的依赖注入
   - 对装饰器模式使用理解
   - express库的简易实现，模拟实现express库各种APIs



实战：

1. [How to perform NestJS Config Validation? - @progressivecoder](https://progressivecoder.com/how-to-perform-nestjs-config-validation/)
   - 如何使用 `Joi` 对 `@nestjs/config` 配置文件进行验证
   - 如何结合 `class-transformer & class-validator` 自定义验证逻辑

