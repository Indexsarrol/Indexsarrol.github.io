---
title: 项目中常用方法，可收藏一下😘
author: Indexsarrol
date: 2021-01-07 11:20:20
categories: 
- 常用方法
tags:
- JavaScript
- 手写代码
---

## 前言

在我们日常的项目开发中，我们喜欢将一些公用的方法，例如，请求方法，统一操作缓存的方法等等，将这写方法会单独抽离出来，然后放到统一的utils文件中，然后暴露给外部去使用，这就使得我们的项目更加的完善，也更加的有层次感。接下来，我将一些目前在我的项目中常用的方法贴出来，可供以后使用。

<!-- more -->

## 常用方法

### 常用工具方法

所谓工具方法，就是说在项目中，我们可能需要通过一些方法执行一些特定的事，比如判断一个对象是否为空、动态生成一个`uuid`、以及通过文件流下载的方法等等

#### 下载公共方法

首先先安装我们的`fetch`请求库

```bash
npm install cross-fetch --save-d
```

方法如下：

```js
/**
 * 方法下载
 * @param url 下载的url
 * @param opt 配置项
 * @returns {Promise<any>}
 */
export function fetchDownload(url, opt) {
    let promise = new Promise((resolve) => {
        fetch(url, opt).then(res => res.blob().then(blob => {
            let a = document.createElement('a');
            let url = window.URL.createObjectURL(blob); // 获取 blob 本地文件连接 (blob 为纯二进制对象，不能够直接保存到磁盘上)
            let filename = window.decodeURI(res.headers.get('Content-Disposition').split('=')[1]); // 获取文件名并处理文件名编码问题
            filename = filename.replace('utf-8\'zh_cn\'', '');
            a.href = url;
            a.download = filename;
            a.click();
            window.URL.revokeObjectURL(url);
        }).then(() => {
            resolve();
        }));
    });
    return promise;
} 
```

#### 判断方法

```js
// 内部函数, 用于判断对象类型
function _getType(object) {
    return Object.prototype.toString.call(object).match(/^\[object\s(.*)\]$/)[1];
}
// 判断是否为数组
export function isArray(obj) {
    return _getType(obj).toLowerCase() === 'array';
}
// 判断是否为字符串
export function isString(obj) {
    return _getType(obj).toLowerCase() === 'string';
}
// 判断是否为日期
export function isDate(obj) {
    return _getType(obj).toLowerCase() === 'date';
}
// 判断是否为对象
export function isObject(obj) {
    return _getType(obj).toLowerCase() === 'object';
}
// 判断是否为数值
export function isNumber(obj) {
    return _getType(obj).toLowerCase() === 'number' && !isNaN(obj);
}


/**
 * @desc 判断参数是否为空, 包括null, undefined, [], '', {}
 * @param {object} obj 需判断的对象
 */
export function isEmpty(obj) {
    let empty = false;
    if (obj === null || obj === undefined) {    // null and undefined
        empty = true;
    } else if ((isArray(obj) || isString(obj)) && obj.length === 0) {
        empty = true;
    } else if (isObject(obj)) {
        let hasProp = false;
        for (let prop in obj) {
            if (prop) {
                hasProp = true;
                break;
            }
        }
        if (!hasProp) {
            empty = true;
        }
    }
    return empty;
}

/**
 * @desc 判断参数是否不为空
 */
export function isNotEmpty(obj) {
    return !isEmpty(obj);
}

/**
 * @desc 判断参数是否为空字符串, 比isEmpty()多判断字符串中有空格的情况, 如: '   '.
 * @param {string} str 需判断的字符串
 */
export function isBlank(str) {
    if (isEmpty(str)) {
        return true;
    } else if (isString(str) && str.trim().length === 0) {
        return true;
    }
    return false;
}

/**
 * @desc 判断参数是否不为空字符串
 */
export function isNotBlank(obj) {
    return !isBlank(obj);
}
/**
 * @desc 判断参数是否为FormData对象
 */
export function isFormData(obj) {
    return obj instanceof FormData;
}

```

#### 动态生成`uuid`

```js
/**
 * @desc 生成一个随机id
 */
export function uuid() {
    return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, (c) => {
        let r = Math.random() * 16 | 0, v = c === 'x' ? r : (r & 0x3 | 0x8);
        return v.toString(16);
    });
}
```

#### 获取地址栏参数对象以及值

```js
/**
 * @desc 获取 url 参数，因为 this.props.location.query 不能得到带有 # 的参数，所以添加此方法
 */
export function getQueryParams() {
    let obj = {}, name, value;
    let str = location.href;
    let num = str.indexOf('?');
    str = str.substr(num + 1);
    const arr = str.split('&');
    for (let i = 0; i < arr.length; i++) {
        num = arr[i].indexOf('=');
        if (num > 0) {
            name = arr[i].substring(0, num);
            value = arr[i].substr(num + 1);
            obj[name] = value;
        }
    }
    return obj;
}

/**
 * @desc 通过URL搜索对象获取url参数, 如www.xxx.com?a=1&b=2, getURLParam('a') return 1
 */
export function getURLParam(name) {
    if (isBlank(name)) {
        return;
    }
    let urlQuery = getQueryParams();
    return urlQuery[name];
}
```

