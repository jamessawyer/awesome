gsap常用的一些工具函数介绍



## 1️⃣ gsap.utils.wrap & gsap.utils.wrapYoyo

这个工具函数主要用于产生 *交替效果*，有以下几种签名：下面以 `wrap` 为例子，`wrapYoyo` 区别在于它可以使用yoyo的效果

```js
// 返回值wrappper是一个函数，value1, value2, value3 依次作用于目标对象
const wrapper = gasp.utils.wrap([value1, value2, value3, ...])
```

假设有10个box，依次改变其 `background` 值

```js
// 默认10个小球的背景色是 #fff
gsap.to('.box', {
  x: 500,
  duration: 2,
  background: gsap.utils.wrap(['red', 'yellow', 'green'])
})
// 使用上面的效果后，我们将发现 背景色依次交替作用在小球身上
box0: red
box1: yellow
box2: green
box3: red
box4: yellow
box5: green
box6: red
box7: yellow
box8: green
box9: red

// 如果将 wrap 换成 wrapYoyo
// 则效果为
box0: red
box1: yellow
box2: green
box3: yellow
box4: red
box5: yellow
box6: green
box7: yellow
box8: red
box9: yellow
```
✏️codepen：
 - [16 - GSAP utils wrap & wrapYoyo](https://codepen.io/JamesSawyer/pen/XWEjqBp)

上面的 `wrapper` 是一个函数，它可以接收一个 `index` 值，用于获取对应索引位置的值，以上面的背景色为例：

```js
const wrapper = gsap.utils.wrap(['red', 'yellow', 'green'])
var bgColor = wrapper(0) // 'red'
bgColor = wrapper(8)  // 'green'
bgColor = wrapper(9)  // 'red'
```

**上面的 `wrap` 可以直接添加一个索引值，获取相应的值**：

```js
// someValue 是依据index计算出的 value1, value2 ... 中的某一个
const someValue = wrap([value1, value2, value3, ...], [index: Number])
```

还有一种形式就是前2个参数传入一个 `Range` 区间，第3个参数如果超过最大值，则其取值等于 `rangeStart + (lastValue - rangeEnd)`; 如果第3个参数小于 `rangeStart`, 则等于 `rangeEnd - (rangeStart - lastValue)`: 

```js
// 12大于10， 则返回值等于 = 5 + （12 - 10） // 7
gsap.utils.wrap(5, 10, 12)

// 3小于rangeStart 5, 返回值 = 10 - (5 - 3) // 8
gsap.utils.wrap(5, 10, 3)
```

**利用 `wrap` 交替赋值的特点，可以制作出如下stragger动画效果**:

- [17 - GSAP utils wrap + SplitText effect - codepen](https://codepen.io/JamesSawyer/pen/dympeQe)

2022年07月13日16:13:17
