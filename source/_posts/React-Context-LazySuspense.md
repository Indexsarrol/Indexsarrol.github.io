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

![image-20210406153348856](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20210406153348856.png)

<!-- more -->

## 写在前面

虽然目前`React`已经更新到了第17个版本，但是丝毫不影响我今天写这个文章，因为这写东西在日常工作中还是使用的太少了，就当作笔记来记录一下吧。


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

### React.lazy()

`lazy`函数可以让你像渲染常规组件一样处理动态引入的组件，换句话说，就是懒加载，它使得我们在页面刚进入加载的时候无需加载指定组件，以便节省性能的消耗，当我们使用某个组件的时候，再进行动态加载。其实在`webpack`中，我们其实也有这种方式，那就是`Code-Spliting`，当我们在项目中使用了很多第三方包的时候，如果没有代码分割就会导致系统体积过大从而造成加载时间过长。

那么怎么进行代码分割呢？最佳方式就是通过`import()`语法。`import()` 有两种使用方式：

- 正常import

  ```jsx
  import { Button } from 'antd';
  ```

- 动态引入方式

  ```jsx
  import('./component.jsx').then(_ => {
  	console.log('loading...');
  })
  ```

当我们在组件中使用了动态引入方式后，`webpack`解析到该语法后（前提是已经配置好了代码分割），会自动的进行代码分割。

说了这么多，那么lazy()方法到底该如何使用呢？我们首先看一下我们在组件中引入其他组件的方式：

```jsx
import xxx from 'xxx';
```

> `lazy()`方法接收一个函数，而这个函数去动态的调用`import()`，并返回一个`Promise`，该 `Promise` 需要 `resolve` 一个 `default` `export` 的 `React` 组件。

示例：

```jsx
import React, { lazy } from 'react';
const MyComponent = lazy(() => import('./MyConponent.jsx'));

export default class App extends Component {
    render() {
        return (
        	<div>
            	<MyComponent />
            </div>
        )
    }
}
```

如果我们按照上面的代码执行的话，我们会看到控制台有报错：

![image-20210203145047213](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20210203145047213.png)

大致意思就是说，如果我们使用了`lazy`函数，就需要和`Suspense`进行搭配，所以我们需要再去引入`Suspense`，然后用<Suspense></Suspense>去包裹使用`lazy`函数生成的组件。但是，我们需要在<Suspense>组件中传递`fallback`参数，用来渲染未加载到已加载的空隙代码，例如加载态。

> 注意：这里`fallback`参数只接受`jsx`语法！

代码如下：

```jsx
import React, { lazy } from 'react';
const MyComponent = lazy(() => import('./MyConponent.jsx'));

export default class App extends Component {
    render() {
        return (
        	<Suspense fallback={<div>Loading...</div>}>
            	<MyComponent />
            </Suspense>
        )
    }
}
```

接下来，我们运用日常开发的一个小示例来演示一下，使用`lazy`和使用普通`import()`的区别吧。在我们开发中，我们避免不了的使用模态框这类的组件，但是有时候模态框组件内部的代码可能比较复杂，如果我们使用普通的导入方式，就需要在页面加载的时候就进行模态框相关资源的引入，再加上内部逻辑复杂，就大大的降低了加载速度，这个时候我们可以使用`lazy()`方法去进行一个懒加载。

我们首先看一下普通导入的状态，打开我们的控制台，找到Network => All:

![image-20210203151607707](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20210203151607707.png)

此时我们按F5刷新页面后，弹框的资源已经加载出来了，这就会导致加载时间变长，从而影响性能。

我们再来看一下使用`lazy`和`Suspense`搭配的方式：

```jsx
// Modal Component
import React, { Component } from 'react';
import { Modal } from 'antd';

export default class Modal extends Component {
    state = {};
	render() {
        return (
        	<Modal>
            	do somethings
            </Modal>
        )
    }
}
```

```jsx
// Main Component
import React, { Component, lazy, Suspense } from 'react';
const Modal = lazy(() => import('./Modal.jsx'));
class App extends Component {
    state = {
        show: false
    }
	render () {
        const { show } = this.state;
        return (
        	<Suspense fallback={<div>Loading...</div>}>
            	<button onClick={() => this.setState({ show: true })}>press</button>
                { show && <Modal /> }
            </Suspense>
        )
    }
}
```

我们打开控制台后，我们发现并没有去加载我们的`Modal`组件，当我们点击后才会出现`Modal`组件的资源：

![image-20210203152141571](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20210203152141571.png)

![image-20210203152430025](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20210203152430025.png)

这边还要穿插一个小tips：我们这里可以看到我们的资源名称其实是没有语义化的，看着比较费力，我们可以用我们的魔法注释来修改一下`chunk`的名称。

语法：

```jsx
import(/* webpackChunkName: "modal" */'./xxx.jsx')
```

再打开控制台，点击`press`按钮，我们就可以看到我们的`chunk`名称已经变了。

![image-20210203152912079](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20210203152912079.png)

