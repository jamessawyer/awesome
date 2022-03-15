## #1 what's Mongoose?



优点：

1. 自带schema验证
2. Models
3. Change tracking
4. 中间件 Middleware
5. 插件 Plugins



`collections` & `documents`:

- `collections` 相当于数据库中的表 tables
- `documents` 相当于表中的row



常见步骤：

1. 连接MongoDB
2. 定义Schema
3. 创建model
4. 创建documents，存储documents到MongoDB
5. 从MongoDB加载documents，即查找



`schema` & `model` & `document` 关系：

- schemas是用于配置models的对象
- models是创建和加载documents的主要工具
- models本质是一个js类
- document相当于数据库中的 `row`



## #2 核心概念

5大核心概念：

1. `Model`: 表示存储在MongoDB中的对象的类，它是使用Mongoose和MongoDB交互的主要方式
2. `Schema`: 用于表示 `model` 配置信息的类，当插件schema时，有2个隐式类：`SchemaTypes & VirtualTypes`
3. `Connection`: 表示连接到MongoDB服务进程的一系列sockets的类
4. `Document`: `model` 的实例，可以通过 `save()` 方法将document存储到MongoDB中
5. `Query`: 表示Mongoose发送到MongoDB的一个操作的类

eg:

```js
const mongoose = require('mongoose')

// connection
const connection = mongoose.createConnection()
connection instanceof mongoose.Connection // true

// schema
const schema = new mongoose.Schema({ name: String })

// 创建一个Model 需要一个connection 和 schema
// model 继承自 mongoose.Model
const MyModel = connection.model('ModelName', schema)

// document是model的实例
const document = new MyModel()
document instanceof mongoose.Document // true

// query
const query = MyModel.find()
query instanceof mongoose.Query // true
```



创建model的2种方式：

```js
// 方式1
const Person = mongoose.model('Person', new mongoose.Schema({}))

// 方式2
const conn = mongoose.createConnection()
const Person = conn..model('Person', new mongoose.Schema({}))
```

**创建一个Model 需要一个connection 和 schema**: 因此方式1等价于 使用默认的connection

```js
const Person = mongoose.model('Person', new mongoose.Schema({}))
// 等价于
const Person = mongoose.connection.model('Person', new mongoose.Schema({}))
```



> Documents 3大功能：Casting, Validating, Change Tracks

1️⃣ **Casting**: mongoose会对doc中的字段按照schema中定义的类型进行转换，比如：字符串数字转换为数字类型

```js
const schema = new mongoose.Schema({
  name: String,
  age: Number
})
const MyModel = mongoose.model('MyModel', schema)


const doc = new MyModel({ name: 'james', age: '20' })
// 会将 字符串类型 '20' 转换为 20
await doc.save()
```

如果转换失败，则会抛出 `CastError` 错误。



2️⃣ **Validating**: 在转换后，会进一步对doc进行验证操作，验证失败会抛出 `ValidationError` 错误：

```js
const schema = new mongoose.Schema({
  name: {
    type: String,
    // 内置的 `enum` 验证器，确保name是以下枚举值
    enum: ['aa', 'bb', 'cc']
  }
})
const Person = mongoose.model('Person', schema)
const doc = new Person({ name: 'dd' })
await doc.save().catch(error => {
  error.errors['name'] // ValidationError 'dd' is not a valid enum value for path 'name'
  Object.keys(error.errors) // ['name']
})
```

当调用 `save()` 方法时，内部其实调用的是 `validate()` 方法，它是一个异步的方法，我们可以手动的调用这个方法 `await doc.validate()`。

✅ mongoose内置了一些验证器，不同数据类型存在的验证器不一样：对Strings类型使用Numbers类型验证器是无效的。

- Strings: `enum & match & minlength & maxlength`
- Numbers: `min & max`
- Dates: `min & max`
- 所有类型通用： `required`



3️⃣ `Change Tracks`: 只追踪变化部分，更新操作时，只将变化部分发送给MongoDB，节省带宽：

```js
const schema = new mongoose.Schema({
  name: String,
  age: Number
})
const MyModel = mongoose.model('MyModel', schema)

// 获取数据
const doc = await MyModel.findOne() // 假设返回 { name: 'aa', age: 20 }
doc.age = 30 // 只更新 age
doc.modifiedPaths() // 获取更新路径 ['name']
await doc.save() // 更新时，只会发送 `age` 字段到MongoDB
```



### 2.1 Getters & Setters

