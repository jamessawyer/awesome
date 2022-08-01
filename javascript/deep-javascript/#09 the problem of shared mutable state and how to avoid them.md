目录：

1. [What is shared mutable state and why is it problematic](#1)
2. [Avoiding sharing by copying data](#2)
   1. [How does copying help with shared mutable state?](#2.1)
3. [Avoiding mutations by updating non-destructively](#3)
   1. [How does non-destructive updating help with shared mutable state?](#3.1)
4. [Preventing mutations by mkaing data immutable](#4)
   1. [How does immutability help with shared mutable state?](#4.1)
5. [Libraries for avoiding shared mutable state](#5)
   1. [Immutable.js](#5.1)
   2. [Immer](#5.2)



本章将回答如下3个问题：

1. 什么是共享可变状态？
2. 为什么共享可变状态会成为麻烦？
3. 如何避免这些麻烦？



<p id="1"></p>



## 1️⃣ 什么是共享可变状态及为什么这会成为问题?

共享可变状态效果如下：

- 如果2方或更多方（`parties`）可以改变相同的数据（变量，对象等）
- 并且它们的生命周期有重叠
- 则会存在某一方的修改会导致其它方不能正常工作

🚨注意这个定义适用于函数调用，协作式多任务（比如，JS中的异步函数）等。风险都类似。

下面代码是个例子。这个例子不太现实，仅做演示和便于理解使用：

```js {2-4,13,16}
function logElements(arr) {
  while (arr.length > 0) {
    console.log(arr.shift())
  }
}

function main() {
  const arr = ['banana', 'orange', 'apple']
  
  console.log('Before sorting')
  logElements(arr)
  
  arr.sort() // 改变了arr
  
  console.log('After sorting:')
  logElements(arr)  // A
}
main()

// 输出结果
// 'Before sorting:'
// 'banana'
// 'orange'
// 'apple'
// 'After sorting:'
```

在这种情况下，有两个独立的当事方:

- 函数 `main()` 想要在数组排序前后记录它
- 函数 `logElements()` 记录其参数 `arr` 中的元素，但是这样做的时候却移除了数组中的元素😅

`logElements()` 破坏了 `main()`，并导致 `A` 行打印了一个空数组。

在本章后续部分，我们将看3种避免共享可变状态引发的问题的3种方式：

1. 通过拷贝数据避免共享
2. 通过非破坏性更新避免突变（`mutations`）
3. 使数据不可变（`immutable`）以便阻止突变

特别的，我们将稍后回顾上面的例子，并修复它。



<p id="2"></p>



## 2️⃣ 通过拷贝数据避免共享

*拷贝数据是避免共享的一种方式。*

💡对拷贝数据，可以参考本书的下面2章：

- [Copying objects and Arrays](https://exploringjs.com/deep-js/ch_copying-objects-and-arrays.html)
- [Copying instances of classes: .clone() vs. copy constructors](https://exploringjs.com/deep-js/ch_copying-class-instances.html)



<p id="2.1"></p>



### 2.1 拷贝是如何帮助解决共享可变状态的？

**只要我们只从共享数据中 读（`read`）数据，我们什么问题也没有**。在 *修改（`modifying`）* 数据前，我们可通过拷贝（必要时需深拷贝）方式 *取消共享（`un-share`）* 它。

🚀🚀 *防御性拷贝（`Defensive copying`）* 是一种在可能出现问题时始终进行复制的技术。它的目的是保证当前实体（函数，类，等）的安全：

- 输入：拷贝潜在传给我们的共享数据，使得我们可以在不打扰外部数据的前提下，使用该数据
- 输出：在暴露给外界前， 我们拷贝内部数据，这意味着外部不能干扰到我们内部的活动

注意，这些措施保护我们不受其他方面的影响，但它们也保护其他方面不受我们的影响。

下面阐述这2种防御性拷贝。



#### 2.1.1 拷贝共享输入

还记得在本章开始的例子中，我们遇到了麻烦，因为`logElements()`修改了它的参数`arr`:

```js
function logElements(arr) {
  while (arr.length > 0) {
    console.log(arr.shift())
  }
}
```

😎让我们给这个函数添加防御性拷贝：

```js {2}
function logElements(arr) {
  arr = [...arr] // 防御性拷贝
  while (arr.length) {
    console.log(arr.shift())
  }
}
```

现在，我们在 `main()` 中再调用 `logElements()` 不会存在任何问题了：

```js {20-22}
function main() {
  const arr = ['banana', 'orange', 'apple']
  
  console.log('Before sorting')
  logElements(arr)
  
  arr.sort() // 改变了arr
  
  console.log('After sorting:')
  logElements(arr)  // A
}
main()

// 打印结果
// 'Before sorting:'
// 'banana'
// 'orange'
// 'apple'
// 'After sorting:'
// 'apple'
// 'banana'
// 'orange'
```



#### 2.1.2 对暴露的内部数据进行拷贝

我们先从类 `StringBuilder` 开始，它在 `A` 行没有拷贝暴露给外部的内部数据：

```js {6-10}
class StringBuilder {
  _data = []
  add(str) {
    this._data.push(str)
  }
  getParts() {
    // 🚨 这里向外部暴露了内部数据
    //    但是没有对其进行拷贝
    return this._data  // A
  }
  
  toString() {
    return this._data.join('')
  }
}
```

只要不使用 `getParts()` 方法，一切正常:

```js
const sb1 = new StringBuilderr()
sb1.add('Hello')
sb1.add(' world!')
assert.equal(sb1.toString(), 'Hello world!')
```

然而，如果改变了 `.getParts()` 的返回结果（`A` 行），会使得 `StringBuilder` 将变得不正常：

```js {4-5}
const sb2 = new StringBuilderr()
sb2.add('Hello')
sb2.add(' world!')
sb2.getParts().length = 0 // A
assert.equal(sb2.toString(), '') // ❌ 不正常
```

😎解决办法就是，在暴露内部数据 `_data` 之前（A行）对其进行防御性拷贝：

```js {7-8}
class StringBuilder {
  _data = []
  add(str) {
    this._data.push(str)
  }
  getParts() {
    // 🚨 防御性拷贝
    return [...this._data ] // A
  }
  
  toString() {
    return this._data.join('')
  }
}
```

现在再改变 `.getParts()` 的结果不再会影响到实例的操作：

```js {4-5}
const sb = new StringBuilderr()
sb.add('Hello')
sb.add(' world!')
sb.getParts().length = 0 // A 不再影响实例sb
assert.equal(sb.toString(), 'Hello world!') // ✅
```

<p id="3"></p>



## 3️⃣ 通过非破坏性更新避免突变

如果我们非破坏性的更新数据，我们可以避免突变。

💡关于更新数据，可参考上一章：

- [Updating data destructively and non-destructively](https://exploringjs.com/deep-js/ch_updating-destructively-and-nondestructively.html)



<p id="3.1"></p>



### 3.1 非破坏性更新如何对共享可变状态有帮助的？

通过非破坏性更新，共享可变状态不再是问题，因为我们永远不会改变该共享可变状态。（注意：只有每个访问数据的操作都这样做，才有效）。

有趣的是，拷贝数据变得十分简单：

```js
const original = {city: 'Berlin', country: 'Germany'}
const copy = original
```

这是可行的，因为我们只进行非破坏性的更改，因此根据需要复制数据。



<p id="4"></p>



## 4️⃣ 使数据不可变的方式阻止突变

**📚我们可以将共享数据变为不可变，以阻止其突变。**

JS中如何使数据不可变，可参考下面2章：

- [Protecting objects from being changed](https://exploringjs.com/deep-js/ch_protecting-objects.html)
- [Immutable wrappers for collections](https://exploringjs.com/deep-js/ch_immutable-collection-wrappers.html)



<p id="4.1"></p>



### 4.1 不可变性如何帮助共享可变状态的？

如果数据不可变，它可以无任何风险共享😎。特别是，没有必要再进行防御性的拷贝了。

💡 *非破坏性更新时对不可变数据的一个重要补充*：

如果我们将2者结合起来，不可变数据将和可变数据一样变得十分有用，并且还没有相关的风险。🤩



<p id="5"></p>



## 5️⃣ 避免共享可变状态的库

JS中有几个支持不可变数据+非破坏性更新的库。最流行的2个是：

1. [Immutable.js](https://github.com/immutable-js/immutable-js) 对lists, stacks, sets, maps等提供不可变数据结构
2. [Immer.js](https://immerjs.github.io/immer/) 也支持不可变数据+非破坏性更新，*但只针对普通的对象，数组，Sets，Maps*。即不需要新的数据结构

下面更详细的介绍一下这2个库。



<p id="5.1"></p>



### 5.1 Immutable.js

[Immutable.js](https://github.com/immutable-js/immutable-js) 仓库描述如下：

- JavaScript的不可变持久数据集合提高了效率和简单性。

Immutable.js提供了如下不可变数据结构：

- Lists
- Stack
- Set (不同于JS内置的Set)
- Map (不同于JS内置的Map)
- 等等

下面我们使用不可变的 `Map`:

```js {7,10,14,17-18}
import { Map } from 'immutable/dist/immutable.es.js'
const map0 = Map([
  [false, 'no'],
  [true, 'yes'],
])

// 我们创建一个修改版本的map0
const map1 = map0.set(true, 'maybe')

// 💡 修改版本不同于原版本
assert.ok(map1 !== map0)
assert.equal(map1.equals(map0), false) // A

// 我们撤销之前的改变
const map2 = map1.set(true, 'yes')

// 💡 map2是不同于map0的新对象
// 但是它们内容相同
assert.ok(map2 !== map0);
assert.equal(map2.equals(map0), true); // (B)
```

注意：

- 像`.set()`这样的“破坏性”操作不会修改接收方，而是返回修改后的副本
- 为了检查两个数据结构是否具有相同的内容，我们使用内置的`.equals()`方法(第A行和第B行)



<p id="5.2"></p>



### 5.2 Immer

[Immer](https://immerjs.github.io/immer/) 仓库描述如下：

- 通过改变当前状态创建下一个不可变状态

Immer帮助非破坏性地更新(可能嵌套)普通对象、数组、Sets和Maps。即不涉及到自定义数据结构。

🌰：

```js {7-10}
import { produce } from 'immer/dist/immer.module.js'

const people = [
  {name: 'Jane', work: {employer: 'Acme'}}
]

const modifiedPeople = produce(people, (draft) => {
  draft[0].work.employer = 'Cyberdyne'
  draft.push({name: 'John', work: {employer: 'Spectre'}})
})

assert.deepEqual(modifiedPeople, [
  {name: 'Jane', work: {employer: 'Cyberdyne'}},
  {name: 'John', work: {employer: 'Spectre'}},
])
assert.deepEqual(people, [
  {name: 'Jane', work: {employer: 'Acme'}},
])
```

存储在 `people.produce()` 中的原数据给我们提供了一个变量 `draft`。我们假装这个变量是 `people`，然后使用正常的破坏性改变操作。**Immer会拦截这些操作**。它不会改变 `draft`，而是对 `people` 进行非破坏性的改变。结果通过 `modifiedPeople` 进行引用。额外的好处是，它是深度不可变的🚀。

`assert.deepEqual()` 能正常运作，是因为Immer返回的是普通对象和数组🤩。



2022年07月29日21:39:12

