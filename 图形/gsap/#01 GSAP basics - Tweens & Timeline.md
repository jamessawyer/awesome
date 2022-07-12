`gsap` 对象功能：

1. 创建动画
2. 配置设置
3. 注册插件，eases & effects
4. 全局控制所有动画

下面主要讲解创建动画 `Tweens` & `Timelines` 的几种方法：

1. `gsap.to()`
2. `gsap.from()`
3. `gsap.fromTo()`
4. `gsap.timeline()`



## 1 Tweens



`Tween` 功能有多个作用： 可开始，暂停，返回，加速动画

1. **沿着时间轴改变一个单一对象的一个单一属性**

   ```js
   // 这里的 'x' 相当于 'translateX'
   gsap.to('.star', { x: 400, duration: 3 })
   ```

2. 沿着时间轴改变单一对象的多个属性

   ```js
   gsap.to('.star', { x: 400, background: 'purple', duration: 3 })
   ```

3. 沿着时间轴改变多个对象的多个属性

   ```js
   // 改变所有 class="star" 的⭐️的多个属性
   gsap.to('.star', { x: 400, rotation: 360, fill: 'yellow', duration: 3 })
   ```

4. 多个对象stagger动画效果

   ```js
   // 多个 圆 stagger效果
   // stagger: 1 表示间隔1s
   <div class="container">
     <div class="box"></div>
     <div class="box"></div>
     <div class="box"></div>
   </div>
   
   .container {
     display: grid;
     gap: 20px 0px;
   }
   .box {
     width: 50px;
     height: 50px;
     border-radius: 50%;
     background: green;
   }
   
   // 向右边700px进行动画
   gsap.to('.box', { stagger: 1, x: 700, background: 'yellow', duration: 5 })
   ```

   

`Timeline`: 是多个 `Tweens` 的组合的一个容器，沿着时间轴进行组合或叠加



### 1.1 Basic Tween gsap.to

最简单的Tween就是使用 `gsap.to()`, 其本质是改变目标对象的内联样式。其语法为：

```js
gsap.to(target, options)
```

默认 `duration` 是 `0.5s`。可以通过全局的方法 `gsap.default({ duration: 1 })` 改变默认配置。

为了获取更好的性能加速，使用GPU加速，优先设置css `Transforms & Opacity` 属性：

- x & y (即 `translateX & translateY`)
- rotation, rotationX, rotationY
- scale, scaleX, scaleY
- skewX, skewY

除了transform和opacity外，gsap可对任何数值类型添加动画：

- width & height
- top & left (前提条件是将 `position` 设置为 `relative | fixed | absolute`)
- borderRadius
- color & backgroundColor
- vh & vw 等等属性

相关文档：

