vue3开发中常用的tricks，提升开发效率：

1. [⭐️⭐️ 5个知识点，让 Vue3 开发更加丝滑 - 小学生study@掘金](https://juejin.cn/post/7054317318343491615)
   - 使用 [vite-plugin-vue-setup-extends](https://github.com/anncwb/vite-plugin-vue-setup-extend) 给 `setup` 语法添加 `name` 属性，以便在使用 `keep-alive` 特性时，重复定义2次 `script` 的写法
   - 使用 [unplugin-auto-import](https://github.com/antfu/unplugin-auto-import) 自动导入API，比如vue相关的api `ref | computed` 等等，不用在每个SFC中进行import（⚠️：eslint报错，可以在eslintrc中extends [vue-global-api](https://github.com/antfu/vue-global-api)）
   - 使用 [vite-plugin-vue-images](https://www.npmjs.com/package/vite-plugin-vue-images) 自动导入图片