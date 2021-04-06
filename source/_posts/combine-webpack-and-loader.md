---
title: 将手写Webpack和手写loader结合使用
author: Indexsarrol
date: 2020-12-25 19:11:58
categories: 
- Webpack
tags:
- Webpack
---
## 将手写Webpack和手写loader结合使用

​		在之前的文章中，我们手写了一个`Webpack`和独立的实现了`style-loader`和`less-loader`，在这里呢，我们准备把两个手写案例结合到一起，看看是否能够用我们手写的`Webpack`通过我们编写的`loader`来打包文件。

​		让我们新建一个名称为`webpackUseSelfLoader`的文件夹，首先`npm init`一下，我们会获取到`package.json`的文件，然后我们按照之前的文件目录分别创建好我们的文件。然后安装一下`less`。我们在`index.js`文件中引入一下之前新建好的`index.less`文件。
>
<!-- more -->

```less
// index.less
html, body {
    width: 100%;
    height: 100%;
    div {
        width: 100px;
        height: 100px;
        background: red
    }
}
```
```js
// index.js
require('./index.less');
```

​		紧接着，我们将`webpack.config.js`中的`module`模块，先注释，然后将我们之前写好的`jspack`放入到当前项目的`node_modules`中，稍微等待一下初始化加载过程后，我们进入`node_modules/jspack`，然后打开命令行，输入命令：`npm link`。



​		接下来我们来试一下能否打包成功，先修改`index.js`中的代码，然后再项目的根目录中打开控制台输入：`npx jspack`

```js
// index.js
// require('./index.less');
console.log('jspack 已经打包结束')
```

 ​		如果上述打包没有问题，就可以进入我们的下一步了。首先我们将`webpack.config.js`中的`module`注释放开，然后来思考一下这边的逻辑：如果我们读取到的是`js`文件，则直接返回代码交由`jspack`打包即可；如果我们读取到的不是`js`文件，则需要交由`loader`去处理过后，再交给`jspack`来进行打包；所以我们需要在读取文件的时候去做这个判断才可以。
>

​		我们找到之前写的`Complier.js`找到`getSource`方法，我们进行如下改造：

```js
getSource(path) { // path为./index.less
    const content = fs.readFileSync(path, "utf8");
    // 1.需要判断当前文件是否是js文件，我们可以通过path的后缀去判断
    // 1.1 我们需要获取到module内配置的规则
    const rules = this.config.module.rules; // 因为是数组，我们需要遍历一下
    rules.forEach((rule) => {
        // 1.2 获取到的rule为{ test: /\.less$/, use: [.....] }，可以使用结构取出校验正则和需要使用的loader
        const { test, use } = rule;
        // 1.3 进行判断
        if (test.test(path)) {
            // 1.4 如果验证通过，则需要使用loader处理，然后loader是从右至左，从下至上来使用的，所以这里需要我们去倒序遍历
            for(let use.length - 1; i >=0 ; i++) {
                // 1.5 我们需要在这里拿到每个loader的方法，然后将我们获取到的content传递进去
                const loader = require(use[i].loader);
                if (typeof loader === 'function') {
                    content = loader(content);
                } else {
                    throw new Error(`${use[i].loader}获取失败`)
                }
                
            }
        }
    })
    return content
}
```

​		接着我们可以试一下将`index.js`中的代码变为引入`index.less`文件，再次进行打包。这个时候会发现我们打包成功了，然后生成一个`index.html`并将打包过后的代码引入，我们会发现样式已经生效了，到此。自定义`webpack`和自定义`loader`就到此结束了。