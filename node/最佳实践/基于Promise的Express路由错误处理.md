## 1. Express.js Routes的一般架构

实际项目中我们可能使用MongoDB存储数据，这里为了简便，使用mock数据。下面使用 `express-generator` 生成项目骨架，移除一些不必要的文件：

```bash
bin
  start.js
node_modules
routes
  users.js
services
  userService.js
app.js
package-lock.json
package.json
```

下面是 `app.js` 文件：

```js
const createError = require('http-error')
const express = require('express')
const cookieParser = require('cookie-parser')
const usersRouter = require('./routes/users')

const app = express()
app.use(express.json())
app.use(express.urlencoded({ extended: false }))
app.use(cookieParser())
app.use(function(req, res, next) {
  next(createError(404))
})

app.use(function(err, req, res, next) {
  res.status(err.status || 500)
  res.send(err)
})

module.exports = app
```

上面创建了express.js app,添加了中间件支持JSON，URL编码，cookie解析。同时对 `users` 添加了 `usersRouter`。最后，如果没有找到路由，如何处理错误，我们将之后更改错误处理的方式。

`bin/start.js` 开启服务：

```js
const app = require('../app')
const http = require('http')

const port = process.env.PORT || 3000

const server = http.createServer(app)
server.listen(port)
```

`package.json` 基本内容如下：

```json
{
  "name": "express-promises-example",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "start": "node ./bin/start.js"
  },
  "dependencies": {
    "cookie-parser": "~1.4.4",
    "express": "~4.16.1",
    "http-errors": "~1.6.3"
  }
}
```

`routes/users.js` 是一个一般的user路由实现：

```js
const express = require('express')
const router = express.Router()

const userService = require('../services/userService')

router.get('/', function(req, res) {
  userService.getAll()
  	.then(result => res.status(200).send(result))
  	.catch(err => res.status(500).send(err))
})

router.get('/:id', function(req, res) {
  userService.getById(req.params.id)
  	.then(result => res.status(200).send(result))
  	.catch(err => res.status(500).send(err))
})

module.exports = router
```

它有2个路由: `/`用户获取所有users，`/:id` 获取单个用户，同时使用 `/services/userServices.js`, 它是一个基于promise的获取数据的方法：

```js
const users = [
  {id: '1', fullName: 'User The First'},
  {id: '2', fullName: 'User The Second'}
]

const getAll = () => Promise.resolve(users)
const getById = (id) => Promise.resolve(users.find(u => u.id === id))

module.exports = {
  getAll,
  getById
}
```

这里我们没有使用ORM（比如Mongoose或者Sequelize）,只是简单的模拟数据的获取



##  2. Express.js路由的问题

观察我们的路由handlers，每个服务调用，都重复的用到了 `.then(...) & .catch(...)` 回调发送数据或者返回错误给客户端。

初看之下，这没什么问题，但是真实项目中，我们可能需要显式特定错误或者忽略通用的 `50X` 错误。另外，可能依据环境是否使用这一逻辑。想象一下，如果我们项目路由从2个增加到200个？



### 2.1 方式1：使用工具函数

我们可能触及单独的工具函数处理 `resolve & reject`, 然后在express.js路由中使用：

```js
// 位于 '/utils' 目录下的某些response handler
const handleResponse = (res, data) => res.status(200).send(data)
const handleError = (res, err) => res.status(500).send(err)

// routes/users.js
router.get('/', function(req, res) {
  userService.getAll()
  	.then(data => handleResponse(res, data))
  	.catch(err => handleError(res, error))
})

router.get('/:id', function(req, res) {
  userService.getById(req.params.id)
  	.then(data => handleResponse(res, data))
  	.catch(err => handleError(res, error))
})
```

看起来还不错，我们不必重复实现数据和错误的放松，但是我们必须在每个路由中都引入这些handlers，并将其传入到 `then() & catch()` 中。



### 2.2 中间件

