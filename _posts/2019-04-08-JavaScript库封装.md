---
layout: post
title: "JavaScript 库封装"
tags: [JavaScript]
comments: true
---

JavaScript 库封装

[^1]: https://www.jianshu.com/u/f6c54f846755

#### 基本结构

```javascript
// jQuery
// 给页面中所有的div设置字体颜色为红色
// $("div").css("color", "red")

// 1.封装的库应该是独立的单元：模块化
// 独立：不依赖任何其他三方库
//		  里面的东西大部分也是与世隔绝的，只有：$、jQuery

(function(global) {
	// global 保存了 window 对象的引用
  function jQuery() {
    
  }
  
  global.$ = global.jQuery = jQuery;
  // 等价于
  global.jQuery = jQuery;
  global.$ = jQuery;
})(window)
```



##### 基本使用-内存浪费 — 不采用

```javascript
(function(global) {
	// global 保存了 window 对象的引用
  function jQuery(selector) {
    // 1.获取页面中所有的元素
    // 2.把这个元素放在一个特定的对象中
    const elements = document.getElementsByTagName(selector);
    // 随着$()的操作频繁增加，产生无数个相同的css方法，造成内存增加
    elements.css = () => {
      
    }
    return elements;
  }
  
  global.$ = global.jQuery = jQuery;
})(window)

$("div")
$("p")
$("span")
$("img")
```



##### 基本使用-原型 — 不采用

``````javascript
// 解决方案，把DOM操作的方法放在原型中，这样看似正常访问，但是依然存在问题： 
// ---> 它破坏了原生的对象结构
HTMLCollection.prototype.css = () => {
  // ...
}
``````

 

#####  基本使用 — 解决方案

```javascript
(function(global) {
	// global 保存了 window 对象的引用
  function jQuery(selector) {
    return new F(selector);
  }
  
  global.$ = global.jQuery = jQuery;
})(window)

// jQuery对象的构造函数
function F(selector) {
  // 在jquery中 内部封装了一个Sizzle引擎（封装了无数正则表达式）
  
	// 1.获取页面中所有的元素
  // 系统支持标签遍历，支持class，id的标签查找
  const elements = document.querySelectorAll(selector);
  // 仅支持固定的TagName
  // const elements = document.getElementsByTagName(selector);
  
  // 为了让这些元素可以在css方法中进行访问，所以需要把这些元素放在对象上面进行传递
  // this.elements = elements;
  
  // jQuery为了后续操作的方便，将这些获取到的DOM元素全部放在对象身上，让自己变成一个数组一样，可以使用for进行遍历，我们把这种对象特性称之为【伪数组】
  // 实现把所有DOM元素都添加到自己身上
  for (let i = 0; i < elements.length; i++) {
    // ele: DOM元素
    var ele = elements[i];
    this[i] = ele;
  }
  
  this.length = elements.length;
}

// F 原型
F.prototype.css = function(name, value) {
  for(let i = 0; i < this.length; i++) {
    let element = this[i];
		// 设置样式
		element.style[name] = value;
  }
}
// 优化如下
F.prototype = {
  constructor: F,
  css(name, value) {
    for(let i = 0; i < this.length; i++) {
      let element = this[i];
      // 设置样式
      element.style[name] = value;
  	}
  }
}

$("div").css("color", "red")
$(".header").css("color", "red")
$("#header").css("color", "red")

// 需求： 支持$("div")[0]
/*
	jQuery为了后续操作的方便，将这些获取到的DOM元素全部放在对象身上，让自己变成一个数组一样，可以使用for进行遍历，我们把这种对象特性称之为【伪数组】
	实现把所有DOM元素都添加到自己身上
	
	for (let i = 0; i < elements.length; i++) {
    // ele: DOM元素
    var ele = elements[i];
    this[i] = ele;
  }
  
  this.length = elements.length;
  
  F.prototype.css = function(name, value) {
  for(let i = 0; i < this.length; i++) {
    let element = this[i];
		// 设置样式
		element.style[name] = value;
  }
}
*/
```



##### 优化

```javascript
(function(global) {
	// global 保存了 window 对象的引用
  function jQuery(selector) {
  	var _init = jQuery.prototype.init;
  	return new _init(selector);
  	// 等价于
    return new jQuery.fn.init(selector);
  }
  
  global.$ = global.jQuery = jQuery;
})(window)

// 给jQuery添加了一个fn属性，fn属性等价于prototype
jQuery.fn = jQuery.prototype = {
  constructor: jQuery,
  init: function(selector) {
    // 在jquery中 内部封装了一个Sizzle引擎（封装了无数正则表达式）

    // 1.获取页面中所有的元素
    // 系统支持标签遍历，支持class，id的标签查找
    const elements = document.querySelectorAll(selector);
    // 仅支持固定的TagName
    // const elements = document.getElementsByTagName(selector);

    // 为了让这些元素可以在css方法中进行访问，所以需要把这些元素放在对象上面进行传递
    // this.elements = elements;

    // jQuery为了后续操作的方便，将这些获取到的DOM元素全部放在对象身上，让自己变成一个数组一样，可以使用for进行遍历，我们把这种对象特性称之为【伪数组】
    // 实现把所有DOM元素都添加到自己身上
    for (let i = 0; i < elements.length; i++) {
      // ele: DOM元素
      var ele = elements[i];
      this[i] = ele;
    }

    this.length = elements.length;
  },
  css(name, value) {
    for(let i = 0; i < this.length; i++) {
    	let element = this[i];
    	// 设置样式
    	element.style[name] = value;
  	}
  }
}

// 此时创建的jQuery是init构造函数的实例
// css方法在jQuery.prototype对象中
// --> 为了让jQuery对象可以访问到css方法
// 		--> 让init的原型继承自jQuery.prototype
jQuery.fn.init.prototype = jQuery.fn

// ---> 1.创建了一个init的对象
// ---> 2.执行css方法
//    ---> 找对象本身有没有css方法，并没有
//    ---> 找对象的原型： init.prototype ---> jQuery.prototype
//    ---> 发现jQuery.prototype中有一个css方法

$("div").css("color", "red")
$(".header").css("color", "red")
$("#header").css("color", "red")
```



