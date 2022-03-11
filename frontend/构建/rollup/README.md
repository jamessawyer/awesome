基础：

1. [rollup能输出哪6种格式 - 春哥的梦想是摸鱼@掘金](https://juejin.cn/post/7051236803344334862)
   - 介绍了rollup能够生成的6种打包格式：`IIFE` & `Common JS` & `AMD` & `UMD` & `SystemJS` & `ESM`
   - `IIFE`: 立即执行函数，比如 `jquery`，将对象挂载到全局 `window` 上；输出的变量会影响全局变量；引入依赖包时依赖全局变量
   - `CJS`: node.js环境下的文件模块
   - `AMD + require.js`: 前端模块依赖管理系统
   - `UMD`: 既可以在node.js上使用，又可以在浏览器环境使用
   - `SystemJS`: whatwg 上提出的规范，微前端中会使用到
   - `ESM`: 最新的module系统，优势明显，对浏览器要求高



插件：

1. [⭐️⭐️ 一文入门rollup🪀！13组demo带你轻松驾驭 - 春哥的梦想是摸鱼@掘金](https://juejin.cn/post/7069555431303020580) 介绍常用rollup插件的作用和使用方式, 插件的命名一般是 `@rollup/plugin-xxx`
   - [commonjs](https://github.com/rollup/plugins/tree/master/packages/commonjs): 在项目中导入common.js模块文件，rollup默认是esm
   - [node-resolve](https://github.com/rollup/plugins/tree/master/packages/node-resolve): 用于定位第三方依赖的路径
   - [babel](https://github.com/rollup/plugins/tree/master/packages/babel): js语法转换
   - [alias](https://github.com/rollup/plugins/tree/master/packages/alias): 路径别名，很常见
   - [html](https://github.com/rollup/plugins/tree/master/packages/html): rollup创建html文件，并引入构建好的包
   - [inject](https://github.com/rollup/plugins/tree/master/packages/inject): 全局注入，比如全局注入 `lodash | jquery`, 这样在不同的文件中，就不需要每个都引入
   - [multi-entry](https://github.com/rollup/plugins/tree/master/packages/multi-entry): 将多个文件合并到一个打包文件中
   - [replace](https://github.com/rollup/plugins/tree/master/packages/replace)： 字符串替换工具，比如常见的 `env` 变量替换
   - [run](https://github.com/rollup/plugins/tree/master/packages/run): 直接运行打包好的bundle，一般用于node.js中
   - [strip](https://github.com/rollup/plugins/tree/master/packages/strip): 移除console,debugger等
   - [url](https://github.com/rollup/plugins/tree/master/packages/url): 通过data uri 或者esm的形式引入文件，一般用于图片的引入
   - [postcss](https://github.com/egoist/rollup-plugin-postcss): 对css预编译语法进行转换，比如sass，less等
   - 上面插件实操项目地址： [rollup-demos - @github](https://github.com/zhangshichun/rollup-demos)