monorepo用于将多个项目存储在一个仓库中进行管理



## 文章

1. [All in one：项目级 monorepo 策略最佳实践 - libinfs@掘金](https://juejin.cn/post/6924854598268108807)
   - monorepo的优劣势
   - monorepo实践，环境锁定，统一配置tsconfig, eslint, babel等
   - Scripty 命令管理
   - 使用Lerna包管理
   - ⭐️扩展阅读
2. [Monorepo 的这些坑，我们帮你踩过了！ - 字节教育@掘金](https://juejin.cn/post/6972139870231724045)
   - 使用monorepo常见问题：依赖重复，版本冲突等
   - ⭐️常见解决方案：pnmp, rush
   - monorepo常用命令




## 视频资料
1. [Monorepos - A Beginner's Guide - @bilibili](https://www.bilibili.com/video/BV1vq4y1w7Qe) 搬运udemy上的课程，简单的介绍了使用workspace 实现monorepo，以及如何使用lerna进行管理

   1. 设置 `package.json` 中的 `workspaces` 字段，通过 `packages` 指定包的位置
   2. 默认情况下，`packages` 中的模块的依赖会提升的项目的最顶层，如果不想提升，可以使用 `nohoist` 属性
   3. 如何设置 `bin` 命令，并在不同模块或者根目录中使用
   4. 如何使用 `yarn workspace same-module module-command` 去执行某个模块中定义的命令，比如 `yarn workspace module-a build` 执行 `module-a` 模块中的 `build` 命令
   5. 另外 `yarn workspace`  还可以给其它模块安装依赖，比如 `yarn workspace module-a add react react-dom`
   6. 如何使用 `learn` 管理packages，发布packages，`packages `给包添加前缀，比如 `@shannon/module-a` | `@shannon/module-b`
   7. [monirepo-beginners - robdonn@github](https://github.com/robdonn/monorepo-beginners)

   

