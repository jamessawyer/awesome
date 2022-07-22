目录：

1. [How enumerability affects property-iteating constructs](#1)
   1. [Operations that only consider enumerable properties](#1.1)
   2. [Operations that consider both enumerable and non-enumerable properties](#1.2)
   3. [Naming rules for introspective operations](#1.3)
2. [The enumerability of pre-defined and created properties](#2)
3. [Use cases for enumerability](#3)
   1. [Use case: Hiding properties from the for-in loop](#3.1)
   2. [Use case: Marking properties as not to be copied](#3.2)
   3. [Marking properties as private](#3.3)
   4. [Hiding own properties from JSON.stringify()](#3.4)
4. [Conclusion](#4)



可枚举性是对象属性的一个 *特性（`attribute`）*。本章将更进一步看看它是如何使用的，以及它如何影响 `Object.keys()` & `Object.assign()` 这些操作的。



前置知识：属性特性。



<p id="1"></p>



## 1️⃣ 可枚举性是如何影响属性迭代构造的

为了演示各种操作如何被可枚举性影响的，我们使用下面对象 `obj`，它的原型是 `proto`。

```js {4,8,12,15}
// 原型
const protoEnumSymbolKey = Symbol('protoEnumSymbolKey')
const protoNonEnumSymbolKey = Symbol('protoNonEnumSymbolKey')
const proto = Object.defineProperties({}, {
  protoEnumStringKey: { // 字符串可枚举key
    value: 'protoEnumStringKeyValue',
    enumerable: true // 可枚举
  },
  [protoEnumSymbolKey]: { // symbol可枚举key
    value: 'protoEnumSymbolKeyValue',
    enumerable: true
  },
  protoNonEnumStringKey: { // 字符串不可枚举key
    value: 'protoNonEnumStringKeyValue',
    enumerable: false, // 不可枚举
  },
  [protoNonEnumSymbolKey]: { // symbol不可枚举key
    value: 'protoNonEnumSymbolKeyValue',
    enumerable: false
  },
})


const objEnumSymbolKey = Symbol('objEnumSymbolKey')
const objNonEnumSymbolKey = Symbol('objNonEnumSymbolKey')

const obj = Object.create(proto, {
  objEnumStringKey: {
    value: 'objEnumStringKeyValue',
    enumerable: true
  },
  [objEnumSymbolKey]: {
    value: 'objEnumSymbolKeyValue',
    enumerable: true
  },
  objNonEnumStringKey: {
    value: 'objNonEnumStringKey',
    enumerable: false
  },
  [objNonEnumSymbolKey]: {
    value: 'objNonEnumSymbolKeyValue',
    enumerable: false
  }
})
```

<p id="1.1"></p>



### 1.1 ⭐只考虑可枚举属性的操作

👩🏻‍🏫 忽略不可枚举属性的操作：

| 操作             |        | String键 | Symbol键 | 继承的 |
| ---------------- | ------ | :------: | :------: | :----: |
| Object.keys()    | ES5    |    ✅     |    🚫     |   🚫    |
| Object.values()  | ES2017 |    ✅     |    🚫     |   🚫    |
| Object.entries() | ES2017 |    ✅     |    🚫     |   🚫    |
| Spreading {...x} | ES2018 |    ✅     |    ✅     |   🚫    |
| Object.assign()  | ES6    |    ✅     |    ✅     |   🚫    |
| JSON.stringify() | ES5    |    ✅     |    🚫     |   🚫    |
| for-in           | ES1    |    ✅     |    🚫     |   ✅    |

下面操作（👆🏻表）只考虑可枚举属性：

- `Object.keys()` 返回自身可枚举的字符串keys

  ```js
  Object.keys(obj)
  ['objEnumStringKey']
  ```

- `Object.values()` 返回自身可枚举的字符串属性的值

  ```js
  Object.values(obj)
  ['objEnumStringKeyValue']
  ```

- `Object.entries()` 返回自身可枚举字符串属性的 key-value 对。（注意 `Object.fromEntries()` 可以接收 `symbols` 作为keys，但是只创建可枚举属性）

  ```js
  Object.entries(obj)
  [['objEnumStringKey', 'objEnumStringKeyValue']]
  ```

- 对象字面量展开操作符（`Spreading {...x}`）只考虑自身可枚举属性（strings keys 或 symbols keys 😎）

  ```js
  const copy = { ...obj }
  Reflect.ownKeys(copy)
  ['objEnumStringKey', 'objEnumSymbolKey']
  ```

- `JSON.stringify()` 只字符串化自身可枚举的字符串keys

  ```js
  JSON.stringify(obj)
  '{"objEnumStringKey":"objEnumStringKeyValue"}'
  ```

- `for-in` 循环遍历 **自身或继承的** 可枚举的字符串键属性🤩（译者注：唯一包含继承的属性迭代操作！！！）

  ```js {2}
  const propKeys = []
  // 🚀 继承的可枚举字符串属性也会被遍历
  for (const propKey in obj) {
    propKeys.push(propKey)
  }
  
  assert.deepEqual(
    propKeys,
    // 可以看出 原型 上的可枚举字符串key 也被遍历了 😎
    ['objEnumStringKey', 'protoEnumStringKey']
  )
  ```

💡 `for-in` 是唯一能对继承的属性的可枚举字符串keys有影响的操作。其余所有操作都**只对自身属性**有效。



<p id="1.2"></p>



### 1.2 ⭐同时考虑可枚举和不可枚举属性的操作

👩🏻‍🏫 同时考虑可枚举和不可枚举属性的操作：

| 操作                               |        | String键 | Symbol键 | 继承的 |
| :--------------------------------- | ------ | :------: | :------: | :----: |
| Object.getOwnPropertyNames()       | ES5    |    ✅     |    🚫     |   🚫    |
| Object.getOwnPropertySymbols()     | ES6    |    🚫     |    ✅     |   🚫    |
| Reflect.ownKeys()                  | ES6    |    ✅     |    ✅     |   🚫    |
| Object.getOwnPropertyDescriptors() | ES2017 |    ✅     |    ✅     |   🚫    |

下面操作（👆🏻表）既考虑可枚举属性，也考虑不可枚举属性：

- `Object.getOwnPropertyNames()` 列举出所有 **自身** 字符串属性keys

  ```js
  Object.getOwnPropertyNames(obj)
  ['objEnumStringKey', 'objNonEnumStringKey']
  ```

- `Object.getOwnPropertySymbols()` 列举出所有 **自身** Symbol-keys 属性键

  ```js
  Object.getOwnPropertySymbols(obj)
  ['objEnumSymbolKey', 'objNonEnumSymbolKey']
  ```

- `Reflect.keys()` 列举出所有的自身属性keys 🚀

  ```js
  Reflect.keys(obj)
  
  [
    'objEnumStringKey',
    'objNonEnumStringKey',
    'objEnumSymbolKey',
    'objNonEnumSymbolKey'
  ]
  ```

- `Object.getOwnPropertyDescriptors()` 列举出所有自身属性描述器

  ```js
  Object.getOwnPropertyDescriptors(obj)
  
  {
    objEnumStringKey: {
      value: 'objEnumStringKeyValue',
      writable: false,
      enumerable: true,
      configurable: false
    },
    objNonEnumStringKey: {
      value: 'objNonEnumStringKeyValue',
      writable: false,
      enumerable: false,
      configurable: false
    },
    [objEnumSymbolKey]: {
      value: 'objEnumSymbolKeyValue',
      writable: false,
      enumerable: true,
      configurable: false
    },
    [objNonEnumSymbolKey]: {
      value: 'objNonEnumSymbolKeyValue',
      writable: false,
      enumerable: false,
      configurable: false
    }
  }
  ```



<p id="1.3"></p>



### 1.3 内省操作命名规则

**内省（`introspection`）** 使程序能在运行时检测值的结构。这是一种**元编程**：正常程序是关于写程序；元编程是关于检测或者改变程序。

📚 在JS中，常见的内省操作有较短的名称，而很少使用的操作有较长的名称。**忽略不可枚举属性是常态**，这就是为什么有短名称的操作和没有长名称的操作:

- `Object.keys()` 忽略不可枚举属性
- `Object.getOwnPropertyNames()` 列举所有自身字符串keys

然而，`Reflect` 方法（例如 `Reflect.ownKeys()`） 偏离这个规则，**因为 `Reflect` 提供的操作更加 `元（meta）`， 并和代理相关**。

此外，还做了以下区分(从ES6开始，引入了Symbol)：

- *Property keys* 要么是 strings，要么是 symbols
- *Property names* 为字符串属性keys
- *Property symbols* 为symbols属性keys

因此，`Object.keys()` 更好的名字可能是 `Object.names()` 😅





<p id="2"></p>



## 2️⃣ 预定义和创建的属性的枚举性

这一节中，我们将 `Object.getOwnPropertyDescriptor()` 缩写如下：

```js
const desc = Object.getOwnPropertyDescriptor.bind(Object)
```

大多数属性创建伴随着下面特性：

```js
{
  writable: true,
  enumerable: false,
  configurable: true
}
```

👩🏻‍🏫 包括：

- 赋值（`Assignment`）
- 对象字面量（`Object literals`）
- 类公有字段
- `Object.fromEntries()`

最重要的不可枚举属性有：

- **内置类的原型属性**

  ```js
  desc(Object.prototype, 'toSztring').enumerable
  
  // false
  ```

- 通过用户定义的类创建的原型属性

  ```js
  // 类方法放在类原型上
  desc(class {foo() {}}.prototype, 'foo').enumerable
  
  // false
  ```

- 数组的 `.length` 属性

  ```js
  Object.getOwnPropertyDescriptor([], 'length')
  
  {
    value: 0,
    writable: false,
    enumerable: false,
    configurable: false
  }
  ```

接下来我们将看看枚举型的使用场景，也会告诉我们为什么某些属性是可枚举的，而有些属性不是的。



<p id="3"></p>



## 3️⃣ 可枚举性的使用场景

**可枚举性是一个不一致的功能**。它存在使用场景，但是总是存在某种缺陷。下面我们看看它的使用场景和其缺陷。



<p id="3.1"></p>



### 3.1 使用场景：对 for-in 循环隐藏属性

📚 `for-in` 循环会遍历对象自身的或继承的所有可枚举字符串keys。

因此，特性 `enumerable` 可用于隐藏不想被遍历的属性。这也是为什么在ECMAScript 1版本中引入了可枚举性这个概念。

通常，最好避免使用 `for-in` 🤔。下面2个小节将解释为什么。下面函数帮助我们展示 `for-in` 是如何运作的：

```js
function listPropertiesViaForIn(obj) {
  const result = []
  for (const key in obj) {
    result.push(key)
  }
  return result
}
```



> 1.对对象使用 for-in 的缺陷⭐

`for-in` 遍历所有属性，**也包括继承的属性**：

```js {3}
const proto = { enumerableProtoProp: 1 }
const obj = {
  __proto__: proto, // 继承proto
  enumerableObjProp: 2
}

listPropertiesViaForIn(obj)

// ['enumerableObjProp', 'enumerableProtoProp']
```

对于普通对象， `for-in` 不会看到继承的方法，比如 `Object.prototype.toString()`, 因为它们是 **不可枚举的**：

```js
const obj = {}
listPropertiesViaForIn(obj)

// []
```

👩🏻‍🏫 在用户定义的类中，所有继承属性都是不可枚举的，因此它们也会被忽略：

```js {12-14}
class Person {
  constructor(first, last) {
    this.first = first
    this.last = last
  }
  
  getName() {
    return this.first + ' ' + this.last
  }
}

// jane 实例的原型是 Person.prototype
// 只有 getName 是继承的属性，它是不可枚举的 😅
// first last 是实例自身的属性，它们是可枚举的
const jane = new Person('Jane', 'Doe')
listPropertiesViaForIn(jane)
// ['first', 'last']
```

💡总结：在对象中，`for-in` 会考虑继承的属性，但我们一般希望忽略继承的属性。因此最好使用 `for-of` 循环 + `Object.keys() | Object.entries()`  等结合起来使用😎。



> 2.对数组使用for-in的缺陷

数组和字符串自身属性 `.length` 是不可枚举的，因此会在 `for-in` 中被忽略：

```js {2-3}
listPropertiesViaForIn(['a', 'b'])
// 🚨 译者注： 这里不是使用 for-of
// 因此打印的是索引值
// ['0', '1']

listPropertiesViaForIn('ab')
// ['0', '1']
```

但是，使用 `for-in` 遍历数组索引通常并不安全，因为它会同时考虑哪些不是索引的继承的和自身的属性。

下面示例展示了，假如数组自身有*非索引属性（`Non-index property`）*:

```js
const arr1 = ['a', 'b']
assert.deepEqual(
  listPropertiesViaForIn(arr1),
  ['0', '1']
)

const arr2 = ['a', 'b']
// 定义一个数组非索引属性
arr2.nonIndexProp = 'yes'
assert.deepEqual(
  listPropertiesViaForIn(arr2),
  ['0', '1', 'nonIndexProp']
)
```

💡总结：`for-in` 不应该用于迭代数组索引，因为它同时考虑到了索引属性和非索引属性：

- 📚如果你对数组的keys感兴趣，可以使用数组方法 `.keys()`: 

  ```js
  [...['a', 'b', 'c'].keys()]
  [0, 1, 2]
  ```

- 如果你想迭代数组的元素，请使用 `for-of` 循环，它还可以对其它可迭代数据结构生效



<p id="3.2"></p>



### 3.2 使用场景：将属性标记为不可拷贝

通过将属性标记为不可枚举，我们可以将其在某些拷贝操作中进行隐藏。

在看更现代化拷贝操作前，我们先看看2个历史性的拷贝操作。



> A. 历史性拷贝操作1：Prototype的 Object.extend()

[Prototype](https://en.wikipedia.org/wiki/Prototype_JavaScript_Framework) 是一个很老的JS框架。

Prototype的 [Object.extend(destination, source)](http://api.prototypejs.org/language/Object/extend/) 会拷贝所有自身和继承的可枚举属性，它的 [实现](https://github.com/prototypejs/prototype/blob/5fddd3e/src/prototype/lang/object.js#L88) 如下：

```js
function extend(destination, source) {
  for (var property in source)
    destination[property] = source[property]
  
  return destination
}
```

如果我们对对象使用 `Object.extend()`，我们可以看到它会拷贝继承属性到自身上，并且忽略非枚举属性（它同样会忽略symbol keys 属性）。这其实是 `for-in` 的原因：

```js
const proto = Object.defineProperties({}, {
  enumProtoProp: {
    value: 1,
    enumerable: true
  },
  nonEnumProtoProp: {
    value: 2,
    enumerable: false
  },
})

const obj = Object.create(proto, {
  enumObjProp: {
    value: 3,
    enumerable: true
  },
  nonEnumObjProp: {
    value: 4,
    enumerable: false
  },
})

extend({}, obj)

// { enumObjProp: 3, enumProtoProp: 1 }
```

> B. 历史性拷贝操作2：jQuery的 $.extend()

jQuery 的 [$.extend(target, source1, source2, ...)](https://api.jquery.com/jquery.extend/) 和 `Object.extend()` 类似：

- 它会拷贝所有自身的和继承的可枚举属性
- 先将 `source1` 拷贝到 `target`，然后将 `source2` 拷贝到 `target`，依次类推



> C. ⭐ 可枚举性驱动拷贝的缺点

基于可枚举性拷贝的方式有几个缺点：

- 可枚举性用于隐藏继承的属性，这是它主要的使用方式，因为我们通常希望拷贝自身属性到自身属性
- 哪些属性被拷贝通常取决于具体的任务；对所有用例使用一个标志很少有意义。更好的选择是提供一个 `predicate` 函数 (返回布尔值的回调)的复制操作，该 `predicate` 告诉复制操作何时忽略属性
- 当拷贝数组时，可枚举性对隐藏自身属性 `.length` 很方便。但是存在一种很少见的例外情况：一个同时影响相连属性和被相连属性影响的魔术属性。如果我们自己去实现这样一个魔术属性，我们将使用（继承的）`getters |& setters`，而不是（自身的） 数据属性



> D. ⭐ Object.assign()

在ES6中，[Object.assign(target, source_1, source_2, ...)](https://exploringjs.com/impatient-js/ch_single-objects.html#object.assign) 可用于将多个sources合并到target中。**sources上所有自身可枚举属性（字符串属性或者symbol keys属性）都会被考虑 📚**。 `Object.assign()` 使用 **`get` 操作** 从source读取值，然后使用 **`set` 操作**将值写入到target上。

关于可枚举性，`Object.assign()` 延续了 Object.extend() 和 $.extend() 的传统：

Object.assign 将为所有已流通的 extend() API 铺平道路。我们认为在这些情况下不复制可枚举方法的先例足以让 Object.assign 有这种行为。

💡 换句话说： `Object.assign()` 是从 `$.extend()` 的升级版本。**它的方式比$.extend更清晰，因为它忽略了继承的属性 🤩**。



> E. 非枚举有用的一个罕见场景：在拷贝时有用的

非枚举有用的一种比较少见的场景。[fs-extra](https://github.com/jprichardson/node-fs-extra/issues/577) 的一个issue:

- Node.js内置模块 `fs` 有一个属性 `promises`，它包含基于Promise版本 `fs` API的对象。在那个issue存在的时候，读取 `.promise` 会导致下面控制台警告：

  ```
  ExperimentalWarning: The fs.promises API is experimental
  ```

- 除了提供自己的功能， `fs-extra` 也重新导出了fs导出的一切。对CommonJS模块，这意味着将fs所有属性都拷贝到 `fs-extra` 的 `module.exports` 对象上（通过 [Object.assign](https://github.com/jprichardson/node-fs-extra/blob/master/lib/index.js) 方法）。当 fs-extra这样做后，就会触发警告。每次加载fs-extra都会触发这个警告，令人感到困惑

- 一个 [快速修复](https://github.com/nodejs/node/pull/20504) 将 `fs.promises` 变为不可枚举。之后需，`fs-extra` 将忽略它



<p id="3.3"></p>



### 3.3 将属性标记为私有

如果你将一个属性标记为不可枚举，则它不会被 `Object.keys` & `for-in` 等等操作看见。对于这些机制，该属性是私有的。

然而，这种方式存在几个问题😅：

- 当拷贝对象时，我们通常也想将私有属性进行拷贝。这和非枚举属性冲突
- **属性并不是真正的私有**。获取，设置和其它对属性的操作，对于可枚举属性和不可枚举属性是没有区别的。
- 当处理代码时，我们不能立即知道一个属性是否是可枚举的。**命名规范（比如下划线）可以帮助我们辨别它们**
- 我们不能用可枚举性来辨别公有方法或私有方法，因为方法在原型上默认就是不可枚举的🤣



<p id="3.4"></p>



### 3.4 JSON.stringify()隐藏自己的属性

📚 **`JSON.stringify()` 返回结果不会包含不可枚举属性。**

因此我们可以用枚举性来决定哪些属性可以导出为JSON。这种使用场景和先前将属性标记为私有类似。但它也是不同的，因为它更多地是关于导出的，并且应用了略微不同的考虑因素。例如:一个对象可以完全从JSON重建吗？

作为枚举型的另一种替代，**对象可以实现 `.toJSON()` 和 `JSON.stringify()` 字符串话任何方法返回的内容，而不必是对象本身。**💡

🌰：

```js
class Point {
  static fromJSON(json) {
    return new Point(json[0], json[1])
  }
  
  constructor(x, y) {
    this.x = x
    this.y = y
  }
  
  toJSON() {
    return [this.x, this.y]
  }
}

JSON.stringify(new Point(8, -3))
// '[8,-3]'
```

我发现 `toJSON()` 要比利用枚举性更加清晰。并且对返回格式更加的自由😎。



<p id="4"></p>



## 4️⃣ 总结

我们已经看到，几乎所有不可枚举的应用程序都是变通方法，现在有了其他更好的解决方案。

👩🏻‍🏫 对于我们自己的代码，我们通常会假装枚举性不存在：

- 使用对象字面量和赋值创建属性总是创建的可枚举属性
- 通过类创建的原型属性（比如方法）总是不可枚举的



2022年07月21日23:47:11

