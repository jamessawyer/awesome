自定义脚手架对于前端基建来说是很常见的需求



1. [leader要我三天时间搭一套“ react-cli ”出来，我答应了 - 郑鱼咚@掘金](https://juejin.cn/post/7033959447017816077)
   - 先分析了 `vue-cli` 的实现细节，lerna + yarn workspace包管理，功能分层
   - 然后依照 `vue-cli` 的结构自定义了自己的脚手架模板
   - 最后谈到 `npm publish` 可能会碰到的坑





## 脚手架开发中常用的一些工具库



1. [Inquirer](https://github.com/SBoudrias/Inquirer.js) 命令行交互工具
2. [comander.js](https://github.com/tj/commander.js)  命令行解析工具
3. [ora](https://github.com/sindresorhus/ora) 命令行loading
4. [chalk](https://github.com/chalk/chalk) console输出添加各种颜色
5. [globby](https://github.com/sindresorhus/globby) 模式匹配，用于快速找到符合正则匹配的文件
6. [semver](https://github.com/npm/node-semver) lib版本查询，比对
7. [log-symbols](https://github.com/sindresorhus/log-symbols) 给打印日志添加小的icon
8. [cli-welcome](https://www.npmjs.com/package/cli-welcome) cli添加一些欢迎信息
