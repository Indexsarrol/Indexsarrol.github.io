---
title: React Context跨层级传递数据以及Lazy与Suspense实现延迟加载 
author: Indexsarrol
date: 2021-02-02 19:11:58
categories: 
- React 
tags:
- React
- ES6
- JavaScript
---

## 写在前面

虽然目前`React`已经更新到了第17个版本，但是丝毫不影响我今天写这个文章，因为这写东西在日常工作中还是使用的太少了，就当作笔记来记录一下吧。
<!-- more -->

## React-Context的使用

### 定义

`Context`在[官方](https://react.docschina.org/docs/context.html)的定义为：

> `Context` 提供了一个无需为每层组件手动添加 `props`，就能在组件树间进行数据传递的方法。

以上的一段话，翻译成通俗易懂的话就是：全局变量。

如果我们之前按照`props`的传值方式的话，我们可能需要传递多个组件才能在我们目标组件中使用到我们想要的值，这未免太过于复杂，而且在传递过程中，一些组件完全没有必要用到这些值，这就导致我们系统性能变差。所以我们可以使用`Context`来进行传值（当然我们可以使用`redux`或者`mobx`，但是有点太臃肿了）。



### 使用

我们先从`react`中导入`context`，这个和我们在`react`中使用`refs`一样：

```js
import React, { Component, createContext } from 'react';
```

然后创建`context`：

```js
const NewContext = createContext(); // 这里后续使用NewContext，是以组件的方式使用的
```

接下来就是使用`context`了，其返回了两个组件，分别是 `Context.Provider` 组件以及`Context.Consumer`组件

#### `Context.Provider`组件

```jsx
<Context.Provider value={123}>{子元素}</Context.Provider>
```

`Context.Provider`接受一个`value`属性，可以将值传递给“子元素”。并在子元素中能够直接获取到由`Provider`传递过来的值。当然`Provider`也是可以嵌套的，只需要我们在重新创建`Context`即可。当`Provider`的`value`发生变化时，它内部的所有组件都会重新进行一次渲染。

> 注意：如果我们在创建`Context.Provider`是忘记传递`value`属性的话，`createContext(defaultProps)`函数中的`defaultProps`则为默认值，目前官方并不推荐我们使用这种方式

#### `Context.Consumer`组件

上述中，我们说到了`Context.Provider`组件是可以嵌套子元素的，那么`Context.Consumer`组件作用也就来了，在该组件中，我们可以监听到`Context.Provider`组件的value值的变更，从而更新数据进行渲染，只不过在`Context.Consumer`组件内部，我们需要用函数作为子元素的方法：代码如下：

```jsx
<Context.Consumer>
{ value => <h1>{value}</h1> }
</Context.Consumer>
```

### 示例代码

```jsx
import React, { Component, createContext } from 'react';
const MyContext = createContext(); // 创建Context

// 子组件
class Children extends Component {
    render() {
        return (
            <MyContext.Consumer> {/* 使用Consumer */}
            	{/* 用函数作为子元素 */}
                {
                    value => <h1>{value}</h1>
                }
            </MyContext.Consumer>
        )
    }
}

class App extends Component {
    state = {
        value: 20
    }
	render () {
        const {value} = this.state;
        return (
        	<MyContext.Provider value={value}> {/* 使用Provider */}
            	<button 
                    onClick={() => this.setState({ value: value - 1 })}
                >
                    press
                </button>
                <Children />
            </MyContext.Provider>
        )
    }
	
}
```

以上代码中，我们使用了`Context.Provider`和`Context.Consumer`搭配完成了数据传输，这里只是垮了一个层级来传递参数，当然可以跨多层传递。我们在页面中打开后，点击按钮，数值会一直减1。这样我们就实现了一个`Context`跨组件传值。

由上述代码，我们可知，如果每次我们想要使用`Provider`，就必须在子组件中使用`Consumer`与之搭配使用，这样总觉得有点繁琐，那有没有什么办法让我们摆脱`Consumer`的束缚，或者说在使用`Consumer`之前就已经获取到由`Provider`传递过来的值了呢？答案是肯定的，`React`给我们提供了一个`Api`：`contextType`，那我们如何使用呢？

只需要在我们的使用`Consumer`组件的地方，声明一个`contextType`静态属性，并将已创建的`MyContext`赋值给`contextType`属性，这个时候会在子组件的类中生成`context`变量，而这个变量内就保存了我们从`Context.Provider`传递过来的`value`值，具体代码如下：

```jsx
// 子组件
class Children extends Component {
    static contextType = MyContext; // 声明ContextType
    render() {
        const value = this.context;
        return (
            <MyContext.Consumer> {/* 使用Consumer */}
            	{/* 用函数作为子元素 */}
                {
                    value => <h1>{value}</h1>
                }
            </MyContext.Consumer>
        )
    }
}
```

> 注意：虽然`Context`确实比较好用，但是[官方](https://react.docschina.org/docs/context.html#before-you-use-context)并不推荐我们使用这种方式来进行组件传值，因为这样会使得组件的复用性变得很差！

## Lazy与Suspense实现延迟加载

后续更新。。。