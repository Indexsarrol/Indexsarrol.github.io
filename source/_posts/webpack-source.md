---
title: Webpack原理以及手写Webpack
author: Indexsarrol
date: 2020-12-23 19:11:58
categories: 
- Webpack
tags:
- Webpack
---
## Webpack如何打包文件

在我们正式开始手写`webpack`之前，我们先来看看`webpack`是如何打包文件的，在之前的文章中，我们说过，如果想利用`webpack`来对我们的项目进行打包，可分为三个步骤：

1. 我们需要安装`webpack`以及`webpack-cli`，这里需要我们注意一点的是，`webpack-cli` 建议安装`3.x`的版本，否则会出现其他异常问题(因为`webpack5`刚出来，可能是兼容没有做好)；

2. 我们需要指定配置文件名称需为`webpack.config.js`，在其内部进行`webpack`的配置；

3. 最后我们通过命令 `$ npx webpack` 来对指定文件进行打包。

所以，综上所述`webpack`就是一个工具模块，提供了`webpack`指令，所以要想实现`webpack`必须先实现一个工具模块。

<!-- more -->

## 如何实现一个基于Node的工具模块

1. 首先，我们先新建一个目录，名称为`jswebpack`，在该文件夹内部新建一个`node_modules`文件夹，在该文件夹下新建`jspack`目录作为工具模块，然后打开控制台，输入以下命令进行初始化：

   ```shell
   $ npm init
   ```

2. 在`jspack`文件夹下，新建一个`bin`目录，内部新建一个`index.js`文件，在`index.js`中添加以下代码：

   ```javascript
   #!/usr/bin/env node
   console.log('hello world！')
   ```

   然后在`package.json`中添加如下代码：

   ```json
   {
     "name": "jspack",
     "version": "1.0.0",
     "description": "",
     "main": "index.js",
     "scripts": {
       "test": "echo \"Error: no test specified\" && exit 1"
     },
     ++"bin": {
     ++ "jspack": "bin/index.js"
     ++ },
     "author": "",
     "license": "ISC",
     "dependencies": {
       "ejs": "^3.1.5"
     }
   }
   ```

3. 最后我们在`jspack`文件夹对应的命令行中输入：

      ```sh
      $ npm link
      ```

到此，我们可以通过在控制台 `npx jspack` 命令，如果控制台输出'`hello world`'则表示我们的工具模块已经实现了。

## 配置Webpack

由上，我们实现了一个工具模块，接下来，我们来配置一个简易版的`webpack.config.js`：

```javascript
// webpack.config.js
const path = require("path");
module.exports = {
    devtool: "none",
    mode: "development",
    entry: "./src/index.js",
    output: {
        filename: "index.js",
        path: path.resolve(__dirname, "bundle")
    }
};
```

然后我们在项目根目录下新建`src`文件夹，在内部新建`index.js`文件，用于编写项目打包内容。

## 分析Webpack打包后的文件

我们在打包之后，可以去观察以下打包过后的源码，有的同学可能会说，那玩意太难懂了，不知道什么意思，其实我们只要把注释和一些没必要的代码干掉，我们就得到了如下的代码：

```js
(function (modules) { // webpackBootstrap
    // 1.初始化一个缓存
    var installedModules = {};
    // 2.自己实现了一个require方法
    function __webpack_require__(moduleId) { // "./src/index.js"
        // 2.1判断缓存中有没有当前需要使用的模块
        if (installedModules[moduleId]) {
            return installedModules[moduleId].exports;
        }
        // 2.2自己创建一个缓存
        var module = installedModules[moduleId] = {
            i: moduleId,
            l: false,
            exports: {}
        };
        // Execute the module function
        modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
        // Flag the module as loaded
        module.l = true;
        // Return the exports of the module
        return module.exports;
    }
    /* 以下代码我们无需理会 */
	/*
    __webpack_require__.m = modules;

    __webpack_require__.c = installedModules;

    __webpack_require__.d = function (exports, name, getter) {
        if (!__webpack_require__.o(exports, name)) {
            Object.defineProperty(exports, name, {enumerable: true, get: getter});
        }
    };

    __webpack_require__.r = function (exports) {
        if (typeof Symbol !== 'undefined' && Symbol.toStringTag) {
            Object.defineProperty(exports, Symbol.toStringTag, {value: 'Module'});
        }
        Object.defineProperty(exports, '__esModule', {value: true});
    };
    __webpack_require__.t = function (value, mode) {
        if (mode & 1) value = __webpack_require__(value);
        if (mode & 8) return value;
        if ((mode & 4) && typeof value === 'object' && value && value.__esModule) return value;
        var ns = Object.create(null);
        __webpack_require__.r(ns);
        Object.defineProperty(ns, 'default', {enumerable: true, value: value});
        if (mode & 2 && typeof value != 'string') for (var key in value) __webpack_require__.d(ns, key, function (key) {
            return value[key];
        }.bind(null, key));
        return ns;
    };
    __webpack_require__.n = function (module) {
        var getter = module && module.__esModule ?
            function getDefault() {
                return module['default'];
            } :
            function getModuleExports() {
                return module;
            };
        __webpack_require__.d(getter, 'a', getter);
        return getter;
    };

    __webpack_require__.o = function (object, property) {
        return Object.prototype.hasOwnProperty.call(object, property);
    };

    __webpack_require__.p = "";
*/
    return __webpack_require__(__webpack_require__.s = "./src/index.js");
})
({
    "./src/index.js": // key
        (function (module, exports, __webpack_require__) { // value
            console.log('手写webpack')
        })
});
```

