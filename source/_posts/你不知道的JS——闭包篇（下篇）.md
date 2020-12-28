---
title: 你不知道的JS——闭包篇（下篇）
author: Indexsarrol
date: 2020-12-28
categories: 
- 你不知道的JS
tags:
- JavaScript
- JS引擎
- 闭包
---

## 写在前面

在上篇 [你不知道的JS——闭包篇（上篇）](/2020/12/28/你不知道的JS——闭包篇（上篇）/)我们事先引入了一个立即执行函数的概念，在本篇中我们将详细的介绍以下闭包：

## 定义

> 由两个函数或多个函数嵌套而成，内部的函数被保存到了外部函数，使得在外部作用域下能够访问到内部的变量，这就形成了闭包。闭包会导致原有的作用域链不释放，从而造成内存泄漏。
>

<!-- more -->

## 为什么会出现闭包

在我们知道了闭包的定义之后，我们就来说说为什么会出现这个东西，这个时候我们要联系之前所说的作用域以及作用域链的知识了。我们用一个简单的闭包代码边看边解释：

```js
function a() {
    var num = 1;
    function b() {
        num = num + 1;    
        console.log(num); // 2
    };
    return b;
}
var test = a();
test();
```

- 首先声明了一个函数`a`，这个时候在作用域链上压入了一个全局执行期上下文`GO`，如下图：

  ![image-20201228140923266](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20201228140923266.png)

- 然后执行`a`函数，这个时候在原有的作用域链上压入`a`函数的执行期上下文`aAO`，如下图：

  ![image-20201228141349464](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20201228141349464.png)

- `a`函数执行的时候出发了`b`函数的定义，`b`函数站在`a`函数基础上生成`a`函数留给它的成果，也就是`a`函数的`AO`以及全局的`GO`，我们的图可以这么画：

  ![image-20201228141414658](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20201228141414658.png)

- 因为b函数仅是定义，并没有被执行，然后我们跳过`function b() {}` 先看`return b`，这个时候b函数被保存到了外部，此时意味着`a`函数执行完成，此时`aAO`函数的作用域链弹出，相当与剪短了`a`函数作用域与`aAo`的连线，因为`a`函数把连线剪短了，但是`b`函数还保留着对`aAO`的引用：如下图所示：

  ![image-20201228141442616](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20201228141442616.png)

- 这个时候`b`函数被保存到了外部，执行`b`函数，在作用域链中压入`b`函数的`AO`，会执行`num = num + 1`。先会去找`b`的执行期上下文，但是`b`没有变量`num`， 就会去找`aAO`，找到后使`num`自增。如图所示：

  ![image-20201228141522075](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20201228141522075.png)

  

以上就是闭包生成的过程。

## 经典案例（打印0~9问题）

```js
function test() {
    var arr = [];
    for (var i = 0; i < 10; i++) {
        arr[i] = function() {
            console.log(i);        
        }    
    }
    return arr;
}
var demo = test();
for (var j = 0; j < 10; j++) {
    demo[j]()
}
```

这道题也算是闭包的经典案例了，在视觉看来，这里的执行应该打印0~9，但是事实却不尽人意，他的结果是10个10，这是为什么呢？

> 原因：首先先执行`test`函数，申明了一个`arr`数组，将10个函数放到`arr`里面，注意：此时该匿名函数并没有执行！！ 然后把`arr`数组给保存到了外部，然后再依次执行函数，这时，i已经变成了10，所以打印出来之后都是10！

如何解决呢？还记得我们在上篇说的立即执行函数吗？这个时候就派上用场了，我们再来回顾一下立即执行函数的特点吧，第一声明过后会立即执行 函数， 第二，执行过后直接销毁。那么针对这个案例我们需要怎么去改呢？请看下面的代码：

```js
// 第一种方式
function test() {
    var arr = [];
    for (var i = 0; i < 10; i++) {
        (function(j){
            arr[j] = function() {
               console.log(j); // 0 1 2 3 4 ... 9       
            }        
        }(i))
           
    }
    return arr;
}
var demo = test();
for (var j = 0; j < 10; j++) {
    demo[j]()
}

// 第二种方式，直接使用ES6的 let 去声明变量，从而形成块状作用域
function test() {
    var arr = [];
    for (let i = 0; i < 10; i++) {
        arr[i] = function() {
            console.log(i); // 0 1 2 3 4 ... 9       
        } 
    }
    return arr;
}
var demo = test();
for (var j = 0; j < 10; j++) {
    demo[j]()
}
```

> 这里就是用闭包去解决闭包的问题了，我们在`arr[i]`赋值的时候外层套了一层立即执行函数，当循环遍历的时候会立即执行这个函数，此时该立即执行函数与`arr[j] = function（） {}` 也形成了闭包，在立即执行函数传入`i`，在函数内部接受，此时j是拿的外层立即执行函数的`AO`，访问到了`i`，所以这里打印就变成了0~9。

## 阿里面试题

这是来自阿里的一道面试题，编写一个函数，给每个`<li></li>`添加点击事件，打印当前点击的索引

```html
<ul>
    <li></li>
    <li></li>
    <li></li>
    <li></li>
</ul>
```

我们认为的方法：

```js
// 错误版本
function test() {
    var oLis = document.getElementsByTagName('li');
    for (var i = 0; i < oLis.length; i++) {
        oLis[i].addEventListener('click', function(){
            console.log(i); // 4 4 4 4
        });
    }
}
test();
```

这时候我们看到打印出来的是4个4，并没有得到我们想要的结果，那现在我们把代码给稍微换个表达方式看看。

```js
function test() {
    var oLis = document.getElementsByTagName('li');
    for (var i = 0; i < oLis.length; i++) {
        oLis[i].onclick = function(){
            console.log(i); // 4 4 4 4
        }
        // oLis[i].addEventListener('click', function(){
        //    console.log(i); // 4 4 4 4
        // });
    }
}
test();
```

我们可以看到其实`addEventListener`等价于`.onclick`，而这个`onclick`是直接绑定到`<li></li>`上的，就像这样：`<li onclick={function() { }}></li>`，这个时候我们看到此时这个函数已经被保存到了外部，这下知道为什么会触发闭包了！

解决方案：

```js
function test() {
    var oLis = document.getElementsByTagName('li');
    for (var i = 0; i < oLis.length; i++) {
        (function (j){
             oLis[j].onclick = function(){
                 console.log(j); // 0, 1, 2, 3
             }      
        }(i))
        
        // oLis[i].addEventListener('click', function(){
        //    console.log(i); // 4 4 4 4
        // });
    }
}
test();
```

这样我们利用了闭包解决了无意之间触发的闭包问题了，闭包到这里就结束了。