另一种方式就是使用Express.js围绕promise的最佳实践：将错误发送逻辑一道Express的 error 中间件中（添加到app.js中），并使用 `next` 回调将异步错误放在中间件中。最简单的错误中间件如下，使用一个匿名函数：

```js
app.use(function(err, req, res, next) {
  res.status(err.status || 500)
  res.send(err)
})
```

Express能理解这是用于处理错误的，因为函数签名参数是4个。

使用 `next` 传递错误，可能如下：

```js
// 位于 '/utils' 目录下的某些response handler
const handleResponse = (res, data) => res.status(200).send(data)

// routes/users.js
router.get('/', function(req, res, next) {
  userService.getAll()
  	.then(data => handleResponse(res, data))
  	.catch(next)
})

router.get('/:id', function(req, res, next) {
  userService.getById(req.params.id)
  	.then(data => handleResponse(res, data))
  	.catch(next)
})
```

即使使用官方最佳实践，我们仍需要在每个路由handler中使用 `handleResponse() & next` 函数。

下面使用更好的方式对其进行简化



### 2.3 基于Promise的中间件

JS的最好的一个特色就是其动态的特性。**我们可以在运行时给对象添加任何字段，我们将利用这一特性扩展Express result对象，而Express中间件是最佳操作位置**



> **promiseMiddleware()** 函数

下面创建promise中间件，使得我们路由更加的灵活，下面将创建一个新的文件 `/middleware/promise.js`:

```js
const handleResponse = (res, data) => 
	res.status(200).send(data)
const handleError = (res, err) => 
	res.status(err.status || 500).send({error: err.message})

module.exports = function promiseMiddleware() {
  return (req, res, next) => {
    res.promise = (p) => {
      let promiseToResolve
      if (p.then && p.catch) {
        promiseToResolve = p
      } else if (typeof p === 'function') {
        promiseToResolve = Promise.resolve().then(() => p())
      } else {
        promiseToResolve = Promise.resolve(p)
      }
      
      return promiseToResolve
      	.then(data => handleResponse(data))
      	.catch(e => handleError(res, e))
    }
    return next()
  }
}
```

在 `app.js` 中，我们对 `app` 对应应用这个中间件，并更新默认错误行为：

```js
// app.js
const promiseMiddleware = require('./middleware/promise')

// ...
app.use(promiseMiddleware)
//...
app.use(function(req, res, next) {
  res.promise(Promise.reject(createError(404)))
})
app.use(function(err, req, res, next) {
  res.promise(Promise.reject(err))
})
```

注意，我们没有忽略错误中间件。对所有同步错误来说，它仍然很重要。为了避免重复，我们使用 `Promise.reject()` 包裹错误，将其传递给 `res.promise()` 方法，然后内部使用 `handleError()` 统一对同步错误进行处理。

这对我们处理类似下面的同步错误很有用：

```js
router.get('/', function(req, res) {
  throw new Error('This is synchronous error!')
})
```

最后，我们在 `/routes/users.js` 中使用 `res.promise()`:

```js
const express = require('express')
const router = express.Router()

const userService = require('../services/userService')

router.get('/', function(req, res) {
  res.promise(userService.getAll())
})

router.get('/:id', function(req, res) {
  res.promise(() => userService.getById(req.params.id))
})

module.exports = router
```

注意上面 `.promise()` 的2中不同用法：我们可以传入一个promise或者函数。对没有promises的方法我们可以传入函数；`.promise` 会将该函数包装成一个promise.

那么它是在给客户端发送错误上更好的地方在哪里呢？这其实是一个代码组织的问题。我们可以在我们的错误中间件（因为它应该处理错误）或我们的promise中间件（因为它已经与我们的响应对象进行了交互）中做到这一点。我决定将所有的response操作都集中存放在promise中间件中进行处理，但决定权在开发者手上。



> 技术上讲，**res.promise()** 是可选的

我们添加了 `res.promise()`， 但我们不一定必须使用它，我们完全可以直接操作我们的response对象。下面我列举这种做法的2种有用的情形：重定向和流管道（stream piping）



