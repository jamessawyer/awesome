目录：

1. [What is type coercion?](#1)
   1. [Dealing with type coercion](#1.1)
2. [Operations that help implement coercion in the ECMAScript specification](#2)
   1. [Converting to primitive types and objects](#2.1)
   2. [Converting to numeric types](#2.2)
   3. [Converting to property keys](#2.3)
   4. [Converting to Array indices](#2.4)
   5. [Converting to Typed Array elements](#2.5)
3. [Intermission: expressing specification algorithms in JavaScript](#3)
4. [Example coercion algorithms](#4)
   1. [ToPrimitive()](#4.1)
   2. [ToString() and related operations](#4.2)
   3. [ToPropertyKey()](#4.3)
   4. [ToNumeric() and related operations](#4.4)
5. [Operations that coerce](#5)
   1. [Addition operator (+)](#5.1)
   2. [Abstract Equality Comparison(==)](#5.2)
6. [Glossary: terms related to type conversion](#6)



本章我们将探索JS中 *类型强转（`type coercion`）*。并且我们将相对深入这个主题，例如，ECMAScript规范是如何处理强转的。



<p id="1"></p>



## 1️⃣ 什么是类型强转？

👩🏻‍🏫 每个操作（函数，操作符等）都期望其参数是正确的类型。如果某个参数的类型不正确，通常会出现3种选项，比如，一个函数：

1. 函数抛出一个异常

   ```js
   function multiply(x, y) {
     if (
       typeof x !== 'number' ||
       typeof y !== 'number'
     ) {
       throw new TypeError()
     }
     // ...
   }
   ```

2. 函数返回一个不正确的值

   ```js
   function multiply(x, y) {
     if (
       typeof x !== 'number' ||
       typeof y !== 'number'
     ) {
       return NaN
     }
     // ...
   }
   ```

3. 函数将其参数转换为正确的类型：

   ```js
   function multiply(x, y) {
     if (typeof x !== 'number') {
       x = Number(x)
     }
     if (typeof y !== 'number') {
       y = Number(y)
     }
     // ...
   }
   ```

**在第3种情况中，操作执行了隐式类型转换，这就是所谓的类型强转（`Type Coercion`）**。

JS开始的时候是没有异常的，这也是为什么JS会对大多数操作使用强转和错误值😅：

```js
// 强转
assert.equal(3 * true, 3)

// 错误值代替
assert.equal(1 / 0, Infinity)
assert.equal(Number('xyz'), NaN)
```

然而，有些情况（尤其是JS最新的一些特性）在参数类型不正确时，会抛出异常：

- 访问 `null` 或 `undefined` 属性：

  ```js
  undefined.prop
  // ❌ TypeError: 不能读取undefined的属性 `prop`
  
  null.prop
  // ❌ TypeError: 不能读取null的属性 `prop`
  
  'prop' in null
  // ❌ TypeError: 不能在 null中使用 `in` 操作符搜索 `prop`
  ```

- 使用symbols:

  ```js
  6 / Symbol()
  // ❌ TypeError: 不能将一个Symbol值转换为number类型
  ```

- 混合使用 `bigints` & `numbers`：

  ```js
  // `数字 + n` 表示 BigInt 类型 
  6 / 3n
  // ❌ TypeError: 不能将BigInt和其它类型混合
  ```

- 不支持该操作的 `New` 调用 或 函数调用

  ```js
  123()
  // ❌ TypeError: 123不是一个函数
  
  (class {})()
  // // ❌ TypeError: 类构造器调用要使用 `new` 操作符
  
  new (() => {})
  // ❌ TypeError: (中间值) 不是一个构造器
  ```

- 改变只读属性（只在严格模式下抛出错误）：

  ```js
  // 可通过 Object.getOwnPropertyDescriptor('abc', 'length') 查看描述器属性
  'abc'.length = 1
  // ❌ TypeError: 不能给只读属性 `length` 赋值
  
  Object.freeze({prop: 3}).prop = 1
  // ❌ TypeError: 不能给只读属性 `prop` 赋值
  ```



<p id="2.1"></p>



### 2.1 处理类型强转

📚 2种处理强转的常见方式：

1. 调用者显式的转换其值，因此让其拥有正确类型。比如：下面🌰，我们像将2个字符串编码的数字相乘：

   ```js
   let x = '3'
   let y = '2'
   assert.equal(Number(x) * Number(y), 6)
   ```

2. *调用者让操作(`*` 操作)帮忙转换类型* 😂

   ```js
   let x = '3'
   let y = '2'
   assert(x * y, 6)
   ```

我通常偏向使用第一种方式，因为它能很好的阐明我的意图：我希望x和y虽然不是数字，但是两个数字能相乘。（译者注：即让字符串数字能正确相乘）



<p id="2"></p>

## 2️⃣ ⭐ ECMAScript规范中帮助实现强转的操作

下面我们将描述ECMAScript规范中用于将实际参数转换为期望类型最重要的内部函数。

比如，在TypeScript中，我们可能写为：

```typescript
function isNaN(number: number) {
  // ...
}
```

在规范中，这可能看着 [如下](https://tc39.es/ecma262/#sec-multiplicative-operators-runtime-semantics-evaluation) （翻译为JS，以便我们能更好的理解）

```js
function isNaN(number) {
  // ToNumber 是后面会定义的一个算法操作
  let num = ToNumber(number)
  // ...
}
```



<p id="2.1"></p>



### 2.1 转换为原始类型和对象

当期望原始类型或者对象时，将使用下面（规范内部）转换函数：

- `ToBoolean()`
- `ToNumber()`
- `ToBigInt()`
- `ToString()`
- `ToObject()`

上面内部函数，JS中存在类似的方法：

```js
Boolean(0)
// false

Boolean(1)
// true

Number('123')
// 123
```

在引入bigints(与数字一起存在)之后，规范通常使用 `ToNumeric()`，而之前使用的是 `ToNumber()`。



<p id="2.2"></p>



### 2.2 转换为数值类型

目前，JS中存在2种 [内置数值类型](https://tc39.es/ecma262/#sec-numeric-types)： *number & bigint*。

👩🏻‍🏫：

1. `ToNumeric()` 返回一个数字值 `num`。它的调用者通常调用规范类型`num`的`mthd`方法: 

   ```js
   Type(num)::mthd(...)
   ```

   其余的，下面操作使用的是 `ToNumeric()`: 🚀

   - `++x` & `x++` 操作符
   - `*` 操作符

2. 当期望数值没有小数部分时，会使用 `ToInteger(x)` 。之后其结果返回通常会进一步限制：

   - 它会先调用 `ToNumber(x)`，然后移除小数部分（类似 `Math.trunc()`）
   - 使用了 `ToInteger()` 的一些操作：
     - `Number.prototype.toString(radix?)`
     - `String.prototype.codePointAt(pos)`
     - `Array.prototype.slice(start, end)`
     - 等等

3. `ToInt32()` & `ToUint32()` 将数值转换为32位整数，用于 **位操作**（如下表）

   - `ToInt32()`：有符，区间 `[−2^31, 2^31−1] `
   - `ToUint32()`： 无符，区间 `[0, 2^32-1]`

位数字操作符的操作数强制转换（BigInt操作符没有限制位的数量）

|    操作符    | 左操作数  |  右操作数  | 结果类型 |
| :----------: | :-------: | :--------: | :------: |
|   左移 <<    | ToInt32() | ToUint32() |  Int32   |
| 有符右移 >>  | ToInt32() | ToUint32() |  Int32   |
| 无符右移 >>> | ToInt32() | ToUint32() |  Uint32  |
|   &, ^, \|   | ToInt32() | ToUint32() |  Int32   |
|      ~       |    --     | ToInt32()  |  Int32   |



<p id="2.3"></p>



### 2.3 转换为属性键

`ToPropertyKey()` 返回一个字符串或symbol，下面操作使用了这个内部方法：

- 中括号操作符 `[]`
- 对象字面量中的计算属性
- `in` 操作符的左手（例如： `'prop' in obj` 中的 `prop`）
- `Object.defineProperty(_, P, _)` 中的 `P` 属性部分
- `Object.fromEntries()`
- `Object.getOwnPropertyDescriptor()`
- `Object.prototype.hasOwnProperty()`
- `Object.prototype.propertyIsEnumerable()`
- `Reflect` 的几个方法



<p id="2.4"></p>



### 2.4 转换为数组索引

- `ToLength()` 主要用于字符串索引
  - `ToIndex()` 的辅助函数
  - 结果区间：`0 <= l <= 2^53-1`
- `ToIndex()` 用于类型数组（`Typed Array`）索引
  - 和 `ToLength()` 主要差别是：如果参数超出范围抛出异常
  - 结果区间：`0 <= i <= 2^53-1`
- `ToUint32()` 用于数组索引
  - 结果区间：`0 <= i < 2^32-1`



<p id="2.5"></p>



### 2.5 转换为类型数组元素

当我们设置一个类型数组元素的值是，会使用下面中某一个转换：

- `ToInt8()`
- `ToUint8()`
- `ToUint8Clamp()`
- `ToInt16()`
- `ToUint16()`
- `ToInt32()`
- `ToUint32()`
- `ToBigInt64()`
- `ToBigUint64()`



<p id="3"></p>



## 3️⃣ ⭐中场休息：以JS形式表示规范中的算法



本章剩下部分，我们会碰到几个规范算法，但是是以JS的方式实现的。**下面介绍如何将规范转换为JS的写法**：

🚀🚀：

- 规范： `If Type(value) is String`
  - JS: `if (TypeOf(value) === 'string')`
- 规范：`If IsCallable(method) is true`
  - JS: `if (IsCallable(method))`
- 规范：`Let numValue be ToNumber(value)`
  - JS：`let numValue = Number(value)`
- 规范：`Let isArray be IsArray(O)`
  - JS：`let isArray = Array.isArray(O)`
- 规范：`If O has [[NumberData]] internal slot`
  - JS：`if ('__NumberData__' in O)`
- 规范：`Let tag be Get(O, @@toStringTag)`
  - JS：`let tag = O[Symbol.toStringTag]`
- 规范：`Return the string-concatenation of "[object",tag,and "]"`
  - JS：`return '[object ' + tag + ']'`

`let`（而不是 `const`） 用于匹配语言规范。

有些细节被忽略了，比如 [ReturnIfAbrupt 缩写符 `？` & `！`](https://tc39.es/ecma262/#sec-returnifabrupt-shorthands)

```js
// typeof 改进版本
function TypecOf(value) {
  const result = typeof value
  switch (result) {
    case 'function':
      return 'object'
    case 'object':
      if (value === null) {
        return 'null'
      } else {
        return 'object'
      }
    default:
      return result
  }
}

function IsCallable(x) {
  return typeof x === 'function'
}
```



<p id="4"></p>



## 4️⃣ ⭐强转算法JS版



<p id="4.1"></p>



### 4.1 ToPrimitive()

[ToPrimitive()](https://tc39.es/ecma262/#sec-toprimitive) 操作是其它强转算法的中间步骤😎。*它将任意值转换为原始类型值。*

`ToPrimitive()` 在规范中经常被使用，因为大多数操作符只能和原始类型的值一起使用🤩。比如，我们经常使用 `+` 操作符让数字相加和字符串拼接，但是我们不能用它将2个数组进行拼接。

下面是JS版本的 `ToPrimitive()`:

```typescript
/**
 * 
 * @param input any 输入值
 * @param hint 'number' | 'string' | 'default' 更倾向于返回哪种类型结果，default 表示不在乎
 */
function ToPrimitive(
  input: any, 
  hint: 'number' | 'string' | 'default' = 'default'
) {
  if (TypeOf(input) === 'object') { // 如果输入是一个对象
    // 先查看对象是是否定义了 Symbol.toPrimitive 方法
    // ToPrimitive - A method that converts an object to a corresponding primitive value.
    let exoticToPrim = input[Symbol.toPrimitive] // A
    if (exoticToPrim !== undefined) {
      let result = exoticToPrim.call(input, hint)
      if (TypeOf(result) !== 'object') {
        // 如果结果不是对象，说明已经转换为了原始类型值
        return result
      }
      throw new TypeError()
    }

    // 如果没有定义 Symbol.toPrimitive 方法
    if (hint === 'default') {
      // 💡如果没有指定想要转换的类型 则优先转换为数值类型
      hint = 'number'
    }
    
    return OrdinaryToPrimitive(input, hint) // 调用另一个方法
  } else {
    // input已经是原始类型了
    return input
  }
}
```

💡 `ToPrimitive()` 允许对象通过 `Symbol.toPrimitive` 方法（A行）覆写转换为原始类型值的结果。如果对象上没有定义这个方法，则继续通过 `OrdinaryToPrimitive()` 处理：

```typescript
function OrdinaryToPrimitive(O: object, hint: 'number' | 'string') {
  let methodNames // 要调用的方法
  if (hint === 'string') {
    // 如果偏向于转换为字符串类型 调用方法顺序如下，
    // 先 'toString'，如果返回结果不正确，再调用 'valueOf'
    methodNames = ['toString', 'valueOf']
  } else {
    methodNames = ['valueOf', 'toString']
  }

  for (let name of methodNames) {
    let method = O[name] // 调用对象上对象的 toString | valueOf 方法
    if (IsCallable(method)) { // 如果是一个方法
      let result = method.call(O)
      if (TypeOf(result) !== 'object') {
        // 如果返回的结果不是一个对象，则说明已经返回了原始类型的值了
        return result
      }
    }
  }
  throw new TypeError()
}
```

#### ⭐4.1.1 调用者的 ToPrimitive() 使用了什么hints？

👩🏻‍🏫 `ToPrimitive` 的参数 `hint` 有3种值：

1. `number` 表示：如果可能的话，`input` 应当转换为数值类型的值
2. `string` 表示：如果可能的话，`input` 应当转换为字符串类型的值
3. `default` 表示：对转换为数字还是字符串无所谓

🚀🚀 下面示例展示各种操作是如何使用 `ToPrimitive()` 的：

1. `hint === 'number'`，下面操作更偏向于转换为数值类型：
   - `ToNumeric()`
   - `ToNumber()`
   - `ToBigInit()` & `BigInt()`
   - 抽象关系比较 （`<`）
2. `hint === 'string'`，下面操作更偏向于转换为字符串类型：
   - `ToString()`
   - `ToPropertyKey()`
3. `hint === 'default'`，下面操作是*中性的*，返回原始值的类型:
   - 抽象相等性比较（`==`）
   - 加操作符（`+`）
   - `new Date(value)` (`value` 可以是数字也可以是字符串)

正如我们在 `ToPrimitive()` 中所见，`default` 默认行为偏向于 `number`。只有 `Symbol` 和 `Date` 的实例覆盖了这种行为🤔（之后会展示）



#### 4.1.2 将对象转换为原始类型调用哪一个方法？

如果转换为原始类型没有被 `Symbol.toPrimitive` 覆写，`OrdinaryToPrimitive()` 会调用下面方法中的一种，或者2种都调用：

1. 如果 `hint` 为字符串，则调用 `toString` 
2. 如果 `hint` 为数值，则调用 `valueOf`

🌰下面代码演示了这一工作过程：

```js
const obj = {
  toString() { return 'a' },
  valueOf() { return 1 }
}

// String() 偏向于转换为字符串
String(obj) // 'a'

// Number() 偏向于转换为数字
Number(obj) // 1
```

方法 `Symbol.toPrimitive` 覆写了正常转换原始类型。这种情况只在标准库中使用了2次：（译者注：可使用 `Object.getOwnPropertyDescriptors(Symbol.prototype)` 进行查看）

1. `Symbol.prototype[Symbol.toPrimitive](hint)`
   - 如果接收者是一个Symbol实例，这个方法总是返回包装的symbol
   - 根本原因是，`Symbol` 实例有一个 `.toString()` 方法返回字符串。但即使其 `hint` 为 `string`, `.toString()` 也不应该被调用，因此我们不会不小心将Symbol实例转换为字符串（*Symbol完全是一种不能类型的属性键*）
2. `Date.prototype[Symbol.toPrimitve](hint)`
   - 下面有更详细的解释



#### 4.1.3 Date.prototype\[Symbol.toPrimitive\]()

下面是 `Dates` 如何处理转换为原始类型值的：

```typescript
Date.prototype[Symbol.toPrimitive] = function(hint: 'default' | 'string' | 'number') {
  let O = this
  if (TypeOf(O) !== 'object') {
    throw new TypeError()
  }

  let tryFirst

  // 可以看出这里如果为 `default` 也更偏向于返回 `string` 
  if (hint === 'string' || hint === 'default') {
    tryFirst = 'string'
  } else if (hint === 'number') {
    tryFirst = 'number'
  } else {
    throw new TypeError()
  }

  return OrdinaryToPrimitive(O, tryFirst)
}
```

📚 这里和默认算法唯一的区别是 `default` 变为了 `string` (而不是 `number`)。这可以通过那些将 `hint` 设置为 `default` 的操作（译者注：即上面提到的中性操作）进行观测：

- 如果其它操作数不是 `undefined | null | boolean `时，  [== 操作符](https://exploringjs.com/deep-js/ch_type-coercion.html#abstract-equality-comparison) 会使用 `default hint` 将对象强制为原始类型。下例中，我们可以看到date强转的结果是一个字符串：

  ```js
  const d = new Date('2222-03-27')
  
  // 可以看出 Date 对于hint为 default时，默认偏向于转换为字符串
  assert.equal(
    d == 'Wed Mar 27 2222 08:00:00 GMT+0800 (中国标准时间)',
    true
  )
  ```

- 🚀🚀 [+ 操作符](https://exploringjs.com/deep-js/ch_type-coercion.html#addition-operator) 会使用 `default hint` 将2个操作数（`operands`） 强转为原始类型。如果其中一个结果为字符串，则执行字符串拼接（否则则执行数字相加）。下例中，我们可以发现date强转的结果是字符串，因为操作符返回一个字符串

  ```js
  const d = new Date('2222-03-27')
  
  assert.equal(
    123 + d，
    '123Wed Mar 27 2222 08:00:00 GMT+0800 (中国标准时间)'
  )
  ```



<p id="4.2"></p>



### 4.2 ToString() 和相关操作

下面是JS版本 `ToString()` 操作：

```typescript

function ToString(argument: any) {
  if (argument === undefined) {
    return 'undefined'
  } else if (argument === null) {
    return 'null'
  } else if (argument === true) {
    return 'true'
  } else if (argument === false) {
    return 'false'
  } else if (TypeOf(argument) === 'number') {
    return Number.toString(argument)
  } else if (TypeOf(argument) === 'string') {
    return argument
  } else if (TypeOf(argument) === 'symbol') {
    throw new TypeError()
  } else if (TypeOf(argument) === 'bigint') {
    return BigInt.toString(argument)
  } else {
    // argument 是一个对象
    // 调用之前定义的 ToPrimitive, hint 为 'string'
    let primValue = ToPrimitive(argument, 'string') // A
    return ToString(primValue)
  }
}
```

注意这个函数在转换原始类型结果为字符串前，会使用 `ToPrimitive()` 作为对对象转换为原始类型的中间步骤（`A` 行）。

`ToString()` 以一种有趣的方式偏离了 `String()` 的工作方式：如果 `argument` 是一个symbol，`ToString()` 抛出一个 `TypeError`, 而 `String()` 则不会。为什么会这样呢？*因为symbol的默认行为是将它们转换为字符串会抛出异常:*

```js
const sym = Symbol('sym')

'' + sym
// ❌ TypeError: 不能将一个Symbol值转换为字符串

`${sym}`
// ❌ TypeError: 不能将一个Symbol值转换为字符串
```

这种默认行为在 `String()` 和 `Symbol.prototype.toString()` 中都被覆写（2种操作都在后面介绍）

```js
String(sym)
// ✅
// 'Symbol(sym)'

sym.toString()
// ✅
// 'Symbol(sym)'
```



#### 4.2.1 String()

```js
function String(value) {
  let s
  if (value === undefined) {
    s = ''
  } else {
    if (new.target === undefined && TypeOf(value) === 'symbol') {
      // 这个函数是函数调用的，值是一个Symbol
      return SymbolDescriptiveString(value)
    }
    s = ToString(value)
  }

  if (new.target === undefined) {
    // 这个函数是函数调用的
    return s
  }
  // This function was new-called 即构造函数
  return StringCreate(s, new.target.prototype) // 简化版本
}
```

`String()` 的工作效果差异取决于是函数调用还是通过 `new` 调用（即构造函数调用）。它使用 [new.target](https://exploringjs.com/es6/ch_classes.html#sec_allocating-and-initializing-instances) 来区分这2种函数。

下面是 `StringCreate()` 和 `SymbolDescriptiveString()` 2个辅助函数：

```js
/**
 * 创建一个包装了 value 的字符串示例，并拥有给定的原型
 */
function StringCreate(value, prototype) {
  // ...
}

function SymbolDescriptiveString(sym) {
  assert.equal(TypeOf(sym), 'symbol')
  let desc = sym.description
  if (desc === undefined) {
    desc = ''
  }
  assert.equal(TypeOf(desc), 'string')
  return 'Symbol(' + desc + ')'
}
```



#### 4.2.2 Symbol.prototype.toString()

除了 `String()`，我们也可以使用 `.toString()` 将一个symbol转换为字符串。它的规范看起来如下：

```js
Symbol.prototype.toString = function() {
  let sym = thisSymbolValue(this)
  return SymbolDescriptiveString(sym)
}

function thisSymbolValue(value) {
  if (TypeOf(value) === 'symbol') {
    return value
  }

  if (TypeOf(value) === 'object' && '__SymbolData__' in value) {
    let s = value.__SymbolData__
    // 确保 s 为 Symbol类型 
    assert.equal(TypeOf(s), 'symbol')
    return s
  }
}
```



#### 4.2.3 ⭐ Object.prototype.toString 

`.toString()` 默认规范看起来如下：🚀🚀

```js
Object.prototype.toString = function() {
  if (this === 'undefined') {
    return '[object Undefined]'
  }

  if (this === 'null') {
    return '[object Null]'
  }

  let O = ToObject(this)
  let isArray = Array.isArray(O)
  let builtinTag // 内置标签
  if (isArray) {
    builtinTag = 'Array'
  } else if ('__ParameterMap__' in O) { // [[ParameterMap]]
    builtinTag = 'Arguments'
  } else if ('__Call__' in O) { // [[Call]]
    builtinTag = 'Function'
  } else if ('__ErrorData__' in O) { // [[ErrorData]]
    builtinTag = 'Error'
  } else if ('__BooleanData__' in O) { // [[BooleanData]]
    builtinTag = 'Boolean'
  } else if ('__NumberData__' in O) { // [[NumberData]]
    builtinTag = 'Number'
  } else if ('__DateValue__' in O) { // [[DateValue]]
    builtinTag = 'Date'
  } else if ('__RegExpMatcher__' in O) { // [[RegExpMatcher]]
    builtinTag = 'RegExp'
  } else { // 默认 Object
    builtinTag = 'Object'
  }

  // 先查看是否存在 Symbol.toStringTag 定义
  let tag = O[Symbol.toStringTag]
  if (TypeOf(tag) !== 'string') {
    // 不存在则使用内置tag
    tag = builtinTag
  }

  return '[object ' + tag + ']'
}
```

可参考：（译者注）

- [Object.prototype.toString - tc39](https://tc39.es/ecma262/multipage/fundamental-objects.html#sec-object.prototype.tostring)

这个操作用于将普通对象转换为字符串时：

```js
String({})
// '[object Object]'
```

默认，它也可用于将类实例转换为字符串：

```js
class MyClass {}
String(new MyClass())
// '[object Object]'
```

正常情况下，我们可能为了配置MyClass的字符串表达形式，覆写 `.toString()`方法，但我们也可以用方括号改变字符串中 `" object "` 后面的内容:

```js
class MyClass {}
MyClass.prototype[Symbol.toStringTag] = 'Custom!'

String(new MyClass())
// '[object Custom!]'
```

对比覆写版本的 `.toString()` 和 `Object.prototype` 原始版本很有趣：

```js
// Array.prototype.toString 覆写了 Object.prototype.toString 的方法 
// 或者 Array.prototype.toString.call(['a', 'b'])
['a', 'b'].toString()
// 'a,b'

Object.prototype.toString.call(['a', 'b'])
// '[object Array]'

/^abc$/.toString()
// '/^abc$/'

Object.prototype.toString.call(/^abc$/)
// '[object RegExp]'
```



<p id="4.3"></p>



### 4.3 ToPropertyKey()

`ToPropertyKey()` 用于括号操作符。这是它工作的方式：

```js
function ToPropertyKey(argument) {
  let key = ToPrimitive(argument, 'string') // 再次使用到了 ToPrimitive 
  if (TypeOf(key) === 'symbol') {
    return key
  }
  // 调用上面的 ToString() 操作
  return ToString(key)
}
```

再一次，在需要原始类型前，将对象转换为了原始类型



<p id="4.4"></p>



### 4.4 ToNumeric() 和相关操作

`ToNumeric()` 被用于乘法操作符（`*`）。工作方式如下：

```js
function ToNumeric(value) {
  let primValue = ToPrimitive(value, 'number') // 再次使用到了 ToPrimitive 
  if (TypeOf(primValue) === 'bigint') {
    // 对 BigInt 单独处理
    return primValue
  }
  // 调用后面定义的 ToNumber 操作
  return ToNumber(primValue)
}
```

#### 4.4.1 ⭐ ToNumber()

`ToNumber()` 工作如下：(相当于调用 `valueOf()`)

```js
function ToNumber(argument) {
  if (argument === undefined) {
    return NaN
  } else if (argument === null) {
    return +0
  } else if (argument === true) {
    return 1
  } else if (argument === false) {
    return +0
  } else if (TypeOf(argument) === 'number') {
    return argument
  } else if (TypeOf(argument) === 'string') {
    // 如果是字符串 就尝试转换为数字
    return parseTheString(argument)
  } else if (TypeOf(argument) === 'symbol') {
    // 如果是Symbol则直接抛出错误
    throw new TypeError()
  } else if (TypeOf(argument) === 'bigint') {
    throw new TypeError()
  } else {
    // argument是一个对象
    // 再次调用 ToPrimitive 将对象转换为原始类型值
    let primValue = ToPrimitive(argument, 'number')
    return ToNumber(primValue) // 递归一下
  }
}
```

可以看出 `ToNumber()` 和 `ToString()` 结构相似。



<p id="5"></p>

## 5️⃣ 强转的操作

<p id="5.1"></p>

### 5.1 ⭐ + 操作

下面就是 `+` 操作算法：

```js
function Addition(leftHandSide, rightHandSide) {
  // 先用ToPrimitive转换为原始类型
  let lprim = ToPrimitive(leftHandSide)
  let rprim = ToPrimitive(rightHandSide)
  
  // 如果左操作数或右操作数有一个是字符串
  if (TypeOf(lprim) === 'string' || TypeOf(rprim) === 'string') { // A
    // 将2个操作数都调用ToString() 然后拼接在一起
    return ToString(lprim) + ToString(rprim)
  }
  
  let lnum = ToNumeric(lprim)
  let rnum = ToNumeric(rprim)
  if (TypeOf(lnum) !== TypeOf(rnum)) {
    // 如果转换为数值类型后，类型还不相同 则抛出错误
    throw new TypeError()
  }
  
  let T = Type(lnum)
  return T.add(lnum, rnum) // B
}
```

算法步骤：

- 2个操作数都转换为原始类型
- 如果其中一个结果是字符串，则2个操作数都转换为字符串，并拼接在一起（行A）
- 否则，2个操作数转换为数值类型相加（行B）, `Type()` 返回 ECMAScript规范中 `lnum.add()` 方法需要的 [numeric 类型](https://tc39.es/ecma262/#sec-numeric-types)

译者注示例🌰：

```js
const obj = {
  toString() { return 'a' },
  valueOf() { return 1 }
}

1 + obj // 2 此时会调用obj valueOf


'x' + obj 
// 'x1' 
// ToPrimitive(obj) 默认 hint为 'number'
// 因此方法调用顺序为 先 valueOf
```



<p id="5.2"></p>



### 5.2 ⭐ 抽象相等性比较（==）

```js
// 松相等性比较（==）
function abstractEqualityComparison(x, y) {
  // 1. 如果类型相同 直接严格比较
  if (TypeOf(x) === TypeOf(y)) {
    // 使用严格相等性 ===
    return strictEqualityComparison(x, y)
  }
  
  // 2. 比较 null 和 undefined
  if (x === null && y === undefined) {
    // js中 null == undefined 返回 true
    return true
  }
  
  if (x === undefined && y === null) {
    return true
  }
  
  // 3. 比较数字和字符串
  if (TypeOf(x) === 'number' && TypeOf(y) === 'string') {
    // 将字符串转换为数字再进行比较
    return abstractEqualityComparison(x, Number(y))
  }
  
   if (TypeOf(x) === 'string' && TypeOf(y) === 'number') {
    // 将字符串转换为数字再进行比较
    return abstractEqualityComparison(Number(x), y)
  }
  
  // 4. 比较 bigint 和 字符串
  if (TypeOf(x) === 'bigint' && TypeOf(y) === 'string') {
    let n = StringToBigInt(y)
    if (Number.isNaN(n)) {
      return false
    }
    return abstractEqualityComparison(x, n)
  }
  
  if (TypeOf(x) === 'string' && TypeOf(y) === 'bigint') {
    return abstractEqualityComparison(y, x)
  }
  
  // 5. 比较布尔值和非布尔值
  if (TypeOf(x) === 'boolean') {
    return abstractEqualityComparison(Number(x), y)
  }
  
   if (TypeOf(y) === 'boolean') {
     return abstractEqualityComparison(x, Number(y))
  }
  
  // 6. 比较对象和原始类型
  // 除了 undefined null & boolean
  if (
    ['string', 'number', 'bigint', 'symbol'].includes(TypeOf(x))
    && TypeOf(y) === 'object'
  ) {
    // 将y转换为原始类型
    return abstractEqualityComparison(x, ToPrimitive(y)) 
  }
  
  if (
    TypeOf(x) === 'object'
    && ['string', 'number', 'bigint', 'symbol'].includes(TypeOf(y))
  ) {
    // 将x转换为原始类型
    return abstractEqualityComparison(ToPrimitive(x), y)
  }
  
  // 8. 比较bigint 和 数字
  if (
    (TypeOf(x) === 'bigint' && TypeOf(y) === 'number')
    || (TypeOf(x) === 'number' && TypeOf(y) === 'bigint')
  ) {
    if (
      [NaN, +Infinity, -Infinity].includes(x)
      || [NaN, +Infinity, -Infinity].includes(y)
    ) {
      // JS中 NaN 自身也不相等
      // 即NaN == NaN  返回false
      // 一般用用 Object.is(NaN, NaN) 返回true 进行判断
      return false
    }
    if (isSameMathematicalValue(x, y)) {
      return true
    } eles {
      return false
    }
  }
  
  return false
}
```

上面没有列举以下操作符：

- [strictEqualityComparison()](https://tc39.es/ecma262/#sec-strict-equality-comparison)
- [StringToBigInt()](https://tc39.es/ecma262/#sec-stringtobigint)
- [isSameMathematicalValue()](https://tc39.es/ecma262/#mathematical-value)

译者注：可以看出 `==` 判断逻辑超级的多，但是整个过程也很清晰



<p id="6"></p>



## 6️⃣ 术语表：关于类型强转的名词

我们已经深入看过了JS类型强转，下面简短的总结一下关于类型强转相关的术语名词：

- 在 *类型转换（`type conversion`）*，我们想要输出值为指定类型。如果输入值已经是该类型，则直接返回即可，否则，输入值将转换为想要的类型
- *显示类型转换（`Explicit type conversion`）* 表示程序员使用操作（函数，操作符等）来触发类型转换。显式转换可以是：
  - *已校验（`Checked`）*: 如果一个值不能被转换，则抛出异常（比如 `1 + Symbol()` 直接抛出错误）
  - *未校验（`Unchecked`）*: 如果一个值不能被转换，返回一个错误的值（比如 `1 / 0` 返回错误值 `Infinity`）
- *类型强转（`Type coercion`）* 是隐式类型转换：一个操作自动将其参数转换为所需要的类型。可以是已校验的，也可以是未校验的，也可以是2者之间的。



原文链接：

- [Type coercion in JavaScript](https://exploringjs.com/deep-js/ch_type-coercion.html#converting-to-typed-array-elements)





译者注：本章对以下知识点的理解提供了极其深入的理解：

1. `+` 操作的怪异行为
2. `==` 操作符的怪异行为
3. 对象转换为基础类型时，到底是调用 `Symbol[toPrimitive] ` 还是 `toString()` 还是 `valueOf()` 的细节问题
4. `Object.prototype.toString` 的内部逻辑是什么，`Symbol.toStringTag` 和 自定义 `toString()` 方法的作用
5. 如何以JS形式理解ECMAScript规范，并将其进行转换
6. `ToPrimitive()` 中的 `hint` 含义，哪些操作 `hint` 是 `number` ，哪些是 `string`；当为 `default` 时，`Date` & `Symbol` 和其它类型不同，它们默认偏向于转换为字符串
7. 规范算法
   1. `ToPrimitive(input: any, hint: 'number' | 'string' | 'default' = 'default')`
   2. `TypeOf(O: any)`
   3. `IsCallable(x)`
   4. `Date.prototype[Symbol.toPrimitive]` & `Symbol.prototype[Symbol.toPrimitive]`
   5. `ToString(argument: any)` & `String(value: any)`
   6. `Object.prototype.toString()`
   7. `ToPropertyKey(argument)`
   8. `ToNumeric(value)` & `ToNumber(argument)` 区别：前置会处理 `bigint` 类型
   9. `+` 操作符
   10. `==` 操作符

2022年07月12日23:45:07

