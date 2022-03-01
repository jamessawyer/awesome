## V6

1. [🚀 「React进阶」react-router v6 通关指南 - 我不是外星人@掘金](https://juejin.cn/post/7069555976717729805)
   - 介绍V5版本整体架构，主要依赖 `Route` 组件去渲染
   - V6整体架构图，主要依赖 `Routes`，而 `Route` 只是作为路由结构，不去做实质的渲染
   - V6路由原理分析，核心代码逻辑：`Router` 函数，`Routes` 内部是通过 `useRoutes` 实现的，并将路由结构扁平化，`matchRoutes` 原理，新的 `Outlet` 路由占位（类似vue中的 `view-router`）
   - 整体渲染链路，从下向上渲染，通过context的方式，当 `location` 发生变化时，Routes重新渲染，通过 `provider` 的方式逐层传递Outlet
   - V6版本全面拥抱Hooks的写法
2. [🚀 Col0ring的V6文章 - @掘金](https://juejin.cn/user/2840793778240253/posts)
   1. [React-Router v6 完全解读指南 - history 篇](https://juejin.cn/post/7065599937265795109)
      - `Path | Action | Location` 等类型，相应的方法 `createPath & parsePath`
      - `History` 对象对原生 history的封装，`createBrowserHistory & createHashHistory & createMemoryHistory` 等等
      - `popstate & hashstate` 事件的处理
      - `block` 方法对 `beforeunload` 事件的处理
   2. [React-Router v6 完全解读指南 - react-router 篇（万字长文，学懂毕业）](https://juejin.cn/post/7067436563457638413)
      - 这个是对history库，react版本的进一步封装，也是react路由的核心
      - `Router` 使用 `NavigationContext & LocationContext`，对所有路由都提供 `navigator & location & navigationType`，一般不直接使用这个组件
      - `Routes & Route` 的内部实现
      - `Route` 根据传递的属性不同，分为： **路径路由 & 布局路由 & 索引路由**，Route只能在 `Routes` 中使用，其余地方单独使用会报错
      - 可以使用 `Routes + Route` 声明式的方式定义路由结构，也可以使用 `useRoutes()` 方法传入路由配置的方式 ✅
      - `useRoutes` 函数的流程： `路由上下文解析 -> 路由匹配阶段 -> 路由渲染阶段`， 而 **路由匹配阶段** 又可以细分为：`路由扁平化 -> 路由权值计算与排序 -> 匹配路由与合并`
      - `Outlet & useOutlet & useOutletContext`: 用于渲染组件和传递额外的context
      - `useNavigate` 路由跳转的原理，主要是如何进行路由匹配 ✅
      - 一些APIs: 
        - `createRoutesFromChildren`(将声明式路由结构转换为数组的形式)
        - `generatePath(path, params)` 根据传递的 params 补齐path的动态参数
        - `matchRoutes(routes, locationArg, basename)`: 通过传入的`routes` 配置项和当前`location`得到`routes`中能与`location`匹配的`matches`数组
        - `matchPath(pattern, pathname)`: 通过判断 pathname 是否匹配传入的 pattern，如果不匹配返回 null，如果匹配经过解析后的`match`对象
        - `resolvePath(ro, from)`: 将`to`与`from`两个路径相结合生成一个最终要跳转的路径
      - Hooks:
        - `useInRouterContext`: 判断当前组件是否在Router上下文中使用
        - `useLocation`: 获取当前浏览器的location ✅
        - `useNavigationType`: 获取 `Action` type
        - `useResolvePath(to)`: 根据当前`location`获取解析传入`to`的路径，返回`Path`对象
        - `useHref(to)`: 用自动添加的 `to` 路径添加 `basename` ，返回一个新的url
        - `useMatch(pattern)`: 一般用于需要根据`pathname`判断组件自身状态时使用。比如 NavLink，当传入的 `pattern`能与当前`pathname`匹配则显示`active`状态
        - `useParams()`: 用于获取当前`url`匹配到的所有`params` ✅
   3. [React-Router v6 完全解读指南 - react-router-dom / native 篇](https://juejin.cn/post/7068101548584206350)
      - `BrowserRouter & HashRouter` 对 `react-router` 中 `Router` 的封装
      - ssr中常用到的 `StaticRouter`
      - `Link & NavLink` 组件对a标签的封装和进一步封装
      - `useSearchParams` 对原生 `URLSearchParams` API的封装，用于帮助我们快速修改浏览器 `pathname` 的`search` 部分



