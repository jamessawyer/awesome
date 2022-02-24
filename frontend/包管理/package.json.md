1. [关于前端大管家 package.json，你知道多少？ - CUGGZ@掘金](https://juejin.cn/post/7023539063424548872)
   - 对package.json中绝大数属性进行了简短的介绍
   - 但是没有对yarn中的 `resolutions` 字段进行说明 以及monorepo项目 `workspaces` 属性
2. [活用yarn resolutions统一版本大幅减小产物包体积（优化之最后的倔强）- 咲奈@csdn](https://blog.csdn.net/qq_21567385/article/details/112644629) resolutions的用法
3. [一文搞懂peerDependencies - astonishqft@segmentfault](https://segmentfault.com/a/1190000022435060) 对等依赖，一般用于自定义库中，使包的依赖去依赖主项目版本，避免重复下载多个不同的包