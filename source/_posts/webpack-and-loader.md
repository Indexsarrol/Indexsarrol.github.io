---
title: webpack原理之手写loader源码
author: Indexsarrol
date: 2020-12-24 19:11:58
categories: 
- Webpack
tags:
- Webpack
---
## 自定义loader

​	在我们正式开始手写`loader`之前，我们先来看看`loader`到底是个什么东西：

​	`loader`：`loader`的本质其实就是一个导出为函数的一个js模块，该函数接收一个参数`source`，`source`的内容，`compiler` 需要得到最后一个 `loader` 产生的处理结果。这个处理结果应该是 `String` 或者 `Buffer`（被转换为一个 `string`），代表了模块的 `JavaScript` 源码。另外还可以传递一个可选的 `SourceMap` 结果（格式为 `JSON` 对象）。
<!-- more -->
那么`loader`分为两种：同步`loader`和异步`loader`。所谓同步`loader`表示只要处理单个结果，使用同步模式然后直接返回结果；如果我们需要处理多个结果，我们则需要使用内置`api-->this.async()，const  callback = this.async()；`其内部原理就只是一个闭包，所以`callback`是一个函数，它接收4个参数：

第一个参数必须是 `Error` 或者 `null`；

第二个参数是一个 `string` 或者 `Buffer`；

可选的：第三个参数必须是一个可以被这个模块解析的 `source map`；

可选的：第四个选项，会被 `webpack` 忽略，可以是任何东西（例如一些元数据）。

```js
// 同步loader
module.export = function(source) {
    const res = do something...
    return res;
}

// 异步loader
module.export = function(source) {
    const callback = this.async();
    setTimeout(function() {
        const res = do something...
        callback(null, res);
    }, 3000);
}
```

## 编写一个简易的loader

 ​	接下来我们来编写一个简单的`loader`，用于将代码中的“monday”替换成“sunday”，首先我们新建一个名称为`loader`的目录，再新建一个`ReplaceLoader.js`的`loader`：
>

```js
// ReplaceLoader.js
module.export = function(source) {
    const result = source.replace(/monday/g, "sunday");
    return result;
}
```

 ​	这样我们的简易版的`loader`就结束了，接下来我们再在`webpack.config.js`中使用这个`loader`：
>

```js
// webpack.config.js
module: {
    rules: [
        {
            test: /\.js$/,
            use: [{
                loader: path.resolve(__dirname, './loader/ReplaceLoader.js')
            }]
        }
    ]
}
```

 ​	接下来我们打包试一下`npm run dev`：


```js
// 打包之后的index.js
({

/***/ "./src/index.js":
/*!**********************!*\
  !*** ./src/index.js ***!
  \**********************/
/*! no static exports found */
/***/ (function(module, exports) {

console.log("www.sunday.com"); // 由此我们可以看到打包过后变成了sunday说明我们的loader是起作用了
// require('./index.less')

/***/ })

/******/ });
```

## 	loader参数处理

​	我们在配置`webpack`的配置项中，当使用到了`loader`之后，我们一般会在`loader`使用内部，添加`options`配置项来传递额外的参数。那么我们在编写`Loader`的时候如何拿到通过`options`传递过来的参数呢?其实有两种方法：

第一种：我们用过`loader`函数中提供的`this`来获取，比如我们配置了如下的`options`：

```js
// webpack.config.js
rules: [{
    loader: 'xxx-loader',
    options: {
        name: 'zhangsan'
    }
}]
```

这样我们就可以通过`this.query.name`获取到`options`传递过来的参数；

第二种：我们可以通过第三方模块`loader-utils`获取：

```js
// 首先进行安装
npm install loader-utils -D
// 在当前编写的loader引入
const loaderUtils = require('loader-utils');
// 通过loaderUtils中的getOptions方法获取，注意：一定要把当前作用域中的this传递给该函数。
const options = loaderUtils.getOptions(this); 
// 使用
console.log(options.name);
```

​	在我们配置`options`的时候，当我们配置项不符合要求时，我们希望控制台能够报错，能够准确无误的提示开发者，配置项哪里出错了，这个时候就需要我们对`options`进行一个校验。校验的话，我们需要使用第三方模块 `schema-utils`。使用方法如下：

```js
// 1.先安装schema-utils
npm install schema-utils -D
// 2.在当前编写的loader引入
const validateOptions = require('schema-utils')
// 3.validateOptions为一个函数，接收3个参数分别为：校验规则rules， 需要校验的配置项options，需要校验的loader的名称
// 4.校验规则如下
let rules = {
  type: "object",
  properties: {
    // 需要校验的属性
    name: {
      // 需要校验的属性的数据类型
      type: "string"
    },
    additionalProperties: false
}
validateOptions(schema, options, 'ReplaceLoader');

```

