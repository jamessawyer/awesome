记录一些优质libs源码分析的blogs：

1. [markdown-it](https://github.com/markdown-it/markdown-it) 滴滴前端团队的解析
   - [markdown-it源码分析1-整体流程](https://juejin.cn/post/6844903921555603470) 入口文件的分析 `src -> [parser -> tokenize -> render] -> html字符串`
   - [markdown-it源码分析2-Ruler & Token](https://juejin.cn/post/6844903921555619847) Ruler & Token 基础型分析：Token是parse的产物，Ruler是职责链管理器，存放解析规则，和render规则
   - [markdown-it源码分析3-ParserCore](https://juejin.cn/post/6844903921555619854) 编译的核心管理者，掌握着不同类型的 token 生成的流程 `CoreState -> Ruler(normalize->block->inline->linkify->replacements->smartquotes) -> Tokens`
   - [markdown源码分析4-ParserBlock](https://juejin.cn/post/6844903921559797768) 处理 `BlockState`, 以换行符作为依据进行解析，经过 `rule_block` 规则处理生成Tokens
   - [markdown-it源码分析5-ParserInline](https://juejin.cn/post/6844903921559797767) 处理 `InlineState`, 对上面BlockState处理后的Token进行更细的处理，经过 `rule_inline` 规则进行处理生成Tokens
   - [markdown-it源码分析6-Renderer](https://juejin.cn/post/6844903921563992078) 最后解析完成的Tokens，使用 `renderers` （RenderInline & RenderByRules & RenderToken） 渲染成最终产物 **html字符串**
   - [markdown-it源码分析7-插件markdown-it-emoji](https://juejin.cn/post/6844903921563992077) markdown-it emoji插件的实现细节