目录：

1. [Wrapping objects](#1)
   1. [Making collections immutable via wrapping](#1.1)
2. [An immutable wrapper for Maps](#2)
3. [An immutable wrpper for Arrays](#3)
4. [Conclusion](#4)

对集合的不可变包装器使集合不可变，通过将其包装到一个新的对象中。

本章，我们将看看它是如何工作的，以及不可变包装的用处。



<p id="1"></p>



## 1️⃣ 包装对象

如果某个对象我们想减少其接口，我们可以采取下面方式：

1. 创建一个新对象，将原对象存储在一个私有字段中。📚新对象即 *包装（`wrap`）* 了原对象。新对象也称之为 **包装器（`wrapper`）**，而原对象称之为 *被包装的对象（`wrapped object`）*
2. 包装器只将它接收到的一些方法调用转发给被包装的对象。

👇🏻下面是包装的方式：

```js
class Wrapper { // 包装器
  #wrapped // 被包装的对象，即原对象
  constructor(wrapped) {
    this.#wrapped = wrapped
  }
  
  allowedMethod1(...args) {
    return this.#wrapped.allowedMethod1(...args)
  }
  
  allowMethod2(...args) {
    return this.#wrapped.allowedMethod2(...args)
  }
}
```

💡相关软件设计模式：

- 包装和 [门面（Facade）模式](https://en.wikipedia.org/wiki/Facade_pattern) 相关
- 我们使用 [转发（delegate）](https://en.wikipedia.org/wiki/Forwarding_(object-oriented_programming)) 实现了 [代理模式](https://en.wikipedia.org/wiki/Delegation_pattern)。委托表示一个对象允许另一个对象（委托）处理某些工作。它是继承共享代码的另一种方式



<p id="1.1"></p>



### 1.1 通过包装使集合不可变

📚 为了使集合不可变，我们可以使用包装，将接口中所有破坏性（`destructive`）操作都移除。

这种技术的一个重要使用场景是，某个对象有个内部可变数据结构，对象想不使用拷贝的方式安全的导出该数据。导出是“实时”的可能是一个目标。对象可以通过包装该内部数据结构，并将其变为不可变完成这些目标。

下面2个小节展示对 `Maps` & `Arrays` 的不可变包装器。它们都存在下面这些限制：

1. 它们是草图。要使它们适合实际使用，还需要做更多的工作:更好的检查，支持更多的方法，等等。
2. 它们的工作方式很简单: **每一种方法都使包装的对象不可变**，但不是它返回的数据不可变。可以通过包装方法返回的一些结果来解决这个问题。



<p id="2"></p>



## 2️⃣ 对Maps的不可变包装器

类 `ImmutableMapWrapper` 对Maps产生包装器：

```js {3,11,17}
class ImmutableMapWrapper {
  static _setUpPrototype() {
    // 💡只转发非破坏性方法给包装的Map
    for (const methodName of ['get', 'has', 'keys', 'size']) {
      ImmutableMapWrapper.prototype[methodName] = function (...args) {
        return this.#wrappedMap[methodName](...args)
      }
    }
  }
  
  #wrappedMap // 被包装的Map
  constructor(wrappedMap) {
    this.#warppedMap = wrappedMap
  }
}

ImmutableMapWrapper._setUpPrototype()
```

设置原型必须通过静态方法执行，因为我们只能从类内部访问私有字段 `#wrappedMap`。

使用 `ImmutableMapWrapper`:

```js {7,13}
const map = new Map([
  [false, 'no'],
  [true, 'yes']
])
const wrapped = new ImmutableMapWrapper(map)

// 非破坏性操作正常工作 😎
assert.equal(wrapped.get(true), 'yes')
assert.equal(wrapped.has(false), true)
assert.deepEqual([...wrapped.keys()], [false, true])


// 破坏性操作不可访问 🚫
assert.throws(
  () => wrapped.set(false, 'never!'),
  /^TypeError: wrapped.set is not a function$/
)
assert.throws(
  () => wrapped.clear(),
  /^TypeError: wrapped.clear is not a function$/
)
```



<p id="3"></p>



## 3️⃣ 对Arrays的不可变包装器

对数组 `arr`，普通包装是不够的，因为我们不仅需要拦截方法调用，也要拦截属性访问，比如 `arr[1] = true`。[JS Proxies](https://exploringjs.com/es6/ch_proxies.html) 允许我们达到这一目标：

```js
const RE_INDEX_PROP_KEY = /^[0-9]+$/
const ALLOWED_PROPERTIES = new Set([
  'length',
  'constructor',
  'slice',
  'concat'
])

function wrapArrayImmutably(arr) {
  const handler = {
    get(target, propKey, receiver) {
      // 假设 propKey 是字符串，而不是symbols
      if (
        RE_INDEX_PROP_KEY.test(propKey)
        || ALLOWED_PROPERTIES.has(propKey)
      ) {
        return Reflect.get(target, propKey, receiver)
      }
      throw new TypeError(`Property "${propKey}" can't be accessed`)
    },
    set(target, propKey, value, receiver) {
      throw new TypeError('Setting is not allowed')
    },
    deleteProperty(target, propKey) {
      throw new TypeError('Deleting is not allowed')
    }
  }
  
  return new Proxy(arr, handler)
}
```

让我们包装数组：

```js
const arr = ['a', 'b', 'c']
const wrapped = wrapArrayImmutably(arr)

// 允许非破坏性操作
assert.deepEqual(
  wrapped.slice(1),
  ['b', 'c']
)
assert.equal(wrapped[1], 'b')

// 不允许破坏性操作 🚫
assert.throws(
  () => wrapped[x] = 'x',
  /^TypeError: Setting is not allowed$/
)
assert.throws(
  () => wrapped.shift(),
  /^TypeError: Property "shift" can’t be accessed$/
)
```

<p id="4"></p>

## 总结（译者注）

- 包装器是什么，被包装是什么意思
- 包装器的本质是什么？使用一个对象代理包装另一个对象
- 包装器涉及到的设计模式：代理模式，门面模式
- 不可变的含义是什么？非破坏性的改变数据结构
- 不可变包装器：包装器只导出非破坏性（不可变）操作，对可变操作进行控制
- 对对象的包装：使用私有字段关联要包装的对象
- 对Maps的包装：类似对象包装，使用静态方法
- 对Arrays的包装：使用 `Proxies` 对数据进行代理，并只保留非破坏性操作



原文链接：

- [Immutable wrappers for collections](https://exploringjs.com/deep-js/ch_immutable-collection-wrappers.html)

2022年07月28日00:42:16