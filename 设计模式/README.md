设计模式是一类通用的解决方案，与语言无关。在看设计模式之前，必须先了解一下UML各种箭头的含义：

- [UML类图详解 - @掘金](https://juejin.cn/post/6844903893327937550)

## 1. 📚 推荐书本

1. 《深入设计模式》: 绘图 + 各种语言示例
   - [Refactoring Guru - online](https://refactoringguru.cn/design-patterns)
2. [设计之路 - 水球潘@youtube](https://www.youtube.com/watch?v=IkG_KuMpQRM&list=PLicQRHHL75d7EXEI9nWfUYJyrPdI79M70)
   - 一位湾湾开发讲的设计模式，简洁逻辑通畅，强烈推荐
   - 但是当前只讲了 "策略模式"，后续模式up准备之后再更新，值得关注



📋 优质博文：

1. [interface vs Implementation - Dmitri Pavlutin](https://dmitripavlutin.com/interface-vs-implementation/) 通过很简单的示例介绍了 **面向接口编程** 的好处 😋
   - 是面向具体实现编程还是面向接口编程呢？如果确认实现不会发生变化，可以面向具体实现编程，不必引入不必要的复杂性；如果确认实现可能会改变，则面向接口编程更加的灵活
   - 面向接口编程符合 **开闭原则** 和 **单一职责原则**
   - 使用组合的模式引入抽象类，具体实现类可以之后传入，这便是面向接口编程的灵活之处