1️⃣ Getters 用于将存储在MongoDB中的数据转换为更可读的形式, **不会影响底层数据的存储**；Setters用于将用户数据转换为自定义形式，以便存储到MongoDB中。

```js
const schema = new mongoose.Schema({email: String})
schema.path('email').get(v => v.replace('@', ' [at] '))

const Model = mongoose.model('User', schema)
const doc = new Model({ email: 'aa@qq.com' })
// 读取时
doc.email // 'aa [at] qq.com'
doc.get['email'] // 'aa [at] qq.com'

// 存储到MongoDB中的仍然是 `aa@qq.com`
await doc.save()
```

禁用getter的2种方式：

```js
// 将doc转换为 JSON时 禁用getters
const userSchema = new mongoose.Schema({
  email: { type: string, get: obfuscate }
}, { toJSON: { getters: false }})

// doc.get 时跳过getters
doc.get('eamil', null, { getters: false })
```



2️⃣ setters用于存储数据前的数据转换：

```js
const userSchema = new mongoose.Schema({ email: String })
// setter 转换为小写
userSchema.path('email').set(v => v.toLowerCase())
const User = mongoose.model('User', userSchema)

const user = new User({ email: 'TEST@gmail.com' })

await user.save() // 存储的为 `test@gmail.com`
```

下面方式都会调用 `setters`:

```js
user.email = 'TEST@gmail.com'
user.set('email', 'TEST@gmail.com')
Object.assign(user, { email: 'TEST@gmail.com' })
```

下面操作也会调用 `setters`:

- `updateOne()`
- `updateMany()`
- `findOneAndUpdate()`
- `findOneAndReplace()`

例如：

```js
const { _id } = await User.create({ email: 'test@gmail.com' })
// 这里会调用上面的setter
await User.updateOne({ _id }, { email: 'NEW@gmail.com' })

const doc = await User.findOne({ _id })
doc.email // 'new@gmail.com'
```



> getters & setters的高级用法

一般MongoDB存储的 `ObjectId` 都是以字符串的形式表示，但是实际上它是一个对象类型，因此比较的时候，不能使用js中的 `=== | ==` 直接去比较：

```js
const str = '5d124083fc741d44eca250fd'
const schema = new mongoose.Schema({ objectid: mongoose.ObjectId })
const Model = mongoose.model('ObjectIdTest', schema)
const doc1 = new Model({ objectid: str })
const doc2 = new Model({ objectid: str })

typeof doc1.objectid // 'object' 类型
doc1.objectid instanceof mongoose.Types.ObjectId // true

doc1.objectid === doc2.objectid // false
doc1.objectid == doc2.objectid // false
```

此时，我们可以使用getters将 `objectid` 属性转换为字符串类型进行比较：

```js
const str = '5d124083fc741d44eca250fd'
const schema = new mongoose.Schema({ objectid: mongoose.ObjectId })
schema.path('objectid').get(v => v.toString())
const Model = mongoose.model('ObjectIdTest', schema)
const doc1 = new Model({ objectid: str })
const doc2 = new Model({ objectid: str })

typeof doc1.objectid // 'string' 类型
doc1.get('objectid', null, { getters: false }) // 'object' 存储到MongoDB中的原始值仍然是 ObjectId 类型

doc1.objectid === doc2.objectid // true
doc1.objectid == doc2.objectid // true
```

⚠️ **如果使用getter将raw value转换成不同类型，则同时应该使用setter将其转换回原来的类型**。比如MongoDB提供了 `Decimal128` 类型存储金融浮点数，我们不能直接用js的number类型去加减Decimal类型：

```js
const accountSchema = new mongoose.Schema({ balance: mongoose.Decimal128 })
const Account = mongoose.model('Account', accountSchema)

await Account.create({ balance: 0.1 })
await Account.update({}, { $inc: { balance: 0.2 } })

const account = await Account.findOne()
account.balance.toString() // 0.3

// 直接将 Decimal128 和js的number类型相加 MongoDB暂不支持
account.balance += 0.5
```

可以使用getter & setter 的方式进行操作：

```js
const accountSchema = new mongoose.Schema({
  balance: {
    type: mongoose.Decimal128,
    // 当访问balance时，将原始的decimal类型先转换为js number类型
    get: v => parseFloat(v.toString()),
    // 当设置 balance 属性时，保留2位小数精度，同时再转换回 decimal 类型
    set: v => mongoose.Types.Decimal128.fromString(v.toFixed(2))
  }
})

const Account = mongoose.model('Account', accountSchema)

const account = new Account({ balance: 0.1 })
account += 02 // 0.3 ✅
account.balance
```