​	通过上述方式配置了之后，我们来修改一下`webpack.config.js`的`options`：

```js
// webpack.config.js
rules: [
    {
        test: /\.js$/,
        use: [{
            loader: path.resolve(__dirname, 'loader/ReplaceLoader.js'),
            // options: {
            //    name: 'wujian'
            // }
            options: {
                age: '18'
            }
        }]
    }
]
// 打包之后我们发现会报错：ValidationError: Invalid configuration object. Object has been initialized using a configuration object that does not match the API schema. -configuration has an unknown property 'age'.

rules: [
    {
        test: /\.js$/,
        use: [{
            loader: path.resolve(__dirname, 'loader/ReplaceLoader.js'),
            options: {
               name: 123
            }
        }]
    }
]
// 打包之后我们发现会报错：Invalid configuration object. Object has been initialized using a configuration object that does not match the API schema. - configuration.name should be a string.
```

​	由上述我们可知，当我们引用了`schema-utils`之后，能够帮我们校验传递的参数类型名称等是否正确，大大的降低了调试难度。

## 	loader导入方式

​	由上面的内容，我们发现了一个问题，我们每次导入`loader`的时候会写很多的代码，但是在`webpack`中，我们导入直接导入当前的`loader`的名称即可，简化了代码的编写量，我们也可以这样吗？答案时肯定的，我们在`webpack`中有一个`resolve`配置，这里面就有个`resolveLoader`的配置项，有两种方式能够帮我们实现直接导入`loader`名称：

```js
// webapck.config.js
// 定义
resolveLoader: {
    // 第一种方式，直接以modules开始，表示，如果要查找loader，先从node_modules中查找，如果找不到，则在指定的loader目录中查找即可；
    modules: ['node_modules', './loader'],
    // 第二种方式，直接给loader一个别名即可  
    alias: {
       ReplaceLoader: path.resolve(__dirname, './loader/ReplaceLoader.js')
    }
}
// 使用
rules: [{
    test: /\.js$/,
    use: [{
        loader: 'ReplaceLoader',
    }]
}]
```

## 实战：手写style-loader和less-loader

​	总算到了手写真实的`loader`环节了，摩拳擦掌，开干！！

 ​	首先我们还是在`loader`文件夹下，新建`style-loader.js`和`less-loader.js`文件，然后我们在新建一个`less`文件，并写点`less`代码。首先我们先来实现`less-loader`，第一步离不开安装：`npm install less -D`，然后我们编写`less-loader`；
>

```js
// less-loader.js
const less = require('less');
module.exports = function(source) {
    // less 提供了一个render方法，接受两个参数，第一个参数为less代码
    // 第二个参数为一个回调函数，这个时候就出现了异步问题，所以我们使
    // 用异步loader的方式编写
    const callback = this.async();
    less.render(source, function(err, data) {
        callback(err, data.css)
    })
}
```

 ​	到此，我们的`less-loader`就编写完了，其实就是调用了一个`less.render()`的方法，配合异步`loader`生成了`css`代码在回传给`webpack`，再经过`style-loader`处理编译后的`css`代码。那么话不多说，开搞`style-loader`，那么`style-loader`是将我们的`css`代码插入到`head`标签下：
>

```js
// style-loader.js
module.exports = function(source) {
    // 注意：这里的source为已经转成的css代码
    // 1.动态的生成style标签
    let style = document.createElement('style');
    // 2.将css代码插入到style标签中
    style.innerHTML = JSON.stringify(source);
    // 3.将style标签插入到head标签中
    document.head.appendChild(style);
    // 4.这个时候我们就会有疑问了，每个loader都需要返回值（String | Buffer）,显然这里将代码变成String更为简单些：
    let style = `
		let style = document.createElement('style');
        style.innerHTML = ${JSON.stringify(source)};
        document.head.appendChild(style);
	`;
    return style;
}
```

 ​	到此，我们的`style-loader`和`less-loader`编写完毕了，是不是so easy，too happy啊！
>

那我们来使用一下吧：

```js
// webpack.config.js
rules: [{
    test: /\.less$/,
    use: [
        {
            loader: path.resolve(__dirname, './loader/style-loader')
        },
        {
            loader: path.resolve(__dirname, './loader/less-loader')
        }
    ]
}]
```

​	这些准备工作完成后，我们开始打包，因为在这边没法展示打包结果，经过我们的打包后发现没有什么问题， `loader`可正常执行。接下来，我们将手写的`webpack`和手写的`loader`放入一起，看看能不能擦出什么火花，一起期待吧！