## 1. Getters & Setters

> 1. paused()

获取当前动画是否处于暂停状态或者设置是否暂停，可以使用 [paused()](https://greensock.com/docs/v3/GSAP/Tween/paused()) 方法，其签名：

```typescript
.paused(value?: Boolean): [Boolean | self]
```

示例：

```js
// 动画是否暂停
console.log(tween.paused())

// 设置为暂停
// 等价于 tween.pause() 方法
tween.paused(true)

// 取消暂停
tween.paused(false)
```



> 2. time()

获取当前时间或者设置动画到某个时间点，可以使用 [time()](https://greensock.com/docs/v3/GSAP/Tween/time()) 方法，其签名：

```typescript
.time(
  value?: Number,
  suppressEvents?: Boolean
): [Number | self]
```

示例：

```js
// 获取当前动画时间位置
consol.log(tween.time())

// 设置动画的位置
// 等价于 tween.seek(2)
tween.time(2)
```



> 3. progress()

获取当前动画进度，或者设置动画进度，可以使用 [progress()](https://greensock.com/docs/v3/GSAP/Tween/progress()) 方法，其签名为：

```typescript
.progress(
  value?: Number,
  suppressEvents?: Boolean
): [Number | self]
```

如果动画完成，则进度为 `1`，示例：

```js
// 假设动画运动中，点击按钮时打印当前动画进度
console.log(tween.progress()) // 0.141

// 设置进度
tween.progress(0.5) // 动画直接到中间位置
```

还有其他的一些方法，比如：

- `tween.duration(5)`: 将时间设置为5s
- `tween.timeScale(2)`: 2倍速播放动画
- `tween.reversed(true)`: 反向播放动画

更多可以参考 [GSAP Tween methods](https://greensock.com/docs/v3/GSAP/Tween)



另外 `gsap.to()` 第一个参数还可以是一个 `tween`，后面再添加上面的属性，eg：

```js
button.addEventListener('click', function() {
  tween.pause() // 暂停动画
  // 1s运动到 0.5位置
  gsap.to(tween, { progress: 0.5, duration: 1 })
})
```

✏️codepen:
 - [Advanced GSAP methods - codepen](https://codepen.io/JamesSawyer/pen/mdBJVYY)



## 2. 回调函数和作用域

使用 `onXXXParams` 的形式对回调函数进行传参，参数以数组的形式，eg:

```js
gsap.to('.box', { 
  x: 400, 
  duration: 2,
  onComplete: callback,
  onCompleteParams: ['hello', 2]
})

function callback(message, num) {
  console.log(message, num)
}
```

可参考： [Tween Special Properties](https://greensock.com/docs/v3/GSAP/Tween)

现有的回调函数有：

1. `onComplete & onCompleteParams`
2. `onRepeat & onRepeatParams`
3. `onReverseComplete & onReverseCompleteParams`
4. `onStart & onStartParams`
5. `onUpdate & onUpdateParams`



> 作用域

回调函数如果不使用箭头函数，则 `this` 默认指向 `Tween` 本身。

```js
gsap.to('.box', { 
  x: 400, 
  duration: 2,
  onComplete: callback
})

function callback(message, num) {
  console.log(this)
}
// 打印
Tween 
{
  data: undefined,
  vars: {},
  ...
}
```

因此可以使用 `Tween` 所有属性和方法，比如获取动画的目标对象:

```js
// ...
function callback(message, num) {
  console.log(this.targets()[0])
}
// 打印
<div class="box"></div>
```

如果回调函数使用 **箭头函数**， 则 `this` 指向 `undefined`。



> 改变作用域

假设在某个对象中定义了Tween动画，`this` 仍旧默认指向 `Tween` 动画自身：

```js
class Fred {
  constructor() {
    this.animation = gsap.to(
      '.box', 
      { x: 100, duration: 2, onComplete: onComplete }
    )
    this.message = 'hello'
    
    function onComplete() {
      console.log(this)
    }
  }
}

const fred = new Fred()
// 打印 this 指向
// 默认指向 Tween 动画自身
Tween 
{
  data: undefined,
  vars: {},
  ...
}
```

如果想要改变this的指向，则可以使用 `callbackScope` 配置项：

```js
class Fred {
  constructor() {
    this.animation = gsap.to(
      '.box', 
      {
        x: 100,
        duration: 2,
        onComplete: onComplete,
        callbackScope: this
      }
    )
    this.message = 'hello'
    
    function onComplete() {
      console.log(this)
    }
  }
}

const fred = new Fred()
// 打印 this 指向 实例对象自身，这样即可访问实例身上的其他属性了
Fred
{
  animation: Tween,
  message: 'hello'
}
```

或者使用 `bind()` 方法改变指向：

```js
class Fred {
  constructor() {
    this.animation = gsap.to(
      '.box',
      {
        x: 100,
        duration: 2,
        onComplete: onComplete.bind(this) 
      }
    )
    this.message = 'hello'
    
    function onComplete() {
      console.log(this)
    }
  }
}

const fred = new Fred()
// 打印 this 指向 实例对象自身，这样即可访问实例身上的其他属性了
Fred
{
  animation: Tween,
  message: 'hello'
}
```



## 3. Timeline.getChildren

用于获取子Tweens 和 timelines，或者其嵌套的Tweens & timelines。

```typescript
/**
* @Params nested: Boolean, default: true, 是否返回嵌套的tweens & timelines
* @Params tweens: Boolean, default: true, 返回的结果是否包含tweens
* @Params timelines: Boolean, default: true, 返回的结果是否包含timelines
* @Params ignoreBeforeTime: Number, default: -Infinity, 返回的结果是否包含timelines
* @Return Array: 包含 tweens & timelines 的数组
*/
.getChildren(
  nested: Boolean,
  tweens: Boolean,
  timelines: Boolean,
  ignoreBeforeTime:Number
): Array
```

通过获取到的 `tweens & timelines` 可以去添加自定义功能，比如自定义时间轴：

```js
const pixelsPerSec = 200
let animation = gsap.timeline()

animation
  .to('#star', { duration: 2, x: 1150 })         // Tween 1
  .to('#circle', { duration: 1, x: 1150 })       // Tween 2
  .to('#square', { duration: 3, x: 1150 }, '<')  // Tween 3

// 监听更新 更新进度条位置
animation.eventCallback('onUpdate', movePlayhead).paused(true)

// 避免FOUC问题
gsap.to('svg', {autoAlpha: 1})
const time = document.getElementById('time')
// 获取时间轴的总长度 = tl时间 * 单位时间长度
const maxX = animation.duration() * pixelsPerSec

// 获取上面定义的3个 Tween的集合
const children = animation.getChildren()

// 设置起始位置，和不同时间对应的长度
children.forEach((tween, i) => {
  gsap.set(`#tween${i}`, { x: tween.startTime() * pixelsPerSec })
  gsap.set(`#rect${i}`, { width: tween.duration() * pixelsPerSec })
})

const dragger = Draggable.create('#playhead', {
  type: 'x',
  cursor: 'pointer',
  trigger: '#timeline',
  bounds: { minX: 0, maxX: maxX},
  onDrag: function() {
    // 拖动时 暂停动画
    animation.pause()
    // 更新时间
    time.textContent = animation.time().toFixed(1)
    // 更新动画进度
    animation.progress(this.x / maxX)
  }
})

function movePlayhead() {
  gsap.set('#playhead', { x: animation.progress() * maxX })
  time.textContent = animation.time().toFixed(1)
}

// ...
```

✏️codepen：

- [GSAP Timeline getChildren method - codepen](https://codepen.io/JamesSawyer/pen/ZEXGVEE)



## 4. gsap.killTweensOf()

[killTweensOf](https://greensock.com/docs/v3/GSAP/gsap.killTweensOf()) 作用：

1. 移除某个目标的Tween动画
2. 或者移除某个Tween动画某些动画属性

eg:

```js
gsap('.red, .blue', 
  { scale: 2, duration: 2, rotation: 360, yoyo: true }
)

// 移除 .blue 的动画
gsap.killTweensOf('.blue')

// 移除 .blue 对象的 rotation和yoyo属性
gsap.killTweensOf('.blue', 'rotation,yoyo')

8: {
    // 5
    width: '218rpx',
    height: '146rpx',
    top: '186rpx',
    left: '394rpx',
    zIndex: 2
  },
```

✏️codepen：

- [07-gsap.killTweensOf basic - codepen](https://codepen.io/JamesSawyer/pen/qBoaEKp)
- [08-gsap.killTweensOf 移除具体的属性 - codepen](https://codepen.io/JamesSawyer/pen/RwMGNBq)



## 5. 基于函数的属性

`gsap.to | gsap.from | gsap.fromTo` 添加属性时，可以直接添加：

```js
<div>
  <div class="box box1"></div>
  <div class="box box2"></div>
  <div class="box box3"></div>
</div>

// .box 会同时对上面3个元素生效
gsap.to('.box', {
  x: 400
})
```

也可以使用函数的形式，这样可以获取多个目标对象中的任意一个：

```js
gsap.to('.box', {
  // targets是所有目标元素的集合
  y: function(index, target, targets) {
    console.log(index)
    console.log(target)
  }
})
// 打印
0 <div class="box box1"></div>
1 <div class="box box2"></div>
2 <div class="box box3"></div>
```

具体可查看：[gsap Function-based values - docs](https://greensock.com/docs/v3/GSAP/gsap.from())





## 6. immediateRender 属性

是否立即渲染，这个对 `gsap.to` 默认是 `false`, 对 `gsap.from & gsap.fromTo` 则默认是 `true`, 大多是情况下，这样没有任何问题。

示例： [immediateRender 默认设置 - codepen](https://codepen.io/JamesSawyer/pen/xxWEbyP)

但是如果对于 **同一个对象，在时间轴中设置相同的属性，可能会导致问题**：

```js
let tl = gsap.timeline()
tl.from('.box1', { opacity: 0 })  // opacity: 0; -> opacity: 1;  (因为immediateRender, 实际opacity立马设置为0)
 .from('.box2', { opacity: 0, y: 50 })
 .from('.box1', { opacity: 0 }) // 再次对 box1 设置相同的属性 (因为immediateRender, 实际opacity立马设置为0)
```

因为默认是 `immediateRender: true`, 导致再执行第3段Tween时，`box1` 的动画效果是 `opacity: 0 -> opacity: 0`, 导致box1被隐藏。

具体效果： [immediateRender 有问题的情况 - codepen](https://codepen.io/snorkltv/pen/vYgJjgd)

为了解决这个问题，将第3段Tween设置 `immediateRender: false` 即可：

```js
let tl = gsap.timeline()
tl.from('.box1', { opacity: 0 })  // opacity: 0; -> opacity: 1;  (因为immediateRender, 实际opacity立马设置为0)
 .from('.box2', { opacity: 0, y: 50 })
 .from('.box1', { opacity: 0, immediateRender: false })  // fix： 这样 opacity: 0 -> opacity: 1
```

✏️codepen:
 - [10-immediateRender demystified (fixed)](https://codepen.io/JamesSawyer/pen/LYdREXm)
