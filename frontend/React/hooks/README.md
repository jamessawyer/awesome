Hooks相关文章：

1. [React官方团队出手，补齐原生Hook短板 - 魔术师卡颂@wx](https://mp.weixin.qq.com/s/J_RUfn-kcynBme5FiE4mRg) 
   - Hooks 中的 **闭包陷阱**： 在事件handler中使用 local state,并且使用 `useCallback` 对函数进行包裹， local state 变化时，因为缓存的原因，导致handler中的 local state 无法更新；如果将local state当做是 `useCallback` 的依赖项，这样又使得 `useCallback` 的功能失去意义
   - 官方可能新出一个 `useEvent` 解决这个问题