

本文将学习如何容器化Node.js应用，先从简单的Dockfile开始，了解一下每个Dockerfile指令中的缺点和不安全性，然后对其进行修复。



## 一个简单的Node.js Docker镜像构建

最常见的构建Nodejs镜像的Dockerfile形式：

```dockerfile
FROM node
WORKDIR /usr/src/app
COPY . /usr/src/app
RUN npm install
CMD "npm" "start"
```

构建并运行：

```bash
docker build . -t nodejs-tutorial
docker run -p 3000:3000 nodejs-tutorial
```

这看起来很简单，并且可以正常运行。

但是这个配置对构建nodejs应用来说，充满了错误和非最佳实践。下面我们一点一点优化这个构建配置。本文 [nodejs-docker-best-practices - github](https://github.com/snyk-labs/nodejs-docker-best-practices)



## 1. 使用显式和确定性Docker基础镜像tags

很明显我们的镜像要基于 `node` 这个基础镜像，如果没有指定tags，docker总是会默认拉取 `:latest` tag.

因此：`FROM node` 等价于 `FROM node:latest`。

这样做的缺点有：

1. 构建的docker镜像可能会不一致，应该和 `package-lock.json` 一样锁定指定版本的镜像
2. 这里的node镜像是一个完整的操作系统，对于node web应用，它包含了太多没有用处的工具和库。这不仅会增大构建体积，和构建时间，还可以引入更多的不安全风险。



推荐做法：

1. 尽可能的使用小的docker镜像，减小体积和风险
2. 使用docker镜像digest，即镜像的静态 `SHA256 hash`。这确保每次构建的基础镜像都是一致的

因此，我们确保使用Node.js的 `LTS` 版本，和体积最小的 `alpine` 镜像类型，即：

```dockerfile
FROM node:lts-alpine
```

尽管这也会拉去最新的tag，但我们可以找到其 `SHA256 hash`,通过下面命令，我们可以将其拉取到本地，获取其hash：

```bash
docker pull node:lts-alpine
lts-alpine: Pulling from library/node
0a6724ff3fcd: Already exists
9383f33fa9f3: Already exists
b6ae88d676fe: Already exists
565e01e00588: Already exists
Digest: sha256:b2da3316acdc2bec442190a1fe10dc094e7ba4121d029cb32075ff59bb27390a
Status: Downloaded newer image for node:lts-alpine
docker.io/library/node:lts-alpine
```

找到 `SHA256` 的另一种方式就是：

```bash
docker images --digests
REPOSITORY                     TAG              DIGEST                                                                    IMAGE ID       CREATED             SIZE
node                           lts-alpine       sha256:b2da3316acdc2bec442190a1fe10dc094e7ba4121d029cb32075ff59bb27390a   51d926a5599d   2 weeks ago         116MB
```

现在我们可以将我们Dockerfile更改为：

```dockerfile
FROM node@sha256:b2da3316acdc2bec442190a1fe10dc094e7ba4121d029cb32075ff59bb27390a
WORKDIR /usr/src/app
COPY . /usr/src/app
RUN npm install
CMD "npm" "start"
```

上面的dockerfile值指定了Node docker镜像的名字，而没有指定 `tag`,这不仅可读性差，不易维护，并且影响开发体验。

下面添加image tag:

```dockerfile
FROM node:lts-alpine@sha256:b2da3316acdc2bec442190a1fe10dc094e7ba4121d029cb32075ff59bb27390a
WORKDIR /usr/src/app
COPY . /usr/src/app
RUN npm install
CMD "npm" "start"
```



## 2. 只在Node镜像中安装production依赖

下面指令会将 `devDependencies` 依赖也安装到容器中，这其实对应用实际功能没什么用，这增加了不必要的安全风险，同时增加了镜像体积：

```bash
RUN npm install
```

如果你看过我先前的 [10 npm security best practices](https://snyk.io/blog/ten-npm-security-best-practices/),你应该知道使用 `npm ci` 进行增强，这可以防止持续集成 (`CI`) 流程中出现意外，因为如果与lockfile有任何偏差，它就会停止。

为了构建生产级别的docker镜像，我们应以确定性的方式安装production依赖，因此我们可以使用下面命名：

```dockerfile
RUN npm ci --only=production
```

因此我们的dockerfile可以改为：

```dockerfile
FROM node:lts-alpine@sha256:b2da3316acdc2bec442190a1fe10dc094e7ba4121d029cb32075ff59bb27390a
WORKDIR /usr/src/app
COPY . /usr/src/app
RUN npm ci --only=production
CMD "npm" "start"
```



## 3. 优化用于生产的Node.js工具

当为生产构建 Node.js Docker 镜像时，希望确保所有框架和库都使用性能和安全性的最佳设置。因此我们可以在dockerfile中添加下面指令：

```dockerfile
ENV NODE_ENV production
```

初看之下，这条指令看起来有点多余，因为我们已经在 `npm install` 阶段指定了production依赖项，那么为什么这是需要的呢？

开发者大多将 `NODE_ENV=production` 环境变量设置与生产相关依赖的安装相关联，但是，此设置还有其他我们需要注意的影响。

如果将该 `NODE_ENV` 环境变量设置为 `production`，则某些框架和库可能仅打开适合生产的优化配置。抛开我们对框架采用的这种做法是好是坏的看法不谈，了解这一点很重要。例如，[Express 文档](https://expressjs.com/en/advanced/best-practice-performance.html#set-node_env-to-production) 概述了设置此环境变量以实现性能和安全相关优化的重要性。

因此 `NODE_ENV` 变量对性能的影响是意义巨大的，依赖的许多其他库也可能希望设置此变量，因此我们应该在 Dockerfile 中设置它。所以我们的dockerfile现在变为：

```dockerfile
FROM node:lts-alpine@sha256:b2da3316acdc2bec442190a1fe10dc094e7ba4121d029cb32075ff59bb27390a
ENV NODE_ENV production
WORKDIR /usr/src/app
COPY . /usr/src/app
RUN npm ci --only=production
CMD "npm" "start"
```


## 4. 不要以root身份运行容器

最小权限原则是 Unix 早期的一项长期安全控制，我们在运行容器化 Node.js Web 应用程序时应始终遵循这一原则。

官方nodejs Docker镜像及其变体（如 `alpine`）包括一个同名的最低权限用户：`node`。然而，仅仅将进程作为 `node` 运行是不够的。例如，以下内容可能不足以使应用程序正常运行：

```dockerfile
USER node
CMD "npm" "start"
```

原因是 `USER` Dockerfile指令仅确保该进程由 `node` 用户拥有。我们之前用 `COPY` 指令复制的所有文件呢？它们归 `root` 所有,这就是Docker默认的工作方式。

删除权限的完整正确方法如下，我们更新Dockerfile：

```dockerfile
FROM node:lts-alpine@sha256:b2da3316acdc2bec442190a1fe10dc094e7ba4121d029cb32075ff59bb27390a
ENV NODE_ENV production
WORKDIR /usr/src/app
COPY --chown=node:node . /usr/src/app
RUN npm ci --only=production
USER node
CMD "npm" "start"
```



## 5. 正确处理事件以安全终止Node.js Docker Web应用

我在有关在Docker容器中运行容器化Node.js应用程序的博客和文章中看到的最常见错误之一是它们调用进程的方式。以下所有指令及其变体都是应该避免的不好模式：

- `CMD "npm" "start"`
- `CMD ["yarn", "start"]`
- `CMD "node" "server.js"`
- `CMD "start-app.sh"`

下面将带你了解它们之间的差异以及为什么它们都是要避免的模式。

以下问题是理解正确运行和终止 Node.js Docker应用程序的上下文的关键：

1. 编排引擎，例如 Docker Swarm、Kubernetes，甚至只是 Docker 引擎本身，都需要一种向容器中的进程发送信号的方法。大多数情况下，这些是终止应用程序的信号，例如 `SIGTERM` 和 `SIGKILL`。
2. 该进程可能会间接运行，如果发生这种情况，则不能总是保证它会收到这些信号。
3. Linux 内核将作为进程 `ID 1` (PID) 运行的进程与任何其他进程 ID 不同。

有了这些知识，让我们开始研究为容器调用进程的方法，从我们正在构建的 Dockerfile 中的示例开始：

```dockerfile
CMD "npm" "start"
```

这里的缺点有两个方面。首先，我们通过直接调用 npm 客户端来间接运行node应用程序。谁能说 `npm CLI` 将所有事件转发到node运行时？实际上它没有，我们可以很容易地对这个说法进行测试。

确保在你的 Node.js 应用程序中为 `SIGHUP` 信号设置了一个事件处理程序，该信号在你每次发送事件时都会记录到控制台。一个简单的代码示例应如下所示：

```js
function handle(signal) {
   console.log(`*^!@4=> Received event: ${signal}`)
}
process.on('SIGHUP', handle)
```

然后运行容器，一旦它启动，就使用 docker CLI 和特殊的 `--signal` 命令行标志专门向它发送 `SIGHUP` 信号：

```bash
docker kill --signal=SIGHUP elastic_archimedes
```

什么都没发生，对吧？那是因为 npm 客户端不会将任何信号转发到它开启的node进程。

另一个缺点与在 Dockerfile 中指定 `CMD` 指令的不同方式有关。有两种方式，它们是不一样的:

1. shellform 表示法，其中容器产生一个封装进程的shell解释器。在这种情况下，shell 可能无法正确地将信号转发到你的进程。
2. execform 表示法，它直接生成一个进程，而无需将其包装在shell中。它使用 `JSON` 数组表示法指定，例如：`CMD ["npm", "start"]`。任何发送到容器的信号都会直接发送到进程

基于这些知识，我们希望改进我们的 Dockerfile 进程执行指令，如下所示：

```dockerfile
CMD ["node", "server.js"]
```

我们现在直接调用 node 进程，确保它接收到所有发送给它的信号，而不是将它包装在 shell 解释器中。然而，这将引入了另一个缺点。

当进程作为 `PID 1` 运行时，它们有效地承担了 init 系统的一些职责，该系统通常负责初始化操作系统和进程。内核以不同于处理其他进程标识符的方式处理 `PID 1`。来自内核的这种特殊处理意味着，如果进程尚未为其设置处理程序，则对正在运行的进程的 `SIGTERM` 信号的处理不会调用杀死进程的默认回退行为。

引用 [Docker and Node.js Best Practices](https://github.com/nodejs/docker-node/blob/main/docs/BestPractices.md#handling-kernel-signals) 中的话：“Node.js 并非设计为作为 PID 1 运行，这会导致在 Docker 内部运行时出现意外行为。例如，作为 `PID 1` 运行的 Node.js 进程不会响应 `SIGINT (CTRL-C)` 和类似信号”。

实现它的方法是使用一个工具，它的作用类似于 init 进程，因为它使用 PID 1 调用，然后将我们的 Node.js 应用程序生成为另一个进程，同时确保所有信号都代理到该node进程。如果可能，我们希望使用尽可能小的工具占用空间，以免将安全漏洞添加到我们的容器映像中。

我们在 Snyk 使用的一个这样的工具是 [dumb-init](https://engineeringblog.yelp.com/2016/01/dumb-init-an-init-for-docker.html)，因为它是静态链接的并且占用空间很小。下面是我们将如何设置它：

```dockerfile
RUN apk add dumb-init
CMD ["dumb-init", "node", "server.js"]
```

我们更新一下dockerfile，你会注意到我们在镜像声明之后放置了 `dumb-init` 包安装，所以我们可以利用 Docker 的层缓存：

```dockerfile
FROM node:lts-alpine@sha256:b2da3316acdc2bec442190a1fe10dc094e7ba4121d029cb32075ff59bb27390a
RUN apk add dumb-init
ENV NODE_ENV production
WORKDIR /usr/src/app
COPY --chown=node:node . .
RUN npm ci --only=production
USER node
CMD ["dumb-init", "node" "server.js"]
```

Tips: `docker kill` 和 `docker stop` 命令仅向具有 `PID 1` 的容器进程发送信号。如果你正在运行一个 shell 脚本来运行你的 Node.js 应用程序，那么请注意一个 shell 实例——比如 `/bin/sh`,例如,不向子进程转发信号，这意味着你的应用程序永远不会得到 `SIGTERM`。



## 6. 优雅终止Node web应用

如果我们已经在讨论终止应用程序的进程信号，那么让我们确保在不干扰用户的情况下正确且优雅地关闭它们。

当 Node.js 应用程序接收到一个中断信号（也称为 `SIGINT` 或 `CTRL+C`）时，它将导致进程突然终止，除非任何事件处理程序被设置为以不同的行为处理它。这意味着连接到 Web 应用程序的客户端将立即断开连接。现在，想象一下由 Kubernetes 编排的数百个 Node.js Web 容器，随着需要扩展或管理错误而开启和关闭。这不是一个好的用户体验。

你可以轻松模拟此问题。这是一个普通的 Fastify Web 应用程序示例，接口的默认延迟响应为 60 秒：

```js
fastify.get('/delayed', async (request, reply) => {
  const SECOND_DELAY = 60
  await new Promise(resolve => {
    setTimeout(() => resolve(), SECOND_DELAY)
  })
  return { hello: 'delayed world' }
})

const start = async () => {
  try {
    await fastify.listen(PORT, HOST)
    console.log(`*^!@4=> Process id: ${process.pid}`)
  } catch (err) {
   fastify.log.error(err)
   process.exit(1)
 }
}

start()
```

运行此应用程序，并在运行后向此接口发送一个简单的 HTTP 请求：

```bash
time curl https://localhost:3000/delayed
```

在正在运行的 Node.js 控制台窗口中按 `CTRL+C`，你将看到 `curl` 请求突然退出。这模拟了当容器关闭时你的用户将获得的相同体验。

为了提供更好的体验，我们可以执行以下操作：

1. 为各种终止信号（如 `SIGINT` 和 `SIGTERM`）设置事件处理程序。
2. 处理程序等待清理操作，如数据库连接、正在进行的 HTTP 请求等。
3. 然后处理程序终止 Node.js 进程。

假设使用 Fastify，我们可以让我们的处理程序调用 `fastify.close()` ，它返回一个我们将等待的promise，Fastify 也会小心地以带有 HTTP 状态码 `503` 去响应每一个新的连接，以表示应用程序此时不可用。即：

```js
async function closeGracefully(signal) {
   console.log(`*^!@4=> Received signal to terminate: ${signal}`)
 
   await fastify.close()
   // await db.close() if we have a db connection in this app
   // await other things we should cleanup nicely
   process.exit()
}
process.on('SIGINT', closeGracefully)
process.on('SIGTERM', closeGracefully)
```

诚然，这更像是一个通用的 Web 应用程序问题，而不是与 Dockerfile 相关的问题，但这在编排环境中更为重要。



## 7. 查找并修复 Node.js docker镜像中的安全漏洞

还记得我们是如何讨论小型 Docker 基础镜像对我们的 Node.js 应用程序的重要性的吗？让我们把这个测试付诸实践。

我将使用 [Snyk CLI](https://support.snyk.io/hc/en-us/articles/360003812458-Getting-started-with-the-CLI) 来测试我们的 Docker 镜像。您可以在这里注册一个免费的 Snyk 帐户：

```bash
npm install -g snyk
snyk auth
snyk container test node@sha256:b2da3316acdc2bec442190a1fe10dc094e7ba4121d029cb32075ff59bb27390a --file=Dockerfile
```

第一个命令安装 Snyk CLI，然后从命令行快速登录以获取 API 密钥，然后我们可以测试容器是否存在任何安全问题。结果如下：

```
Organization:      snyk-demo-567
Package manager:   apk
Target file:       Dockerfile
Project name:      docker-image|node
Docker image: node@sha256:b2da3316acdc2bec442190a1fe10dc094e7ba4121d029cb32075ff59bb27390a
Platform:          linux/amd64
Base image:        node@sha256:b2da3316acdc2bec442190a1fe10dc094e7ba4121d029cb32075ff59bb27390a
✓ Tested 16 dependencies for known issues, no vulnerable paths found.
```

Snyk 检测到 16 个操作系统依赖项，包括我们的 Node.js 运行时可执行文件，并且没有发现任何易受攻击的版本。

这很好，但是如果我们使用 `FROM node` 基本指令会发生什么？ 更好的是，假设你使用了更具体的 Node.js docker 基础映像，例如：`FROM node:14.2.0-slim`。

这看起来是一个更好的位置——我们非常专注于 Node.js 版本以及使用 `slim` 类型的镜像，这意味着 Docker 镜像中的依赖项占用空间更小。让我们用 Snyk 测试一下：

```
…

✗ High severity vulnerability found in node
  Description: Memory Corruption
  Info: https://snyk.io/vuln/SNYK-UPSTREAM-NODE-570870
  Introduced through: node@14.2.0
  From: node@14.2.0
  Introduced by your base image (node:14.2.0-slim)
  Fixed in: 14.4.0

✗ High severity vulnerability found in node
  Description: Denial of Service (DoS)
  Info: https://snyk.io/vuln/SNYK-UPSTREAM-NODE-674659
  Introduced through: node@14.2.0
  From: node@14.2.0
  Introduced by your base image (node:14.2.0-slim)
  Fixed in: 14.11.0


Organization:      snyk-demo-567
Package manager:   deb
Target file:       Dockerfile
Project name:      docker-image|node
Docker image:      node:14.2.0-slim
Platform:          linux/amd64
Base image:        node:14.2.0-slim

Tested 78 dependencies for known issues, found 82 issues.

Base Image        Vulnerabilities  Severity
node:14.2.0-slim  82               23 high, 11 medium, 48 low

Recommendations for base image upgrade:

Minor upgrades
Base Image         Vulnerabilities  Severity
node:14.15.1-slim  71               17 high, 7 medium, 47 low

Major upgrades
Base Image        Vulnerabilities  Severity
node:15.4.0-slim  71               17 high, 7 medium, 47 low

Alternative image types
Base Image                 Vulnerabilities  Severity
node:14.15.1-buster-slim   55               12 high, 4 medium, 39 low
node:14.15.3-stretch-slim  71               17 high, 7 medium, 47 low
```

虽然似乎特定的 Node.js 运行时版本（例如 `FROM node:14.2.0-slim`）已经足够好，但 Snyk 能够从两个主要来源中找到安全漏洞：

1. Node.js 运行时本身——你是否注意到上述报告中的两个主要安全漏洞？这些是 Node.js 运行时中众所周知的安全问题。对这些问题的直接修复是升级到较新的 Node.js 版本，Snyk 会告诉您该版本并告诉您修复它的版本 - 14.11.0，如你在输出中所见。
2. 此 debian 基础映像中安装的工具和库，例如 glibc、bzip2、gcc、perl、bash、tar、libcrypt 等。虽然容器中的这些易受攻击的版本可能不会立即构成威胁，但如果我们不使用它们，为什么还要使用它们呢？

这份 Snyk CLI 报告中最好的部分是什么？ Snyk 还建议切换到其他基本图像，因此你不必自己弄清楚。寻找替代图像可能非常耗时，因此 Snyk 为你省去了这个麻烦。

我现阶段的建议如下：

1. 如果你在registry（例如 Docker Hub 或 Artifactory）中管理 Docker 映像，则可以轻松地将它们导入 Snyk，以便平台为你找到这些漏洞。这还将在 Snyk UI 中为你提供推荐建议，并持续监控您的 Docker 镜像以发现新发现的安全漏洞。
2. 在 CI 自动化中使用 Snyk CLI。 CLI 非常灵活，这正是我们创建它的原因——因此你可以将其应用于你拥有的任何自定义工作流程。我们也有用于 GitHub Actions 的 Snyk，如果你喜欢那些 



## 8. 使用多阶段构建

多阶段构建是从简单但可能有错误的 Dockerfile 转移到构建 Docker 镜像的单独步骤的好方法，因此我们可以避免泄露敏感信息。不仅如此，我们还可以使用更大的 Docker 基础镜像来安装我们的依赖项，如果需要编译任何原生 npm 包，然后将所有这些工件复制到一个小的生产基础镜像中，就像我们的 alpine 示例一样。



> 阻止敏感信息泄露

如果你正在为工作构建 Docker 镜像，那么你很有可能还维护私有 npm 包。如果是这种情况，那么你可能需要找到某种方法使该秘密 `NPM_TOKEN` 可用于 npm 安装。例如：

```dockerfile
FROM node:lts-alpine@sha256:b2da3316acdc2bec442190a1fe10dc094e7ba4121d029cb32075ff59bb27390a
RUN apk add dumb-init
ENV NODE_ENV production
ENV NPM_TOKEN 1234
WORKDIR /usr/src/app
COPY --chown=node:node . .
#RUN npm ci --only=production
RUN echo "//registry.npmjs.org/:_authToken=$NPM_TOKEN" > .npmrc && \
   npm ci --only=production
USER node
CMD ["dumb-init", "node", "server.js"]
```

但是，这样做会在 Docker 镜像中留下带有秘密 npm 令牌的 `.npmrc` 文件。你可以尝试通过之后删除它来改进它，如下所示：

```dockerfile
RUN echo "//registry.npmjs.org/:_authToken=$NPM_TOKEN" > .npmrc && \
   npm ci --only=production
RUN rm -rf .npmrc
```

但是，现在 .npmrc 文件在 Docker 镜像的不同层中是可获取的。如果这个 Docker 镜像是公开的，或者有人能够以某种方式访问它，那么你的令牌就会被泄露。更好的改进如下：

```dockerfile
RUN echo "//registry.npmjs.org/:_authToken=$NPM_TOKEN" > .npmrc && \
   npm ci --only=production; \
   rm -rf .npmrc
```

现在的问题是 Dockerfile 本身需要被视为秘密资源，因为它包含在其中的秘密 npm 令牌。

幸运的是，Docker 支持一种将参数传递到构建过程的方法：

```dockerfile
ARG NPM_TOKEN
RUN echo "//registry.npmjs.org/:_authToken=$NPM_TOKEN" > .npmrc && \
   npm ci --only=production; \
   rm -rf .npmrc
```

然后使用下面命令构建镜像：

```bash
docker build . -t nodejs-tutorial --build-arg NPM_TOKEN=1234
```

我知道你在想我们现在都完成了，但很抱歉让你失望了.

还有什么问题，你想一想？像这样传递给 Docker 的构建参数保存在历史日志中。运行此命令：`docker history nodejs-turorial`

将打印：

```
IMAGE          CREATED              CREATED BY                                      SIZE      COMMENT
b4c2c78acaba   About a minute ago   CMD ["dumb-init" "node" "server.js"]            0B        buildkit.dockerfile.v0
<missing>      About a minute ago   USER node                                       0B        buildkit.dockerfile.v0
<missing>      About a minute ago   RUN |1 NPM_TOKEN=1234 /bin/sh -c echo "//reg…   5.71MB    buildkit.dockerfile.v0
<missing>      About a minute ago   ARG NPM_TOKEN                                   0B        buildkit.dockerfile.v0
<missing>      About a minute ago   COPY . . # buildkit                             15.3kB    buildkit.dockerfile.v0
<missing>      About a minute ago   WORKDIR /usr/src/app                            0B        buildkit.dockerfile.v0
<missing>      About a minute ago   ENV NODE_ENV=production                         0B        buildkit.dockerfile.v0
<missing>      About a minute ago   RUN /bin/sh -c apk add dumb-init # buildkit     1.65MB    buildkit.dockerfile.v0
```

通过历史记录可以看到我们 `NPM_TOKEN`.

有一个很好的方法来管理容器镜像的secrets，是时候引入多阶段构建来缓解这个问题，并展示我们如何构建最小的镜像。



> 引入多阶段构建

就像关注点分离的软件开发原则一样，我们将应用相同的想法来构建我们的 Node.js Docker 镜像。我们将有一个镜像，用于构建运行 Node.js 应用程序所需的一切，在 Node.js 世界中，这意味着安装 npm 包，并在必要时编译本机 npm 模块。**那将是我们的第一阶段。**

第二个 Docker 镜像代表 Docker 构建的第二阶段，将是生产阶段 Docker 镜像。这第二个也是最后一个阶段是我们实际优化并发布到注册表的镜像（如果有的话）。我们将称为 `build` 镜像的第一个镜像丢弃，并在构建它的 Docker 主机中作为悬空镜像留下，直到它被清除。

更新一下我们dockerfile，将其划分为2个阶段：

```dockerfile
# --------------> The build image
FROM node:latest AS build
ARG NPM_TOKEN
WORKDIR /usr/src/app
COPY package*.json /usr/src/app/
RUN echo "//registry.npmjs.org/:_authToken=$NPM_TOKEN" > .npmrc && \
   npm ci --only=production && \
   rm -f .npmrc
   
# --------------> The production image
FROM node:lts-alpine@sha256:b2da3316acdc2bec442190a1fe10dc094e7ba4121d029cb32075ff59bb27390a
RUN apk add dumb-init
ENV NODE_ENV production
USER node
WORKDIR /usr/src/app
COPY --chown=node:node --from=build /usr/src/app/node_modules /usr/src/app/node_modules
COPY --chown=node:node . /usr/src/app
CMD ["dumb-init", "node", "server.js"]
```

如你所见，我为 `build` 阶段选择了一个较大的镜像，因为我可能使用到 `gcc` 工具去编译原生的npm包，或者其它工具。

在第2个阶段，`COPY` 指令有个特殊的注解，用于将 `node_modules/` 文件夹从 `build` docker镜像拷贝到新的生产基础镜像中。

同时，你可以发现 `NPM_TOKEN` 作为 `build` 参数传入到docker镜像中。同时在生产阶段中使用 `docker history nodejs-tutorial` 命令，再也无法找到这些私密信息。



## 9. 在镜像中排除不必要的文件

和 `.gitignore` 一样，docker也有个 `.dockerignore` 用于排除不必要的文件。例如：

```
.dockerignore
node_modules
npm-debug.log
Dockerfile
.git
.gitignore
```

如你所见，忽略 `node_modules/` 文件是很重要的，避免将其拷贝到镜像中。

另外，在使用多阶段构建docker镜像时， `.dockerignore` 的作用更大。我们第2阶段构建如下：

```dockerfile
# --------------> The production image
FROM node:lts-alpine
RUN apk add dumb-init
ENV NODE_ENV production
USER node
WORKDIR /usr/src/app
COPY --chown=node:node --from=build /usr/src/app/node_modules /usr/src/app/node_modules
COPY --chown=node:node . /usr/src/app
CMD ["dumb-init", "node", "server.js"]
```

当我们执行 `COPY . /usr/src/app` 时，这会将 `node_modules/` 拷贝到镜像中。

最重要的是，由于我们使用了通配符 `COPY .` 我们也可能会复制到 Docker 镜像敏感文件中，其中包括凭据或本地配置。

`.dockerignore` 文件的要点是：

1. 跳过潜在可能被修改的 `node_modules/` 文件到镜像中
2. 使你免于泄露机密，例如 `.env` 或 `aws.json` 文件内容中的凭据进入 Node.js Docker 映像。
3. 它有助于加速 Docker 构建，因为它忽略了，否则会导致缓存的文件失效。例如，如果一个日志文件被修改，或者一个本地环境配置文件，所有这些都会导致 Docker 镜像缓存在复制本地目录的那一层失效。



## 10. 将secrets挂载到Docker构建镜像中

关于 `.dockerignore` 文件需要注意的一点是，它是一种全有或全无的方法，不能在 Docker 多阶段构建中的每个构建阶段打开或关闭。

它为什么如此重要？理想情况下，我们希望在构建阶段使用 `.npmrc` 文件，因为我们可能需要它，因为它包含一个秘密 npm 令牌来访问私有 npm 包。也许它还需要一个特定的代理或注册表配置来从中提取包。

这意味着让 .npmrc 文件可用于 `build` 阶段是有意义的,然而，我们在生产镜像的第二阶段根本不需要它，我们也不希望它存在，因为它可能包含敏感信息，比如秘密的 npm 令牌。

缓解这种 .dockerignore 警告的一种方法是挂载一个可用于构建阶段的本地文件系统，但还有更好的方法。

Docker 支持一种相对较新的功能，称为 Docker secrets，并且非常适合我们需要使用 `.npmrc` 的情况。下面是它的工作原理：

- 当我们运行 `docker build` 命令时，我们将指定命令行参数来定义一个新的 secret ID 并引用一个文件作为secret的来源;
- 在 Dockerfile 中，我们将向 `RUN` 指令添加标志以安装生产 npm，它将secret ID 引用的文件安装到目标位置——本地目录 `.npmrc` 文件，这是我们希望它可用的地方;
- `.npmrc` 文件作为机密挂载，永远不会复制到 Docker 镜像中;
- 最后，不要忘记将 `.npmrc` 文件添加到 `.dockerignore` 文件的内容中，这样无论是构建镜像还是生产镜像，它都不会进入镜像

让我们看看所有这些是如何协同工作的。首先是更新的 `.dockerignore` 文件：

```
.dockerignore
node_modules
npm-debug.log
Dockerfile
.git
.gitignore
.npmrc
```

然后，完整的 Dockerfile，带有更新的 `RUN` 指令以在指定 `.npmrc` 挂载点的同时安装 npm 包：

```dockerfile
# --------------> The build image
FROM node:latest AS build
WORKDIR /usr/src/app
COPY package*.json /usr/src/app/
RUN --mount=type=secret,mode=0644,id=npmrc,target=/usr/src/app/.npmrc npm ci --only=production

# --------------> The production image
FROM node:lts-alpine
RUN apk add dumb-init
ENV NODE_ENV production
USER node
WORKDIR /usr/src/app
COPY --chown=node:node --from=build /usr/src/app/node_modules /usr/src/app/node_modules
COPY --chown=node:node . /usr/src/app
CMD ["dumb-init", "node", "server.js"]
```

最后，构建 Node.js Docker 镜像的命令：

```bash
docker build . -t nodejs-tutorial --secret id=npmrc,src=.npmrc
```

注意：`Secrets` 是 Docker 中的一项新功能，如果您使用的是旧版本，则可能需要按如下方式启用它 Buildkit：

```bash
DOCKER_BUILDKIT=1 docker build . -t nodejs-tutorial --build-arg NPM_TOKEN=1234 --secret id=npmrc,src=.npmrc
```



## 总结

最后一步总结了有关容器化 Node.js Docker Web 应用程序的整个指南，考虑了与性能和安全相关的优化，以确保我们构建生产级 Node.js Docker 镜像！

扩展阅读：

- [10 Docker Security Best Practices](https://snyk.io/blog/10-docker-image-security-best-practices/)
- [Docker for Java developers: 5 things you need to know not to fail your security](https://snyk.io/blog/docker-for-java-developers/)



原文链接：

- [10 best practices to containerize Node.js web applications with Docker](https://snyk.io/blog/10-best-practices-to-containerize-nodejs-web-applications-with-docker/)

相关链接：

- [nodejs-docker-best-practices - github](https://github.com/snyk-labs/nodejs-docker-best-practices)



2021年08月30日16:02:17