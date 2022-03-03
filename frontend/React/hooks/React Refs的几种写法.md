[Refs API](https://reactjs.org/docs/refs-and-the-dom.html) 是 `reference` 的简写，refs用于访问React组件中底层DOM元素。访问DOM元素的用处有很多，常见用处就是管理focus和触发动画。这篇文章将学习如何合理的使用refs，如何使用 `current` API,以及何时做出不同的选择。



扩展阅读：

- [🚀 Ref的使用和原理 - 我不是外星人@掘金小册](https://juejin.cn/book/6945998773818490884/section/6953939191776411689)
- [react ref 为什么更新前要先设置为null，然后再赋值 - @github issues](https://github.com/facebook/react/issues/9328#issuecomment-298438237)



## 1. Refs的问题

在js中操作DOM对开发者来讲应该是司空见惯的，为什么开发者使用 refs 会不适应呢？



### 1.1 命令式范式

Refs本质上是命令式的，这和React声明式的特性形成鲜明的对比。例如，下面普通js添加事件的方式和React之间的差别：

```html
<div id="my-custom-button">
  Click me!
</div>
```

js部分：

```js
var onClick = function() { console.log('clicked') }
var button = document.getElementById('my-custom-button')
button.addEventListener('click', onClick)
```

上面就是命令式的写法。

React中的写法：

```jsx
const customButton = () => {
  const onClick = () => console.log(`click`)
  return <div onClick={onClick}>Click me</div>
}
```

除了语法糖，声明式和命令式是2种不同的方式。而使用 `refs` 和第一种情况很类似：定义如何控制元素。



### 1.2 多种创建refs 的 APIs

4种创建refs的方法：

1. 当React诞生之初，React推荐使用 **字符串refs**
2. 但现在这种方式已被废弃。另一种更强大的方式被引入： **回调refs**。但是作用越大，代价也越大：回调refs很冗杂，并且可能表现怪异
3. 为了进一步简化， React引入了 `createRef` API
4. 最后，在hooks引入后，`useRef` 并出现了

因为这4种方式做的是相同的事情，导致开发者对如何使用refs失去了信心。



### 1.3 回调Refs 和 createRef 做出抉择

因为 **字符串refs** 已被React废弃，那么便有了： 该使用 **回调refs** 还是 `createRef` API呢？

这个问题的答案就是 **大多数时间你可以安心的使用 `createRef` , 尽管回调refs可以做相同的事情，但要记住 `createRef` 是了简化这个API才被设计出来的**。

为了本文介绍refs，可以查看这个 [cheat sheet 示例](https://react-refs-cheatsheet.netlify.com/)。

下面看看如何编程式的触发focus：

```jsx
class SimpleRef extends Component {
  constructor() {
    super()
    this.inputRef = React.createRef()
  }
  
  onClick() {
    this.inputRef.current.focus()
  }
  
  render() {
    return (
    	<div>
        <input ref={this.inputRef} />
        <button onClick={this.onClick.bind(this)}>Click to Focus</button>
      </div>
    )
  }
}
```

这个API很简单，你先定义一个 `ref`,然后将其赋值给你想要操控的元素，然后在 `ref` 的 `current` 属性上调用 `focus`.

下面是使用 **回调refs** 的方式：

```js
class SimpleCallbackRef extends Component {

  onClick() {
    this.inputRef.focus()
  }
  
  render() {
    return (
    	<div>
        <input ref={ref => { this.inputRef = ref }} />
        <button onClick={this.onClick.bind(this)}>Click to Focus</button>
      </div>
    )
  }
}
```

注意现在你不用手动的创建ref，回调函数 `ref => { this.inputRef = ref }` 看起来不是很直观，并且有一个讨人厌的缺点：

```jsx
class InlineCallbackRefWithReRender extends Component {
  constructor() {
    super()
    this.state = { count: 0 }
  }

  onClick() {
    this.inputRef.focus()
    this.setState({ count: this.state.count + 1 })
  }
  
  render() {
    return (
    	<div>
        <input ref={ref => { this.inputRef = ref }} />
        <button onClick={this.onClick.bind(this)}>Click to Focus</button>
      </div>
    )
  }
}
```

 现在，当状态发生变化时，会触发重渲染，这会导致回调函数被调用2次：第一次是 `null`, 第二次才是正确值，[React docs对这种情况也有所说明](https://reactjs.org/docs/refs-and-the-dom.html#caveats-with-callback-refs)。

这也就意味着，下面的情况会抛出错误，因为第一次调用时 `ref` 是 `null`:

```jsx
<input ref={ref => ref.focus()} />
```

你可以使用下面方式修正这个错误： `ref => ref && ref.focus()` 或者在构造器中绑定回调。但是如果你直接在 `render` 中绑定就没作用了：

```jsx
class ConstructorBoundCallbackRefWithReRender extends Component {
  constructor() {
    super()
    this.state = { count: 0 }
    // 需要在构造器中绑定
    this.onRefMount = this.onRefMount.bind(this)
  }
  
  onClick() {
    this.inputRef.focus()
    this.setState({ count: this.state + 1 })
  }
  
  onRefMount(ref) {
    // ✅ 在挂载时，只调用一次
    ref.focus()
    this.inputRef = ref
  }
  
  render() {
    return (
    	<div>
        <input ref={this.onRefMount} />
        <button onClick={this.onClick.bind(this)}>Click to Focus</button>
      </div>
    )
  }
}
```

这看起来就和 `createRef` API很像了。现在也许你对回调refs已经很失望了，因为它太易错了，因此 React 对于简单场景 更倾向使用 `createRef` API，大多数时候用这个API就足够了。



## 2. 函数组件

为了进一步简化，假设写一个函数组件：

```jsx
const FunctionComponentWithRef = () => {
  const textInput = React.createRef()
  
  return (
    <div>
    	<input ref={textInput} />
      <button onClick={() => textInput.current.focus()}>Click to Focus</button>
    </div>
  )
}
```

组件函数在hooks出现前，并不能完全替代类组件的所有功能。[React hooks V16.8](https://reactjs.org/docs/hooks-intro.html) 被引入。



### 2.1 使用回调refs的场景

显而易见， `createRef` 提供一个简单的API，但是，回调refs也不会被废弃。那他们有什么特别之处呢？

考虑下面场景，假设你想动态的创建refs。首先使用 `createRef` 创建ref，然后将其赋值给引用。但 [ref不和父组件共享相同的生命周期](https://github.com/reactjs/rfcs/pull/17#issuecomment-362754859) 可能导致一些问题。

假设用户能动态的创建任务列表，彼此堆叠，并且每个task都有一个按钮，点击时滚动到该task。换句话说，[你想要这种效果](https://react-refs-cheatsheet.netlify.app/#dynamic-refs-callback)。

那么你该怎么做呢？

```jsx
import randomColor from 'randomcolor'

class DynamicRefs extends Component {
  constructor() {
    super()
    this.state = {
      tasks: [
        { name: 'Task 1', color: 'red' },
        { name: "Task 2", color: "green" },
        { name: "Task 3", color: "yellow" },
        { name: "Task 4", color: "gray" }
      ]
    }
    
    this.refsArray = []
  }
  
  render() {
    return (
    	<div>
      	<div>
          <button
            onClick={() => {
              const newTasks = this.state.tasks.concat([{
                name: "Task " + this.state.tasks.length + 1,
            		color: randomColor() // Just assign some random color
              }])
              this.setState({tasks: newTasks})
            }}
          >Add new Task</button>
        </div>
        {this.state.tasks.map((task, i) => (
        	<button key={i}
            onClick={() => { this.refsArray[i].scrollIntoView() }}>
            Go to {task.name}
          </button>
        ))}
        {this.state.tasks.map((task, i) => (
        	<div 
            key={i}
            ref={ref => { this.refsArray[i] = ref}}
            style={{height: '300px', backgroundColor: task.color}}
          >
           {task.name}
          </div>
        )}
      </div>
    )
  }
}
```

注意因为 `this.state.tasks` 是动态的，当组件重渲染时，你使用 **回调refs** 存储引用。

虽然使用 `createRef` 也能做同样的事情，但是当 `this.state.tasks` 变化时，你需要负责更新 `this.refsArray`，因此这可能导致错误。下面是这种写法：

```jsx
import randomColor from 'randomcolor'

class DynamicCreateRef extends Component {
  constructor() {
    super()
    const tasks = [
      { name: 'Task 1', color: 'red' },
      { name: "Task 2", color: "green" },
      { name: "Task 3", color: "yellow" },
      { name: "Task 4", color: "gray" }
    ]
    this.state = { tasks }
    
    this.updateRefsArray(tasks)
  }
  
  updateRefsArray(tasks) {
    this.refsArray = tasks.map(() => React.createRef())
  }
  
  render() {
    return (
    	<div>
      	<div>
          <button
            onClick={() => {
              const newTasks = this.state.tasks.concat([{
                name: "Task " + this.state.tasks.length + 1,
            		color: randomColor() // Just assign some random color
              }])
              this.setState({tasks: newTasks})
            }}
          >Add new Task</button>
        </div>
        {this.state.tasks.map((task, i) => (
        	<button key={i}
            onClick={() => { this.refsArray[i].scrollIntoView() }}>
            Go to {task.name}
          </button>
        ))}
        {this.state.tasks.map((task, i) => (
        	<div 
            key={i}
            ref={ref => { this.refsArray[i] = ref}}
            style={{height: '300px', backgroundColor: task.color}}
          >
           {task.name}
          </div>
        )}
      </div>
    )
  }
}
```



### 2.2 给子组件添加Refs

上面提到的情况都是，refs赋值给DOM，而这些DOM元素都是 `render` 中可见的。如果不可见的情况呢？

例如：

```jsx
function CustomInput() {
  return <input />
}

class SimpleRef extends Component {
  constructor(props) {
    super(props)
    this.textInput = React.createRef()
  }
  
  render() {
    // 这样传 不起作用
    return <CustomInput ref={this.textInput} />
  }
}
```

因为函数组件不存在实例，因此添加对它的引用会失败。解决办法就是使用 [forwardRef](https://reactjs.org/docs/forwarding-refs.html) ，用法如下：

```jsx
const CustomInput = React.forwardRef((props, ref) => (
   <input ref={ref} />
))

class SimpleRef extends Component {
  constructor(props) {
    super(props)
    this.inputRef = React.createRef()
  }
  
  onClick() {
    this.inputRef.current.focus()
  }
  
  render() {
    return (
    	<div>
        <CustomInput ref={this.inputRef} />
        <button onClick={this.onClick.bind(this)}>Click to Focus</button>
      </div>
    )
  }
}
```

子元素使用 `forwardRef` 使得 `input ref` 在父组件中也能访问到。

 `forwardRef` 只能用于函数组件上，如果你想在类组件上显示这种效果，可以按照下面的 [例子](https://react-refs-cheatsheet.netlify.app/#forwarding-to-custom-components)

```jsx
class ClassInput extends Component {
  constructor() {
    super()
    this.inputRef = React.createRef()
  }
  
  customFocusMethod() {
    this.inputRef.current.focus()
  }
  
  render() {
    return <input ref={this.inputRef} />
  }
}

class ComponentRef extends Component {
  constructor() {
    super()
    // ⚠️：需要使用2个refs
    this.componentRef = React.createRef()
  }
  
  onClick() {
    // componentRef 表示复合组件，因此能访问其方法
    this.componentRef.current.customFocusMethod()
  }
  
  render() {
    return (
    	<div>
      	<ClassInput ref={this.componentRef} />
        <button onClick={this.onClick.bind(this)}>Click to Focus</button>
      </div>
    )
  }
}
```



### 2.3 useRef Hook

`React V16.8` 提出了Hook的概念，`useRef` 表示一个在组件整个生命周期中可持久化的可变对象。

```jsx
const RefsWithHook = () => {
  const inputRef = useRef(null)
  
  return (
  	<div>
    	<input ref={inputRef} />
      <button onClick={() => inputRef.current.focus()}>Click to Focus</button>
    </div>
  )
}
```

使用Hooks动态的设置refs：

```jsx
import React, { useState, useRef } from 'react';
import randomColor from 'randomcolor';

const DynamicRefsWithHooks = () => {
  const initialTasks = [
    { name: "Task 1", color: "red" },
    { name: "Task 2", color: "green" },
    { name: "Task 3", color: "yellow" },
    { name: "Task 4", color: "gray" }
  ];
  const [tasks, setTasks] = useState(initialTasks);
  const refsArray = useRef([]);
  return (
    <div>
      <div><button onClick={() => {
        const newTasks = tasks.concat([{
          name: "Task " + this.state.tasks.length + 1,
          color: randomColor()
        }]);
        setTasks(newTasks);
      }}>Add new Task</button></div>
      {tasks.map((task, i) => (
        <button
          key={i}
          onClick={() => { refsArray.current[i].scrollIntoView(); }}>
          Go to {task.name}
        </button>
      ))}
      {tasks.map((task, i) => (
        <div 
          key={i}
          ref={ref => { refsArray.current[i] = ref; }} 
          style={{height: "100px", backgroundColor: task.color}}>
          {task.name}
        </div>
      ))}
    </div>
  );
};
```



## 3. 总结

总结refs的所有用法：

1. 不要过度使用refs
2. 停止使用字符串类型的refs
3. 只有在**动态设置refs**时，才使用回调refs
   1. 在类组件中，请使用 `createRef`
   2. 在函数组件中，请使用 `useRef`
4. 当需要访问子组件中的ref时，使用 `forwardRef`
   1. 使用Hooks增强你的函数组件
   2. 如果子组件ref必须不是一个函数组件，请使用自定义方法在父组件中触发focus（记住你得到的是组件实例，而不是DOM元素）





原文链接：

- [The Complete Guide to React Refs](https://rafaelquintanilha.com/the-complete-guide-to-react-refs/?utm_campaign=React%2BNewsletter&utm_medium=web&utm_source=React_Newsletter_166)

文中所有示例在线运行：

- [React Refs Cheat Sheet](https://react-refs-cheatsheet.netlify.app/)



2021-03-21 16:08

