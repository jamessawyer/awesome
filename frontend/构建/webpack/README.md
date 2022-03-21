深入理解webpack:

1. [🚀🚀 [万字总结] 一文吃透 Webpack 核心原理 - 范文杰@掘金](https://juejin.cn/post/6949040393165996040) 强烈推荐多读几遍
   - 将webpack分为3块：构建的核心流程，plugins架构和webpack hooks，loaders的作用
   - 构建的核心流程又分为：
     - 初始化阶段：读取配置，创建Complier对象，确认入口
     - 构建阶段：生成modules，和生成依赖图，**module** 是webpack的最小资源
     - 生成阶段：根据配置生成chunks，最终生成文件资源
   - 插件系统：插件的定义形式，[Tapable](https://github.com/webpack/tapable) 钩子类型；钩子触发时机+参数；因为钩子执行产生副作用，会影响到后续插件的执行
   - loaders作用：资源的读取，资源的转换，AST对象的生成
   - [webpack5 核心知识体系 mindmap](https://gitmind.cn/app/doc/fc4fa821f4732f3f2da7c74706c75bf5)
   - [⭐️⭐️ 精通webpack核心原理 - 专栏 范文杰](https://juejin.cn/column/6978684601921175583) 更多深入理解webpack的文章
2. [Webpack 原理系列二：插件架构原理与应用 - 范文杰@掘金](https://juejin.cn/post/6955421936373465118) 还是这位大佬的
   - 详细的讲解了webpack插件系统依赖的 [Tapable](https://github.com/webpack/tapable) 库
   - Tapable钩子的几种类型：同步钩子 & 异步钩子
   - webpack中对不同钩子的使用情况
   - Tapable动态变异的处理流程
   - **构建一个插件体系需要的考虑的架构能力**
   - 对自定义webpack插件，理解Tapable是必须的




source map:
1. [弄懂 SourceMap，前端开发提效 100% - 3分钟学前端@wx](https://mp.weixin.qq.com/s/6uQY2EW433u6iQAa4Z0CSw)
   - 介绍了source map的生成几种方式：uglifyjs,gulp,webpack等
   - 很简单的介绍了一下sourcemap文件的含义
   - webpack最常见的7种sourcemap类型：`source-map | inline-source-map | hidden-source-map | eval-source-map | nosources-source-map | cheap-source-map | cheap-module-source-map`
   - 可从 **内联和外联** 的方式划分为2类，另外从构建速度和调试友好度方式可以分别用于 **生产和开发**