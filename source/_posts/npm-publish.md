---
title: 手把手教你如何发布属于自己的npm包——react-slider-verify
author: Indexsarrol
date: 2021-01-26 19:11:58
categories: 
- npm 
tags:
- npm
- 发布组件
- React
---

## 写在前面

最近因为公司的工作比较忙以及自己的一些私人的事，导致很长时间没有更新博客的内容了，最近也是因为年底了就准备花点时间在博客上面吧，在npm上发包是我之前一直想做的事，但总是被各种事情打断，正好这几天稍微有点空闲时间，就去找了一些资料，也是成功的发布了属于自己的一个npm包。在此就把过程和总结当作一个笔记来记录一下吧！
<!-- more -->
## 准备

1. 确保当前机器上有`Node`，没有的话就去[下载](https://nodejs.org/zh-cn/download/)安装一下；
2. 确保有一个Github账号，如果没有的话，可以去官网[注册](https://github.com/join?ref_cta=Sign+up&ref_loc=header+logged+out&ref_page=%2F&source=header-home)一个；
3. 确保有一个npm账号，这个是必要条件，可以去官网[注册](https://www.npmjs.com/signup)一下。

在有了以上的三个条件之后，我们就可以正式啦~因为我们是基于React去发布一个组件，所以我们需要到github上创建一个仓库并初始化项目。

## 创建仓库并初始化项目

1. 创建仓库：

   ![image-20210128171431220](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20210128171431220.png)

2. 新建本地仓库：

   ![image-20210128171608789](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20210128171608789.png)

   ```shell
   $ git clone https://github.com/<用户名>/<仓库名>
   $ cd <仓库名>
   ```

3. 初始化项目：

   在当前本地仓库所在目录中输入命令：

   ```shell
   $ npm init -y
   ```

   > 输入完之后，会在目录中生成一个`package.json`，内部记录着项目相关的信息。

4. 在目录下新建`assets`、`config`、`demo`、`src`等目录，新建`.gitignore`文件用于忽略上传文件。具体目录机构如下：

   ```
   assets							// 存放静态文件
   config
       |-webpack.base.js			// webpack基础配置
       |-webpack.config.dev.js		// webpack开发环境配置
       |-webpack.config.prod.js	// webpack线上环境配置
   demo							// 插件演示文件入口
   	|-demo.js					// 入口文件
   	|-demo.less					// 样式文件
   	|-index.html				
   src 						    // 项目源码目录
   	|-index.js					// 插件源文件
   	|-index.less				// 插件源文件样式
   ```

5. 安装需要的包和插件：

   ```shell
   npm install webpack@4.x webpack-cli@3.x @babel/core @babel/plugin-proposal-class-properties @babel/plugin-proposal-decorators @babel/plugin-proposal-object-rest-spread @babel/plugin-transform-modules-commonjs @babel/plugin-transform-runtime @babel/preset-env @babel/preset-react babel-loader css-loader html-webpack-plugin less less-loader style-loader url-loader webpack-merge --save-dev
   ```

   因为是基于`React`的插件，当然肯定少不了我们的主角`React`：

   ```shell
   npm install react react-dom --save
   ```

6. 生成`.babelrc`文件，然后在其内部进行`babel`的配置：

   ```json
   {
       "presets": [
           [
               "@babel/preset-env",
               {
                   "targets": "> 0.25%, not dead"
               }
           ],
           "@babel/preset-react"
       ],
       "plugins": [
           "@babel/plugin-transform-runtime",
           "@babel/plugin-transform-modules-commonjs",
           [
               "@babel/plugin-proposal-decorators",
               {
                   "legacy": true
               }
           ],
           "@babel/plugin-proposal-class-properties",
           "@babel/plugin-proposal-object-rest-spread"
       ]
   }
   ```

7. 配置`webpack`打包规则，并使用`webpack-merge`将配置合并，具体如何配置请参照[webpack官网](https://webpack.docschina.org/)，在此只将代码贴出：

   ```js
   // webpack.base.js
   module.exports = {
       module: {
           rules: [
               { // 在webpack中使用babel需要babel-loader
                   test: /\.js?$/,
                   loader: 'babel-loader',
                   exclude: '/node_modules/',
               },
               { // 用于加载组件或者css中使用的图片
                   test: /\.(jpg|jpeg|png|gif|cur|ico|svg)$/,
                   use: [{
                       loader: 'url-loader', options: {
                           name: "images/[name][hash:8].[ext]"
                       }
                   }]
               },
               { // 编译less
                   test: /\.less$/,
                   exclude: '/node_modules/',
                   use: [{
                       loader: 'style-loader'
                   }, {
                       loader: 'css-loader'
                   }, {
                       loader: 'less-loader'
                   }]
               },
           ]
       }
   }
   ```

   ```js
   // webpack.config.dev.js
   const path = require('path');
   const merge = require('webpack-merge');
   const baseConfig = require('./webpack.base.js'); // 引用公共的配置
   
   const devConfig = {
       devtool: 'cheap-module-eval-source-map',
       entry: './demo/demo.js', // 入口文件
       mode: 'development', // 打包为开发模式
       output: {
           filename: 'demo.bundle.js', // 输出的文件名称
           path: path.resolve(__dirname, '../demo') // 输出的文件目录
       },
       devServer: { // 该字段用于配置webpack-dev-server
           contentBase: path.join(__dirname, '../demo'),
           compress: true,
           port: 9000, // 端口9000
           open: true // 自动打开浏览器
       },
   }
   
   module.exports = merge(devConfig, baseConfig);
   ```

   ```js
   // webpack.config.prod.js
   const path = require('path');
   const merge = require('webpack-merge');
   const baseConfig = require('./webpack.base.js');
   const devConfig = {
       entry: './src/index.js',
       mode: 'production',
       output: {
           path: path.resolve(__dirname, '../dist'),
           filename: 'index.js', // 输出文件
           libraryTarget: 'umd', // 采用通用模块定义, 注意webpack到4.0为止依然不提供输出es module的方法，所以输出的结果必须使用npm安装到node_modules里再用，不然会报错
           library: 'react-slide-verify', // 库名称
           libraryExport: 'default', // 兼容 ES6(ES2015) 的模块系统、CommonJS 和 AMD 模块规范
       },
       externals: {
           react: {
               root: "React",
               commonjs2: "react",
               commonjs: "react",
               amd: "react"
           },
           "react-dom": {
               root: "ReactDOM",
               commonjs2: "react-dom",
               commonjs: "react-dom",
               amd: "react-dom"
           }
       }
   }
   
   module.exports = merge(devConfig, baseConfig);
   ```

8. 配置好`webpack`后，我们需要`package.json`中的一些默认配置：

   - 修改`scripts`命令：

   ```json
   "scripts": {
       "build": "set NODE_ENV=production && webpack --config ./config/webpack.config.prod.js",
       "dev": "webpack-dev-server --config ./config/webpack.config.dev.js"
    },
   ```

   - 修改打包之后入口文件：

   ```json
   {
       "main": "dist/index.js", // 指定入口文件
   }
   ```

9. 接下来在`src/index.jsx`中写入插件的核心逻辑代码（由于篇幅有限，只贴出部分代码）：

   ```jsx
   import React, { Component } from 'react';
   import './index.less';
   const PI = Math.PI;
   
   class ReactSlideVerify extends Component {
       constructor(props) {
           super(props);
           this.canvas = React.createRef();
           this.block = React.createRef();
           this.state = {
               containerActive: false, // container active class
               containerSuccess: false, // container success class
               containerFail: false, // container fail class
   			...
           }
       }
       componentDidMount() {
           this.init();
       }
       // 初始化  
       init = () => {
           this.initDom()
           this.initImg()
           this.bindEvents()
       }
       // do other logic
       
       render() {
           return (
               <div className="verify-container">
               	{/* 插件内部核心html */}
               </div>
           )
       }
   }
       
    export default ReactSlideVerify;
   ```

10. 在`demo/demo.js`中引入我们写好的插件：

    ```jsx
    import React, { Component } from 'react';
    import ReactDom from 'react-dom';
    import ReactSlideVerify from '../src/index'
    import './demo.less';
    import img1 from './../assets/img1.jpg';
    import img2 from './../assets/img2.jpg';
    import img3 from './../assets/img3.jpg';
    import img4 from './../assets/img4.jpg';
    import img5 from './../assets/img5.jpg';
    
    class Demo extends Component {
        constructor(props) {
            super(props);
            this.state = {};
        }
        onSuccess = (time) => {
            console.log(time);
        }
        onFail = () => {
            console.log('失败');
        }
    
        onRefresh = () => {
            console.log('刷新')
        }
    
        render() {
            return (
                <div className="container">
                    <ReactSlideVerify
                        ...other props
                        success={this.onSuccess}
                        fail={this.onFail}
                        refresh={this.onRefresh}
                    />
                </div>
                
            )
        }
    }
    ReactDom.render(<Demo />, document.getElementById('app'));
    ```

    然后运行命令：`npm run dev`，这个时候就可以在浏览器中看到我们写的插件了，可以测试看看有没有什么问题。如果没有问题的话，我们就可以用线上环境的打包规则去打包我们的代码了。

## 发布到npm上

发布到npm上面去其实很简单就只有两个命令：`npm login` 和 `npm publish`：

### `npm login`

首先我们打开控制台，输入命令：`npm login`，这个时候会让我们输入npm注册时候的账号和密码以及邮箱：

![image-20210129090908541](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20210129090908541.png)

当出现最后一行话就代表已经登录成功了。

> 注意：这里再登录的时候，请不要使用`cnpm`淘宝源，否则会出现登录不上的问题！

### `npm publish`

接下来进入到我们的本地代码，打开控制台后，输入命令：`npm publish`：

> 注意：在发布之前请看下`package.json`中的`name`是否在`npm`仓库中已经注册了，如果已经注册了，就换个名字；
>
> ​		  如果是要更新所发布的包，请务必将`package.json`中的`version`更改一下，否则会导致发布不成功!

由于每次都是先进行线上环境的打包，然后再发布，我们可以再新加入一个命令，让它同时执行以上两步：

```json
"scripts": {
    "build": "set NODE_ENV=production && webpack --config ./config/webpack.config.prod.js",
    "dev": "webpack-dev-server --config ./config/webpack.config.dev.js",
    "publish": "npm run build && npm publish"
 },
```

这个时候我们就可以通过执行命令`npm run publish`来进行发布即可。

### `npm unpublish`

既然可以发布，就可以下架，我们可以运行命令：`npm --force unpublish <包名>`:

这个时候npm官方就会提示你一句比较有意思的话：

> `I sure hope you know what you are doing`

到此为止，关于在这次发布组件所遇到的问题就大致这些了，如果以后遇到其他的问题，会及时更新！