#### 生成指定范围的随机整数

```js
/**
 * 生成随机整数
 * @param min
 * @param max
 * @constructor
 */
export function random(min, max) {
    min = min || -90;
    max = max || 90;
    return min + Math.floor(Math.random() * (max - min));
}
```

#### 深度拷贝

```js
// 第一种方式
export function deepClone(obj) {
    return JSON.parse(JSON.stringify(obj));
}
// 第二种方式
export function deepClone(source) {
    if (!source && typeof source !== 'object') {
        throw new Error('error arguments', 'shallowClone');
    }
    const targetObj = source.constructor === Array ? [] : {};
    for (const keys in source) {
        if (source.hasOwnProperty(keys)) {
            if (source[keys] && typeof source[keys] === 'object') {
                targetObj[keys] = source[keys].constructor === Array ? [] : {};
                targetObj[keys] = deepClone(source[keys]);
            } else {
                targetObj[keys] = source[keys];
            }
        }
    }
    return targetObj;
}
```

#### 表格序号索引

```js
/**
 * 获取表格索引序号
 * @param index
 * @param start
 * @param size
 * @returns {*}
 */
export function getTableIndex(index, start, size = 10) {
    return index + 1 + (start - 1) * size;
}
```

### 缓存类方法

项目中，我们需要偶尔去操作一下缓存，`sessionStorage`或者`localStorage`等，所以我们这边特别的封装了几个方法，供以后项目中使用：

#### `sessionStorage`方法

```js
/**
 * 设置session
 * @param name 存储名称
 * @param value 存储值
 */
export function setSession(name, value) {
    if (typeof sessionStorage === 'object') {
        var data = value;
        if (typeof value !== 'string') {
            if (data === undefined) {
                data = null;
            } else {
                data = JSON.stringify(data);
            }
        }
        sessionStorage.setItem(name, data);
    }
}

/**
 * 获取session
 * @param name 存储名称
 */
export function getSession(name) {
    if (typeof sessionStorage === 'object') {
        var data = sessionStorage.getItem(name);
        try {
            return JSON.parse(data);
        } catch (e) {
            return data;
        }
    }
    return null;
}
```

#### `localStorage`方法

```js
/**
 * 设置Local
 * @param name 存储名称
 * @param value 存储值
 */
export function setLocal(name, value) {
    if (typeof localStorage === 'object') {
        var data = value;
        if (typeof value !== 'string') {
            if (data === undefined) {
                data = null;
            } else {
                data = JSON.stringify(data);
            }
        }
        localStorage.setItem(name, data);
    }
}
/**
 * 获取Local
 * @param name 存储名称
 */
export function getLocal(name) {
    if (typeof localStorage === 'object') {
        var data = localStorage.getItem(name);
        try {
            return JSON.parse(data);
        } catch (e) {
            return data;
        }
    }
    return null;
}
```

#### `sessionStorage`和`localStorage`共有方法

```js
/**
 * 移除session或者Local
 * @param name 存储名称
 */
export function remove(name) {
    if (typeof sessionStorage === 'object') {
        if (sessionStorage.getItem(name)) {
            sessionStorage.removeItem(name);
        }
    }
    if (typeof localStorage === 'object') {
        if (localStorage.getItem(name)) {
            localStorage.removeItem(name);
        }
    }
}
/**
 * 移除session或者Local全部内容
 */
export function clear() {
    if (typeof sessionStorage === 'object') {
        sessionStorage.clear();
    }
    if (typeof localStorage === 'object') {
        localStorage.clear();
    }
}
```



### 请求方法类

我们在项目中，必不可免的要和请求打交道，如果我们使用常规的`ajax`或者`axios`去请求的话，就会造成我们代码量非常冗余，所以接下来用`promise`封装一个通用的请求方法吧，这里我们需要用到上述的一些共有方法。在封装之前，我们需要安装两个插件——`axios`和`qs`：

```bash
npm install axios --save
npm install qs --save-dev
```

`axios` 相信大家已经很熟悉了，在这里就不去过多的介绍了，我们来着重看下`qs`是个什么东西：

`qs` 官方给出的定义是序列化一个对象，`qs` 有两个常用方法：`qs.parse()`、`qs.stringify()`

`qs.parse()`方法是用于将一个`url`解析成对象的格式：

```js
let url = https://www.baidu.com?a=1&b=2&c=3;
qs.parse(url);
//{
//    a: 1,
//    b: 2,
//    c: 3
//}
```

`qs.stringify()`方法是`qs.parse()`方法的逆执行：

```js
let obj = {
    a: 1,
    b: 2,
    c: 3
};
qs.stringify(obj); // a=1&b=2&c=3
```

接下来就是请求方法的封装，因为我的项目是react，所以我们是基于`antd`和`react-router`进行的封装，如果使用其他框架，请自行替换：

