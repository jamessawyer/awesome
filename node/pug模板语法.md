Jade和Sass一样通过空格来控制格式，一般推荐使用tab(2个空格键大小)来进行缩进。

代码演示：

- [pug playground](https://pug.vercel.app/)

## 1️⃣ Jade 小技巧

> **:**

**表示不用换行， 在jade中叫block expansion**

```jade
li: a(href="#") Home
// 等同于
li
	a(href="#") Home
```

> **|**

表示这行不用添加任何标签,只有内容存在

示例1：

```jade
.glyphicon.glyphicon-email
|  3 days ago
// 表示图标后面直接添加日期

.glyphicon.glyphicon-email
days ago
// 如果省略"|" jade会认为上面的email图标叫days
// 结果只会显示 图标+ ago
```

示例2：写一个ul>li>a>{文字}

```jade
ul
  li(role="presentation")
    a(href="#"): |Home
  li(role="presentation")
		a(href="#"): |Maps

// 利用 ： 和 |
// 等同于
<ul>
	<li><a>Home</a></li>
	<li><a>Maps</a></li>
</ul>
```


​    

> **添加图标**

比如说添加一些fontAwesome图标, **注意下面的a后面的写法**

```jade
dl
  dd CONTACT US
	dl
	  // Twitter前面2个空格
	  a(href=""): i.fa.fa-twitter  Twitter
	dl
	  a(href=""): i.fa.fa-facebook  Facebook
	dl
	  a(href=""): i.fa.fa-google  Google+
```

相当于

    <dl>
    	<a href="">
    		<i class="fa fa-twitter"></i> Twitter
    	</a>
    </dl>


## 2️⃣ 注释 ##

> **单行注释**

Jade可以像JS一样的注释， 使用 

- **`//`**： 在编译出来的html中显示出来 
- **`//-`**： 在编译出来的html中不显示出来

比如：

```jade
// 注释1 注意 p旁边的 '.' 不能省略 省略后 底下文字会被解析为一个tag
p.
  我们添加一个单行注释，并且注释将在html中显示出来

//- 注释2，这是第二段
p 
  | 我们添加另一个单行注释，这个注释不会在html中显示出来
```


> **多行注释**

多行注释在单行注释的基础上对注释文字进行缩进即可

```jade
//
  这是另一种注释
  这是一个多行注释
p hello world

# 编译结果
<!--
这是另一种注释
这是一个多行注释-->
<p>hello world</p>
# 同样可以通过 //- 来设置在编译之后不显示出来
```


## 三.定义变量 ##

jade 作为模板语言，首先将语句转换成js,然后由js生成dom，我们可以像js一样定义变量， **通过 `- ` 来声明一个变量**。

```jade
- var some_text = 'hello world';
- list = ['bob', 'james', 'kobe']
- obj = {'type': 'text', 'value': 'anything'}
```

**变量的使用**，是通过类似其它模板语言一样， 使用 **`#{}`**.(ES6 中使用**`${}`**)

```jade
p #{some_text}
// 或者
p= some_text
// 都等同于
p 'hello world'
```

和ES6模板一样，jade模板也可以使用计算操作

```jade
p 23 * 10 = #{23 * 10}
// 等同于
p 23 * 10 = 230
```

在元素属性中使用变量我们需要使用ES6语法：**\`${variableName}\`**

```jade
// 比如定义变量
- dog = 'mary'
// 在img 元素中
// '#{dog}'是pug语法
// `images/${dog}` 是ES6
img(src=`images/${dog}`, alt=`${dog}`) 
p #{dog}
```

更多示例：

```jade
- base_url = 'http://www.baidu.com' 
a(href=`${base_url}/maps`) 百度地图

- i = {'type': 'text', 'name': 'Bob'}
input(type=`${i.type}, value=`${i.name}`)

- content = 'whatever the content is'

p #{content}
// 或者
p= content
```

> **转义**

jade为防止XSS攻击， 对于html代码需要进行转义

```jade
- html_content = 'hello <em>world</em>'
p #{html_content}
// 想要转换成html，变为
<p>hello <em>world<\em></p>
// 而实际，在浏览器页面中显示为
<p>hello &lt;em&gt;world&lt;/em&gt;</p>
```

**转义的方法有以下2种：**

```jade
p!= html_content
// 或者
p !{html_content}
```


## 3️⃣ 逻辑操作

jade也可以进行一般的逻辑操作和流程控制，主要由以下几个方面：

- **`if/else`**
- **`unless/else`**
- **`case...when`**
- **`for loop`**
- **`each`**
- **`while`**

> **1.if/else**

一般写法：

```jade
- name = 'Bob'
- if (name == 'Bob'){
    h1 Hello Bob
- } else {
    h1 My name is #{name} 
- }
```

还可以进行三元运算：

```jade
- name = 'Bob'
- greeting = (name == 'Bob' ? 'Hello': 'My name is')
h1 #{greeting} #{name}
```

**简写方法，这个也是我们一般书写的方法**

```jade
- name = 'Bob'
if name == 'Bob'
  h1 hello Bob
else
  h1 My name is #{name}
```

同样， 可以加入 **else if**:

```jade
- name = 'Bob'
if name == 'Bob'
  h1 hello bob
else if name == 'James'
  h1 hello James
else
  h1 my name is #{name}
```

> **2.unless...**

这个其实就是对条件加一层否定， 等同于 **if(!(expr))**

```jade
- name = 'Bob'
unless name == 'Bob'
  h1 my name is #{name}
else
  h1 hello Bob
```


> **3.case...when**

**这个起始等同于switch...case**

```jade
- name = 'Bob'
case name
  when 'Bob'
		p Hi Bob!
  when 'Alice'
    p Hi Alice
  default
		p Hello #{name}!
```

> **4.for loops**

和js中for一样

```jade
- list = ['one', 'two', 'three']

ul
  - for (var i = 0, i < list.length; i++) {
  	  li= list[i]
  - }

// 等同于
ul>li*3
```


> **5.Each loops**

语法为 **`each VAL[, KEY] in OBJ`**, 上面的for循环可写为：

```jade
- list = ['one', 'two', 'three']  
ul
  each item in list
		li= item

// 另一个例子
- books = ['A', 'B', 'C']
select
  each book, i in books
		option(value=i) Book name: #{book}

// 写成对象的形式也可以
- books = {'001': 'A', '002': 'B', '003': 'C'}
select
  each book, i in books
		option(value=i) Book name: #{book}
```

> **6.while loops**

本质上和上面的for loops没有什么差别

```jade
- list = ['one', 'two', 'three']
- i = 0

ul
  while i < list.length
		li=list[i]
		i++; // 注意这里的;不能省略
```

## 4️⃣ Mixins ##

这个功能和sass中的一样，都是为了代码重用设计的。jade中的**mixin**有以下几种形式：

> **1.一般形式**

```jade
mixin book(name, price)
  li #{name}: #{price}$
```

使用mixin,用 **`+`**

```jade
ul#books-list
  +book('Pride and Prejudice', 25.00)
  +book('Jane Eye', 13.67)
```

> **2.不带参数的mixin**

```jade
// 没有参数
mixin copyright
  | ©

// 调用	
p
  +copyright
  |  2015-2016
```

> **3.传递 `block`**

除了传递参数以外，还可以传递整个块作为mixin，**block当作占位符**

```jade
mixin input(name)
  li(id=name.replace(/\s/g, '-'))
	label= name + ':'
	//- 这里的 block 本质上是占位符
	block

form: ul
  +input('favorite color')
	  //- 取代上面的block
		input('type'='text')
  +input('cpmments')
  	//- 取代上面的block
		textarea Type your comment here.
```

 

> **4.arguments类数组对象**

jade中也可以使用**arguments**对象，实现mixin参数可变的情况
    
```jade
mixin list()
  ul
    - var args = [].slice.call(arguments);
    //- 
    for item in args
      li= item

// 调用
+list('one', 'two', 'three')
```

   


## 5️⃣ 模板继承 ##

使用Jade可以将页面分成各个小的部分，然后通过 **extends**, **include**， **mixin** 组装在一起。

> **1.blocks 函数**

blocks函数在jade中是一个小的容器， 通过block我们可以 **追加(append)**, **向前添加(prepend)**， **取代(replace)** 某些内容。其实上面我们已经使用到blocks充当占位符的功能了。

比如：

```jade
// layout.jade
doctype html
html
  head
    block scripts
      script(src='jquery.js')
    block styles
  body
    block content
      p there's no content here
```

等价于：

```html
<!DOCTYPE html>
<html>
  <head>
    <script src="jquery.js"></script>
  </head>
  <body>
  </body>
</html>
<p>there's no content here</p>
```

> **extends**

**`extends`** 关键词允许我们指定一个特定的模板扩展其他模板

例如下面表示

**1.替代(replace)原有的模板**：

```jade
// page1.jade（假设和layout.jade相同路径）

extends layout // .jade扩展名可以省略

block scripts  // 替代原模板中的 block scripts
  script(src='jquery.js')
  script(src='underscore.js')

block content // 替代原模板中的 block content
```

等价于
	
```html
<doctype html>
<html>
  <head>
	<script src='jquery.js'></script>
	<script src='underscore.js'></script>
  </head>
  <body>
  </body>
</html>
```


**2.追加append**:

```jade
// page2.jade
extends layout

append scripts
  script(src='underscore.js')

block content
// 结果同上
```

**3.向前追加 `prepend`**

```jade
// page3.jade
extends layout

prepend scripts
  script(src='underscore.js')

block content

// 结果
<doctype html>
<html>
  <head>
	<script src='underscore'></script> // 注意两者的顺序
	<script src='jquery.js'></script>
  </head>
  <body>
	there's no content here
  </body>
</html>
```


> **2.include 关键词**

通过**include**关键词是最简单的组合方式，当这样会使语言动态性降低， 当然这也是一种好的方法。**可以include 各种类型的文件， 如html, js, css等**

比如：

`style.css` 文件

```css
p {
  color： white;
  text-decoration: underline;
}
```

`content.html` 文件

```html
<h1>includes something</h1>
<p>this is a file for demonstrationg the use of includes</p>
```

`example.jade` 文件

```jade
doctype html
html
  head
	style(type='text/css')
	  include style.css
  body
		include content.html
```

等同于：
    
```html
<doctype html>
<html>
  <head>
	<style type='text/css'>
		p {
		  color: white;
		  text-decoration: underline; 
		}
	</style>
  </head>
  <body>
	<h1>includes something</h1>
	<p>this is a file for demonstrationg the use of includes</p>
  </body>
</html>
```

**当然最常用的还是将页面的 `header, footer` 等各部分分成各个小的组件来写，来组成一个页面，实现重用**。


> **总结**

Jade作为模板语言写html感觉还是十分的简洁的，同时语法也十分容易上手，学习成本低，用处大。现在Jade更名为pug了，原来漂亮的兔子图标也没有了，但是语言本身没有太多的变化。

参考连接：

- [Jade语言文档](http://naltatis.github.io/jade-syntax-docs/)
- [Pug官网](https://pugjs.org)
- [个人的jade学习笔记](http://codepen.io/JamesSawyer/pen/gwbpmx)
- [原简书地址](https://www.jianshu.com/p/e36dd0b9396c) 在此基础上对有些错误进行了修正