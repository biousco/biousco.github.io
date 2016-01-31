title: JavaScript事件(1)
date: 2015-03-28 09:46:16
categories: 前端
tags: JS
description: 本来以为页面做的差不多了。后来发现移动端和PC端一样有好多坑要填=。= 干了几天感觉恶补了JS什么的。这次主要总结一下JS中的事件。
---

本来以为页面做的差不多了。后来发现移动端和PC端一样有好多坑要填=。=
干了几天感觉恶补了JS什么的。这次主要总结一下JS中的事件。

## 事件流
事件流描述的是从页面中接受事件的顺序。
### 事件冒泡
IE的事件流，事件开始时由最具体的元素接收，逐级向上传播到较为不具体的结点。还是很好理解的。。比如：
```html
<!DCTYPE html>
<html>
<head></head>
<body>
    <div>ddd</div>
</body>
</html>
```
单击`<div>`元素，Click事件就会从`<div>`到`<body>`到`<html>`到document。就像冒泡一样。
所有浏览器都支持，IE5.5之前会跳过`<html>`。现代浏览器冒泡到window对象。
### 事件捕获
顺序和冒泡相反，由Netscape Communicator团队提出，现代浏览器也支持这种事件流模型。"DOM2级事件"规范要求事件从document对象开始传播，但是现代浏览器从window对象开始。
### DOM事件流
包括三个阶段：事件捕获阶段、处于目标阶段、事件冒泡阶段。
事件捕获阶段为截获事件提供了机会；然后是实际的目标接收到事件；最后的冒泡阶段可以在这个阶段对事件做出相应。
就像先事件捕获（不包括目标div）然后在处于目标阶段，事件在div上发生，然后开始冒泡。
规范要求捕获阶段不会涉及事件目标，但是现代浏览器会在捕获阶段触发事件对象上的事件。因此有两个机会在目标对象上操作事件。
兼容性：IE8及之前不支持。
<!-- more -->
## 事件处理程序
相应某个事件的函数就是事件处理程序（事件侦听器）。名字以"on"开头。
### HTML事件处理程序
`<input type="button" value="Click me" onclick="alert('Clicked')" />`
某个元素支持的每种事件，都可以使用一个与相应事件处理程序同名的HTML特性来指定，这个特性的值是能够执行的JavaScript代码。不能使用未转义的HTML语法字符：& "" < >。可以改写：`&quot`代替双引号。
也可以调用函数：`onclick="showMessage()`代码执行时有权利访问全局作用域中的任何代码。
使用这种方法会创建一个封装着元素属性值的函数。包含一个局部变量event（事件对象）。通过这个变量可以直接访问事件对象。比如：`onclick="alert(event.type)"`会输出"click"。在这个函数内部，this为事件的目标元素。
另外，这个动态创建的函数可以访问document及该元素本身的成员。这个函数内部实现是通过`with()`来延长作用域。如果当前元素是表单输入元素，作用域还会包含表单元素的入口：
```javascript
function () {
    with(document){
        with(this.form){
            with(this){
                //元素属性值
            }
        }
    }
}
```
所以可以直接访问其他表单的字段：`username.value`。
这种方式的缺点：
1. 用户在点击按钮触发事件的时候函数还未解析完成，会导致错误。因此需要放在`try-catch`块中。
2. HTML代码和JavaScript代码紧密耦合。

### DOM0级事件处理程序
传统的通过JavaScript指定事件处理程序的方式，将一个函数赋值给一个事件处理程序属性。具有简单，跨浏览器的优势。
```javascript
var btn = document.getElementById('mybtn');
btn.onclick = function () {
	alert(this.id);
}
```
先取得一个对象的引用，再指定事件处理程序。在这些代码运行之前不会指定事件处理程序，所以如果代码在按钮后面，一段时间内是不会触发事件的。
这种方法事件处理程序被认为是元素的方法，在元素的作用域中运行，this指向当前元素。
这种方式添加的事件处理程序会在冒泡阶段处理。
删除：`btn.onclick = null;`

### DOM2级事件处理程序
定义了两个方法：`addEventListener()`和`removeEventListener()`。所有DOM结点都包含这两种方法，接受三个参数：要处理的事件名称，事件处理函数，是否在捕获阶段调用事件处理函数的布尔值（当为false时，在冒泡阶段调用）。
```javascript
var btn = document.getElementById('mybtn');
btn.addEventListener("click",function () {
	alert(this.id);
},false);
```
这种方法的好处是可以添加多个时间处理程序，并且按照添加的顺序触发。
移除事件处理程序只能用`removeEventListener()`，并且传入的参数要和添加的参数相同。所以一般是这样：
```javascript
var btn = document.getElementById('mybtn');
var handler = function () {
	alert(this.id);
}
btn.addEventListener("click",handler,false);
btn.removeEventListener("click",handler,false);
```
大多数情况下在冒泡阶段处理，保证兼容性。

### IE事件处理程序
两个方法：`attachEvent()`和`detachEvent()`。接受两个参数：事件处理名称和事件处理函数。因为IE8-只支持冒泡，所以这种方式添加的事件处理程序都会被添加到冒泡阶段。
```javascript
var btn = document.getElementById('mybtn');
btn.attachEvent("onclick",function () {
	alert("Clicked");
})
```
这种方法和DOM0级方法的区别在于事件处理程序的作用域。这里的方法会在全局作用域内运行，所以this等于window。
这种方法也可以添加多个事件处理程序，但按照相反的添加顺序执行。

参考书目：《JavaScript高级程序设计》

