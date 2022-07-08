目录：

1. [The problem: initializing a perperty asynchronously](#1)
2. [Solution: Promise-based constructor](#2)
   1. [Using an immediately-invoked asynchronous arrow function]()
3. [Solution: static factory method](#3)
   1. [Improvement: private constructor via secret token](#3.1)
   2. [Improvement: constructor throws, factory method borrows the class prototype](#3.2)
   3. [Improvement: instances are inactive by default, activated by factory methoed](#3.3)
   4. [Variant: separate factory function](#3.4)
4. [Subclassing a Promised-based constructor(optional)](#4)
5. [Conclusion](#5)
6. [Further reading](#6)



这一章中，探索几种创建类实例的方式：构造器，工厂函数等等。通过多种方式解决同一个实际问题。本章的关注点是类，所以其它和类不相关的都被忽略。



<p id="1"></p>



## 1️⃣ 问题：异步初始化一个属性

下面容器类应该 **异步的** 接受它的属性 `data` 的内容。下面是第一次尝试：

```js
class DataContainer {
  #data // A
  constructor() {
    Promise.resolve('downloaded')
      .then(data => this.#data = data) // B
  }

  getData() {
    return 'Data: ' + this.#data // C
  }
}

const dc = new DataContainer()
console.log(dc.getData())

setTimeout(() => {
  console.log(dc.getData())
}, 0)

// 打印结果
// DATA: undefined
// Data: downloaded
```

`A行` 声明了一个 [私有属性](https://2ality.com/2019/07/private-class-fields.html) `#data`, 在 `B` & `C` 行使用了该私有属性。

🤔 在 `DataContainer` 构造器内的Promise异步敲定（`settlement`）, 这也是为什么只有当我们完成当前任务，然后通过 `setTimeout` 开启一个新的任务才能看到最终的 `.data` 值。换句话说，*当我们第一次看到 `DataContainer` 实例时，它没有完全实例化😥*。



<p id="2"></p>



## 2️⃣ ⭐️解决方案：基于Promise的构造器

那假如我们延迟访问 `DataContainer` 实例直到它完全实例化呢？**📚我们可以通过在构造器中返回一个Promise就可以完成。** 默认情况下，构造函数返回它所属的类的新实例，我们可以显式的返回一个对象覆盖这种默认行为。

```js
class DataContainer {
  #data;
  constructor() {
    // 😎 返回一个Promise 覆盖默认返回的类的实例
    return Promise.resolve('downloaded')
      .then(data => {
        this.#data = data
        return this // A
      })
  }

  getData() {
    return 'DATA: ' + this.#data
  }
}

new DataContainer()
  .then(dc => { // B
  	console.log(dc.getData())
  })

// 打印
// Data: downloaded
```

现在我们必须等到我们可以访问我们的实例（`B行`）。数据 “下载完”（`downloaded`）后传给我们（`A行`）。上面代码存在2种可能错误的来源：

1. 下载可能失败，产生一个拒绝状态（`rejection`）
2. 在第一个 `.then()` 回调函数体中可能抛出异常

不论那种情况，错误都会成为从构造函数中返回的Promise的拒绝状态（`rejection`）。

👩🏻‍🏫 优点和缺点：

- 这种方式的一个好处就是，只有该实例完全初始化后我们才能可以访问。并且没有其它方式创建 `DataContainer` 实例。
- 缺点就是，该类构造器返回的是一个Promise，而不是该类的一个实例，这可能令人感到惊讶。😮



<p id="2.1"></p>



### 2.1 使用一个立即调用的异步箭头函数

除了直接使用 Promise API 来创建从构造函数返回的 Promise，我们还可以使用我们 [立即调用](https://exploringjs.com/impatient-js/ch_async-functions.html#immediately-invoked-async-arrow-functions) 的异步箭头函数：

```js
class DataContainer {
  #data
  constructor() {
    return (async () => {
      this.#data = await Promise.resolve('downloaded')
      return this // A
    })()
  }

  getData() {
    return 'DATA: ' + this.#data
  }
}

const dc = new DataContainer()
dc.then(dc => {
  console.log(dc.getData())
})

// 打印
// DATA: downloaded
```



<p id="3"></p>



## 3️⃣ 解决方案：静态工厂方法 🤩

类 `C` 的静态工厂方法会创建一个类 `C` 的实例，这是使用 `new C()` 外的另一种选择。JS中常见的静态工厂🏭方法名字有：

1. `.create()`：创建一个新的实例。例子： `Object.create()`
2. `.from()`：基于不同的对象通过拷贝或者转换的方式创建一个新的实例。例子：`Array.from()`
3. `.of()`：通过参数组合的方式创建一个新的实例。例子：`Array.of()`

下面例子中，`DataContainer.create()` 是一个静态工厂函数。它返回 DataContainer 实例的 Promises：

```js
class DataContainer {
  #data

  static async create() {
    const data = await Promise.resolve('downloaded')
    return new this(data) // 💡 这里调用了DataContainer的构造函数
  }

  constructor(data) {
    this.#data = data
  }

  getData() {
    return 'DATA: ' + this.#data
  }
}

DataContainer.create()
  .then(dc => dc.getData())
  .then(console.log)

// 打印
// DATA: downloaded
```

现在，所有的异步功能都包含在了 `.create()` 中，这也使得类的其余部分都完全同步，因此更加简洁。

👩🏻‍🏫 优点和缺点：

- 优点：构造器变得十分简洁
- 缺点：可能通过 `new DataContainer()` 的方式，导致不能正确的创建实例



<p id="3.1"></p>



### 3.1 ⭐️改进：通过私密token创建私有构造器

如果你想实例总是被正确的创建，我们必须保证只有 `DataContainer.create()` 才能调用构造器函数。我们可以通过私密token的方式完成：

```js
// 🤩 秘密token 不向外导出 因此外部无法使用该token 从而保证构造器无法直接调用 只能调用工厂函数
const secretToken = Symbol('DataContainer')

class DataContainer {
  #data

  static async create() {
    const data = await Promise.resolve('downloaded')
    return new this(secretToken, data)
  }

  constructor(token, data) {
    if (token !== secretToken) {
      throw new Error('请使用DataContainer.create()创建实例')
    }
    this.#data = data
  }

  getData() {
    return 'DATA: ' + this.#data
  }
}

DataContainer.create()
  .then(dc => dc.getData())
  .then(console.log)

// 打印
// DATA: downloaded

// ❌
// 如果使用构造器调用
const dc = new DataContainer(Symbol('DataContainer'), 'xxx')

// 直接抛出错误
// Error: 请使用DataContainer.create()创建实例
```

如果 `secretToken` 和 `DataContainer` 在同一个模块下，只导出 `DataContainer`，则外部模板是无法访问 `secretToken`，因此是无法创建 `DataContainer` 实例的。



👩🏻‍🏫 优点和缺点：

- 优点：安全和直白
- 缺点：有点复杂冗余



<p id="3.2"></p>



### 3.2 改进：构造器抛出错误，工厂函数借用类原型

下面解决方案变种中，`DataContainer` 构造函数被禁用，而是通过一个技巧的方式创建实例（`A行`）:

```js
class DataContainer {
  static async create() {
    const data = await Promise.resolve('downloaded')
    return Object.create(this.prototype)._init(data) // A
  }

  constructor() {
    throw new Error('构造器是私有的，请通过DataContainer.create()创建实例')
  }

  _init(data) {
    this._data = data
    return this
  }

  getData() {
    return 'DATA: ' + this._data
  }
}

DataContainer.create()
  .then(dc => {
    console.log(dc instanceof DataContainer) // B
    return dc.getData()
  })
  .then(console.log)


// 打印
// true
// DATA: downloaded
```

在内部，`DataContainer` 的实例是其原型为 `DataContainer.prototype` 的任何对象。这也是为什么我们可以通过 `Object.create()` (`A行`) 创建实例，也是为什么 `B` 行返回 `true`。



👩🏻‍🏫 优点和缺点：

- 优点：优雅；`instanceof` 也能正常工作（译者注：其余的方式都是通过返回 `Promise` 的方式）
- 缺点：
  - 创建实例没有被完全被阻止。不过，公平地说，通过 `Object.create()` 的变通方法也可以用于我们之前的解决方案。
  - 不能在DataContainer中使用 [私有字段](https://2ality.com/2019/07/private-class-fields.html) 和 [私有方法](https://2ality.com/2019/07/private-methods-accessors-in-classes.html)，因为这些仅针对通过构造函数创建的实例才能被正确设置。



<p id="3.3"></p>



### 3.3 改进：实例默认不激活，只能通过工厂方法激活

另一种更冗余的变种方式是，通过 `.#active` 标志关闭实例。开启实例的方法 `.#init()` 不能被访问访问，但是 `Data.container()` 可以调用它

```js
class DataContainer {
  #data
  static async create() {
    const data = await Promise.resolve('downloaded')
    return new this().#init(data) // 调用私有方法
  }

  #active = false

  constructor() {}

  #init(data) {
    this.#active = true
    this.#data = data
    return this
  }

  getData() {
    this.#check()
    return 'DATA: ' + this.#data
  }

  #check() {
    if (!this.#active) {
      throw new Error('请使用DataContainer.create()创建实例')
    }
  }
}


DataContainer.create()
  .then(dc => dc.getData())
  .then(console.log)

// 打印
// DATA: downloaded
```

标志 `.#active` 强制在每个方法开始前都必须调用私有方法 `.#check()`.

这种方法的缺点就是太过冗余。有可能在某个方法中忘记调用 `#check()`。





<p id="3.4"></p>



### 3.4 变种：分离工厂函数

为了完整性，下面展示另一种变种：除了使用静态方法作为工厂外，还可以使用分离版本的工厂函数：

```js
const secretToken = Symbol('secretToken')

class DataContainer {
  #data
  constructor(token, data) {
    if (token !== secretToken) {
      throw new Error('构造器是私有的')
    }
    this.#data = data
  }

  getData() {
    return 'DATA: ' + this.#data
  }
}

async function createDataContainer() {
  const data = await Promise.resolve('downloaded')
  return new DataContainer(secretToken, data)
}

createDataContainer()
  .then(dc => dc.getData())
  .then(console.log)
  
// 打印
// DATA: downloaded
```

单独函数作为工厂优势很有用，但在这里这种情况下，我更偏好静态方法的形式：

- 单独函数不能访问DataContainer中的私有成员
- `DataContainer.create()` 这种形式更美观😅



<p id="4"></p>



## 4️⃣ 继承一个基于Promise的构造器（可选）

通常，继承一般用的很少。

使用一个单独的工厂函数，相对来说继承 `DataContainer` 很简单。

使用基于 Promise 的构造函数扩展类会导致严重的限制。在下面的示例中，我们继承了 `DataContainer`。子类 `SubDataContainer` 有自己的私有字段 `.#moreData` ，它通过挂钩到其父类的构造函数返回的 `Promise` 来异步初始化。

```js
class DataContainer {
  #data
  constructor() {
    return Promise.resolve('downloaded')
      .then(data => {
        this.#data = data
        return this
      })
  }

  getData() {
    return 'DATA: ' + this.#data
  }
}

class SubDataContainer extends DataContainer {
  #moreData

  constructor() {
    super()

    const promise = this
    return promise.then(_this => {
      return Promise.resolve('more')
        .then(moreData => {
          _this.#moreData = moreData
          return _this
        })
    })
  }

  getData() {
    return super.getData() + ', ' + this.#moreData
  }
}
```

我们不能实例化这个类：

```js
assert.rejects(
  () => new SubDataContainer(),
  {
    name: 'TypeError',
    message: 'Cannot write private member #moreData ' +
      'to an object whose class did not declare it',
  }
)
```

为什么会失败呢？一个构造器总是将它的私有属性添加到它的 `this` 中。但是这里，在子构造器的 `this` 是父构造器中返回的Promise(而不是通过 `Promise` 传递的 `SubDataContainer` 的实例)。

但是，如果 `SubDataContainer` 没有任何私有字段，这种方法仍然有效。



<p id="6"></p>



## 6️⃣ 总结

对于本章研究的场景，我更喜欢基于Promise的构造函数（即 [2.1](#2.1)）或静态工厂方法加上通过秘密令牌的私有构造函数（即 [3.1](#3.1)）。

但是，此处介绍的其他技术在其他场景中仍然有用。



<p id="7"></p>

## 7️⃣ 扩展阅读

- 异步编程：
  - [Promise for asynchronous programming](https://exploringjs.com/impatient-js/ch_promises.html) - “JavaScript for impatient programmers” 📚
  - [Async Functions](https://exploringjs.com/impatient-js/ch_async-functions.html) - “JavaScript for impatient programmers” 📚
  - [Immediately invoked async arrow function](https://exploringjs.com/impatient-js/ch_async-functions.html#immediately-invoked-async-arrow-functions) - “JavaScript for impatient programmers” 📚
- OOP:
  - [Prototype chains and classed](https://exploringjs.com/impatient-js/ch_proto-chains-classes.html) - “JavaScript for impatient programmers” 📚
  - [博客： ES proposal: private class fields](https://2ality.com/2019/07/private-class-fields.html)
  - [博客： ES proposal: private methods and accessors in JavaScript classes](https://2ality.com/2019/07/private-methods-accessors-in-classes.html)



译者注：关于这一章，比较有意思的是，
 - 构造器中返回的不是实例自身，而是自定义的Promise，加上异步的特性。
 - 还有些奇奇怪怪的用法，比如 [Object.create(this.prototype)._init(data)](#3.2) 借用类原型的方式。
 - 另外如何使用 `secretToken` 的方式保证构造器的正确调用。
 - 私有方法和属性的使用



2022年07月08日11:24:33