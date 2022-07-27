目录：

1. [Shallow copying vs. deep copying](#1)
2. [Shallow copying in JavaScript](#2)
   1. [Copying plain objects and Arrays via spreading](#2.1)
   2. [Shallow copying via Object.assign() (Optional)](#2.2)
   3. [Shallow copying via Object.getOwnPropertyDescriptors() and Object.defineProperties() (Optional)](#2.3)
3. [Deep copying in JavaScript](#3)
   1. [Manual deep copying vai nested spreading](#3.1)
   2. [Hack: generic deep copying via JSON](#3.2)
   3. [🔥 Implementing generic deep copying](#3.3)
4. [Further reading](#4)

本章我们将学习JS中如何拷贝对象和数组。



<p id="1"></p>



## 1️⃣ 浅拷贝 vs. 深拷贝

拷贝数据存在2种 **深度**：

1. **浅拷贝（`Shallow copying`）**：只拷贝对象和数组最外层的项（`entries`）。原始值和副本值一样
2. **深拷贝（`Deep copying`）**：拷贝所有的项。即遍历所有项进行拷贝

下面会介绍这2种拷贝。不幸的是，JS中只内置支持浅拷贝，如果想要深拷贝，则我们需要自己实现😅。



<p id="2"></p>



## 2️⃣ JS中的浅拷贝

我们看看几种浅拷贝数据的方式。



<p id="2.1"></p>



### 2.1 通过spreading操作符拷贝对象和数组

我们可以使用 [展开对象字面量](https://exploringjs.com/impatient-js/ch_single-objects.html#spreading-into-object-literals) 和 [展开数组字面量](https://exploringjs.com/impatient-js/ch_arrays.html#spreading-into-array-literals) 方式进行拷贝。

```js
const copyOfObject = { ...originalObject }
const copyOfArray = [ ...originalArray ]
```

spreading操作符存在几个问题。其中一些是实实在在的限制，而有一些则是特性。



#### 2.1.1 spreading 不会拷贝对象原型上的属性

🌰：

```js {7}
class MyClass {}

const original = new MyClass()
assert.equal(original instanceof MyClass, true)

const copy = { ...original }
assert.equal(copy instanceof MyClass, false)
```

注意，下面2个表达式是相等的：

```js
obj instanceof SomeClass
SomeClass.prototype.isPrototypeOf(obj)
```

因此，我们可以通过给副本添加和原始对象一样的原型来修复这个问题😎：

```js {6,9}
class MyClass {}

const original = new MyClass()

const copy = {
  __proto__: Object.getPrototypeOf(original),
  ...original
}
assert.equal(copy instanceof MyClass, true)
```

💡另外，我们还可以使用 `Object.setPrototypeOf()` 方法在副本创建之后再设置其原型：

```js
const copy = {
  ...original
}
Object.setPrototypeOf(
  copy,
  Object.getPrototypeOf(original)
)
```



#### 2.1.2 很多内置对象有特殊的内置槽不能通过spreading操作符拷贝

这样的内置对象有 **正则表达式和Date**。如果你拷贝它们，我们将丢失大多是其内部存储的数据😅



#### 2.1.3 ⭐只有自身（非继承的）属性才能被Spreading拷贝

鉴于 [原型链](https://exploringjs.com/impatient-js/ch_proto-chains-classes.html#prototype-chains) 的工作原理，这通常是我们想要的正确方式😀。但是我们仍需要注意这一点。

下面示例中，`original` 继承的属性 `.inherietedProp` 在副本 `copy` 中是不存在的，因为我们只会拷贝自身的属性，不会保留原型链属性。📚

```js {8}
const proto = { inheritedProp: 'a' }
const original = { __proto__: proto, ownProp: 'b' }
assert.equal(original.inheritedProp, 'a')
assert.equal(original.ownProp, 'b')

const copy = { ...original }
assert.equal(copy.inheritedProp, undefined)
assert.equal(copy.ownProp, 'b')
```

#### 2.1.4 ⭐ Spreading只拷贝可枚举属性

💡数组的 `.length` 属性是不可枚举的自身属性，它不会被拷贝。

🌰，我们通过spreading拷贝数组 `arr`（行A）：

```js {6}
const arr = ['a', 'b']
assert.equal(arr.length, 2)
assert.equal({}.hasOwnProperty.call(arr, 'length'), true)

const copy = {...arr} // A
assert.equal({}.hasOwnProperty.call(copy, 'length'), false)
```

这其实并不是什么限制，因为大多数属性都是可枚举的。

💡 **如果我们需要不可枚举属性，我们需要使用 `Object.getOwnPropertyDescriptors()` + `Object.defineProperties()` 拷贝对象** （下面会讲）：

- 它会包含所有特性（不止是 `value`），因此它们能正确的拷贝 `getters` & `setters` & 只读属性，等等
- `Object.getOwnPropertyDescriptors()` 会同时获取可枚举属性和不可枚举属性

关于枚举性，可以查看 [属性的可枚举性](https://jamessawyer.github.io/deep-javascript-cn/4/Enumerability-of-Properties.html) 这一章。



#### 2.1.5 ⭐ 使用Spreading时属性特性不会被准确的拷贝

📚 独立于 [属性特性](https://jamessawyer.github.io/deep-javascript-cn/4/Property-attributes-an-Introduction.html)，**它的副本总是变为可写可配置的数据属性**。

比如，我们将 `original.prop` 特性设置为 `writable = false` & `configurable = false`:

```js {6-7}
const original = Object.defineProeprty(
  {},
  {
    prop: {
      value: 1,
      writable: false,
      configurable: false,
      enumerable: true
    }
  }
)

assert.deepEqual(original, { prop: 1 })
```

💡 如果我们拷贝 `.prop`, 则它的 `writable & configurable` 特性都将变为 `true`:

```js {9-10}
const copy = { ...original }

// 💡特性 `writable` & `configurable` 将被改写
assert.deepEqaul(
  Object.getOwnPropertyDescriptors(copy),
  {
    prop: {
      value: 1,
      writable: true,
      configurable: true,
      enumerable: true
    }
  }
)
```

💡，作为结果，`getters & setters` 将不会被正确的拷贝：

```js {10-11}
const original = {
  get myGetter() { return 123 },
  set mySetter(x) {}
}

const copy = { ...original }
assert.deepEqual(
  copy,
  {
    myGetter: 123, // 从访问器属性变为了数据属性😅
    mySetter: undefined
  }
)
```

后面提到的 `Object.getOwnPropertyDescriptorss()` + `Object.defineProperties()` 总是会完整无缺的传输自身属性的所有特性😎。



#### 2.1.6 Spreading拷贝是浅拷贝

副本对original中的键值对拥有全新的版本，但是嵌套的部分不会被拷贝。比如：

```js {9,14,23,30,37,46}
const original = {
  name: 'Jane',
  work: {
    employer: 'Acme'
  }
}
const copy = { ...original }

// 💡属性 .name 是一个副本，改变副本不会影响original
copy.name = 'John'
assert.deepEqual(
  original,
  {
    name: 'Jane', // original.name 不受影响
    work: {
      employer: 'Acme'
    }
  }
)
assert.deepEqual(
  copy,
  {
    name: 'John', // 副本变化了
    work: {
      employer: 'Acme'
    }
  }
)

// 🚨 .work的值是共享的：改变副本会影响到original
copy.work.employer = 'Spectre'
assert.deepEqual(
  original,
  {
    name: 'Jane',
    work: {
      employer: 'Spectre' // 受影响了
    }
  }
)
assert.deepEqual(
  copy,
  {
    name: 'John',
    work: {
      employer: 'Spectre'
    }
  }
)
```

我们将稍后看如何深拷贝。



<p id="2.2"></p>



### 2.2 使用 Object.assign() 进行浅拷贝

`Object.assign()` 和 `spreading` 工作效果类似。即，下面2种拷贝操作几乎相同：

```js
const copy1 = { ...original }
const copy2 = Object.assign({}, oirginal)
```

使用方法形式的优势在于，我们可以在老JS引擎中使用垫片（`polyfill`）的方式。

🤔 `Object.assign()` 和 `Spreading` 并不是完全相同。在某一个方面存在差异，相对微妙的点：**它们创建属性的方式不同**

- `Object.assign()` 使用 **赋值（`assignment`）** 创建副本的属性
- `Spreading` 通过 **定义（`definition`）** 方式创建副本属性

💡 *赋值会调用自身和继承的 `setters`，而定义则不会。[assignment vs. definition](https://jamessawyer.github.io/deep-javascript-cn/4/Properties-assignment-vs-definition.html)* 

这种差异并不是很显著。下面代码是个例子，但这个例子比较刻意：

```js {2,4,6,12-18}
const original = {
  ['__proto__']: null // A
}
// 定义方式创建属性，不会调用继承的setter
const copy1 = { ...original } 
// 💡 copy1 有自身属性 '__proto__'
assert.deepEqual(
  Object.keys(copy1),
  ['__proto__']
)

// 赋值方式创建属性，会调用继承的setter
const copy2 = Object.assign({}, original) 
// copy2 原型为 null
assert.deepEqual(
  Object.getPrototypeOf(copy2),
  null
)

```

通过使用 `A` 行的计算属性，我们创建了一个 `.__proto__` 作为自身属性，并且不会调用其继承的setter。然而，当 `Object.assign()` 拷贝该属性时，它会调用 `setter`。（关于 `.__proto__` 可查看 [JS for impatient programmers](https://exploringjs.com/impatient-js/ch_proto-chains-classes.html#proto)）

<p id="2.3"></p>



### 2.3 ⭐ 使用Object.getOwnPropertyDescriptors()+Object.defineProperties()进行浅拷贝

JS允许我们通过 [属性描述器](https://jamessawyer.github.io/deep-javascript-cn/4/Property-attributes-an-Introduction.html#_2%EF%B8%8F%E2%83%A3-%E5%B1%9E%E6%80%A7%E6%8F%8F%E8%BF%B0%E7%AC%A6%EF%BC%88property-descriptors%EF%BC%89) 创建属性，它是指定了属性特性的一个对象。比如，通过 `Object.defineProperties()`，在实战中我们已经见过这个方法。如果结合 `Object.getOwnPropertyDescriptors()` 我们可以更准确的进行拷贝：

```js
function copyAllOwnProperties(original) {
  return Object.defineProperties(
    {},
    Object.getOwnPropertyDescriptors(original)
  )
}
```

🚀这消除了通过Spreading拷贝的2个问题。

第一，自身属性所有特性都被正确的拷贝。因此，我们也可以拷贝getters & setters 🤩:

```js
const original = {
  get myGetter() { return 123 },
  set mySetter(x) {}
}

assert.deepEqual(
  copyAllProperties(original),
  original
)
```

第二，因为 `Object.getOwnPropertyDescriptors()` 非枚举属性也可以被拷贝了：

```js {9-12}
const arr = ['a', 'b']
assert.equal(arr.length, 2)
assert.equal(
  {}.hasOwnProperty.call(arr, 'length'),
  true
)

const copy = copyAllProperties(arr)
assert.equal(
  {}.hasOwnProperty.call(copy, 'length'),
  true
)
```



<p id="3"></p>



## 3️⃣ JS中的深拷贝

现在该处理深拷贝问题了。首先我们将手动深拷贝，然后我们看看通用的一些方式。



<p id="3.1"></p>



### 3.1 使用嵌套spreading手动深拷贝

如果我们使用嵌套的spreading，我们可以深拷贝：

```js {10,16-17}
const original = {
  name: 'Jane',
  work: {
    employer: 'Acme'
  }
}

const copy = {
  name: original.name,
  work: { ...original.work }
}

// 我们成功拷贝了
assert.deepEqual(original, copy)

// 并且拷贝是深拷贝的
assert.ok(original.work !== copy.work)
```



<p id="3.2"></p>



### 3.2 🤔 Hack方式：使用JSON深拷贝

这是一种hack方法，但是它提供了一种快速的解决方案：为了深拷贝对象 `original`，我们先将其转换为 **JSON字符串**，然后解析该字符串：

```js {1-3}
function jsonDeepCopy(original) {
  return JSON.parse(JSON.stringify(original))
}

const original = {
  name: 'Jane',
  work: {
    employer: 'Acme'
  }
}

const copy = jsonDeepCopy(original)
assert.deepEqual(original, copy)
```

**这种方式的弊端是，我们只能拷贝JSON支持的合法属性keys和属性values** 😅。

👩🏻‍🏫 下面是不支持的keys & values将被直接忽略：

```js
assert.deepEqual(
  jsonDeepCopy({
    // Symbols作为keys JSON不支持 🚫
    [Symbol('a')]: 'abc',
    // 函数作为值，JSON不支持 🚫
    b: function() {},
    // undefined | null值 JSON不支持 🚫
    c: undefined,
  }),
  {} // 得到一个空对象
)
```

其它情况会导致异常：

```js
assert.throws(
  // JSON不支持 BigInt 类型  🚫
  () => jsonDeepCopy({ a: 123n }),
  /^TypeError: Do not know how to serialize a BigInt$/
)
```



<p id="6.3"></p>



### 3.3 🚀实现通用深拷贝

下面函数时一种通用的深拷贝：

```js {15}
function deepCopy(original) {
  if (Array.isArray(original)) {
    const copy = []
    for (const [index, value] of original.entries()) {
      copy[index] = deepCopy(value)
    }
    return copy
  } else if (typeof original === 'object' && original !== null) {
    const copy = {}
    for (const [key, value] of Object.entries(original)) {
      copy[key] = deepCopy(value)
    }
    return copy
  } else {
    // 原始类型则直接返回，不需要进行拷贝
    return original
  }
}
```

这个函数处理了3中情形：

1. 如果 `original` 是数组：我们创建一个新数组，深拷贝其元素到新数组中
2. 如果 `original` 是对象，我们使用类似的方式
3. 如果 `original` 是原始类型值，我们什么也不做

示例：

```js
const original = {
  a: 1,
  b: {
    c: 2,
    d: {
      e: 3
    }
  }
}

const copy = deepCopy(original)

// 副本和original完全一样?
assert.deepEqual(copy, original) // true

// 我们是否真的完全拷贝了所有层级？
// （内容相同，但是是不同对象？） 
assert.ok(copy     !== original)
assert.ok(copy.b   !== original.b)
assert.ok(copy.b.d !== original.b.d)
```

注意 `deepCopy()` 只修复了spreading的一个问题： **浅拷贝**。其余问题仍存在😅：

- 原型不会被拷贝
- 特殊对象只部分拷贝
- 非枚举属性被忽略
- 大多数属性特性被忽略

完全实现完整的通用拷贝是不可能的，可能原因为：不是所有数据都是一棵树，有时我们也不想拷贝所有属性等。



#### 3.3.1 🔥 更简洁版本deepCopy()

我们可以使用 `.map()` & `Object.fromEntries()` 使上面的deepCopy更简洁：

```js {5-8}
function deepCopy(original) {
  if (Array.isArray(original)) {
    return original.map(elem => deepCopy(elem)) // 递归
  } else if (typeof original === 'object' && original !== null) {
    return Object.fromEntries(
      Object.entries(original)
      	.map(([k, v]) => [k, deepCopy(v)]) // 递归
    )
  } else {
    // 原始类型值：原子性的，不需要拷贝
    return original
  }
}
```



<p id="4"></p>



## 4️⃣ 进一步阅读

- [拷贝类实例：.clone() vs. 拷贝构造器](https://exploringjs.com/deep-js/ch_copying-class-instances.html) 解释基于类的拷贝模式
- [Spreading对象字面量](https://exploringjs.com/impatient-js/ch_single-objects.html#spreading-into-object-literals)
- [Spreading数组字面量](https://exploringjs.com/impatient-js/ch_arrays.html#spreading-into-array-literals)
- [原型链](https://exploringjs.com/impatient-js/ch_proto-chains-classes.html#prototype-chains)


## 5️⃣ 总结（译者注）

- 对象和数组的拷贝方式：浅拷贝 & 深拷贝
- spreading 拷贝
  - 不会拷贝原型上的属性
  - 不会拷贝某些对象内部槽数据（比如正则 & Date对象）
  - spreading对属性特性拷贝不准确，比如将 `writable=false` | `configurable=false` 拷贝后变为 `writable=true` | `configurable=true`
- `Object.assign()` 浅拷贝
  - Object.assign() 通过 **赋值（`assignment`）** 创建副本属性
  - Spreading 通过 **定义（`definition`）** (比如 `Object.defineProperties()`) 方式创建副本属性
  - 赋值 vs. 定义 区别：赋值会调用自身或继承的setter，而定义则不会
- `Object.getOwnPropertyDescriptors()` + `Object.defineProperties()` 浅拷贝方式解决spreading拷贝的问题：
  - 属性特性拷贝正确
  - 可以对访问器属性（`getters & setters`）进行正确的拷贝
- 深拷贝的几种方式：
  - 嵌套spreading拷贝
  - Hack方式：JSON，存在几个问题 - 只能拷贝JSON支持的keys & values，其余忽略或抛出异常
  - 🔥递归方式进行深拷贝，使用到了 [Object.fromEntries()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/fromEntries) 等方法






2022年07月26日22:53:36
