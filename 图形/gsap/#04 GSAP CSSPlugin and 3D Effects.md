## 1. clearProps

通过这个属性，可以清除元素内联样式定义部分或者全部的属性。(搭配使用 `gsap.set()`)

```js
<div>
  <div class="box" style="color:red;"></div>
</div>
<button id="btn">清除样式</button>

gsap.to('.box', {
  opacity: 1,
  x: 500,
  backgroundColor: 'red'
})

// 上面动画本质上是添加内联样式
<div class="box"
  style="color: red; transform: translate3d(500, 0, 0); opacity: 1; background-color: red;"
></div>
```

🚀 清除背景色：

```js
document.querySelector('#btn').onclick = () => {
  gsap.set('.box', { clearProps: 'backgroundColor' })
}
// 当点击按钮后，box内敛样式中的 'background-color' 将被移除
<div class="box" style="color: red; transform: translate3d(500, 0, 0); opacity: 1;"></div>
```

🚀 清除多个属性，使用 **`,`** 隔离，*注意不要添加空格：*

```js
document.querySelector('#btn').onclick = () => {
  gsap.set('.box', { clearProps: 'backgroundColor,opacity' })
}
// 当点击按钮后，box内联样式中的 'background-color' & 'opacity' 将被移除
<div class="box" style="color: red; transform: translate3d(500, 0, 0);"></div>
```

🚀 清除所有的属性，可以使用 `clearProps: true | clearProps: 'all'`:

```js
ocument.querySelector('#btn').onclick = () => {
  gsap.set('.box', { clearProps: 'backgroundColor,opacity' })
}
// 当点击按钮后，box元素的 style 属性将被移除
<div class="box"></div>
```

**🚨 因此使用时需要注意的是，元素原有的内联样式也将被移除**。



**🚨 另外还有一点需要注意的是**，`x, y, scale, rotation` 等和 `transform` 相关的属性，只要移除其中一个，整个transform都将被移除:

```js
gsap.to('.box', {
  x: 500,
  rotation: 180,
  duration: 2,
  backgroundColor: 'red'
});
// 动画结束后 style 为
<div class="box" style="background-color: red; transform: translate(500px, 0px) rotate(180deg);"></div>

// 清除 'x' 属性
document.querySelector('#btn').onclick = () => {
  gsap.set('.box', { clearProps: 'x' })
}
// 点击按钮后 元素变为
<div class="box" style="background-color: red;"></div>
// 整个 transform 都被清除了
```

另外 `clearProps` 除了在 `gsap.set` 中使用外，还可以在Tween动画结束了之后使用：

```js
//if you clear one transform value it will clear them all
gsap.to("#herman", { duration: 1, scale: 2, x: 300, clearProps: "x" })
```



文档：

- [GSAP CSSPlugin clearProps - docs](https://greensock.com/docs/v3/GSAP/CorePlugins/CSSPlugin)

Demo:

- [12 - clearProps - fork Snorkl.tv codepen](https://codepen.io/JamesSawyer/pen/XWEjEKO)
- [clearProps on end of Tween](https://codepen.io/snorkltv/pen/ExKGNOq?editors=0110)





## 2. 3D Transform 效果

说到3D效果，首先能想到的是CSS3中定义的一些属性：

- `rotationX & rotationY & rotationZ`: 沿着哪个轴旋转
- `transform-origin`: 设置旋转的中心点，除了一般的 `transform-origin: center center` 设置x轴和y轴位置外，其实还可以设置z之轴位置，`transform-origin: center center -200px`
- `perspective`: 透视距离属性，它是提供3D效果最重要的一个属性😎
- `backface-visibility`: 旋转后背面是否可见属性，这个使用时要注意兼容性问题
- `transform-style: preserve-3d`: 这个用于指示子元素位于3D空间（实验性属性）

gsap中提供了 `transformPerspective` 属性，它可以用于设置在各个元素上，它的效果和css在父元素上设置 `perspective` 显示效果是不同的。

```js
<div class="container">
  <div class="box"></div>
  <div class="box"></div>	
</div>

// 对每个 .box设置 perspective
gsap.set('.box', { transformPerspective: '200' })
gsap.to('.box', {
  rotationY: 360,
  transformOrigin: '50% 50%'
})
```

css对父元素设置perspective：

```js
<div class="container">
  <div class="box"></div>
  <div class="box"></div>	
</div>

.container {
  perspective: 400px;
}

// 对每个 .box设置 perspective
gsap.set('.box', { transformPerspective: '200' })
gsap.to('.box', {
  rotationY: 360,
  transformOrigin: '50% 50%'
})
```

css中所有子元素将共享一个透视角度，因此各个子元素之间翻转的显示效果是不一致的，而使用gsap给每个子元素设置perspective，则每个子元素的显示效果将是一致的。

具体效果对比：

- [13 - GSAP 3D transform Perspective vs CSS perspective(Advanced) - codepen](https://codepen.io/JamesSawyer/pen/MWVjVmz)

更多3D效果demo:

1. [14 - SplitText插件 + Stagger + transformPerspective 3D文字效果(Advanced) - codepen](https://codepen.io/JamesSawyer/pen/rNdMdoX)
2. [15 - CSS3属性3D翻转卡片效果(Advanced) - codepen](https://codepen.io/JamesSawyer/pen/WNzGJeY) 🚀🚀





关于css3 Transform 3D 的兼容性和css 3d效果和gsap 3d的差异可以参考：

- [transform3D - caniuse](https://caniuse.com/transforms3d)
- [3D Transform - gsap blogs](https://greensock.com/css3/)







2021年12月04日15:45:28