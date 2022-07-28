目录：

1. [.clone() methods](#1)
2. [Static factory methods](#2)
3. [Acknowledgements](#3)

本章，我们将看2个用于实现对类实例的拷贝的技术

- `.clone()` 方法
- 所谓的 **拷贝构造器（`copy constructors`）**,构造器接收另一个当前类的实例，然后用它初始化当前实例



<p id="1"></p>



# 1️⃣ .clone()方法

这个技术在需要被拷贝的类中引入一个 `.clone()` 方法。它返回一个 `this`（即类实例） 的深拷贝。下面示例，展示了3个可被拷贝的类。

```js {7-9,17-19,28-30}
class Point {
  constructor(x, y) {
    this.x = x
    this.y = y
  }
  
  clone() {
    return new Point(this.x, this.y)
  }
}

class Color {
  constructor(name) {
    this.name = name
  }
  
  clone() {
    return new Color(this.name)
  }
}

class ColorPoint extends Point {
  constructor(x, y, color) {
    super(x, y)
    this.color = color
  }
  
  clone() {
    return new ColorPoint(this.x, this.y, this.color.clone()) // A 🚨
  }
}
```

行 `A` 展示了这个技术的一个重点：复合实例属性值必须**递归地克隆**。



<p id="2"></p>



# 2️⃣ 静态工厂方法

👩🏻‍🏫 **拷贝构造器** 是使用当前类的另一个实例来设置当前实例的构造器。

拷贝构造器在一些静态语言，比如C++，Java等语言中比较流行，你可以通过 **静态重载（`static overloading`）** 的方式提供多个版本的构造器。这里，**静态** 意味着在编译时选择使用哪个版本。

在JS中，我们必须在**运行时**做出决定，这也导致一些不优雅的代码😅

```js {3-5}
class Point {
  constructor(...args) {
    if (args[0] instanceof Point) {
      // 拷贝构造器（Copy Constructor）
      const [other] = args
      this.x = other.x
      this.y = other.y
    } else {
      const [x, y] = args
      this.x = x
      this.y = y
    }
  }
}
```

使用：

```js {3}
const original = new Point(-1, 4)

// 💡传入另一个Point实例到构造器中创建当前实例 实现对original实例的拷贝
const copy = new Point(original)

assert.deepEqual(copy, original)
```

**静态工厂方法（`static factory methods`）** 是构造器的另一种方式，并且效果更好，因为我们可以直接调用想要的功能🤩。（这里，**静态** 表示工厂方法是类方法）

下面示例，3个类 `Point`, `Color` & `ColorPoint` 每个都有一个静态工厂方法 `.from()`:

```js {7-9,17-19,28-34}
class Point {
  constructor(x, y) {
    this.x = x
    this.y = y
  }
  
  static from(other) {
    return new Point(other.x, other.y)
  }
}

class Color {
  constructor(name) {
    this.name = name
  }
  
  static from(other) {
    return new Color(other.name)
  }
}

class ColorPoint exentds Point {
  constructor(x, y, color) {
    super(x, y)
    this.color = color
  }
  
  static from(other) {
    return new ColorPoint(
      other.x,
      other.y,
      Color.from(other.color) // A 🚨
    )
  }
}
```

行 `A` 我们再一次递归的拷贝了💡。

示例：

```js {2-3}
const original = new ColorPoint(-1, 4, new Color('red'))
// 静态工厂方法创建副本
const copy = ColorPoint.from(original)

assert.deepEqual(copy, original)
```



<p id="3"></p>



# 3️⃣ 鸣谢（Acknowledgements）

[Ron Korvig](https://github.com/ronkorving) 提醒我在JavaScript中使用静态工厂方法，而不是重载构造函数进行深度复制。





总结（译者注）：

- 本章提供了2种类实例拷贝的2种方式
- `clone()` 方法中调用构造器创建新的实例，返回当前类 `this` 的深拷贝
- 静态工厂方法拷贝，一般命名 `static from(other) {}`，类似于 `Array.from()` （推荐😎） 

2022年07月27日23:50:51

