这篇文章将学习到关于 `useEffect` 的一切。

假设有2个组件，父组件使用 [useState](https://www.robinwieruch.de/react-usestate-hook) 管理状态，子组件使用该组件，并通过 [callback event handler](https://www.robinwieruch.de/react-event-handler/) 修改状态

```jsx
import * as React from 'react'

const App = () => {
  const [toggle, setToggle] = React.useState(true)
  
  const handleToggle = () => {
    setToggle(!toggle)
  }
  
  return <Toggler toggle={toggle} onToggle={handleToggle} />
}

const Toggler = ({toggle, onToggle}) => {
  return (
    <div>
      <button type="button" onClick={onToggle}>Toggle</button>
      {toggle && <div>Hello React</div>}
    </div>
  )
}
```

根据父组件传入的 `toggle` 属性，子组件条件的渲染 `Hello React` 组件。下面我们深入学习 `useEffect`， 它本质上就是运行一个副作用函数。它可以只在组件挂载，或者组件渲染时，又或者只在组件重渲染时，等等条件下被调用。



## 1. 情形1: 始终调用

下面情况下，`useEffect` 每次渲染都被调用：

```jsx
const Toggler = ({toggle, onToggle}) => {
  React.useEffect(() => {
    console.log(`每次渲染都会被渲染： 挂载 + 更新`)
  })
  
  return (
    <div>
      <button type="button" onClick={onToggle}>Toggle</button>
      {toggle && <div>Hello React</div>}
    </div>
  )
}
```

这种方式很直白，我们只传入一个函数到 `useEffect` 中。它会在函数第一次渲染（即挂载mount）时，以及每次重渲染时（即更新update）都会被调用。



## 2. 情形2: 只在挂载时调用

如果你想它 **只在第一次渲染** 时被调用（即 only on mount）, 可以在 `useEffect` 中传入第2个参数：

```jsx
const Toggler = ({toggle, onToggle}) => {
  React.useEffect(() => {
    console.log(`只在组件第一次渲染时调用： 挂载`)
  }, [])
  
  return (
    <div>
      <button type="button" onClick={onToggle}>Toggle</button>
      {toggle && <div>Hello React</div>}
    </div>
  )
}
```

这里第2个参数是一个 **空数组**，它被称之为 **依赖数组(`dependency array`)**: 如果依赖数组为空，则表示 `useEffect` 的副作用函数中不存在依赖，因此只会在组件第一次渲染时被调用。



## 3. 情形3: 更新时调用（包含第一次渲染）

上面我们知道了依赖数组，这个数组可用于在某些变量发生变化时，重新运行副作用函数：

```jsx
const Toggler = ({toggle, onToggle}) => {
  React.useEffect(() => {
    console.log(`调用： 挂载 + toggle变量发生变化时`)
  }, [toggle])
  
  return (
    <div>
      <button type="button" onClick={onToggle}>Toggle</button>
      {toggle && <div>Hello React</div>}
    </div>
  )
}
```

现在这个组件的副作用函数 **在组件第一次渲染时，副作用函数会执行，另外当依赖数组中的变量发生变化时也会调用**。

另外，依赖数组可以添加多个依赖变量，例如：

```jsx
const Toggler = ({toggle, onToggle}) => {
  const [title, setTitle] = React.useState('Hello React')
  
  React.useEffect(() => {
    console.log(`调用： 挂载 + toggle或title变量发生变化时`)
  }, [toggle, title])
  
	const handleChange = (e) => {
    setTitle(e.target.value)
  }
  
  return (
    <div>
      <input type="text" value={title} onChange={handleChange} />
      <button type="button" onClick={onToggle}>Toggle</button>
      {toggle && <div>{title}</div>}
    </div>
  )
}
```



## 4. 情形4: 只在更新时调用（不包含第一次渲染）

通过上面的例子，我们知道存在第二个参数：依赖数组的情况时，第一次渲染也会调用副作用函数。那么怎么在第一次渲染时不调用，而只在依赖数组发生变化时调用呢？

**我们可以使用 [ref](https://www.robinwieruch.de/react-ref) 实现：因为 `ref` 在整个重渲染时，值不会丢失**：

```jsx
const Toggler = ({toggle, onToggle}) => {
  const didMount = React.useRef(false)
  
  React.useEffect(() => {
    if (didMount.current) {
       // 第一次渲染时 didMount.current 为false
       // 之后更新阶段 didMount.current 为true
       // 因此确保了 只有更新时调用
       console.log(`调用：title变量发生变化时`)
    } else {
      // 第一次渲染时，会走到这里
      didMount.current = true
    }
   
  }, [toggle])
  
	const handleChange = (e) => {
    setTitle(e.target.value)
  }
  
  return (
    <div>
      <input type="text" value={title} onChange={handleChange} />
      <button type="button" onClick={onToggle}>Toggle</button>
      {toggle && <div>{title}</div>}
    </div>
  )
}
```

[下面是对这个需求的自定义hook](https://www.robinwieruch.de/react-useeffect-only-on-update):

```jsx
const useEffectOnlyOnUpdate = (callback, dependencies) => {
  const didMount = React.useRef(false)
  
  React.useEffect(() => {
    if (didMount.current) {
      callback(dependencies)
    } else {
      didMount.current = true
    }
  }, [callback, dependencies])
}
```

使用：

```jsx
const App = () => {
  const [toggle, setToggle] = React.useState(true)
  
  const handleToggle = () => {
    setToggle(!toggle)
  }
  
  useEffectOnlyOnUpdate((dependencies) => {
    console.log(`调用：title依赖变量发生变化时`)
  }, [toggle])
  
  return (
    <div>
      <button type="button" onClick={onToggle}>Toggle</button>
      {toggle && <div>Hello React</div>}
    </div>
  )
}
```

## 5. 情形5: 只调用一次

上面的例子我们已经知道，我们可以传入一个空数组依赖，这样就只在组件第一次渲染时调用一次。**那么怎么实现，只在依赖数组更新时，只调用一次呢？**

```jsx
const Toggler = ({toggle, onToggle}) => {
  const calledOnce = React.useRef(false)
  
  React.useEffect(() => {
    if (calledOnce.current) {
      // 如果 calledOnce.current 为 true 表示已经调用过了
      return
    } 
    // 因为父组件一开始传入的是 true,
    // 因此发生变化时 toggle 将变为false
    if (toggle === false) {
      console.log(`只在 toggle 属性为 false 时调用`)
      // 将其设置为 true， 这样后续 toggle变化时 上面直接返回了
      calledOnce.current = true 
    }
   
  }, [toggle])
  
  
  return (
    <div>
      <button type="button" onClick={onToggle}>Toggle</button>
      {toggle && <div>{title}</div>}
    </div>
  )
}
```

和之前一样，使用 `useRef` 追踪飞状态信息，一旦条件满足，就不在调用副作用函数。可以使用 [自定义hook: useEffect only on update once](https://www.robinwieruch.de/react-useeffect-only-once):

```jsx
const useEffectOnlyOnce = (callback, dependencies, condition) => {
  const calledOnce = React.useRef(false)
  
  React.useEffect(() => {
    if (calledOnce.current) {
      return
    }
    
    if (condition(denpendencies)) {
      callback(denpendencies)
      
      calledOnce.current = true
    }
  }, [callback, condition, dependencies])
}
```

使用：

```jsx
const Toggler = ({toggle, onToggle}) => {
  const [toggle, setToggle] = React.useState(true)
  
  const handleToggle = () => {
    setToggle(!toggle)
  }
 
  useEffectOnlyOnce(
  	(dependencies) => {
      console.log(`只在toggle为false时运行一次`)
    },
    [toggle],
    (dependencies) => dependencies[0] === false
  )
  
  return (
    <div>
      <button type="button" onClick={onToggle}>Toggle</button>
      {toggle && <div>{title}</div>}
    </div>
  )
}
```

## 6. 情形6: 清理工作

有时可能在组件重渲染时做一些清理工作，`useEffect` 内置了这个功能，它的副作用函数会返回一个清理函数：

```jsx
const App = () => {
  const [timer, setTimer] = React.useState(0)
  
  React.useEffect(() => {
    const interval = setInterval(() => setTimer(timer + 1), 1000)
    
    return () => clearInterval(interval)
  }, [timer])
  
  return <div>{timer}</div>
}
```

当组件第一次渲染时，设置一个interval，每1秒跳动一次，当interval开启后，timer的状态会+1。状态的变化，会触发重渲染。如果不做不清理，会导致重新设置另一个interval。这不是我们想要的行为，因为我们只需要一个interval。这就是为什么useEffect在组件更新前会清理interval，然后组件设置一个新的interval。本质上，在此示例中，interval在清除之前仅运行一秒钟。



## 7. 情形7: 卸载时调用

`useEffect` 的清理函数在组件卸载时也会调用，这对于在组件不再存在之后应该停止运行interval或任何其他消耗内存的对象来说是有意义的。

将上面例子，稍作修改：

```jsx
const App = () => {
  const [timer, setTimer] = React.useState(0)
  
  React.useEffect(() => {
    const interval = setInterval(
      () => setTimer(currentTimer => currentTimer + 1),
     	1000
    )
    
    return () => clearInterval(interval)
  }, [])
  
  return <div>{timer}</div>
}
```

这里 `useState` 传入的是一个函数去更新状态，而不是一个值。这个函数有一个 `currentTime`, 因此这里不需要将 `timer` 作为依赖。因此这里的清理函数只有在组件卸载时才会被调用。



原文链接：

- [How to useEffect in React - robinwieruch](https://www.robinwieruch.de/react-useeffect-hook)

2021-03-22 16:27

