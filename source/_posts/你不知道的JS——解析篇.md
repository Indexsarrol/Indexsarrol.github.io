---
title: 你不知道的JS——解析篇
author: Indexsarrol
categories: 
- 你不知道的JS
tags:
- JavaScript
- JS引擎
---



​		众所周知，JS是单线程的一种语言，所谓单线程就是需要根据代码一行一行去执行并输出结果，如果有一系列的代码，其中有一个因为某种原因导致阻塞，那么后续的模块将无法执行。今天我们就来聊一聊JS的解析方式，知道JS是怎么去一步一步的将我们的代码编译出来的。


​	当我们编写出我们自己逻辑代码时，JS引擎会经历*词法分析、预编译、解释执行*三个阶段：
<!-- more -->
## **词法分析**

​		所谓词法分析就是一个扫描的过程，可以理解成JS引擎拿到这串代码后，大致检查一下有没有一些低级的语法错误等等。

## **预编译**

​		以前我们刚学JS的时候也大致都了解过预编译的过程：1.函数声明提升；2.变量 声明提升；大致意思就是执行之前，会将变量与函数给单独提出来，然后放到该作用域的最顶端，然后进行下一步的解释执行。这两句话确实能解决一部分问题，但是真的能解决所有问题嘛？答案是不能，咱们接着往下看。

```js
function test(a) {
    console.log(a);
    var a = 123;
    console.log(a);
    function a() {}
    console.log(a);
    var b = function () {}
    console.log(b);
    function d() {}
}
test(1)
```

如上代码块，如果我们遇到了这种问题，该怎么去解决呢，以上的那两句话就无法用于这种情况了。这时候就需要我们要了解一下预编译的四个步骤了：

1. **创建AO（Activation Object）执行期上下文（作用域）对象；**
2. **查找形参和变量的声明，作为AO的key，挂载到AO对象上去，并赋值为undefined；**
3. **将实参值和形参统一；**
4. **在函数体里面找函数声明，值赋予函数体。**

接下来我们对照上面的代码，按照这个步骤一步一步的来分析这段代码是如何解释执行的：

- 首先，第一步创建AO对象：

  ```js
  // create Activity Object
  AO {}
  ```

- 第二步，查找形参与变量的声明，可以看到上述代码中，有一个形参a，声明的变量a，以及声明的变量b，并赋值为undefined，那可以有如下写法:

  ```js
  AO {
      a: undefined,
      b: undefined
  }
  ```
  
- 第三步，将实参值和形参值统一：

  ```js
  AO {
      a: 1, // 由undefined 变为实参 1
      b: undefined
  }
  ```

- 第四步，在函数体内找函数声明，有上述代码我们可以看到函数体有a和d，值赋予函数体，于是这个AO又发生了变化了：

  ```js
  AO {
      a: function a() {}, // 由实参 1 变为了a函数体
      b: undefined,
      d: function d() {}
  }
  ```

## 解释执行

```js
function test(a) {
    console.log(a); // function a() {}
    var a = 123;
    console.log(a); // 123
    function a() {}
    console.log(a); // 123
    var b = function () {}
    console.log(b); // function () {}
    function d() {}
}
test(1)
```

## 练习

```js
function test(a, b) {
    console.log(a); // 1
    c = 0;
    var c;
    a = 3;
    b = 2;
    console.log(b); // 2
    function b() {};
    function d() {};
    console.log(b); // 2
}
test(1);
```



```js
var c;
var a = 123;
// 执行到此处a变成了123,这个时候a is not a function 
a();
function a() {
    console.log(a);
    var c = 234;
    c = function() {}
    console.log(c);
    if (a) {
        b = function() {}    
    }
}
console.log(c);
// 执行结果：报错
```

