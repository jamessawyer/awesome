深入理解打包：

1. [基于 esbuild 的 universal bundler 设计 - 字节前端@掘金](https://juejin.cn/post/6940218189921910797)
   - 理解这篇文章有点难，要求知识比较深，感觉吃透很难，建议多看几遍
   - 介绍了现有前端各种bundlers的工作原理：生成module graph，然后基于module graph进行各种优化，比如tree shaking，transfromer，code spliting & minify等等，然后最后format成各种js代码格式（esm,cjs,amd等等）
   - 各种插件系统的实现
   - 针对浏览器端运行node需要的环境的方案，virtual module
   - filesystem & node module resolution算法
   - prebundle工具
   - esbuild存在的一些问题：调试麻烦，target针对现代浏览器，生成的wasm包运行速度低且大
   - 几款工具 [esm.sh](https://esm.sh/) & [rollup/repl](https://rollupjs.org/repl) & [memfs](https://github.com/streamich/memfs)





如何写一个库：

1. [Creating React Libraries from Scratch - @bilibili](https://www.bilibili.com/video/BV1sa411y7TM)

   - 这个课程来自 [newline](https://www.newline.co/courses/creating-react-libraries-from-scratch) 

   - 介绍写一个库的基本配置：typescript + jest + eslint + lint-staged

   - 通过学习最后几节的视频，上面配置过程有个大致的了解

   - 顺带介绍了一下 `storybook`

     