1. [gsap.to docs](https://greensock.com/docs/v3/GSAP/gsap.to()?ref=6234)
2. [gsap.defaults docs](https://greensock.com/docs/v3/GSAP/gsap.defaults())



### 1.2 gsap.from & gsap.fromTo

`gsap.to()` 用于定义目标的要动画的最终状态，其初始状态是目标的初始状态，因此只需要定义最终状态的options即可；而 `gsap.from()` 则表示动画的初始状态，其最终状态是目标的自然状态，因此只需定义最初状态的options即可，比如

```js
// 多个 圆 stagger效果
// stagger: 1 表示间隔1s
<div class="container">
  <div class="box"></div>
  <div class="box"></div>
  <div class="box"></div>
</div>

.container {
  display: grid;
}
.box {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background: green;
}

// 则小球从 700px 的位置向左边进行stagger动画
gsap.from('.box', { stagger: 1, x: 700, background: 'yellow', duration: 5 })
```

而 `gsap.fromTo()` 则可同时定义初始状态和最终状态，对动画进行全面的控制，这对连接其他动画很有用：

```js
gsap.fromTo('.box', { opacity: 0 }, { opacity: 1, scale: 1.2 })
```

相关文档：

- [gsap.fromTo docs](https://greensock.com/docs/v3/GSAP/gsap.fromTo())



### 1.3 Delay & Repeat &  yoyo 属性

这3个属性用于定义动画的行为：

1. `delay`: 延迟多少秒开始动画
2. `repeat`: 重复多少次动画，`repeat: -1`, 则表示无限次的重复
3. `yoyo`: 是否像yoyo球一样来回的进行动画，如果 `yoyo: false`（默认值），则 `repeat`每次都从初始状态开始重复动画
4. `repeatDelay`: 重复动画之前延迟时间

eg:

```js
// 重复2次 yoyo
gsap.to('.box', { x: 500, duration: 3, repeat: 2, yoyo: true, repeatDelay: 1 })

// 无线重复
gsap.to('.box', { x: 500, duration: 3, repeat: -1, yoyo: true, repeatDelay: 1 })
```



### 1.4 Eases 缓动函数

用于定义动画运动曲线效果，可使用 [gsap Eases](https://greensock.com/docs/v3/Eases?ref=6234) 进行查看或自定义想要的效果

```js
gsap.to(graph, { duration: 2.5, ease: "back.out(1.7)", y: -500 })
```



### 1.5 Stagger

`stagger` 除了可以直接添加一个时间外，还可以添加一个对象配置

```js
<div class="container">
  <div class="box box1"></div>
  <div class="box box2"></div>
  <div class="box box3"></div>
</div>

.container {
  display: grid;
  gap: 20px 0px;
}
.box {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background: green;
}

// 每个球之间间隔 0.2s
gsap.to('.box', { x: 500, stagger: 0.2 })
// 等价于
gsap.to('.box', { x: 500, stagger: { each: 0.2 } })

// amount 属性， 表示间隔时间的总量
// amount: 3 表示，3个球之间，每个间隔 3 / 2 = 1.5s时间
gsap.to('.box', { x: 500, stagger: { amount: 3 } })

// from 属性，表示动画从哪里开始
// from: end 表示从最后面开始， 即这里是从 box3 开始动画
gsap.to('.box', { x: 500, stagger: { each: 0.2, from: 'end' } })

// from: center 表示从中间开始， 即这里是从 box2 开始动画，如果是偶数，则中间2个同时开始动画
gsap.to('.box', { x: 500, stagger: { each: 0.2, from: 'center' } })

// from: center 表示从边缘开始， 即这里是从 box1 & box3 开始动画，然后box2
gsap.to('.box', { x: 500, stagger: { each: 0.2, from: 'edges' } })
```
✏️codepen:
 - [01-gsap-stagger](https://codepen.io/JamesSawyer/pen/jOzMOrK)



stagger的几个属性：

1. `each`: 每个对象之间的间隔时间
2. `amount`: 对象之间间隔的时间总量 `time = (amount) / (n - 1)`，n表示对象的总数
3. `from`: `end | center | edges` 动画的起始位置



### 1.6 Tween的方法

Tween有很多方法，下面列举几个常用的：`tween` 实例可以通过 `gsap.to() | gsap.from() | gsap.fromTo()` 等方法返回

1. `play(from: Number, suppressEvents: Boolean): self`: 开始动画 [tween.play](https://greensock.com/docs/v3/GSAP/Tween/play())
2. `pause(from: Number, suppressEvents: Boolean): self:` 暂停动画 [tween.pause](https://greensock.com/docs/v3/GSAP/Tween/pause())
3. `reverse(from: *, suppressEvents: Boolean): self`: 反向动画 [tween.reverse](https://greensock.com/docs/v3/GSAP/Tween/reverse())
4. `restart(includeDelay: Boolean, suppressEvents: Boolean): self`: 重新开始动画 [tween.restart](https://greensock.com/docs/v3/GSAP/Tween/restart())

```js
<div class="box"></div>
<div class="buttons">
  <button id="play">play</button>
  <button id="pause">pause</button>
  <button id="reverse">reverse</button>
  <button id="restart">restart</button>
</div>

body {
  margin: 0;
  background: pink;
}

.box {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background: green;
}
.buttons {
  display: flex;
  flex-direction: row;
  margin-top: 20px;
}
button {
  outline: 0;
  border: 0;
  width: 100px;
  height: 44px;
  line-height: 1;
  background: purple;
  color: #fff;
  border-radius: 4px;
  cursor: pointer;
  color: #fff;
}
button + button {
  margin-left: 10px;
}

// 这里设置 paused
const tween = gsap.to('.box', {
  x: 700,
  background: 'green',
  paused: true, // 初始为暂停状态
  duration: 5
})
const $ = (selector) => document.querySelector(selector)
$('#play').onclick = () => tween.play()
$('#pause').onclick = () => tween.pause()
$('#reverse').onclick = () => tween.reverse()
$('#restart').onclick = () => tween.restart()
```
✏️codepen:
 - [02-gsap-methods](https://codepen.io/JamesSawyer/pen/mdxrdWW)


## 2 Timelines

时间轴默认将一组Tweens进行连接，一个动画接着一个动画依次执行

```js
<div class="container">
  <div class="box box1"></div>
  <div class="box box2"></div>
  <div class="box box3"></div>
</div>

body {
  margin: 0;
  background: pink;
}
.container {
  display: grid;
}
.box {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background: green;
}

gsap.timeline()
  .from('.box1', {x: 500, duration: 1})
  .from('.box2', {x: 400, duration: 2})
  .from('.box3', {x: 400, opacity: 0, backgroundColor: 'yellow'})
```



### 2.1 相对时间轴位置和绝对位置

timeline默认依次执行Tweens动画，可以通过设置第3个参数，来改变tween动画的开始位置。

假设有下面动画：

```js
gsap.timeline()
	.to('.A', { x: 400, duration: 1 })
	.to('.B', { x: 400, duration: 2 })
	.to('.C', { x: 400, duration: 1 })
// 默认执行动画时间轴 '-' 表示 0.5s
// 时间轴如下
A--
   B----
        C--
```

使用相对位置（**相对于前一个Tween**）：

```js
gsap.timeline()
	.to('.A', { x: 400, duration: 1 })
	.to('.B', { x: 400, duration: 2 }, '+=1') // 相对于前一个Tween，延后1s开始
	.to('.C', { x: 400, duration: 1 })
// 默认执行动画时间轴C '-' 表示0.5s '.' 表示 0.5s
// '+=1' 表示向后延后1s
// 时间轴如下
A--
   ..B----
          C--

// 在A完成后 B延迟1s C 提前 1s
gsap.timeline()
	.to('.A', { x: 400, duration: 1 })
	.to('.B', { x: 400, duration: 2 }, '+=1')
	.to('.C', { x: 400, duration: 1 }, '-=1') // 相对于前一个Tween结束位置，提前1s开始
// 时间轴如下
A--
   ..B-- --
        C--

// 和前一个Tween 同时开始
// 使用 '<'
gsap.timeline()
	.to('.A', { x: 400, duration: 1 })
	.to('.B', { x: 400, duration: 2 }, '+=1')
	.to('.C', { x: 400, duration: 1 }, '<') // 和前一个Tween同时开始
// 时间轴如下
A--
   ..B----
     C--

// 和前一个Tween开始之后 0.5s开始
// 使用 '<0.5'
gsap.timeline()
	.to('.A', { x: 400, duration: 1 })
	.to('.B', { x: 400, duration: 2 }, '+=1')
	.to('.C', { x: 400, duration: 1 }, '<0.5') // 相对于前一个开始位置，延后0.5s
// 时间轴如下
A--
   ..B- ---
       C--

// 在前一个Tween 结束之前 0.5s开始
// 使用 '>-0.5'， 这个其实等价于 '-=0.5' 
gsap.timeline()
	.to('.A', { x: 400, duration: 1 })
	.to('.B', { x: 400, duration: 2 }, '+=1')
	.to('.C', { x: 400, duration: 1 }, '>-0.5') // 相对于前一个结束位置，提前0.5s
// 时间轴如下
A--
   ..B--- -
         C--
```

相对位置除了可以用时间外，还可以使用百分比,比如 `<25%` 表示前一个Tween动画开始位置的 `25%` 位置开始。

📚相对位置小结：

- `+=1`： 相对于前一个Tween结束位置延后1s
- `-=1`: 相对于前一个Tween结束位置，提前1s
- `<`: 表示和前一个Tween一同开始
- `<0.5`: 在前一个Tween开始位置的0.5s后开始
- `>-0.5`: 在前一个Tween结束位置的0.5s前开始

除了使用相对位置，还可以使用绝对位置

```js
gsap.timeline()
	.to('.A', { x: 400, duration: 2 })
	.to('.B', { x: 400, duration: 3 }, 1) // 在1s位置开始
	.to('.C', { x: 400, duration: 1 }, 4) // 在4s位置开始
// 时间轴 总共时间是6s
 -- ----------
A-- --
   B------
          C--
```

**当然相对位置和绝对位置是可以一起使用的**：

```js
gsap.timeline()
	.to('.A', { x: 400, duration: 2 })
	.to('.B', { x: 400, duration: 3 }, '+=1') // 前一个动画结束后1s开始
	.to('.C', { x: 400, duration: 1 }, 0) // 在0位置就开始动画，即和A同时动画
// 时间轴
A----
     ..B------
C--
```





### 2.2 Timeline方法

除了和 `Tween` 一样的几个常用的 `play & restart & reverse & pause` 方法外，timeline可以使用 `add` 方法 给Tween添加label，这样可以直接跳到label指定的位置进行动画



```js
const animation = gsap.timeline()
  .to('.box1', { x: 500, duration: 2 })
  .to('.box2', { x: 500, duration: 1 }, '+=1') // 相对前一个tween 延后1s开始动画
  .add('customLabel') // 自定义Label 方便动画直接从Label处开始
  .to('.box3', { x: 500, duration: 2 }, '>-0.5') // 在上一个tween结束前0.5s开始

const $ = (selector) => document.querySelector(selector)

// 假设box1正在动画，如果点击按钮，则可以直接跳到标签指定的位置开始动画
$('#play').onclick = () => animation.play('customLabel') // 直接跳到Label位置
```
✏️codepen:
 - [03-gsap-timeline](https://codepen.io/JamesSawyer/pen/xxWExQV)

另外还有很多有用的方法，可以参考官方文档：

- [timelien - GSAP docs](https://greensock.com/docs/v3/GSAP/Timeline)





### 2.3 Timeline options

设置 [.vars](https://greensock.com/docs/v3/GSAP/Timeline/vars) ，传入timeline构造器，则整个timeline都将拥有这些属性，比如：

```js
gsap.timeline({
  delay: 1,
  repeat: 1,
  yoyo: true,
  onComplete: () => {
    console.log('timeline done')
  }
})
```

除了传入配置属性，timeline还可以设置默认值，比如下面的每个Tween都使用了 `opacity: 0`:

```js
const tl = gsap.timeline()
tl.to('.box1', {x: 400, opacity: 0, duration: 1})
	.to('.box2', {x: 300, scale: 1.2, opacity: 0, duration: 2})
	.to('.box3', {x: 400, opacity: 0, duration: 1})

// 使用defaults则可以写为：
// 子Tweens都将继承 opacity: 0 的属性
const tl = gsap.timeline({ defaults: { opacity: 0, ease: 'bounce' }})
tl.to('.box1', {x: 400, duration: 1})
	.to('.box2', {x: 300, scale: 1.2, duration: 2})
	.to('.box3', {x: 400, duration: 1})
```



## 3. GSDevTools

gsap开发调试工具栏，提供了暂停，开始，加速，放缓动画等等调试功能，这个直接引入即可，这个工具是gsap会员才能使用的，但是可以通过 [GreenSock Plugins](https://codepen.io/GreenSock/full/OPqpRJ/) 获取到本地版本，或者使用 [gsap cdns](https://gist.github.com/cassieevans/43237473fba253ec6d7161090ee35ff9)

使用也很简单：

```js
GSDevTools.create()
```



## 4. FOUC 问题

FOUC 表示 `Flash of Unstyled Content`, 一般是html渲染了，而js还没有加载，导致js没有执行导致的。比如现在的动画效果使用 `gsap.from()`, 因为js没有执行，页面会先渲染为css定义的位置，然后js执行时，再从js指定位置动画到自然位置，这样就造成了困惑。

解决办法：

1. 将js放在 `window.onload` 函数中，确保页面加载之后再执行js
2. 将动画部分使用 `visibility: hidden`, 先进行隐藏
3. 对容器使用gsap提供的 `autoAlpha: 0` 属性，它会自动添加 `opacity: 1; visiblity: inherit;` 属性

这样就可以确保不会出现 FOUC 问题。

```js
<div class="container">
  <div class="box box1"></div>
  <div class="box box2"></div>
  <div class="box box3"></div>
</div>
<button id="test">test</button>

body {
  margin: 0;
  background: pink;
}
.container {
  display: grid;
  gap: 20px 0;
  /* 将容器visibility设置为hidden */ 
  visibility: hidden; 
}
.box {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background: green;
}
button {
  outline: 0;
  border: 0;
  height: 44px;
  line-height: 1;
  width: 150px;
  background: rgba(124,89,123, 0.7);
  border-radius: 6px;
  color: #fff;
  font-size: 16px;
  margin-left: 40px;
}

// 在 onload 中执行方法
window.onload = function() {
  const $ = (selector) => document.querySelector(selector)
  
  const animation = gsap.timeline()
    // 对容器使用 autoAlpha: 0,
    // 它会自动将容器css设置为 opacity: 1; visibility: inherit;
    .from('.container', { autoAlpha: 0 })
    .from('.box1', { x: 500, duration: 1 })
    .from('.box2', { x: 500, duration: 2}, '+=1') // 延后1s开始
    .addLabel('test')
    .from('.box3', { x: 500, duration: 2, background: 'purple', ease: 'bounce' })
  
  // 开发者工具
  GSDevTools.create()
  
  $('#test').onclick = () => {
    animation.play('test')
  }
}
```
✏️codepen:
 - [04-gsap-fouc-problem](https://codepen.io/JamesSawyer/pen/qBoaBzR)