### 2.2 Virtuals

这个相当于 **计算属性** ，特点：

- 不会真正存储到MongoDB中，因此直接通过计算属性去查，是没有结果的
- 转换为 `JSON` 类型时（比如express中的 `res.json()`）,默认是不会带上virtuals的，但是可以进行设置

```js
const userSchema = new mongoose.schema({ email: String })
userSchema.virtuals('domain').get(function() {
  return this.email.slice(this.email.indexOf('@') + 1)
})
const User = mongoose.Model('User', userSchema)

let doc = await User.create({ email: 'test@gmail.com' })
doc.domain // 'gmail.com'

// 通过 domain 查找，是获取不到结果的 因为domain并没有真正被存储到MongoDB中
const doc1 = await User.findOne({ domain: 'gmail.com' }) // null
```

JSON类型：

```js
const opts = { toJSON: {  virtuals: true }} // 设置json
const userSchema = new mongoose.schema({ email: String }, opts)
userSchema.virtuals('domain').get(function() {
  return this.email.slice(this.email.indexOf('@') + 1)
})
const User = mongoose.Model('User', userSchema)

let doc = await User.create({ email: 'test@gmail.com' })
// { _id: ..., email: 'test@gmail.com', domain: 'gmail.com' }
doc.toJSON()
```

也可以全局设置这个属性：

```js
mongoose.set('toJSON', { virtuals: true })
```



### 2.3 Queries

当我们上面调用 `Model.findOne()` 时，看起来返回的是一个promise，但实际上，它返回的是 **`Query`** 的实例：

```js
const query = Model.findOne()
query instanceof mongoose.Query // true
query instanceof Model.Query // true

const doc = await = query // 执行query
```

返回Query相比于Promise的优势是， **可以进行一些链式调用，比如 `where() | equals()`**:

```js
const doc = await Model.findOne()
	.where('name').equals('Jimmy')
```

Query 有一个 `op` 属性，用于表示发送给MongoDB的是什么操作：

```js
const query = Model.findOne()
query.op // 'findOne'
```

Query类存在下面一些 `op` 值：

- `find`
- `findOne`
- `findOneAndUpdate`
- `findOneAndDelete`
- `findOneAndReplace`
- `deleteOne`
- `deleteMany`
- `replaceOne`
- `updateOne`
- `updateMany`

Mongoose 不会自动的执行query，可以通过 `exec()` | `then()` 显式的执行，当调用 `await` 时，js运行时底层会调用 `then()`:

```js
// exec() 执行query 返回一个promise
const promise1 = Model.findOne().exec()

// then() 底层调用 exec() 返回一个promise
const promise2 = Model.findOne().then(doc => doc)

// await 底层调用 then()
const doc = await Model.findOne()
```



### 2.4 Mongoose Globals & 多Connections

mongoose全局有个 `connection` 属性，它是 `mongoose.Connection` 类实例，当调用 `mongoose.connect()` 时，相当于调用 `mongoose.connection.openUri()`:

```js
const mongoose = require('mongoose')

const m1 = new mongoose.Mongoose()
const m2 = new mongoose.Mongoose()

m1.connection instanceof mongoose.Connection // true
m1.connections.length // 1
m1.connections[0] === m1.connection // true

m1.connection.readyState // 0 'disconnected'
m1.connect('mongodb://localhost:27017/test', { useNewUrlParser: true })
m1.connection.readyState // 2 'connecting'
```

当调用 `mongoose.createConnection()` 时，会创建一个新 `connection` 对象，Mongoose 追踪 `mongoose.connections` 属性：

```js
const m1 = new mongoose.Mongoose()

const conn = m1.createConnection('mongodb://localhost:27017/test', { useNewUrlParser: true })
m1.connections.length // 2
m1.connections[1] === conn // true
```

一般情况下我们只需要1个connection，可能需要多connections的情景：

1. 应用需要访问不同的databases，一个connection只能访问一个database，如果models在不同的databases中，则需要多个connections
2. 利用多connections提升查询性能

```js
const con1 = mongoose.createConnection('mongodb://localhost:27017/db1', { useNewUrlParser: true })
const con2 = mongoose.createConnection('mongodb://localhost:27017/db2', { useNewUrlParser: true })

const Model1 = con1.model('Test', mongoose.Schema({ name: String }))
const Model2 = con1.model('Test', mongoose.Schema({ name: String }))
```

