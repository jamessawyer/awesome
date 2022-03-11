基础：

1. [不可多得的 Babel 小抄 - 码农小余@wx](https://mp.weixin.qq.com/s/_0KHBNvm-jQSwD0Zp4Bgjw)
   - `@babel/plugin-transform-destructuring` & `@babel/plugin-proposal-object-rest-spread` 插件的作用
   - `@babel/preset-env`  预设的作用，`targets` 配置项和 `browserslist` 的关系
   - `@babel/plugin-transform-runtime` 插件的作用：将转换函数变成外部模块化，避免重复生成
   - `@babel/polyfill` 去配置 `@babel/preset-env`，使用 `useBuiltIns` 动态引入使用到的polyfill
   - [polyfull bundle online](https://polyfill.io/v3/url-builder/)
   - [babel插件手册 - @github](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md)