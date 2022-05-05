关于react中使用ts的一些优质文章：
1. [⭐️⭐️⭐️如何优雅地在 React 中使用TypeScript，看这一篇就够了！- CUGGZ@掘金](https://juejin.cn/post/7021674818621669389)
   - React内置类型：还有类组件的写法
     - `React.ReactNode` 组件实例
     - `React.FC<P>`: 函数组件
     - `React.CSSProperties`： 组件style
   - hooks ts写法
   - 事件相关的类型，React事件类型 + 原生事件类型 + HTML标签类型
   - **axios** 的ts封装
   - 扩展阅读其ts专栏： [深入浅出TypeScript](https://juejin.cn/column/6997066245245763597)



常见问题：
1. [JSX.Element vs ReactNode vs ReactElement 类型的区别 - @stackoverflow](https://stackoverflow.com/a/58123882)

2. 关于使用 `useRef` 可能碰到的2个问题

   - [useRef React Hook Cannot Assign to Read Only Property](https://www.designcise.com/web/tutorial/how-to-fix-useref-react-hook-cannot-assign-to-read-only-property-typescript-error) useRef返回类型存在 `RefObject` & `MutableRefObject` 2种类型，初始化时不存入值，则是 `MutableRefObject`  类型
   - [ref.current possibly null](https://www.designcise.com/web/tutorial/how-to-fix-object-is-possibly-null-typescript-error-when-using-useref-react-hook) 可能为空的情况，这个时候就需要进行断言去处理

   ```typescript
   export default function Dong() {
     const [count, setCount] = useState(0)
   
     useEffect(() => {
       setInterval(() => {
         setCount(count => count + 1)
       }, 500)
     }, [])
   
     const fn = () => {
       console.log(count)
     }
   
     // 1️⃣ const ref = useRef() 直接这样会报下面错误
     // https://www.designcise.com/web/tutorial/how-to-fix-useref-react-hook-cannot-assign-to-read-only-property-typescript-error
     // Cannot Assign to Read Only Property
     const ref = useRef<() => void | null>()
   
     // useLayoutEffect } from 在 render之前 同步执行
     useLayoutEffect(() => {
       ref.current = fn
     })
   
     // useEffect 在 render 之后 异步执行
     useEffect(() => {
       setInterval(() => {
         // 2️⃣ 直接使用 ref.current() 会报如下错误
         // How to Fix "Object is possibly 'null'" TypeScript Error When Using useRef React Hook?
         // https://www.designcise.com/web/tutorial/how-to-fix-object-is-possibly-null-typescript-error-when-using-useref-react-hook
         if ( ref && ref.current) {
           ref.current()
         }
       }, 500)
     }, [])
   
     return <h1>Dong {count}</h1>
   }
   ```



## 📚 资料

1. [🚀 React Typescript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/docs/basic/setup)
2. [React18 Types - @github](https://github.com/DefinitelyTyped/DefinitelyTyped/pull/56210) React18中类型的breaking changes

