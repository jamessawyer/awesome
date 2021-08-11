类 `vs` hooks

### 1. state

```jsx
// class
class CounterButton extends Component {
  constructor() {
    super()
    this.state = {
      count: 0
    }
  }
  
  render() {
    return (
    	<button onClick={() => this.setState({ count: this.state.count + 1 })}>
      	{this.state.count}
      </button>
    )
  }
}

// Hooks
function CounterButton() {
  const [count, setCount] = useState(0)
  
  return (
    <button onClick={() => setCount(count + 1)}>
      {count}
    </button>
  )
}
```



### 2. ComponentDidMount

```jsx
// Class
componentDidMount() {
  console.log('Mounted!')
}

// Hooks
useEffect(() => {
  console.log('Mounted!')
}, [])
```



### 3. ComponentWillUnmount

```jsx
// Class
componentWillUnmount() {
  console.log('Unmounted!')
}

// Hooks
useEffect(() => {
  return () => console.log('Unmounted!')
}, [])
```



### 4. ComponentWillReceiveProps | ComponentDidUpdate

`componentWillReceiveProps`

```jsx
// Class
componentWillReceiveProps(nextProps) {
  if (nextProps.count !== this.props.count) {
    console.log(`count changed`, nextProps.count)
  }
}

// Hooks
// 第一次迭代就开始打印
useEffect(() => {
  console.log(`count changed`, props.count)
}, [props.count])

// 跳过第一次打印（这个和componentWillReceiveProps一样）
const isFirstRun = useRef(true)
useEffect(() => {
  if (isFirstRun.current) {
    isFirstRun.current = false
    return
  }
  console.log(`count changed`, props.count)
}, [props.count])
```

`ComponentDidUpdate:`

```jsx
// Class
componentDidUpdate() {
  console.log(`Just updated..`)
}

// Hooks
useEffect(() => {
  console.log(`Just updated..`)
})
```



### 5. DOM Refs

```jsx
// Class
class InputWithFocus extends Component {
  constructor() {
    super()
    this.inputRef = null
  }
  render() {
    return (
    	<div>
      	<input type="text" ref={inputRef => { this.inputRef = inputRef }} />
        <button onClick={() => this.inputRef.focus()}>Focus the input</button>
      </div>
    )
  }
}

// Hooks
const InputWithFocus = (props) => {
  const inputRef = useRef(null)
  
  return (
    <div>
      <input type="text" ref={inputRef} />
      <button onClick={() => inputRef.current.focus()}>Focus the input</button>
    </div>
  )
}
```



### 6. this.myVar

除了DOM refs之外， `useRef` 还可以作为可变属性的容器存储任何值，类似于class实例中的实例属性。例如，保持interval id：

```jsx
const Timer = (props) => {
  const intervalRef = useRef()
  
  useEffect(() => {
    const id = setInterval(() => {
      // ...
    })
    intervalRef.current = id
    
    return () => {
      clearInterval(intervalRef.current)
    }
  })
}
```



### 7. comparing with previous state|props

有些生命周期，比如 `componentDidUpdate` 提供了先前状态和属性。

如果想在Hooks中使用先前的值，可以通过useRef进行模拟：

```jsx
const Counter = props => {
  const [count, setCount] = useState(0)
  
  const prevCountRef = useRef()
  
  useEffect(() => {
    prevCountRef.current = count
  })
  // 先前的状态值
  const prevCount = prevCountRef.current
  
  return <h1>Now: {count}; before: {prevCount}</h1>
}
```



### 8. ShouldComponentUpdate

可以使用 `memo` 模拟：

```jsx
// Class
shouldComponentUpdate(nextProps) {
  return nextProps.count !== this.props.count
}

// memo
import React, { memo } from 'react'

const MyComponent = memo(
	_MyComponent,
  // 注意这里的判断条件和 shouldComponentUpdate 是相反的，
  // shouldComponentUpdate 相同则不更新；memo则是相同的时候，就使用缓存的值
  (prevProps, nextProps) => nextPros.count === prevProps.count
)
```



原文链接：

- [React Class features vs Hooks equivalents - Nir Hadassi@medium](https://medium.com/soluto-engineering/react-class-features-vs-hooks-equivalents-745368dafdb3)

