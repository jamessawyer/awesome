🚀 核心原理：

1. [⭐️⭐️ 超详细的《Express》核心原理解析（上） - 愣锤@掘金](https://juejin.cn/post/7095550340883283976)
   - 对中间件的整个执行流程进行了很详细的讲解， `app.use()` + `app[method]()` 添加中间件的原理
   - 分清 `Router` & `Layer` & `Application` 等核心代码进行细致的讲解
   - `Router` & `Route` 的关系，以及真正存放中间件的 `stacks`；真正触发中间件调用的 `router.dispatch` 函数
   - `next()` & `next(error)` 普通中间件和错误中间件的触发条件
   - 多张流程图很清晰，跟着一起画一遍会有收获