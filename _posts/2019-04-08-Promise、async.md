---
layout: post
title: "Promise、async"
tags: [JavaScript]
comments: true
---

Promise、async

[^1]: https://www.jianshu.com/u/f6c54f846755

### Promise：用来解决回调地狱的问题(ES6语法)

##### 解释【回调地狱】

```javascript
// 解释【回调地狱】
$.get("/getUser", function(res) {
  $.get("/getUserDetail", function(res) {
  	$.get("/getUserAcccount", function(res) {
  		$.get("/getBooks", function(res) {
  			// ...
      })
    })
  })
})
```



##### Promise 基础

```javascript
function fn() {
	// 把异步操作封装在一个Promise对象中
	// (构造函数) new来创建一个Promise的实例，传入一个参数： 函数
	// 函数存在两个参数resolve, reject
  return new Promise(function(resolve, reject) {
  	// 把需要执行的异步操作放在这里
    setTimeout(() => {
      console.log("Hello");
      // 如何告诉外界，执行结束了 --> 执行resolve
      resolve();
    }, 1000) 
  })
}

// 调用函数，执行异步操作
// fn()

// 存在问题，不知道异步操作什么时候结束
fn().then(res=> {
	// 执行到下一步
  console.log("下一步");
  
  fn().then(res=> {
    // 执行到第二步
    console.log("第二步");
  })
})

// Hello
// 下一步
// Hello
// 第二步
```



##### Promise 解决回调地狱

```javascript 
function f1() {
  return new Promise(function(resolve, reject) {
    // 把需要执行的异步操作放在这里
    setTimeout(() => {
      console.log("第一步");
      // 异步操作执行完，必须告诉外界执行完，不然不会执行then
      // 调用relsove就是为了调用then的回调
      // 下一步操作也就是写在then的回调中  
      resolve();
    }, 1000)
  })
}

function f2() {
  return new Promise(function(resolve, reject) {
    // 把需要执行的异步操作放在这里
    setTimeout(() => {
      console.log("第二步");
      // 异步操作执行完，必须告诉外界执行完，不然不会执行then
      // 调用relsove就是为了调用then的回调
      // 下一步操作也就是写在then的回调中  
      resolve();
    }, 1000)
  })
}

f1().then(res => {
	// 返回一个promise对象
	return f2();
}).then(res=>{ // 链式调用
  console.log("完成");
})

// 第一步
// 第二步
// 完成

f1().then(res => {
	// 返回一个promise对象
	return f2();
}).then(res=>{ 
  return f1();
}).then(res=>{
  return f2();
}).then(res=>{
  console.log("完成");
})

// 第一步
// 第二步
// 第一步
// 第二步
// 完成
```



##### Promise 传参

```javascript
function getUser() {
  return new Promise(resolve => {
    $.get("/getUser", function(res) {
    	// res 服务器返回的数据
    	// resolve告诉外界异步操作执行结束
    	// 把数据传到下一步操作中
    	resolve(res);
  	})
  })
}

getUser().then(res => {
  // res 表示上一步异步操作返回的参数值，即服务器获取的数据
})
```



##### Promise 错误处理

```javascript
function getUser() {
  return new Promise((resolve, reject) => {
    $.ajax({
      url: "/getUser",
      success(res) {
        // 成功获取数据
        resolve(res); // 表示异步操作是成功的
      },
      error(res) {
        // res表示错误信息
        // 如果失败执行error方法
        // 通过执行reject把错误信息传递到外界
        reject(res); // 表示异步操作是失败的
      }
    })
  })
}

// 第一种处理错误的方式
getUser().then(res => {
  // res 表示请求成功的时候获取到的数据
}, rej => {
  // rej 表示失败的错误信息
})

// 第二种处理错误的方式 --> 推荐
getUser().then(res => {
  // res 表示请求成功的时候获取到的数据
}).catch(rej => {
	// 这里也可以获取到错误信息
  // rej 表示失败的错误信息
})

// 上面两种错误处理方式，更加推荐第二种
// 第二种方式更强大地方在于：
// 不仅仅可以捕获到reject传递的参数，还可以捕获到成功的回调中发生的错误

// 多层异步中使用catch
getUser().then(res => {
  // res 表示请求成功的时候获取到的数据
}).then(res => {
  // ...
}).then(res => {
  // ...
}).catch(rej => {
	// 这里也可以获取到错误信息
  // rej 表示失败的错误信息
})

// 例子
new Promise((resolve, reject) => {
  setTimeout(()=> {
    console.log("第一步");
    resolve("第一步完成");
  })
}).then(res => {
  console.log(res); // 第一步完成
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log("第二步");
      reject("第二步失败");
    })
  })
}).then(res => {
  // 并不会执行到这里
  console.log("第二步完成");
}).catch(rej => {
  console.log(rej); // 第二步失败
})

// 第一步
// 第一步完成
// 第二步
// 第二步失败
```



##### async(ES8)

async其实是一个Promise的语法糖

```javascript
// 认识async
function f1() {
  return new Promise(resolve => {
    setTimeout(()=>{
      console.log("Hello");
      resolve();
    }, 1000)
  })
}

f1().then(res => {
  console.log("第二步");
})

// 你好
// 第二步

// async 实现方式
(async function() {
	// await 表示这行代码是一个异步操作
	// --> 这里的异步操作执行完毕其实就是resolve
  await f1();
  // 下面的代码会在这个异步操作之后执行
  console.log("第二步");
  await f1();
  await f1();
  console.log("第三步");
})()

// Hello
// 第二步
// Hello
// Hello
// 第三步
```



##### async处理返回值

```
function q() {
  return new Promise((resolve) => {
    setTimeout(() => {
      console.log("Hello");
    }, 1000)
  })
}

(async function() {
  const res = await q();
  console.log("第一次：", res); // 第一次：Hello
  
  const res1 = await q();
  console.log("第二次：", res); // 第二次：Hello
  
  const res2 = await q();
  console.log("第三次：", res); // 第三次：Hello
})()

// 第一次：Hello
// 第二次：Hello
// 第三次：Hello

// 注意，await必须在async的函数内部

var o1 = {
  say: async () => {
  	console.log("say function");
    const res = await q();
    console.log(res);
  },
  run: async function() {
  	console.log("run function");
    await q();
  }
}

// 需求，先执行完毕say，在执行完毕run
var fn = async function(){
  await o1.say()
  await o1.run()
}

// say function
// Hello
// run function
// Hello
```



##### async错误处理

```javascript
function q() {
	return new Promise((resolve, reject) => {
    setTimeout(()=>{
      reject("Error Message");
    }, 1000)
	})
}


(async function() {
  try {
    const res = await q();
	  console.log(res);
  } catch(e) {
    console.log(e); // Error Message
  }
})

// Error Message
```





