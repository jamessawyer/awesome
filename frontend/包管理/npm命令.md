可以使用下面命令查看帮助文档，更详细文档可以参考 [官方npm cli](https://docs.npmjs.com/cli/v8/commands/)

```bash
# ⭐️npm符号链接的一些指令
# usage: `npm link` 或者 `npm ln`
npm help link

# 查看npm和版本相关的信息
npm help version

# ⭐️查看package.json文件中的每个字段的含义
npm help package-json

# ⭐️查看package.json中 "scripts" 属性的定义
# scripts钩子 "pre" "post" lifecycle等
npm help scripts

# 初始化 package.json 文件的一些指令
npm help init

# 管理npm配置文件
npm help config

# npm的配置文件 npmrc
npm help npmrc

# 包安装卸载的一些配置命令
npm help install
npm help uninstall

# ⭐️查看包的注册信息
npm help view

# 用于搜索包
npm help search

# 使用monorepo时 会使用到workspaces
npm help workspaces

# 发布一个npm包到 https://www.npmjs.com/
npm help publish
```



> 查看某个包的版本信息

实际上是使用 `npm view` 命令进行查看

```bash
# 当前稳定版本
npm view react version

# 查看所有版本
npm view react versions
```



> 版本升级

包一般依照semver规范进行升级，即 `major.minor.patch`, 假设当前版本是 `v1.0.0`（即package.json中的 `version` 字段对应的值），在发布新包时，可以手动的升级版本，但是一般使用下面命令行进行版本升级

```bash
# 升级 `patch`
# 执行完命令后，version将变为 `v1.0.1`
npm version patch

# 升级 `minor`
# 执行完命令后，version将变为 `v1.1.1`
npm version minor

# 升级 `major`
# 执行完命令后，version将变为 `v2.1.1`
npm version major
```

