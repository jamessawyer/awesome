## 基础概念



lucifer的系列文章：

1. [上帝角度看typescript - @segments](https://segmentfault.com/a/1190000023489694)
   1. 从输入输出看：ts文件经过typescript编译器生成 `.js` 文件和 `.d.ts` 文件
   2. 从功能上看：对 **变量** 提供丰富的类型系统，而js是对 **值** 提供类型
   3. 编译器如何工作的：扫描 -> 分词 -> 解析 -> AST抽象树
2. [typeRoots & types & @types是什么？ - lucifer@segments](https://segmentfault.com/a/1190000023722037)
   1. `typeRoots` 用于指定默认的类型生命文件查找路径，默认是 `node_modules/@types`
   2. `types` 用于指定只引入部分类型文件
3. [tsconfig.json配置文件怎么写](https://segmentfault.com/a/1190000023750243)
   1. tsconfig 如何被解析，查看类型的顺序类似js对象原型链
   2. 分类：文件选项；严格检测；模块解析；项目配置

如果看完基础后：

1. [TypeScript 高级用法 - 字节前端@掘金](https://juejin.cn/post/6926794697553739784)



## 高级概念

1. [⭐️Ts高手篇：22个示例深入讲解Ts最晦涩难懂的高级类型工具 - 愣锤@掘金](https://juejin.cn/post/6994102811218673700?utm_source=gold_browser_extension#heading-8) 强烈推荐，高级类型的使用方式和原理，并自定义很多ts高级类型

   1. 对内置的泛型 `Partial | Readonly` 等等原理进行解析
   2. 自定义了一些高级类型：`SymmetricDifference | OptionalKeys | UnionToIntersection` 等等

2. [⭐️⭐️你不知道的 TypeScript 泛型（万字长文，建议收藏）- lucifier@segments](https://segmentfault.com/a/1190000022993503) 读完这些会对泛型的理解有质的飞越（作者微信公众号：`脑洞前端`）

   1. 将泛型类比成函数定义去理解

   2. 泛型默认参数

   3. 泛型递归

   4. 泛型嵌套，一个泛型传入另一个泛型，类比一个函数传入另一个函数作为参数

   5. tricks: 封装接口

      ```typescript
      interface Seal {
        name: string;
        url: string;
      }
      interface API {
        "/user": { name: string; age: number; phone: string; };
        "/seals": { seals: Seal[]}
      }
      
      const api = <URL extends keyof API>(url: URL) => Promise<API[URL]> => {
        return fetch(url).then(res => res.json())
      }
      
      // 下面方式会出现提示 '/user' | '/seals'
      api('')
      
      // res. 会出现提示 name | age | phone
      api('/user').then(res => res.)
      ```



看完上面的文章后可以尝试做一下下面的练习题进行巩固：

1. [typescript-exercises 练习地址](https://typescript-exercises.github.io/#exercise=1&file=%2Findex.ts)
   1. [练习题讲解 Part1 - lucifier](https://segmentfault.com/a/1190000025157672)
   2. [练习题讲解 Part2 - lucifier](https://segmentfault.com/a/1190000037521679?utm_source=sf-similar-article)

## 书籍和参考资料

1. [深入理解 TypeScript - 中文版](https://jkchao.github.io/typescript-book-chinese/)
2. [Typescript Deep Dive - 英文原版](https://basarat.gitbook.io/typescript/)



## 第三方工具



1. [type-fest - @github](https://github.com/sindresorhus/type-fest) 封装了很多工具类型
2. [piotrwitek/utility-type - @github](https://github.com/piotrwitek/utility-types) 在原生ts基础上补充了不少常用类型
3. [ts-toolbelt - @github](https://github.com/millsp/ts-toolbelt) 很全的类型库