```js
import { isString, isBlank, isEmpty, isNotEmpty, isFormData } from './util';
import { getSession } from './storage.js';
import qs from 'qs';
import axios from 'axios';
import { message } from 'antd';
import { browserHistory } from 'react-router';

/**
 * @desc 使用axios第三方库访问后台服务器, 返回封装过后的Promise对象.
 * @param {string} url 请求的接口地址, 格式: "/xxx..."
 * @param urlType
 * @param {string} domain 跨域请求的域名地址, 如: http://www.baidu.com
 * @param {string} type HTTP请求方式, 默认GET.
 * @param {object} data 请求的数据, object对象格式
 * @param contentType
 * @param {function} onUpload 上传文件过程中的回调函数, 接收progressEvent参数.
 * @param {function} onDownload 下载文件过程中的回调函数, 接收progressEvent参数.
 * @param {function} cancel 取消请求的回调函数, 接收cancel参数, 当执行cancel()参数时请求被取消.
 * @param {number} timeout 配置请求超时时间, 为毫秒数, 默认从配置文件读取.
 * @param closeTips
 * @param {boolean} cache 是否开启缓存, 开启后同样的请求(url相同, 参数相同), 第二次请求时会直接返回缓存数据, 不会请求后台数据, 默认false.
 * @param {boolean} handleError 是否自动处理接口报错情况, 默认true.
 * @return {object} - 返回一个promise的实例对象
 */

const ContentType = {
    JSON: 'application/json',
    FORM_URLENCODED: 'application/x-www-form-urlencoded'
}
export function Request({
                              url = '',
                              urlType = null,
                              type = HttpMethod.GET,
                              data = null,
                              contentType = ContentType.JSON,
                              onUpload = null,
                              onDownload = null,
                              cancel = null,
                              closeTips = false,
                              handleError = true
                          }) {
    let getData;
    let postData;
    let cancelToken;
    let crossDomain = false;
    if (isEmpty(url)) {
        return Promise.resolve();
    }

    if (type === HttpMethod.POST) {
        if (isNotEmpty(data)) {
            postData = data;
            if (ContentType.FORM_URLENCODED === contentType) {
                if (isNotEmpty(postData) && !isFormData(postData)) {
                    postData = qs.stringify(postData, { allowDots: true });
                }
            }
        }
    } else {
        getData = data === null ? {} : data;
    }
    if (isNotEmpty(cancel)) {
        cancelToken = new axios.CancelToken(cancel);
    }
    let promise = new Promise((resolve, reject) => {
        axios.defaults.headers.common['User-Token'] = getSession('User-Token'); // 添加token
        axios.defaults.headers.common['X-Requested-With'] = 'XMLHttpRequest';
        axios.defaults.headers.post['Content-Type'] = contentType + ';charset=UTF-8';
        axios({
            method: type,
            baseURL: '/api/v1.0/', // 看项目是否配置了ngix反向代理
            url: url,
            timeout: timeout,
            params: getData,
            data: postData,
            withCredentials: crossDomain,
            onUploadProgress: onUpload,
            onDownloadProgress: onDownload,
            cancelToken: cancelToken
        }).then((response) => {
            if (isBlank(response.data)) {
                reject(response);
            } else {
                let responseData = response.data;
                if (isString(responseData)) {
                    try {
                        responseData = JSON.parse(responseData);
                    } catch (e) {
                        try {
                            if (urlType) {
                                responseData = JSON.parse(responseData.split('=')[1].split(';')[0]);
                            }
                        } catch (e) {
                            if (handleError) {
                                message.error('数据异常');
                            }
                            reject(e);
                            return;
                        }
                    }
                }
                if (!urlType) {
                    if (responseData.code === 200 || responseData.code === 403) {
                        resolve(responseData);
                    } else if (responseData.code === 332 || responseData.code === 302) {
                        browserHistory.push('/login');
                    } else {
                        reject(responseData);
                    }
                } else {
                    resolve(responseData);
                }
            }
        }).catch((error) => {
            // 服务端返回的异常
            if (error.response) {
                reject(error.response.data);
            } else {
                reject(error);
            }
        });
    });
    return promise;
}
```

方法已经封装好了，那么怎么去使用呢？

首先，我们可以新建一个请求方法的js文件，引入刚才的请求方法：

```js
import { Request } from 'utils';

export function getData(data) {
    return Request({
        url: '/getList',
        type: 'get',
        data
    });
}
```

在需要调用接口的地方，我们可以这么使用：

```js
import React, { Component } from 'react';
import { getData } from './services'
class Demo extends Component{
    constructor(props) {
        this.state = {}
    }
    
    componentDidMount() {
        this.getDataList();
    }
    
    getDataList() {
        // 调用接口
        getData({ search: '' }).then(res => {
            console.log(res); // 返回值
        }).catch(err => {
            console.log(err);
        })
    }
    
    render() {
        return <div></div>
    }
}
```

## 结尾

以上这些就是目前项目中使用的共有方法，后续还会继续更新...