**a.特殊情形1：重定向**:

假设我们将用户重定向到另一个URL,我们在 `userService.js` 中添加一个 `getUserProfilePicUrl()` 函数：

```js
const getUserProfilePicUrl = id => Promise.resolve('/img/${id}')
```

我们在users路由中使用 `async-await` 的形式处理response:

```js
router.get('/:id/profilePic', async function(req, res) {
  try {
    const url = await userService.getUserProfilePicUrl(req.params.id)
    res.redirect(url)
  } catch (e) {
    res.promise(Promise.reject(e))
  }
})
```

现在我们可以使用 `async-await` 实现重定向，更重要的是，我们有过一个中心化的地方传递使用 `res.promise()` 任何错误.



**b.特殊情形2：Stream Piping**

和重定向一样，流管道也是需要我们直接操作response的一种情形。

为了操作我们重定向的url，我们添加了一个路由。

首先我们将 `profilePic.jpg` 添加到 `/assets/img` 目录下（实际项目我们可能使用云存储，比如AWS S3，但是流机制是一样的）。

下面我们对 `/img/profilePic/:id` 请求，使用管道该图片进行响应。我们创建一个新路由 `/routes/img.js`:

```js
const express = require('express')
const router = express.Router()

const fs = require('fs')
const path = require('path')

router.get('/:id', function(req, res) {
  // 注意我们基于当前工作路径创建一个path，而不是router文件的位置
  const fileStream = fs.createReadStream(
  	path.join(process.cwd(), './assets/img/profilePic.jpg')
  )
  fileStream.pipe(res)
})

module.exports = router
```

我们将 `/img` 路由添加到 `app.js` 中：

```js
app.use('/users', require('./routes/users'))
app.use('/img', require('./routes/img'))
```

不同于重定向的情形是：我们在img路由中没有使用 `res.promise()`。这是因为一个已经打开的管道response对象传递错误的方式不同于在流中出现错误的方式。

Express开发者在处理流的时候应当注意，**应依据错误发生的时机来不同的处理错误**。我们需要在管道之前处理错误（这里可以使用 `res.promise`）, 在管道中时（使用 `.on('errpr')` handler），这超出了这篇文章讨论的范围。



> 增强 **res.promise()**

当调用 `res.promise()` 时，我们不要被本文的实现所限定，我们可以增强 `promiseMiddleware.js`，比如在 `res.promise()` 中接收一些可选性，允许调用者指定response状态码，内容类型，或者项目中其它需要的。这取决于开发者。



## 3. Express错误处理碰见基于Promise的编码风格

这里介绍的方法允许比我们开始时更优雅的路由handlers和处理结果和错误的单点——即使是那些在 res.promise(...)外，感谢app.js的错误处理。当然，我们并没有强制使用它，我们也可以处理边缘情形。

完整代码在 [github](https://github.com/vssenko/express-promises-example). 在哪里，开发者可以在 `handleResponse()` 中添加自定义逻辑，比如没有数据时返回状态码 `204`，而不是 `200`。

但是对错误的控制更加有用。这种方式帮助我简化实现功能：

- 一致性的格式化错误，比如 `{error: {message}}`
- 如果没有状态，发送通用的消息或者发送特定的消息
- 如果环境变量是 `dev | test`, 生成 `error.stack` 字段
- 处理数据库索引错误，比如某个实体唯一索引字段已经存在，并且优雅的返送有意义的用户错误

这个 Express.js 路由逻辑都在一个地方，没有触及任何服务——一种解耦使代码更容易维护和扩展。这就是简单但优雅的解决方案可以极大地改善项目结构。



进一步阅读：

- [How to Build a Node.js Error Handling System?](https://www.toptal.com/nodejs/node-js-error-handling)

原文链接：

- [Using Express.js Routes for Promise-based Error Handling](https://www.toptal.com/express-js/routes-js-promises-error-handling)
- [express-promises-example - github](https://github.com/vssenko/express-promises-example)

2021年08月24日23:56:39

