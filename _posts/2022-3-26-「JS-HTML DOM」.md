---
layout: "post"
title: "「前端基础」HTML"
subtitle: "JS-HTML-DOM"
author: "wyfame"
date: 2022-3-26

tags: ["前端","北航"]
lang: zh
catalog: true
header-image: ""
header-style: text
---

# JS-HTML DOM

## 一、DOM介绍

DOM is short of Document Object Model（文档对象模型）

- DOM 是 W3C（万维网联盟）的标准。
- "W3C 文档对象模型 （DOM） 是中立于平台和语言的接口，它允许程序和脚本动态地访问和更新文档的内容、结构和样式。"

### HTML DOM

- HTML 的标准对象模型
- HTML 的标准编程接口
- W3C 标准

HTML DOM 定义了所有 HTML 元素的对象和属性，以及访问它们的方法

> 换言之，HTML DOM 是关于如何获取、修改、添加或删除 HTML 元素的标准。

------



## 二、查找HTML元素

- **通过Id找到HTML元素**

  ```javascript
  var x = document.getElementById('instr');
  ```

  > 若查找成功，将在x中以对象形式返回该元素
  >
  > 若查找失败，则x为Null

- **本例查找 id="main" 的元素，然后查找 id="main" 元素中的所有 <p> 元素（通过标签名查找）：**

  ```html
  <div id="main">
  <p> DOM 是非常有用的。</p>
  <p>该实例展示了  <b>getElementsByTagName</b> 方法</p>
  </div>
  <script>
  var x=document.getElementById("main");
  var y=x.getElementsByTagName("p");
  document.write('id="main"元素中的第一个段落为：' + y[0].innerHTML);
  </script>
  ```

  > 所有符合条件的标签以数组形式返回

- **通过`getElementsByClassName()`函数来查找 class="intro" 的元素**

  ```javascript
  var x=document.getElementsByClassName("intro");
  ```

------

## 三、触发事件

- **通过点击按钮改变id1的HTML元素样式**

```html
<h1 id="id1">我的标题 1</h1>
<button type="button" onclick="document.getElementById('id1').style.color='red'">
点我!</button>
```

### onload和onunload事件

> onload 和 onunload 事件会在用户进入或离开页面时被触发。
>
> onload 事件可用于检测访问者的浏览器类型和浏览器版本，并基于这些信息来加载网页的正确版本。
>
> onload 和 onunload 事件可用于处理 cookie。

```html
<body onload="checkCookies()">

<script>
function checkCookies(){
	if (navigator.cookieEnabled==true){
		alert("Cookies 可用")
	}
	else{
		alert("Cookies 不可用")
	}
}
</script>
```

------

### onchange事件

> onchange 事件常结合对输入字段的验证来使用。

当用户改变输入字段的内容时，会调用 `myFunction()` 函数:

```html
<script>
function myFunction(){
	var x=document.getElementById("fname");
	x.value=x.value.toUpperCase();
}
</script>
输入你的名字: <input type="text" id="fname" onchange="myFunction()">
```

------

### onmouseover 和 onmouseout 事件

> onmouseover 和 onmouseout 事件可用于在用户的鼠标移至 HTML 元素上方或移出元素时触发函数。

```html
<div onmouseover="mOver(this)" onmouseout="mOut(this)" style="background-color:#D94A38;width:120px;height:20px;padding:40px;">Mouse Over Me</div>
<script>
function mOver(obj){
	obj.innerHTML="Thank You"
}
function mOut(obj){
	obj.innerHTML="Mouse Over Me"
}
</script>
```

------

## onmousedown、onmouseup 以及 onclick 事件

> onmousedown, onmouseup 以及 onclick 构成了鼠标点击事件的所有部分。首先当点击鼠标按钮时，会触发 onmousedown 事件，当释放鼠标按钮时，会触发 onmouseup 事件，最后，当完成鼠标点击时，会触发 onclick 事件。

```html
<script>
function lighton(){
	document.getElementById('myimage').src="bulbon.gif";
}
function lightoff(){
	document.getElementById('myimage').src="bulboff.gif";
}
</script>
</head>

<body>
<img id="myimage" onmousedown="lighton()" onmouseup="lightoff()" src="bulboff.gif" width="100" height="180" />
<p>点击不释放鼠标灯将一直亮着!</p>
```

------

### onfocus事件

> 当获取焦点时，将触发onfocus事件

```html
function myFunction(x){
	x.style.background="yellow";
}
</script>
输入你的名字: <input type="text" onfocus="myFunction(this)">
```

- 上例为当鼠标选中输入框时，背景将变为黄色

------

## 四、DOM EventListener

> 添加监听事件

```javascript
element.addEventListener(event, function, useCapture);
```

- event为触发事件类型，如（“click"）
- function为事件触发后调用的函数
- useCapture用于描述事件是冒泡还是捕获

例如：用户点击按钮时显示时间

```html
<button id="myBtn">点我</button>
<p id="demo"></p>
<script>
document.getElementById("myBtn").addEventListener("click", displayDate);
function displayDate() {
    document.getElementById("demo").innerHTML = Date();
}
</script>
```

### 基本语法

- addEventListener() 方法添加的事件句柄不会覆盖已存在的事件句柄。
- 你可以向一个元素添加多个事件句柄
- 你可以向同个元素添加多个同类型的事件句柄，如：两个 "click" 事件。
- addEventListener() 方法允许你在 HTML DOM 对象添加事件监听， HTML DOM 对象如： HTML 元素, HTML 文档, window 对象。或者其他支持的事件对象如: xmlHttpRequest 对象

### 事件传递方式

> 事件传递有两种方式：冒泡与捕获。

事件传递定义了元素事件触发的顺序。 如果你将 <p> 元素插入到 <div> 元素中，用户点击 <p> 元素, 哪个元素的 "click" 事件先被触发便是事件传递。

- 在 **冒泡** 中，内部元素的事件会先被触发，然后再触发外部元素，即： <p> 元素的点击事件先触发，然后会触发 <div> 元素的点击事件。
- 在 **捕获** 中，外部元素的事件会先被触发，然后才会触发内部元素的事件，即： <div> 元素的点击事件先触发 ，然后再触发 <p> 元素的点击事件。

useCapture参数默认值为false，即冒泡传递，当值为 true 时, 事件使用捕获传递

## removeEventListener() 方法

removeEventListener() 方法移除由 addEventListener() 方法添加的事件句柄，如：

```html
element.removeEventListener("mousemove", myFunction);
```

