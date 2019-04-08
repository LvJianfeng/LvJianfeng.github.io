---
layout: post
title: "解答"
tags: [JavaScript]
comments: true
---

记录一些问题的解答

[^1]: https://www.jianshu.com/u/f6c54f846755

### 解答

##### 为什么对象方法中不加this就取的是全局的属性？

```javascript
var age = 18;

var p = {
  age: 10,
  say() {
  	// age 是一个属性
    console.log(this.age); // 10
    
    // age 是一个变量
    /// age这个变量--作用域
    /// 发现age是全局作用域中声明的变量，age = 18
    console.log(age); // 18
  }
}

p.say();
```



##### 声明一个全局变量，使用window？

```javascript
// 是的
window.[name] 也可以用来声明全局变量

// window上有一个name，说明有一个全局变量name
// 如果声明了一个全局变量，name这个变量就变成了window上的属性
```

