目录：

1. [The structure of objects](#1)
   1. [Internal slots](#1.1)
   2. [Property keys](#1.2)
   3. [Property attributes](#1.3)
2. [Property descriptors](#2)
3. [Retrieving descriptors of properties](#3)
   1. [Object.getOwnPropertyDescriptor(): retrieving a descriptor for a single property](#3.1)
   2. [Object.getOwnPropertyDescriptors(): retrieving for all properties of an object](#3.2)
4. [Degining properties via descriptors](#4)
   1. [Object.defineProperty(): defining single properties via descriptors](#4.1)
   2. [Object.defineProperties(): defining multiple properties via descriptors](#4.2)
5. [Object.create(): Creating objects via descriptors](#5)
6. [Use cases for Object.getOwnPropertyDescriptors](#6)
   1. [Use case: copying properties into an object](#6.1)
   2. [Use case for Object.getOwnPropertyDescriptors(): cloning objects](#6.2)
7. [Omitting descriptor properties](#7)
   1. [Omitting descriptor properties when creating properties](#7.1)
   2. [Omitting descriptor properties when changing properties](#7.2)
8. [What property attributes do built-in constructs use](#8)
   1. [Own properties created via assignment](#8.1)
   2. [Own properties created via object literals](#8.2)
   3. [The own property .length of Arrays](#7.3)
   4. [Prototype properties of built-in classes](#8.4)
   5. [Prototype properties and instance properties of user-defined classes](#8.5)
9. [API: property descriptors](#9)
10. [Further reading](#10)



这一章，我们将近距离看一看ECMAScript规范是如何看待JS对象的。*特别的：在规范中属性不是原子性（`atomic`）的，而是由多个特性（`attributes`）组合而成（可理解为record中的字段）*。甚至是一个数据属性的值也是存储在一个特性中。



<p id="1"></p>



## 1️⃣ 对象结构

👩🏻‍🏫 [在ECMAScript规范中](https://tc39.es/ecma262/#sec-object-type)，一个对象由以下2个部分组成：

1. **内部插槽（`Internal slots`）**: 不能被JS所访问到的存储位置，只能被规范中的操作（`operations`）访问
2. **属性（`properties`）集合**： 每个属性都将键与特性（`attributes`）关联起来(可理解为记录中的字段)。



<p id="1.1"></p>



### 1.1 内部插槽

规范对[内部插槽](https://tc39.es/ecma262/#sec-object-internal-methods-and-internal-slots)的描述如下：

- 内部槽对应于与对象关联并被各种ECMAScript规范算法使用的**内部状态**
- **内部插槽不是对象属性，它们不被继承**
- 根据具体的内部槽规范，这种状态可能包括:
  - 任何ECMAScript语言类型的值
  - 或者具体ECMAScript规范类型值
- 除非显式的指出，否则内部插槽作为创建对象的一部分被分配，并且可能不能动态的添加到对象中
- 😎 除非特意指出，否则内部插槽的初始值是 `undefined`
- 该规范中的各种算法创建具有内部槽的对象。但是，**ECMAScript语言没有提供将内部槽与对象关联的直接方法。** 📚
- 内部方法和内部槽在规范中是等同的，使用双中括号的形式 `[[name]]`

👩🏻‍🏫 存在2种类型的内部槽：

- 方法槽用于操作对象（获取属性，设置属性等等）
- 数据槽存储值

📚 普通对象有如下数据槽（`data slots`）:

- `.[[Prototype]]: null | object`
  - 存储对象的原型
  - 可以直接通过 `Object.getPrototypeOf()` 和 `Object.setPrototypeOf()` 进行操作
- `.[[Extensible]]: boolean`
  - 表示是否可以给对象添加属性
  - 可以通过 `Object.preventExtensions()` 设置为 `false`
- `.[[PrivateFieldValues]]: EntryList`
  - 用于管理 [class私有字段](https://2ality.com/2019/07/private-class-fields.html)



<p id="1.2"></p>



### 1.2 属性键

属性存在2种类型的键：

- 字符串类型
- Symbol类型



<p id="1.3"></p>



### 1.3 属性特性

存在2种类型的属性，可以依据其特性分类：

- 数据属性（`data property`） 存储数据。它的特性 `value` 可存储任何JS值
- 访问器属性（`accessor property`） 由 `getter` |& `setter` 函数组成。前者存储在 `get` 特性中，后者存储在 `set` 特性中

🚀 另外一些特性是所有属性都存在的。下标列举出所有特性和其默认值：

| 属性类型 | 特性名称和类型                   | 默认值    |
| -------- | -------------------------------- | --------- |
| 数据属性 | value: any                       | undefined |
|          | writable: boolean                | false     |
| 访问属性 | get: (this: any) => any          | undefined |
|          | set: (this: any, v: any) => void | undefined |
| 所有属性 | configurable: boolean            | false     |
|          | enumerable: boolean              | False     |

我们已经碰到过了 `value` & `get` & `set` 特性。其余特性效果如下：

- `writable`：决定一个数据属性是否可以被更改
- `configurable`：决定一个属性的特性是否可以被更改。如果为 `false`，则：
  - 我们不能删除属性（译者注：`delete obj.prop` 松散模式下失败，严格模式下会报错）
  - 不能将一个数据属性更改为访问器属性，反之亦然
  - 除了 `value` 特性外，其它特性都不能再被更改 😎
  - 然而，还有一个特性是可以被更改：我们可以将 `writable` 特性从 `true` 更改到 `false`。这种异常现象背后的基本原理是[历史性](https://stackoverflow.com/questions/9829817/why-can-i-set-enumerability-and-writability-of-unconfigurable-property-descrip/9843191#9843191)的：数组的 `.length` 属性总是可写不可配置的。允许将它的 `writable` 特性更改，使我们能够冻结数组。
- `enumerable`：会影响到某些操作（比如 `Object.keys()`）。如果为 `false`，则这些操作将忽略该属性。大多数属性是可枚举的（比如：*通过赋值或者对象字面量创建的属性*），这也是为什么在实际中你很少会注意到这个特性。如果你对这个特性如何运作的很感兴趣，可以查看 [Enumerability of properties](https://exploringjs.com/deep-js/ch_enumerability.html) 这一章。

> 陷阱: 继承的不可写属性会阻止通过赋值创建自己的属性

如果一个继承的属性是不可写的，我们不能使用赋值来创建具有相同键值的自己的属性:

🌰

```js
const proto = {
  prop: 1
}

// 使 proto.prop 变为不可写
Object.defineProperty(
  proto, 'prop', { writable: false }
)

// obj 继承 proto
const obj = Object.create(proto)

// ❌ 不能对只读属性 `prop` 进行赋值
assert.throws(
  () => obj.prop = 2,
    /^TypeError: Cannot assign to read only property 'prop'/
)
```

更多信息可参考：

- [11.3.4 Inherited read-only properties prevent creating own properties via assignment](https://exploringjs.com/deep-js/ch_property-assignment-vs-definition.html#inherited-properties-prevent-assignment)



<p id="2"></p>



## 2️⃣ 属性描述符

*属性描述符（`property descriptor`）将属性的特性编码为 JavaScript 对象。* 它们的TypeScript接口看着如下：

👩🏻‍🏫：

```typescript
// 数据属性
interface DataPropertyDescriptor {
  value?: any;
  writable?: boolean;
  configurable?: boolean;
  enumerable?: boolean;
}

// 访问器属性
interface AccessorPropertyDescriptor {
  get?: (this: any) => any;
  set?: (this: any, v: any) => void;
  configurable?: boolean;
  enumerable?: boolean;
}

// 💡 可以看出属性描述符分为2类
type PropertyDescriptor = DataPropertyDescriptor | AccessorPropertyDescriptor
```

上面属性的 `?:` 表明所有特性都是可选的。[Omitting descriptor properties](#7) 描述如果特性被忽略了会发生什么。



<p id="3"></p>



## 3️⃣ 取回属性的特性

<p id="3.1"></p>



### 3.1 Object.getOwnPropertyDescriptor() 获取一个属性的描述符

🌰：

```js
const legoBrick = {
  kind: 'Plate 1x3',
  color: 'yellow',
  get description() {
    return `${this.kind} (${this.color})`
  }
}
```

先获取数据属性 `.color` 的描述符：

```js
assert.deepEqual(
  Object.getOwnPropertyDescriptor(legoBlock, 'color'),
  {
    value: 'yellow',
    writable: true,
    enumerable: true,
    configurable: true
  }
)
```

访问属性 `.description` 的描述符：

```js
const desc = Object.getOwnPropertyDescriptor.bind(Object)

assert.deepEqual(
  Object.getOwnPropertyDescriptor(legoBrick, 'description'),
  {
    get: desc(legoBrick, 'description').get, // A
    set: undefined,
    enumerable: true,
    configurable: true
  }
)
```

在 `A` 行使用工具函数 `desc()` 确保 `deepEqual()` 能正常运行。



<p id="3.2"></p>



### 3.2 Object.getOwnPropertyDescriptors()：获取对象上所有属性的描述器

🌰：

```js
const legoBrick = {
  kind: 'Plate 1x3',
  color: 'yellow',
  get description() {
    return `${this.kind} (${this.color})`
  }
}

const desc = Object.getOwnPropertyDescriptor.bind(Object)

assert.deepEqual(
  Object.getOwnPropertyDescriptors(legoBrick),
  {
    kind: {
      value: 'Plate 1x3',
      writable: true,
      enumerable: true,
      configurable: true,
    },
    color: {
      value: 'yellow',
      writable: true,
      enumerable: true,
      configurable: true,
    },
    description: {
      get: desc(legoBrick, 'description').get, // (A)
      set: undefined,
      enumerable: true,
      configurable: true,
    },
  })
```



<p id="4"></p>



## 4️⃣ 通过描述器定义属性

📚 如果我们通过描述器 `propDesc` 定义属性 `k`，会发生如下事情：

- 如果不存在属性 `k`，则创建一个新的属性，特性为 `propDesc`
- 如果已经存在属性 `k`，将会改变属性的特性，以匹配 `propDesc`



<p id="4.1"></p>



### 4.1 Object.defineProperty()：通过描述器定义单一属性

首先，我们通过描述器创建一个新属性：

```js
const car = {}

Object.defineProperty(car, 'color', {
  value: 'bluue',
  writable: true,
  enumerable: true,
  configurable: true
})

assert.deepEqual(
  car,
  {
    color: 'blue'
  }
)
```

😎接着，我们通过描述器改变属性类型，将数据属性变为一个 `getter`（访问器属性）：

```js
const car = {
  color: 'blue'
}

let readCount = 0
Object.defineProperty(car, 'color', {
  get() {
    readCount++
    return 'red'
  }
})

assert.equal(car.color, 'red')
assert.equal(readCount, 1)
```

最后，我们通过描述器改变一个数据属性的值：

```js
const car = { color: 'blue' }

// 使用相同的特性作为赋值
Object.defineProperty(
  car,
  'color',
  {
    value: 'green',
    writable: true,
    enumerable: true,
    configurable: true
  }
)

assert.deepEqual(
  car,
  { color: 'green' }
)
```

我们使用相同的属性特性作为赋值



<p id="4.2"></p>



### 4.2 Object.defineProperties()：通过描述器定义多个属性

`Object.defineProperties()` 是 `Object.defineProperty()` 多属性版本：

```js
const legoBrick1 = {}

Object.defineProperties(
  legoBrick1,
  {
    kind: {
      value: 'Plate 1x3',
      writable: true,
      enumerable: true,
      configurable: true
    },
    color: {
      value: 'yellow',
      writable: true,
      enumerable: true,
      configurable: true
    },
    description: {
      get: function() {
        return `${this.kind} (${this.color})`
      },
      enumerable: true,
      configurable: true
    }
  }
)

assert.deepEqual(
  kind: 'Plate 1x3',
  color: 'yellow',
  get description() {
    return `${this.kind} (${this.color})`
  }
)
```



<p id="5"></p>



## 5️⃣ Object.create()：通过描述器创建对象

📒 `Object.create()` 创建一个新对象：

- 第一个参数指定新对象的原型
- 第二个可选参数指定该对象的属性描述器

下面例子，我们使用之前相同的对象：

```js
const legoBrick2 = Object.create(
  Object.prototype,
  {
    kind: {
      value: 'Plate 1x3',
      writable: true,
      enumerable: true,
      configurable: true
    },
    color: {
      value: 'yellow',
      writable: true,
      enumerable: true,
      configurable: true
    },
    description: {
      get: function() {
        return `${this.kind} (${this.color})`
      },
      enumerable: true,
      configurable: true
    }
  }
)

// 🤔 我们真的创建一个相同的对象吗?
// 译者注：deepEqual只会对对象内容进行比较，对象的地址是不同的
assert.deepEqual(legoBrick1, legoBrick2) // 是的
```

<p id="6"></p>



## 6️⃣ Object.getOwnPropertyDescriptors()使用场景

如果结合 `Object.defineProperties()` 或者 `Object.create()` 一起使用的话， `Object.getOwnPropertyDescriptors()` 主要有2个使用场景：

1. 拷贝属性到对象中
2. 拷贝对象 



<p id="6.1"></p>



### 6.1 使用场景：拷贝属性到对象中

从ES6开始，JS已经存在一个工具用于拷贝属性了： `Object.assign()`。但是，这个方法使用简单的 get & set 操作拷贝属性：

```js
target[key] = source[key]
```

这意味着它只在以下情况下创建属性的可靠副本:

- 它的特性 `writable = true`，它的特性 `enumerable = true`(因为这是赋值创建属性的原因)
- 它是一个数据属性

下面示例展示了这一限制🥲。对象 `source` 有一个key为 `data` 的 `setter`:

```js
const source = {
  set data(value) {
    this._data = valuue
  }
}

// 属性 `data` 存在
// 因为只有一个setter，还没有赋值，所以其值为 `undefined`
assert.equal('data' in source, true)
assert.eqaul(source.data, undefined)
```

如果我们使用 `Object.assign()` 拷贝 `data` 属性，*则访问器属性 `data` 转换为一个数据属性*：😅

```js
const target1 = {}
Object.assign(target1, source)

// 拷贝后的对象转换为了数据属性
assert.deepEqual(
  Object.getOwnPropertyDescriptor(target1, 'data'),
  {
    value: undefined,
    writable: true,
    enumerable: true,
    configurable: true
  }
)

// 作为对比，source为：
// 💡 原对象为访问器属性
const desc = Object.getOwnPropertyDescriptor.bind(Object)

assert.deepEqual(
  Object.getOwnPropertyDescriptor(source, 'data'),
  {
    get: undefined,
    set: desc(source, 'data').set,
    enumerable: true,
    configurable: true,
  })
```

🚀 幸运的是，使用 `Object.getOwnPropertyDescriptors()` + `Object.defineProperties()` 可以完成对属性 `data` 可靠的拷贝:

```js
const target2 = {}
Object.defineProperties(
  target2, Object.getOwnPropertyDescriptors(source)
)

assert.deepEqual(
  Object.getOwnPropertyDescriptor(target2, 'data'),
  {
    get: undefined,
    set: desc(source, 'data').set,
    enumerable: true,
    configurable: true,
  })
```

> 陷阱：拷贝使用了 super 的方法

使用了 `super` 的方法和其 **家对象 （`home object`）**（存储它的对象） 紧紧的联系在一起。目前还没有办法将这样的方法移动或者拷贝到不同的对象上😅。



<p id="6.2"></p>



### 6.2 Object.getOwnPropertyDescriptors() 使用场景：拷贝对象

*浅拷贝类似于拷贝属性，这也是为什么 `Object.getOwnPropertyDescriptors()` 在这里是一个好选择的原因*。

为了创建克隆，我们使用 `Object.clone()`:

```js
const original = {
  set data(value) {
    return this._data = valuue
  }
}

const clone = Object.create(
  Object.getPrototypeOf(original),
  Object.getOwnPropertyDescriptors(original)
)

// 🚨 deepEqual 只对比属性值是否相同，不会对比对象的引用是否相同
assert.deepEqual(original, clone)
```

关于这个话题，可参考：

- [Copying objects and Arrays](https://exploringjs.com/deep-js/ch_copying-objects-and-arrays.html)



<p id="7"></p>



## 7️⃣ 忽略描述器属性

📚 **描述器的所有属性都是可选的。** 当忽略属性发生什么取决于具体操作。



<p id="7.1"></p>



### 7.1 当创建属性时忽略描述器属性

👩🏻‍🏫 当通过描述器创建一个新的属性，**忽略特性意味着使用忽略特性的默认值**：

```js
const car = {}
Object.defineProperty(
  car,
  'color',
  {
    value: 'red'
  }
)
// 💡 默认值全部为false
assert.deepEqual(
  Object.getOwnPropertyDescriptor(car, 'color'),
  {
    value: 'red',
    writable: false,
    enumerable: false,
    configurable: false,
  })
```



<p id="7.2"></p>



### 7.2 当改变属性时忽略描述器属性

如果我们改变一个已经存在的属性，忽略描述器属性意味着不会去触碰相应的特性：

```js
const car = {
  color: 'yellow'
}
// 💡 通过字面量赋值 特性默认值均为 true
assert.deepEqual(
  Object.getOwnPropertyDescriptor(car, 'color'),
  {
    value: 'yellow',
    writable: true,
    enumerable: true,
    configurable: true,
  })

// 改变已经存在的属性的特性
Object.defineProperty(
  car,
  'color',
  {
    value: 'pink'
  }
)

assert.deepEqual(
  Object.getOwnPropertyDescriptor(car, 'color'),
  {
    value: 'pink',
    writable: true,
    enumerable: true,
    configurable: true,
  })
```



<p id="8"></p>



## 8️⃣ 内置构造使用哪些属性特性？

👩🏻‍🏫 对属性特性的通用规则（存在某些例外）：

- 原型链开头的对象属性通常是可写的、可枚举的和可配置的
- 如 [枚举型](https://exploringjs.com/deep-js/ch_enumerability.html) 这章描述的，大多数继承的属性都是不可枚举的，为了将它们在传统的构造（比如 `for-in` 循环）中隐藏。继承的属性通常是可写可配置的



<p id="8.1"></p>



### 8.1 通过赋值（assignment）创建的属性

```js
const obj = {}
obj.prop = 3

assert.deepEqual(
  Object.getOwnPropertyDescriptors(obj),
  {
    prop: {
      value: 3,
      writable: true,
      enumerable: true,
      configurable: true,
    }
  })
```



<p id="8.2"></p>



### 8.2 通过对象字面量（Object literals）创建的属性

```js
const obj = { prop: 'yes' }

assert.deepEqual(
  Object.getOwnPropertyDescriptors(obj),
  {
    prop: {
      value: 'yes',
      writable: true,
      enumerable: true,
      configurable: true
    }
  })
```



<p id="8.3"></p>



### 8.3 数组自身的 .length 属性

👩🏻‍🏫 *数组的 `.length` 属性是不可枚举的*， 因此它不能通过 `Object.assign()` 或 `spreading` 或类似的操作拷贝。它也是不可配置的：

```js
Object.getOwnPropertyDescriptor([], 'length')
// { value: 0, writable: true, enumerable: false, configurable: false }

Object.getOwnPropertyDescriptor('abc', 'length')
// { value: 3, writable: false, enumerable: false, configurable: false }
```

`.length` 是一个特别的数据属性，它会受自身属性（具体讲是 **索引属性**）影响，或者它会影响自身属性。



<p id="8.4"></p>



### 8.4 内置类原型属性

```js
assert.deepEqual(
  Object.getOwnPropertyDescriptor(Array.prototype, 'map'),
  {
    value: Array.prototype.map,
    writable: true,
    enumerable: false,
    configurable: true
  }
)
```



<p id="8.5"></p>



### 8.5 ⭐️用户自定义类的原型属性和实例属性

```js
class DataContainer {
  accessCount = 0
  constructor(data) {
    this.data = data
  }
  getData() {
    this.accessCount++
    return this.data
  }
}

// 💡 可以看出原型上的属性都是可写可配置，不可枚举的
assert.deepEqual(
 Object.getOwnPropertyDescriptors(DataContainer.prototype),
  // 🚨 构造器
  constructor: {
    value: DataContainer,
    writable: true,
    enumerable: false,
    configurable: true,
  },
  getData: {
    value: DataContainer.prototype.getData,
    writable: true,
    enumerable: false,
    configurable: true,
  }
)
```

💡而实例属性都是可写可配置可枚举的：

```js
const dc = new DataContainer('abc')

assert.deepEqual(
  Object.getOwnPropertyDescriptors(dc),
  {
    accessCount: {
      value: 0,
      writable: true,
      enumerable: true,
      configurable: true,
    },
    data: {
      value: 'abc',
      writable: true,
      enumerable: true,
      configurable: true,
    }
  })
```



<p id="9"></p>



## 9️⃣ API：属性描述器

API1：

```js
// ES5
Object.defineProperty(
  obj: object,
  key: string|symbol,
  propDesc: PropertyDescriptor
): object
```

创建或者改变 `obj` 上属性 `key`,其特性通过 `propDesc`。返回被修改的对象：

```js
const obj = {}
const result = Object.defineProperty(
  obj,
  'happy',
  {
    value: 'yes',
    writable: true,
    enumerable: true,
    configurable: true
  }
)
// 对象被修改并返回
assert.equal(result, obj)
assert.deepEqual(obj, {
  happy: 'yes',
})
```

API2：

```typescript
// ES5
Object.defineProperties(
  obj: object,
  properties: { [K: string|symbol]: PropertyDescriptor }
): object
```

*`Object.defineProperty()` 的批量版本*。对象属性的每个属性 `p` 指定一个要添加到 obj 的属性：

```js
const address1 = Object.defineProperties({}, {
  street: { value: 'Evergreen Terrace', enumerable: true },
  number: { value: 742, enumerable: true },
})
```

API3:

```typescript
// ES5
Object.create(
  proto: null|object, 
  properties?: { [k: string|symbol]: PropertyDescriptor }
): object
```

首先，创建一个原型为 `proto` 的对象。然后，如果提供了第二个可选参数 `properties`，则以 `Object.defineProperties()` 相同的方式给新建的对象添加属性。最后返回结果。比如，下面代码产生和之前代码一样的结果：

```js
const address2 = Object.create(
  Object.prototype,
  {
    street: { value: 'Evergreen Terrace', enumerable: true },
    number: { value: 742, enumerable: true }
  }
)

assert.deepEqual(address1, address2)
```

API4：

```typescript
// ES5
Object.getOwnPropertyDescriptor(
  obj: Object,
  key: string|symbol
): undefined | PropertyDescriptor
```

返回对象 `obj` 自身属性 `key` (**非继承属性**) 的描述器。如果对象不存在该属性，则返回 `undefined`：

```js
assert.deepEqual(
  Object.getOwnPropertyDescriptor(Object.prototype, 'toString'),
  {
    value: {}.toString,
    writable: true,
    enumerable: false,
    configurable: true
  }
)

// 非自身属性 返回undefined
assert.equal(
  Object.getOwnPropertyDescriptor({}, 'toString'),
  undefined
)
```

API5:

```typescript
// ES7
Object.getOwnPropertyDescriptors(
  obj: Object
): { [k: string|symbol]: PropertyDescriptor }
```

返回对象 `obj` 自身所有属性描述器。返回结果可用作 `Object.defineProperties()` & `Object.create()` 的输入 🚀🚀：

```js
const propertyKey = Symbol('propertyKey')
const obj = {
  [propertyKey]: 'abc',
  get count() { return 123 }
}

const desc = Object.getOwnPropertyDescriptors.bind(Object)

assert.deepEqual(
  Object.getOwnPropertyDescriptors(obj),
  {
    [propertyKey]: {
      value: 'abc',
      writable: true,
      enumerable: true,
      configurable: true
    },
    count: {
      get: desc(obj, 'count').get, // A
      set: undefined,
      enumerable: true,
      configurable: true
    }
  }
)
```



<p id="10"></p>



## 🔟 进一步阅读

下面3个章节提供了对属性特性更多详细的细节：

- [Protecting objects from being changed](https://exploringjs.com/deep-js/ch_protecting-objects.html)
- [Properties: assignment vs. definition](https://exploringjs.com/deep-js/ch_protecting-objects.html)
- [Enumerability of properties](https://exploringjs.com/deep-js/ch_protecting-objects.html)



2022年07月10日10:22:25