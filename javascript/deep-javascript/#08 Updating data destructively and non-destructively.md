目录：

1. [Examples: updating an object destructively and non-destructively](#1)
2. [Examples: updating an Array destructively and non-destructively](#2)
3. [Manual deep updating](#3)
4. [Implementing generic deep updating](#4)



本章，我们将学习2种不同的更新数据方式：

1. **破坏性更新（`destructive update`）** 数据 - 直接改变数据，以便得到想要的形式
2. **非破坏性更新（`non-destructive update`）** 数据 - 创建数据的副本，以得到想要的形式

💡第2种方式类似于：先拷贝数据，得到一个副本，然后再破坏性的更改副本数据，但是这些操作是同时发生的😎



<p id="1"></p>



## 1️⃣ 例子：破坏性和非破坏性更新对象

下面函数破坏性的更新对象属性：

```js {8}
function setPropertyDestructively(obj, key, value) {
  obj[key] = value
  return obj
}

const obj = {city: 'Berlin', country: 'Germany'}
setPropertyDestructively(obj, 'city', 'Munich')
// 💡 直接改变了原对象 这称之为破坏性改变
assert.deepEqual(obj, {city: 'Munich', country: 'Germany'})
```

而下面代码演示非破坏性更新对象：

```js {12,15}
function setPropertyNonDestructively(obj, key, value) {
  const updatedObj = {}
  for (const [k, v] of Object.entries(obj)) {
    updateObj[k] = (k === key ? value : v)
  }
  return updateObj
}

const obj = {city: 'Berlin', country: 'Germany'}
const updatedObj = setPropertyNonDestructively(obj, 'city', 'Munich')

// 我们创建了一个新的更新对象
assert.deepEqual(updatedObj, {city: 'Munich', country: 'Germany'})

// 💡 原对象并没有没改变 这称之为非破坏性更新
assert.deepEqual(obj, {city: 'Berlin', country: 'Germany'})
```

😎使用 `Spreading` 操作符可以更简洁：

```js
function setPropertyNonDestructively(obj, key, value) {
  return { ...obj, [key]: value }
}
```

上面2个版本的 `setPropertyNonDestructively` 更新都是浅更新（`shallowly`）：即它们只能改变对象最上层的部分。



<p id="2"></p>



## 2️⃣ 例子：破坏性和非破坏性更新数组

下面函数展示破坏性的更新数组元素：

```js
function setElementDestructively(arr, index, value) {
  arr[index] = value
}

const arr = ['a', 'b', 'c', 'd', 'e']
setElementDestructively(arr, 2, 'x')
//💡 直接改变了原数组arr
assert.deepEqual(arr, ['a', 'b', 'x', 'd', 'e'])
```

而下面演示非破坏性的更新数组：

```js {13,16}
function setElementNonDestructively(arr, index, value) {
  const updatedArr = []
  // 注意 arr.entries() 写法
  for (const [i, v] of arr.entries()) {
    updatedArr.push(i === index ? value : v)
  }
  return updatedArr
}

const arr = ['a', 'b', 'c', 'd', 'e']
const updatedArr = setElementNonDestructively(arr, 2, 'x')

// 我们创建了新的数组，在新数组上进行改变
assert.deepEqual(updatedArr, ['a', 'b', 'x', 'd', 'e'])

// 💡 原数组并没有没改变 这称之为非破坏性更新
assert.deepEqual(arr, ['a', 'b', 'c', 'd', 'e'])
```

😎 使用 `.slice()` + `Spreading` 可以使函数更简洁：

```js
function setElementNonDestructively(arr, index, value) {
  return [
    ...arr.slice(0, index),
    value,
    ...arr.slice(index + 1)
  ]
}
```

上面2个版本的 `setElementNonDestructively` 更新都是浅更新（`shallowly`）：即它们只能改变数组最上层的部分。



<p id="3"></p>



## 3️⃣ 手动深度更新

目前，我们只是对数据进行浅更新。现在处理深度更新（`deep updating`）。下面代码展示了如何手动处理深度更新。我们改变name和employer：

```js {4,7}
const original = {name: 'Jane', work: {employer: 'Acme'}}

const updatedOriginal = {
  ...original,
  name: 'John',
  work: {
    ...original.work,
    employer: 'Spectre'
  }
}

assert.deepEqual(
  original,
  {name: 'Jane', work: {employer: 'Acme'}}
)
assert.updatedOriginal(
  original,
  {name: 'John', work: {employer: 'Spectre'}}
)
```



<p id="4"></p>



## 4️⃣ 🔥实现通用深度更新

下面函数实现通用深度更新：

```js
function deepUpdate(original, keys, value) {
  if (keys.length === 0) {
    return value
  }
  const currentKey = keys[0]
  
  if (Array.isArray(original)) {
    return original.map(
      (v, index) => index === currentKey
      	? deepUpdate(v, keys.slice(1), value) // A 递归
      	: v // B
    )
  } else if (typeof original === 'object' && original !== null) {
    return Object.fromEntries(
       Object.entries(original).map(
        (keyValuePair) => {
          const [k, v] = keyValuePair
          if (k === currentKey) {
            return [k, deepUpdate(v, keys.slice(1), value)] // C
          } else {
            return keyValuePair  // D
          }
        }
      )
    )
  } else {
    // 原始类型值
    return original
  }
}
```

如果我们将 `value` 视为正在更新的树的根，那么 `deepUpdate()` 只会深度更改单个分支(`A`行和`C`行)。所有其它分支都被浅拷贝（`B`行 和 `D` 行）。

使用：

```js
const original = {name: 'Jane', work: {employer: 'Acme'}}
const copy = deepUpdate(original, ['work', 'employer'], 'Spectre')
assert.deepEqual(
  original,
  {name: 'Jane', work: {employer: 'Acme'}}
)
assert.updatedOriginal(
  copy,
  {name: 'John', work: {employer: 'Spectre'}}
)
```

译者注：

- 这个深度更新，和上一章的 **深度拷贝** 几乎类似，其原理都是，先拷贝一个副本，然后再在副本上进行操作



2022年07月29日00:24:51

