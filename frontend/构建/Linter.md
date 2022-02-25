关于eslint,prettier等等一些配置规则

1. [🚀Linter上手完全指南 - haixiangyan@github](https://github.yanhaixiang.com/linter-guide/) 这篇小册讲的很透彻
   - eslint 和 prettier 的区别
   - eslint-config-xxx & eslint-plugin-xxx & xxx-config-prettier 的区别和作用分别是什么
   - linters结合typescript如何使用
   - linters + vue | react 所需要的插件和config
   - husky & lintstaged 的用法
2. [一文彻底读懂ESLint - 程序员成长指北@wx](https://mp.weixin.qq.com/s/uUSTt_4ClYj7uqMCU1JEPw) 看完上面文章再看这篇就很好理解了
3. [手摸手教你使用最新版husky(v7.0.1)让代码更优雅规范 - 啥也不是的小垃圾@掘金](https://juejin.cn/post/6982192362583752741) 讲解最新版本 husky + lint-staged 的使用方式



可以使用 `mrm2` 创建 `lint-staged` 命令：

```bash
npx mrm@2 lint-staged
```

