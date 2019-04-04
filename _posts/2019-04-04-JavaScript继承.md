---
layout: post
title: "JavaScript继承"
tags: [JavaScript]
comments: true

---

原型链继承 / 拷贝继承 / 原型式继承 / 借用构造函数继承

[^1]: https://www.jianshu.com/u/f6c54f846755

1.原型链继承：prototype

```javascript
var p1 = new Person()

Person.prototype.(method_name) = function() {
  // ...
}

Person.prototype.(method_name_2) = function() {
  // ...
}

p1.(method_name)()
p1.(method_name_2)()

// 确定就是方法很多的时候会造成代码冗余
```



2.原型链继承2：prototype

```javascript
Person.prototype = {
  // constructor 重要！
  constructor: Person,
   (method_name): function() {
     // ...
   }
}

var p1 = new Person()
p1.(method_name)()

/** 注意：
1. 一般情况下，应该先改变原型对象，在创建对象
2.一般情况下，对于新原型，会添加一个constructor属性，从而不破坏原有的原型对象的结构
*/
```

###### ```注意：JavaScript的类继承其实本质还是用原型继承来包装的```



3.拷贝继承（混入继承）

```javascript
var p1 = {age: 18}
var p2 = p1;
p2.age = 20;
// 上面代码p1的age也会被修改了，因为p1,p2是同一对象

// 如果想使用某个对象中的属性，又不能直接修改，就可以创建一个该对象的拷贝
// jquery: $.extend： 编写jquery插件的必经之路
// 拷贝继承（混入继承）
var p1 = {name: 'T1', age: 18}

// 实现拷贝继承
var p2 = {}

for (var key in p1) {
  var value = p1[key];
  p2[key] = value;
}

p2[age] = 22;
// ...
```

```es6``` 提供了<对象扩展运算符>

```javascript
var p1 = {name: 'T1', age: 18}
var p2 = {...p1}
// or
var p3 = {...p1, age: 22}
```



4.原型式继承

```javascript
// 场景：
// 创建一个纯洁的对象
// 创建一个继承自某个父对象的子对象 
var p1 = {name: 'T1', age: 18}
var p2 = Object.create(p1);
```



5.借用构造函数继承

```javascript
function Person(name, age) {
  this.name = name
  this.age = age
} 

function Student(name, age, sex) {
  // 将Person函数内部的this指向Student的实例
	Person.call(this, name, age)
  // or (等价于)
  Person.apply(this, [name, age, sex])
	this.sex = sex
}

var p1 = new Student('T1', 22, '男')
// ...
// 局限性： Person(父类构造函数)的代码必须完全适用于Student(子类构造函数)
```



6.寄生继承

7.寄生组合继承



闭包

1.认识闭包

```javascript
function fn() {
  var a = 5;
  return function() {
    a++;
    console.log(a);
  }
}

var f1 = fn();
f1();   // 6
f1();   // 7
f1();   // 8

// 一般认为函数执行完毕，变量就会释放，但是此时由于js引擎发现匿名函数要使用a变量，所以a变量并不能得到释放，而是把a变量放到匿名函数可以访问到的地方去了
// a变量存在于f1函数可以访问到的地方，此时a变量只能被f1访问
```











