下面介绍一下pinia相关的文章：



基础篇：

1. [新一代状态管理工具，Pinia.js 上手指南 - 小学生study@掘金](https://juejin.cn/post/7049196967770980389)
   - pinia的基本配置：state, actions（同步 & 异步），getters
   - 多个stores之间如何相互调用
   - pinia持久化插件： [pinia-plugin-persist](https://github.com/Seb-L/pinia-plugin-persist) 的用法
   - pinia使用注意事项，比如state，actions等的定义方式，可以参考 [Pinia 的学习路程 - 流量小生@掘金](https://juejin.cn/post/7069739313553997838)
   - 在线示例： [pinia-ts - @stackblitz](https://stackblitz.com/edit/vitejs-vite-cthpkx)



进阶：

1. [🚀 Pinia进阶：优雅的setup（函数式）写法+封装到你的企业项目 - 南山种子外卖选手@掘金](https://juejin.cn/post/7057439040911441957)
   - 介绍了pinia的2中写法： `options & setup` 写法
   - pinia代码分割，避免store重复打包，定义一个 `registerStore`方法，在 `main.ts` 中调用，打包优化
   - 多个stores进行组合，父store的模式，通过父store暴露子store
   - [pinia高阶用法 - @stackblitz](https://stackblitz.com/edit/vitejs-vite-rueqgk)

