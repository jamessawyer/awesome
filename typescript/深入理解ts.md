## 1. `keyof any` 为什么是 `string | number | symbol`

我们有时看到 `keyof any` 返回类型是 `string | number | symbol`, 这是因为 `keyof any` 的依据是 **object 所支持的key的类型**， 目前object支持的key的类型就3种：`string | number | symbol`, 比如：

```js
const symbol = Symbol('whatever')
// 对象支持的3种类型的key
const obj = {
  1: 'one',
  'name': 'kobe',
  [symbol]: 'whatever'
}
```

因此：

```typescript
type MyType = keyof any // string | number | symbol
```

参考：

- [Why does 'keyof any' have type of 'string | number | symbol' in typescript? - @stackoverflow](https://stackoverflow.com/a/55535722/7185283)



## 2. typescript 中 `is` 关键词的作用

常常在函数返回类型中看到 `is` 关键词，比如 `arg is number` 等等，它的作用是什么呢？

根据官网 [using type predicates](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates) 解释，表示 `is` 用于类型预测，这样听起来很抽象，**它的主要作用是，类型保障，对类型进行缩窄**，示例：

```typescript
function isNumber(arg: any): boolean {
  return typeof arg === 'number'
}

function example(foo: any) {
  // 这里 isNumber 返回 true，但是 foo 仍然是 any 类型
  if (isNumber(foo)) {
    console.log(foo.toFixed(2)) // ✅
    
    // 因为 foo 是any类型 编译时ok
    // 这里会出现 运行时错误
    console.log(foo.length) // ❌
  }
}
```



如果使用 `is` 进行类型保障：

```typescript
// 这里使用 is
function isNumber(arg: any): arg is number {
  return typeof arg === 'number'
}

function example(foo: any) {
  if (isNumber(foo)) {
     // 这里已经能确保 foo 是 number 类型

    console.log(foo.toFixed(2)) // ✅
    
    // 因为 foo 是 number 类型 ，不存在 length 属性
    // 这里会出现 编译时错误， 直接将错误扼杀在摇篮中 🎉
    console.log(foo.length) // ❌
  }
}
```

参考：

- [What does the 'is' keyword do in typescript? - @stackoverflow](https://stackoverflow.com/a/45748366/7185283)



## 3. `infer` 关键词的作用

关于 `infer` 我们知道：

- 只能用于 **条件类型** 的 `extends` 语句中



## 4. 裸类型


## 5. 类型分发
