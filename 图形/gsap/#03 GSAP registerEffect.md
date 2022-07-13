`gsap.registerEffect()` 方法类似于动画的组件化方式，这样在任何地方都可以使用 `gsap.effects.customEffect()` 进行使用。其签名如下：

```js
gsap.registerEffect({
  name: String, // 注册的效果名
  extendTimeline: Boolean, // 是否将效果注册到 gsap.timeline 的原型链上
  defaults: Object, // 下面函数中config中属性的默认值，用于传递自定义属性
  effect: (targets, config) => Tween | Timeline // 在这里写组装的动画效果
})
```

比如我们注册一个常见的 `move` 动画：

```js
gsap.registerEffect({
  name: 'move',
  defaults: {
    x: 400,
    y: 0,
    duration: 2
  },
  effect: function(targets, config) {
    return gsap.from(
      targets, 
      { 
        x: config.x,
        y: config.y,
        duration: config.duration,
        opacity: 1
      }
    )
  }
})
// 使用
// 假设 .box1 .box2 是个小球
gsap.effects.move('.box1') // 使用默认值
gsap.effects.move('.box2', { x: 500, duration: 3 }) // 自定义配置
```

假设我们想要将上面2个Tween动画加入到时间轴中：我们将使用 `Timeline.add()` 方法

```js
let tl = gsap.timeline()
tl.move('.box1')
tl.add(gsap.effects.move('.box2', { x: 500, duration: 3 }))
```

这样显得不是很优雅，我们可以使用 `extendTimeline: true` 属性，这样我们就可以进行链式调用了。

```js
gsap.registerEffect({
  name: 'move',
  extendTimeline: true, // 设置为true
  defaults: {
    x: 400,
    y: 0,
    duration: 2
  },
   effect: function(targets, config) {
    return gsap.from(
      targets, 
      { 
        x: config.x,
        y: config.y,
        duration: config.duration,
        opacity: 1
      }
    )
  }
})

// 链式调用
let tl = gsap.timeline()
tl.move('.box1')
	.move('.box2', { x: 500, duration: 3 })
```

另外对于时间轴，还可以使用绝对或者相对位置参数：

```js
let tl = gsap.timeline()
tl.move('.box1')
	.move('.box2', { x: 500, duration: 3 }, '<') // 和box1一起开始动画
```

注册effect可以说是很强大的一种组件化的方式，具体可以参考文档：

- [gsap.registerEffect - docs](https://greensock.com/docs/v3/GSAP/gsap.registerEffect())

另外演示地址：(例子中还用到了 [SplitText](https://greensock.com/splittext/) 插件)

- [rainbow register effect - snorkl.tv codepen](https://codepen.io/snorkltv/pen/LYNBZww)
- [11 - rainbow register effect](https://codepen.io/JamesSawyer/pen/GRxjQVZ)



2021年12月04日15:12:37