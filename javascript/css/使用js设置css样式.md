使用js设置css样式的4种方式

比如下面一段html，使用js设置 `p` 标签的颜色

```html
<p id="target">
  rainbow 🌈
</p>
```



## 1. 内联样式(Inline style)

基本步骤：

1. 选中元素
2. 添加样式

```js
const p = document.querySelector("#target")
p.style.color = 'red'
```



## 2. 全局样式(Global style)

创建 `<style>` 标签，添加CSS规则，然后添加到DOM中

```js
const style = document.createElement('style')
style.innerHTML = `
  #target {
		color: red;
	}
`
document.head.appendChild(style)
```



## 3. CSSOM insertRule

使用 **`CSSStyleSheet`** 的 `insertRule` 方法

```js
const style = document.createElement('style')
document.head.appendChild(style)
style.sheet.insertRule(`
	#target {
		color: red;
	}
`)
```

打开开发者工具，插入的 `<style>` 标签是空的，但是样式会添加给元素，并且通过devtools不能更改该样式，著名的 `styled-components` 就是使用这种技术



## 4. 可构建StyleSheets（chrome only）

`chrome` 可以使用 **`CSSStyleSheet`** 对象：

```js
// 创建共享的 stylesheet
const sheet = new CSSStyleSheet()
sheet.replaceSync(`
	#target {
		color: red;
	}
`)

// 将stylesheet添加给document
document.apotedStyleSheets = [sheet]
```



文章来源： [Set css styles with javascripts](https://dev.to/karataev/set-css-styles-with-javascript-3nl5)

2020年11月17日15:11:12