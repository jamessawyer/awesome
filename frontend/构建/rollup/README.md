基础：

1. [rollup能输出哪6种格式 - 春哥的梦想是摸鱼@掘金](https://juejin.cn/post/7051236803344334862)
   - 介绍了rollup能够生成的6种打包格式：`IIFE` & `Common JS` & `AMD` & `UMD` & `SystemJS` & `ESM`
   - `IIFE`: 立即执行函数，比如 `jquery`，将对象挂载到全局 `window` 上；输出的变量会影响全局变量；引入依赖包时依赖全局变量
   - `CJS`: node.js环境下的文件模块
   - `AMD + require.js`: 前端模块依赖管理系统
   - `UMD`: 既可以在node.js上使用，又可以在浏览器环境使用
   - `SystemJS`: whatwg 上提出的规范，微前端中会使用到
   - `ESM`: 最新的module系统，优势明显，对浏览器要求高