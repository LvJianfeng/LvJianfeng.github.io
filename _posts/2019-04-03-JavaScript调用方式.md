---
layout: post
title: "JavaScript的几种调用方式"
tags: [JavaScript]
comments: true
---

[^1]: <https://www.jianshu.com/u/f6c54f846755>

1.函数调用

```javascript
// 例子
function Person(name) {
	// this 指向的是 window
  this.name = name;
}

Person.prototype = {
  constructor: Person,
  say: function() {
    console.log(this.name);
  }
}

// 函数调用
Person(name)  

// 例子
function Student(age) {
  this.age = age
}

// 函数调用 
Student(22);
```



2.方法调用

```javascript
// 例子
function Person(name) {
	// this 指向的是 window
  this.name = name;
}

Person.prototype = {
  constructor: Person,
  say: function() {
    console.log(this.name);
  }
}

var p1 = new Person();
// 方法调用
p1.say();
```



3.构造函数   



4.上下文调用： ```call```, ```apply```, ```bind```

```javascript
function f1() {
  console.log(this);
}
// call方法的第一个参数决定了函数内部的this的值
f1.call([1, 2, 3]);
// or
f1.call({ name: 'T1', age: 22})
// or
f1.call(1) // -> new Number(1)
f1.call("abc") // -> new String("abc")
f1.call(true) // -> new Boolean(true)
// 总结：
// 1.如果是一个对象类型，函数内部的this指向该对象
// 2.如果是undefined，null，函数内部的this指向window
// 3.如果是数字，this指向new Number(1)
// 4.如果是布尔，this指向new Boolean(true) 

// 上述代码都可以将call替换为apply
// call 和 apply都可以改变函数内部this的值，不同的是传参形式不同
function toString(a, b, c) {
  console.log(a + '' + b + '' + c)
}

toString.call(null, 1, 2, 3) // 1 2 3
// or
toString.apply(null, [1, 2, 3]) // 1 2 3
```

```javascript
var obj = { 
	age: 18,
	run: function() {
    console.log(this); // this.obj
    setTimeout(function() {
      // this -> window
      console.log(this.age); // undefined
    }, 50)
	}
}

obj.run()

// 以前的方式
var obj = { 
	age: 18,
	run: function() {
    var _that = this;
    setTimeout(function() {
      // this -> window
      // _that -> obj
      console.log(_that.age); // 18
    }, 50) 
	}
}

obj.run()

// bind
var obj1 = { 
	age: 18,
	run: function() {
    console.log(this); // this.obj
    setTimeout(function() {
      // this -> window
      console.log(this.age); // this.obj
    }.bind(this), 50)
    // 通过执行了bind的方法，匿名函数本身没有执行，只是改变了该函数内部的this值，指向obj
	}
}

obj1.run()

// bind 基本用法
function speed() {
  console.log(this.seconds);
}
// 执行bind方法之后，产生一个新函数，这个新函数里面的逻辑和原来的还是一样
// 唯一不同的是this指向了{ seconds: 100 }
var speedBind = speed.bind({ seconds: 100 })

speedBind();

// --->

(function eat() {
  console.log(this.seconds)
  // this 指向了 { seconds: 200}
}).bind({ seconds: 200} ) // 200

var obj2 = {
  name: '西瓜',
  drink: (function() {
    // this 指向了 { name: '橙汁' }
    console.log(this.name);
  }).bind({name: '橙汁'})
}

obj.drink(); // 橙汁

var p10 = {
  height: 88,
  run: function() {
  	// this
    setInterval((function() {
      	console.log(this.height); // 88
    }).bind(this), 50)
  }
}

p10.run(); // 88
```

<script src="https://gist.github.com/mmistakes/43a355923921d22cd993.js"></script>