如果我们改变了`config`配置文件，我们再重新打包几次观察以下，其实变化的只有以下部分:

```js
(function (modules) {
    // ...其他代码
    return __webpack_require__(__webpack_require__.s = "./src/index.js"); // 路径变化
})({
    "./src/index.js": // key 变化
        (function (module, exports, __webpack_require__) { 
            console.log('手写webpack') // value 变化
        })
});
```

所以，我们通过观察打包之后的代码我们可以清楚的知道，其实我们只需要两个变量即可，一个路径和内部代码。所以我们可以把这些变量放到一个对象中，对象的`Key`就是入口文件路径，`Value`就是入口文件内部的代码。其他剩下的代码，我们可以利用`EJS`当作模板去传到生成的文件中。

首先，我们需要安装`ejs`：

```shell
$ npm install ejs -S
```

然后修改内部变量为`<%-variable%>`，并将该模板后缀改为`.ejs`，然后保存在lib文件夹下:

```js
(function (modules) {
    // ...其他代码
    return __webpack_require__(__webpack_require__.s = "<%-entryId%>"); // 路径变化
})({
    "<%-entryId%>": // key 变化
        (function (module, exports, __webpack_require__) { 
            <%-modules[entryId]%> // value 变化
        })
});
```

## 开始手写Webpack
接下来，我们就正式进入手写简易`webpack`。还记得我们在`jspack`中`bin`目录下的`index.js`文件吗？这个就是我们`jspack`的入口文件，让我们回想以下`Webpack`如何打包文件，我们应该先去读取`webpack.config.js`的配置项：
>

​	那么`index.js`中应该这么写，首先我们肯定是要操作某一个文件的，所以我们就需要用到`Node`中的`path`模块。若想读取文件内容我们则要知道该文件的路径是什么，代码如下：

```javascript
// bin/index.js
#! /usr/bin/env node
const path = require('path');
const configPath = path.resolve(process.cwd(), 'webpack.config.js');
const configContent = require(configPath);
```

那么既然配置项我们获取到了，那就需要有一个函数或者方法来充当编译器，所以我们在`jspack`下新建一个`lib/Complier.js`：

```js
// lib/Complier.js
class Complier {
    constructor(config) {
        this.config = config
    }
}

module.exports = Complier
```

声明好了编译器之后，我们将 `Complier` 引入到 `bin/index.js` 中，并且实例化：

```js
// bin/index.js
#! /usr/bin/env node
const path = require('path');
const Complier = require('../lib/Complier.js');
const configPath = path.resolve(process.cwd(), 'webpack.config.js');
const configContent = require(configPath);
// 进行实例化,并将获取到的配置文件传给Complier类
const complier = new Complier(configContent);
// 当使用命令行 npx jspack 时会调用 Complier 类中的run方法
```

所以这个时候我们再返回到 `lib/Complier.js` 处理编译的逻辑：

```js
// lib/Complier.js
const fs = require('fs');
const path = require('path');
const ejs = require('ejs');
class Complier {
    constructor(config) {
        this.config = config;
        this.modules = {}; // 用于构建key：value对象
    }
    run() {
        this.buildModule(); // 开始构建
        this.commitFile(); // 获取模板并将其变量匹配之后生成打包之后的目录
    }
    // 获取需要打包的文件路径以及源码，并拼成key：value格式
    buildModule() {
        const entryPath = this.config.entry; // 获取需要打包文件的路径,作为Key
        const code = fs.readFileSync(entryPath, 'utf8'); // 获取需要打包文件内容，作为Value
        this.modules[entryPath] = code;
    }
    commitFile() {
        // 1.获取ejs模板
        // 1.1获取ejs路径
        const ejsPath = path.resolve(__dirname, 'main.ejs');
        // 1.2根据路径来啊过去ejs模板
        const ejsTemplate = fs.readFileSync(ejsPath, 'utf8');
        // 1.3将ejs模板与变量相匹配
        const matchVariable = {
            entryId: this.config.entry,
            modules: this.modules
        };
        const sourceCode = ejs.render(ejsTemplate, matchVariable);
        // 2.将获取到的模板输出至指定文件夹
        // 2.1 获取从配置项中输出的文件夹目录,并且需要判断该目录是否存在
        const outputDir = this.config.output.path;
        if (!fs.existsSync(outputDir)) {
            fs.mkdirSync(outputDir);
        }
        // 2.2 新建一个文件，名称与配置项中的pathname保持一致
        const outputPath = path.resolve(outputDir, this.config.output.pathname);
        // 2.3 写入文件
        fs.writeFileSync(outputPath, sourceCode)
    }
    
}

module.exports = Complier;
```

这个时候，我们的编译器的功能就完成了，接下来我们来看看效果：

![image-20201224205107321](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20201224205107321.png)

这样，我们就编写了一个简易的`webpack`，在下篇文章中我们来看看如何打包多文件。尽请期待吧~