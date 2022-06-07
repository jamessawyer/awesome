[🌟 Framer Motion](https://www.framer.com/docs/introduction/) 动画：

1. [Guide to createing animations that spark joy with Framer Motion - MaximeHeckel](https://blog.maximeheckel.com/posts/guide-animations-spark-joy-framer-motion/) 示例 + 在线编辑的形式

   - 使用 `motion` 的 `initial`（初始状态） + `animate`（最终状态） + `transition`（动画属性） 定义简单动画
   - `variants` 变种，声明式定义，动态改变动画，`variants` + `custom` + `whileHover` + `whileTap` 属性
   - 插值动画，使用 `useMotionValue` + `useTransform` 进行组合插值，例如缩放 `[0, 0.5]`, 透明度对应插值 `[0, 1]`,表示当缩放到一半的时候，opacity变为1
   - 动画编排：**父子动画，stagger动画**， 使用 `duration` + `delayChildren` + `delayDuration` 定义stagger效果

2. [Advanced animation patterns with Framer Motion - MaximeHeckel](https://blog.maximeheckel.com/posts/advanced-animation-patterns-with-framer-motion/) 进阶动画效果

   - 传播机制，在父元素设置 `initial | whileHover` 属性，子元素可以直接使用父元素设置的这些属性
   - `AnimatePresence` 包裹组件，使用 `exit` 属性添加离场动画效果
   - 使用 `layout` 属性添加 LayoutAnimation 效果： 位置属性，`flex | grid` 属性， `width | height` 属性
   - 共享布局动画效果，使用 `layoutId` 属性，类似hero动画效果

    

   

   

   

   