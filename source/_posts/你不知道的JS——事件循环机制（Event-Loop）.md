---
title: 你不知道的JS——事件循环机制（Event Loop）
author: Indexsarrol
date: 2020-12-27 19:11:58
categories: 
- 你不知道的JS
tags:
- JavaScript
- JS引擎
---

## 前言

​		Event Loop，也就是事件循环机制，相信大家都多多少少听过一些，事件循环机制分为浏览器的以及Node环境下的，今天我们就只先针对浏览器来解释。一起来看看吧！

## 定义		

​		事件循环机制简单理解的话就是一种浏览器对代码的执行顺序（还有Node下面也有Event Loop，我们可以先忽略）,我们都知道js的任务可以分为同步和异步：

> 同步：按照主线程来一步一步依次执行代码；

> 异步：把要执行的部分先放到任务队列中，让主线程的任务和任务队列中的任务分开执行。
<!-- more -->

还记得我们之前说过的作用域链吗？在这里它有一个新的名字——调用栈。

这下我们就能组织出来事件循环机制的定义了：**不管是同步还是异步任务，将这些任务放入调用栈执行，执行完之后去检查有无新的任务，如果有，则继续循环执行，如果没有，循环结束。**

## 宏任务（Macro-Task）与微任务（Micro-Task）

​		由上可知，事件循环机制是基于任务来执行的，任务可分成两类：宏任务与微任务，这两种任务有他们自己独有的队列，在我们的业务代码中，常见的宏任务与微任务有：

> 宏任务：`setTimeout`函数、`setInterval`函数、`script`（整体代码）、`I/O`（输入/输出操作）、`UI`渲染等等

> 微任务：`new Promise().then(() => {})`、`async/await`函数等。

## 执行过程

1. 先执行宏任务的同步任务的代码（script标签内的代码）；
2. 遇到宏任务，先把宏任务丢到宏任务的队列中；
3. 检查当前宏任务下有没有微任务，如果有执行微任务，如果没有则该宏任务结束，去执行下一个宏任务；
4. 如果下一个宏任务找到了就继续执行，进行第三步，如果没有则进行下一轮event loop；

这样单靠言语描述可能有些难理解，在这里，我画了一张简易的流程图供大家参考：

​	![image-20201228105407166](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/event-loop2.png)

## 示例

```js
console.log('start');
setTimeout(() => { // 宏任务1
    console.log('children2');
    Promise.resolve().then(() => { // 微任务1
        console.log('children3');
    });
}, 0);
new Promise(function(resolve, reject) {
    console.log('children4');
    resolve('children6');
    setTimeout(function() { // 宏任务2
        console.log('children5');
        // resolve('children6');
    }, 0);
}).then(res => {
    console.log('children7');
    setTimeout(() => { // 宏任务3
        console.log(res)
    }, 0);
});
```

执行顺序：

- 第一步，先执行整体代码，首先打印‘start’；
- 第二步，我们遇到了`setTimeout()`，这是一个宏任务，我们把他先放到宏任务队列里，先不管；
- 第三步，我们遇到了`promise`，因为promise内部函数默认是同步的，所以打印‘children4’;
- 第四步，执行`resolve()`，这个时候我们遇到了.`then()`函数，这是一个微任务，暂时存到微任务队列中；
- 第五步，再次遇到了`setTimeout()`，又是一个宏任务，再次放入宏任务队列中；
- 第六步，整体代码执行完毕，该执行当前任务下面的微任务了，所以执行.then()函数 打印‘children7'，这个时候我们又遇到了`setTimeout()`，先加入宏任务队列中；
- 第七步，现在整体代码的宏任务已经结束，我们该执行宏任务队列中的宏任务1了，打印'children2'，然后我们又遇到了.`then()` 先加入微任务队列，当前宏任务1已经执行结束了，开始执行微任务1，打印'children3'，到此宏任务1执行结束；
- 第八步，执行宏任务2，打印'children5', 并没有属于当前宏任务的微任务，到这里宏任务2执行结束；
- 第九步，执行宏任务3，打印'children6',并没有属于当前宏任务的微任务，到这里宏任务3执行结束;

到此所有任务全部结束。

按照上面的执行结果，可以得到：

```js
// start
// children4
// children7
// children2
// children3
// children5
// children6
```

接下来我们再来看一个例子：

```js
console.log('script start');
async function async1() {
    console.log('async1 start');
    await async2();
    console.log('async1 end'); // 微任务1
};
async function async2() {
    console.log('async2 end');
};
async1()
setTimeout(() => {
    console.log('setTimeout');
}, 0)
new Promise((resolve, reject) => {
    console.log('promise start');
    resolve()
}).then(() => console.log('promise end'))// 微任务2
console.log('script end')
```

如上述代码所示，我们加入了`async/await`，唯一需要我们注意的就是`await()` 之后的代码加入微服务队列；我们再来一步一步分析一下：

- 第一步，先执行整体代码宏任务，打印了'script start';
- 第二步，执行`async1()`，打印了'async1 start'，再执行`async2()`，打印'async2 end'，同时将`await async2()`后面的代码加入微任务队列中；
- 第三步，遇到了`setTimeout()`，加入宏任务队列；
- 第四步，执行`new Promise`，打印'promise start'，然后执行.`then()`函数加入微服务队列，执行`console.log('script end')`， 打印'script end'；
- 第五步，执行微任务1，打印'async1 end'，再次执行微任务2，打印'promise end'；
- 第六步，执行宏任务队列中的`setTimeout()`，打印'setTimeout';到此整个任务执行结束。

上述代码的输出顺序如下：

```js
// script start
// async1 start
// async2 end
// promise start
// script end
// async1 end
// promise end
// setTimeout
```

到此就是基于浏览器的事件循环机制，当然还有基于Node的版本，后续会放进来。