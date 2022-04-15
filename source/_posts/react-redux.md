---
title: 都2021年了，你还不会Redux？
author: Indexsarrol
date: 2021-04-08
categories: 
- React
tags:
- React
- Redux
---

![image-20210408160440589](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20210408160440589.png)

<!-- more -->

## 前言

​	学习或使用过`react`的同学对`redux`肯定多多少少有点了解，今天我就结合我目前公司的可视化BI项目来重新复习一下`redux`，因为我们这个项目目前的技术栈主要是`React + TypeScript + React-Hooks + React-redux + Redux-thunk + Ant-Design`，而且内部功能逻辑复杂，所以肯定不会很详细的解释，只会挑出一些比较容易理解以及比较值得学习的地方来着重说一下，那么话不多说开始吧。

## Redux简介

相信大家已经对redux已经并不陌生了，官网对`redux`的定义为：

> `Redux` 是 `JavaScript` 状态容器，提供可预测化的状态管理。

那我们为什么要用`redux`呢？两个方面概括：第一，当一个`React`组件内部的状态太过于复杂时，对组件的性能以及可复用性将会变得很差，这个时候我们需要将这些状态统一放到一个公共的仓库下进行管理，这样组件内部的逻辑也不会特别复杂，而且可用性将会提高很多；第二，我们在日常开发中，必不可少的需要组件之间的传值，显得很臃肿，如果我们将所有的数据集中放到一个地方去管理，就无需我们再去一层一层传值了。如下图所示：

![image-20210408154516247](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20210408154516247.png)

当我们未使用`redux`时，当绿色的组件状态发生了变化时，如果我们想更新顶层组件，我们需要经过一层又一层的传值才能达到我们想要的目的。

当我们使用`redux`时，我们可以创建一个新的类似仓库一样的`Store`模块，将所有的数据都放到`store`中，如果绿色的组件更新了状态，这个时候会派发一个更新操作到`store`中，去更新数据，然后我们在其他组件享用到这个数据时，只需从`store`中获取即可，大大的节省了代码量以及性能。

## Redux的基础使用

别的我们先不多说，直接上张图，如下图所示：

![image-20210408170614982](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20210408170614982.png)

对于`redux`老玩家来说，这张图可能再熟悉不过了，但对于那些还停留在打人机地步的同学会满脸黑人问号，这**是个啥？别着急，我们来一步一步说明：

这其实就是我们`redux`数据流向，说白了，就是把我们的数据怎么一步一步存到`redux`中，并且如何更新，如果获取`redux`中的值的。其实看懂这个图并不复杂，我们只需要理解怎么去往`redux`中去塞值（更新`redux`的数据）,我们可以结合日常生活中的你去图书馆借书的流程即可。

假设我们现在要去图书馆借一本武侠小说，我们肯定要和图书馆管理员说，我们要借什么书，然而呢图书馆里面的书肯定很多，管理员也不可能每一本都记得很清楚，他就需要去借助其他工具（小手册，电脑）来帮我们查找这本书在哪，找到书之后，图书馆管理员才能把书拿给你。对比这个日常生活的例子，我们就可以很轻松地理解这张图了。

最下面的`ReactComponents`就是我们自己，接下来来到了代码中的`actionCreators`，我们可以把这个当作我们对图书馆管理员说的话：”我要借一本武侠小说“，那么这句话就从`actionCreators`中出来变成了`action`，通过声音的传播将我要什么书的信息（数据）传到了管理员的耳朵里（`Store`），但是呢，图书管的书实在有太多太多，管理员也不知道我们要的书在哪，所以他就要借助其他工具（`Reducers`）来去帮我们查找相关书籍的信息，通过工具查找后，工具会图书馆管理员一个信息，告知管理员这个书是在几楼那个位置，这个时候图书馆管理员去这个位置拿到小说之后，就会把书那给你。这个就是`redux`数据流向和日常生活中的例子结和起来了。接下来我们再用代码的方式来描述一下。

首先对于`Components`组件来说，必不可免的会有数据的产生，那么在组件去拿`redux`中的数据，其实很简单，主要是复杂在更新`redux`中的数据，首先我们要通过`actionCreators`去生成一个`action`，那么`action`是什么呢？

> `Action`就是一个普通对象，其内必须使用一个字符串类型的`type` 字段来表示将要执行的动作。

生成`action`之后，通过`redux`提供的`dispatch()` 方法将`action`传递到`store`，并由`store`传到`reducers`中，由`reducers`统一处理之后返回一个新的状态更新到`store`中去。这就是一个完整的`redux`数据流向。

接下来我们通过简单的小案例——`todolist`来熟悉一下整个`redux`是如何使用的吧！

首先，我们先安装官方提供的脚手架——`create-react-app`，当然了，你想自己手动搭一个也可以（手动狗头）。

### 安装`create-react-app`

```bash
$ npx create-react-app todolist
```

然后我们进入项目并启动。

```bash
$ cd todolist
$ yarn start
```

### 实现简易版的todolist

为了等一下布局方面，我们直接采用蚂蚁金服的`antd UI` 库，安装过程就不说了，照着官网的一步一步配置就可以了；

接下来，我们先用传统的`class`方式实现一下`todolist`，代码如下就不一一解释了，相信大家都能看懂：

```jsx
import React, { Component } from 'react';
import { Input, Button, List } from 'antd';
import './App.css';
class App extends Component {
  constructor(props) {
    super(props);
    this.state = {
      inputValue: '',
      list: ['1', '2']
    }
  } 

  onChange = e => {
    this.setState({ inputValue: e.target.value });
  }

  addTodo = () => {
    const { list, inputValue } = this.state;
    if (inputValue.length === 0) return;
    list.push(inputValue);
    this.setState({ list, inputValue: '' });
  }

  deleteTodo = index => {
    const { list } = this.state;
    list.splice(index, 1);
    this.setState({ list });
  }
  render() {
    const { inputValue, list } = this.state;
    return (
      <div className="todo-container">
        <div className="add-todo">
          <Input 
            placeholder="输入内容"
            onChange={this.onChange}
            value={inputValue}
          />
          <Button onClick={this.addTodo}>提交</Button>
        </div>
        <div className="todo-list">
          <List
              bordered
              dataSource={list}
              renderItem={(item, index) => (
                <List.Item onClick={this.deleteTodo.bind(this, index)}>{item}</List.Item>
              )}
            />
        </div>
      </div>
    )
  }
}
export default App;
```

具体布局如下：

![image-20210409141033132](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20210409141033132.png)

### 安装`redux`

```bash
npm install redux -S
```

### 使用`redux`

我们在根目录下新建一个`store`文件夹，新建`index.js`文件，写入：

```js
import { createStore } from 'redux';
export default const store = createStore();
```

其次，我们现在已经有了图书馆管理员，那还需要给这个图书馆搭配一个工具`Reducers`，在当前目录下新建`reducer.js`文件：

```js
let initValue = {
    inputValue: '',
    list: [],
}; // store的默认值
export default (state = initValue, action) => {
    return state;
}
```

这时，我们需要将我们的`store`和`reducer`结合起来，为了更直观地看到我们地数据流向，我们需要在Chrome网上应用店搜索安装`redux-devtools`插件，如果你的浏览器插件栏有这个图标就代表安装成功了：

![image-20210409151643949](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20210409151643949.png)

我们需要在`createStore`方法中的第二个参数中传入`window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()`语句；

```js
import { createStore } from 'redux';
import reducer from './reducer';
const devTools = window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__();
export default const store = createStore(reducer, devTools);
```

此时打开我们的控制台就会有一个`Redux`的标签栏，打开之后就是这个样子：

![image-20210409152000782](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20210409152000782.png)

这样，我们一个简易仓库就搭建好了，接下来的时间就是将我们`todolist`和`redux`结合使用了

我们在`App.js`中引入我们创建好的`store`：

```diff
// app.js
+ import store from './store';

- this.state = {
-	inputValue: '',
-   list: ['1', '2']
- };

+ this.state = store.getState(); // 该方法是用于获取store中的默认值，也就是reducer里定义的默认值
```

我们就可以获取到`redux`中的数据了，接下来我们就是需要去更改`redux`中的数据了，大家还记得我们的流程嘛？先有`action`然后通过`store`的`dispatch()`到`reducer`中，让`reducer`处理数据之后，再返回给组件。

```diff
// app.js
- onChange = e => {
-   this.setState({ inputValue: e.target.value });
- }

+ onChange = e => {
+	const action = { type: 'change_input_value', payload: e.target.value };
+	store.dispatch(action);
+ }
```

这样我们的数据就被派发到了`reducer`中，在`reducer`中进行处理，`reducer`其实就是一个纯函数函数，所谓的纯函数就是输入与输出结果一样，其接收两个参数`state`和`action`，`state`就是当前在`store`中的存储的状态值，而`action`就是从`dispatch()`方法派发过来的，这里我们需要注意：

> `reducer`中我们绝对不能修改原有的`state`！！！

所以我们这里先选择一个比较简单的深拷贝方式：`JSON.parse(JSON.stringify(data))`，当然这种方式有点弊端，我们暂且先用着，后面我们讲借助一个第三方库来更优雅地去处理这个问题。

```diff
// reducer.js
export default (state = initValue, action) => {
- return state;
+ switch(action.type) {
+ 	case 'change_input_value':
+ 		const newState = JSON.parse(JSON.stringify(state));
+		newState.inputValue = action.payload;
+		return newState; // 切记一定要将新的state返回出去
+ 	default :
+   	return state;
+ }
}
```

这个时候当我们在输入框输入时，我们可以仔细观察一下`devtools`工具，发现其内部的数据在不停的变化，但是，在页面上并没有任何变化，输入框内输入不上值：

![image-20210409153736150](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20210409153736150.png)

这是因为，我们只是更改了`store`里面的数据，但是页面上面并不知道我们更改了数据，所以我们就用到了`store`的第三个方法：`store.subscribe()`，该方法接收一个回调函数，当`dispatch action`的时候就会执行这个函数，我们可以使用这个函数来监听数据的变化从而达到更新数据的效果。

```diff
// app.js
this.state = store.getState(); // 该方法是用于获取store中的默认值，也就是reducer里定义的默认值
+ store.subscribe(this.handleSubscribe);
+ handleSubscribe = () => {
+    this.setState(store.getState())
+ }
```

这时，我们看一下`devtools`里的数据也发生了变化，并且输入框就可以输入了，达到了我们的目的。

接下来我们完成第二个功能，点击按钮新增，我们需要在按钮上绑定一个`onClick`事件，同样走一遍刚才的流程，先派发`action`到`reducer`，再经过`reducer`处理之后返回给`store`，最后有组件拿到值:

```jsx
<Button onClick={this.addToto}>提交</Button>

addToto = () => {
    const action = {
        type: 'add_todo'
    };
    store.dispatch(action);
}

// reducer.js
let initialValue = {
    inputValue: '',
    list: []
};

export default (state = initialValue, action) => {
    const newState = JSON.parse(JSON.stringify(state));
    switch(action.type) {
        case 'change_input_value':
            newState.inputValue = action.payload;
            return newState;
        case 'add_todo':
            if (newState.inputValue.length === 0) return ;
            newState.list.push(newState.inputValue);
            newState.inputValue = '';
            return newState;
        default: 
            return state;
    }
}
```

这样大致走了两遍流程之后，应该都了解得差不多了，删除功能和这些类似，只不过`action`中得`payload`需要传递一个索引值`index`，具体代码在这里就不写了，感兴趣的可以自己试一下。说了这么多，大家有没有发现一个不太友好的地方，没错，就是`action`，每次都需要手动的创建一个`action`，确实挺麻烦的，我们可以将这些`action`统一的放到一个文件中，然后需要的时候就从这个文件中引入即可，于是，我们在`store`目录中新创建一个`actionCreators.js`文件，里面用于保存这些`action`， 但是我们很快就能发现新的问题，有些`action`是需要进行传值的，所以我们直接采用函数的方式去返回`action`：

```js
// actionCreators.js
export const changeInputValue = (value) => ({
   type: 'change_input_value',
   payload: value
});

export const addTodoItem = () => ({
    type: 'add_todo_item'
})
```

那在派发`action`的地方我们可以直接先引入`actionCreators.js`:

```diff
// app.js
import * as actionCreators from './store/actionCreators';

onChange = e => {
- const action = {
-	type: 'change_input_value',
-	payload: e.target.value
- };

+ store.dispatch(actionCreators.changeInputValue(e.target.value));
}

addTodo = () => {
- const action = {
-	type: 'add_todo_item'
- };

+   store.dispatch(actionCreators.addTodoItem());
}
```

不知道大家有没有发现，其实这么写还是有一点点缺陷的，缺陷在于如果我们把`type`这个字符串不小心拼写错误的话，我们在`reducer`中就不会做出任何反应，如果出问题的话，我们可能不能及时发现错误，所以我们在这里可以新建一个`constants.js`文件，用于存储`type`字符串的常量值：

```js
// constants.js
export const CHANGE_INPUT_VALUE = 'change_input_value';
export const ADD_TODO_ITEM = 'add_todo_item';
```

于此同时，我们需要更改`actionCreators.js`文件和`reducer.js`文件：

```js
// actionCreators.js
import * as contants from './constants';
export const changeInputValue = (value) => ({
   type: contants.CHANGE_INPUT_VALUE,
   payload: value
});

export const addTodoItem = () => ({
    type: contants.ADD_TODO_ITEM,
})

// reducer.js
import * as contants from './constants';
let initialValue = {
    inputValue: '',
    list: []
};

export default (state = initialValue, action) => {
    const newState = JSON.parse(JSON.stringify(state));
    switch(action.type) {
        case contants.CHANGE_INPUT_VALUE:
            newState.inputValue = action.payload;
            return newState;
        case contants.ADD_TODO_ITEM:
            if (newState.inputValue.length === 0) return ;
            newState.list.push(newState.inputValue);
            newState.inputValue = '';
            return newState;
        default: 
            return state;
    }
}
```

这样我们就不会因为我们拼写的错误导致长时间定位不到问题在哪个位置了。完美！到此`redux`就到此结束了，肯定有不足的地方，我也会继续去改进这些地方。接下来我们就要去了解`react-redux`的使用方式了，`redux`和`react-redux`多多少少还是有点区别的（个人理解），接下来进入正题吧！

## React-Redux的使用

上述我们了解了`redux`的基本使用以及利用`redux`做了一个简单的`todolist`，当然了，`redux`不仅可以在`react`中使用，你可以在`vue`、`jq`等其他框架里去使用，我们接下来介绍的`react-redux`是相当于专门为`react`封装了一个`redux`，话不多说，先安装再来了解相关的`api`：

```bash
$ npm install react-redux --save-dev
```

### api——Provider

`Provider`是一个组件，其实很好理解，通俗来说就是你想让哪些组件使用`react-redux`里面的数据，一般来说，我们都会放到根组件外部，让其包裹根组件，它需要传递一个`store`参数，这个也很好理解，就是将我们的`store`仓库传递给这个参数即可。那么我们如何使用呢？

```jsx
import store from './store';

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
)
```

### api——connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])

要说`redux`和`react-redux`最大的区别，我认为应该就是这个`connect()`方法了，这个方法使用于将我们当前的组件和`react-redux`做一个连接，让彼此可以相互传递数据，这个操作并不会改变原来的组件类，而是返回一个新的并且与`redux store`做好连接的组件类，其内部原理就是我们`react`中的高阶组件，由于本文章以会使用为目的，在这里我们就不细说了。

#### [`mapStateToProps(state, [ownProps]): stateProps`] (*Function*):

`mapStateToProps`是一个回调函数，如果我们调用了这个`connect`方法并且传入了`mapStateToProps`这个参数，该组件就会监听`store`中数据的变化，不管任何情况，只要`store`里面的数据发生了变化，那么该函数就会被执行一次，这个函数的返回值必须是一个纯对象，这个对象将会和`props`进行一个合并，如果你省略了这个参数，你的组件将不会监听 `Redux store`。如果指定了该回调函数中的第二个参数 `ownProps`，则该参数的值为传递到组件的 `props`，而且只要组件接收到新的 `props`，`mapStateToProps` 也会被调用。说了这么多，我们用一段通俗易懂的话来概括一下：

> `mapStateToProps`是用来筛选`store`中的数据，并且将需要的数据注入到当前的`props`中。

#### [`mapDispatchToProps(dispatch, [ownProps]): dispatchProps`] (*Object* or *Function*): 

`mapDispatchToProps`同样也是一个回调函数，如果传递的是一个对象，那么每个定义在该对象的函数都将被当作 Redux action creator，对象所定义的方法名将作为属性名；每个方法将返回一个新的函数，函数中`dispatch`方法会将`action creator`的返回值作为参数执行。这些属性会被合并到组件的 `props` 中。概括一下：

> `mapDispatchToProps`充当的是`actionCreators`的角色，将`action`派发到`reducer`中，并将这个动作派发给当前组件`props`中。

#### [`mergeProps(stateProps, dispatchProps, ownProps): props`] (*Function*): 

如果指定了这个参数，`mapStateToProps()` 与 `mapDispatchToProps()` 的执行结果和组件自身的 `props` 将传入到这个回调函数中。该回调函数返回的对象将作为 `props` 传递到被包装的组件中。你也许可以用这个回调函数，根据组件的 `props` 来筛选部分的 `state` 数据，或者把 `props` 中的某个特定变量与 `action creator` 绑定在一起。如果你省略这个参数，默认情况下返回 `Object.assign({}, ownProps, stateProps, dispatchProps)` 的结果。

#### [`options`] *(Object)* :

如果指定这个参数，可以定制 `connector` 的行为。

- [`pure = true`] *(Boolean)*: 如果为 `true`，`connector` 将执行 `shouldComponentUpdate` 并且浅对比 `mergeProps` 的结果，避免不必要的更新，前提是当前组件是一个“纯”组件，它不依赖于任何的输入或 `state` 而只依赖于 `props` 和 Redux store 的 `state`。*默认值为 `true`。*
- [`withRef = false`] *(Boolean)*: 如果为 `true`，`connector` 会保存一个对被包装组件实例的引用，该引用通过 `getWrappedInstance()` 方法获得。*默认值为 `false`。*

在我们日常开发中，我们一般只会使用到前两个参数`mapStateToProps`和`mapDispatchToProps`，我们来通过我们上面写的`todolist`的例子来看些这个到底怎么用

首先我们需要在根组件中导入`Provider`并且提供`store`：

```jsx
// 项目入口根目录
import React from 'react';
import { Provider } from 'react-redux';
import store from './store';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
ReactDOM.render(
  <Provider store={store}>
    <App /> 
  </Provider>,
  document.getElementById('root')
);

```

然后我们清除一下上面使用`redux`遗留的代码，直接引入`connect`方法：

```jsx
import React, { Component } from 'react';
import { connect } from 'react-redux';
import * as actionCreators from './store/actionCreators';
import './App.css';

class App extends Component {
  constructor(props) {
    super(props);
  } 
  onChange = e => {
    this.props.onChange(e);
  }
  addTodo = () => {
    this.props.addTodo()
  }
  render() {
    const { inputValue, list } = this.props;
    return (
      <div className="todo-container">
        <div className="add-todo">
          <input 
            placeholder="输入内容"
            onChange={this.onChange}
            value={inputValue}
          />
          <button onClick={this.addTodo}>提交</button>
        </div>
        <div className="todo-list">
          {
            list.map((item, index) => (
              <div>{item}</div>
            ))
          }
        </div>
      </div>
    )
  }
}
const mapStateToProps = (state) => ({
  inputValue: state.inputValue,
  list: state.list
})
const mapDispatchToProps = (dispatch) => ({
  onChange(e) {
    dispatch(actionCreators.changeInputValue(e.target.value));
  },
  addTodo() {
    dispatch(actionCreators.addTodoItem());
  }
})
export default connect(mapStateToProps, mapDispatchToProps)(App);

```

由上面的代码我们看到`mapStateToProps`和`mapDispatch`方法的使用：

前者将`store`中的数据筛选到了该组件的`props`中，供组件内部使用；

后者相当于把派发`action`到`store`这个过程封装成了一个放在`props`中的函数，当我们点击提交按钮时，就会直接帮我们派发这个`action`。我们只要在找到`props`里的这个函数，然后直接调用即可。有了上面`redux`的基础，这一块是不是很快就能轻松地理解了呢？

## 使用Redux-thunk派发异步Action

### 什么是redux的中间件

在了解`redux-thunk`之前，我们先来看看`redux`的中间件，很多同学在初期了解中间件的时候总喜欢说`react`的中间件，包括我现在的一些同事也会说`react`的中间件，这个是不对的。那么什么是`redux`的中间件呢？我们可以理解为它是一个工厂，会将我们当前的`action`做出相应的处理然后再去交给`reducer`进行进一步的处理。那么我们日常开发中一般会将请求数据的部分放到这个工厂内部进行处理，通过处理之后将拿到的数据再次的通过`reducer`进行数据的更新。如下图所示：



<img src="https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/redux-thunk.gif" alt="redux-middleware" />

由上图gif我们可以清楚的看到`redux`的数据流向以及中间件的工作机制。上述View层触发了一个事件（`event`）然后到`actions`中，通过`actionCreators`创建了一个`action`，如果我们不适用中间件的情况，那么这个`action`将会被直接派发到`reducer`中进行数据的更新等操作。但是有了中间件之后，我们不难发现，由`actionCreators`创建的`action`并没有直接到达`reducer`中，而是通过中间件去进行数据的处理（调用数据接口），然后将获取到的数据通过中间件的`dispatcher`模块再次派发到`reducer`中进行数据的更新。

### redux-thunk使用

想要使用`redux-thunk`，那肯定是先要安装了：

```bash
$ npm install redux-thunk -S
```

安装成功后，我们需要更改一下`store/index.js`，我们需要将`redux-thunk`引入：

```diff
import reducer from './reducer';
- import { createStore } from 'redux';
+ import { createStore, compose, applyMiddleware } from 'redux';
+ import thunk from 'redux-thunk';
- const devTools = window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__();
+ const composeEnhancers = window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose;
- export default const store = createStore(reducer, devTools);
+ const store = createStore(reducer, composeEnhancers(
+	applyMiddleware(thunk)
+ ));
```

现在我们还是借助上面的`todolist`，一起来过一下`redux-thunk`的使用方式，现在有这样的场景，需要拿到之前保存过的`todo-item`，这里就需要去请求数据接口了，为了方便这里就直接使用`axios`，不进行一层封装了，主要的目的是学习如何使用

首先安装一下`axios`：

```bash
npm install axios -S 
```

还记得上面我们用的`actionCreators`文件以及那张`redux`中间件的运行机制吗？我们直接上代码：

```js
// actionCreators
import axios from 'axios';
import * as constants from './constants'
getPreviousTodoItemsActions = (list) => ({
    type: constants.GET_PREVIOUS_TODO_ITEM,
    payload: list
});

export const getPreviousTodoItems = () => {
    return (dispatch) => {
        axios.get('/api/todo.json').then((res) => {
			const result = res.data.data;
			dispatch(getPreviousTodoItemsActions(result));
		});
    }
}
```

上面代码多多少少看着有点懵逼，我们来解释一下，我们暴露了一个名称为`getPreviousTodoItems`的方法，如果我们没有使用`thunk`之前，我们这里肯定返回的是一个对象，也就是`action`，但是我们使用了`thunk`之后，这里就变成了返回一个函数，该函数接受一个`dispatch`函数，那么在这个函数内部我们就可以进行接口数据的请求，然后拿到数据之后，我们还是通过函数的方式动态的创建一个`action`，将这个`action`通过回调函数的参数`dispatch`去派发给`reducer`，再由`reducer`进行数据更新。我们在`app.js`中调用：

```js
// import React, { Component } from 'react';
import { connect } from 'react-redux';
import * as actionCreators from './store/actionCreators';
import './App.css';

class App extends Component {
  constructor(props) {
    super(props);
  } 
  componentDidMount() {
      this.props.getTodoItems()
  }
  onChange = e => {
    this.props.onChange(e);
  }
  addTodo = () => {
    this.props.addTodo()
  }
  render() {
    const { inputValue, list } = this.props;
    return (
 		...html
    )
  }
}
const mapStateToProps = (state) => ({
  inputValue: state.inputValue,
  list: state.list
})
const mapDispatchToProps = (dispatch) => ({
  getTodoItems() {
      dispatch(actionCreators.getPreviousTodoItems())
  },
  onChange(e) {
    dispatch(actionCreators.changeInputValue(e.target.value));
  },
  addTodo() {
    dispatch(actionCreators.addTodoItem());
  }
})
export default connect(mapStateToProps, mapDispatchToProps)(App);
```

这样我们就完成了一个数据请求。在`mapDispatchToProps`中添加`getTodoItem`方法，然后再`componentDidMount`（或者`Hooks`中的`useEffect`）中调用即可。

但是有的同学可能会说，这样会将`actionCreators`和请求混再一起，能不能分开，这样看着会舒服点，如果你有这种想法的话，建议看一下[redux-saga](https://github.com/redux-saga/redux-saga#readme)，这也是一个`redux`的中间件，它会将数据请求和`action`分开，感兴趣的小伙伴可以看看。

## 可视化BI项目中值得学习相关的库

### immer——提高性能的神器

相信大家在开发过程中肯定会遇到一些过于复杂的数据类型，例如这样的：

```js
const userInfo = {
    name: 'indexsarrol',
    sex: '男',
    telephone: '1888888888',
    hobit: 'LOL',
    address: {
        country: 'China',
        city: {
          name: 'Nanjing',
          area: 'Jiangning',
          postcode: 210000, // 邮编号码
        },
    }
};
```

如果这些出现在`react`中的`state`时，就变成了这样：

```jsx
class UserInfo extends Component {
	constructor(props) {
        super(props);
        this.state = {
            name: 'indexsarrol',
            sex: '男',
            telephone: '1888888888',
            hobit: 'LOL',
            address: {
                country: 'China',
                city: {
                  name: 'Nanjing',
                  area: 'Jiangning',
                  postcode: 210000, // 邮编号码
                },
            }
        }
    }
}
```

如果现在有一个需求时让我们修改我们的邮编号码，这个时候我们一般会这么做：

```jsx
this.setState({
    address: {
        ...this.state.address,
        city: {
            ...this.state.address.city,
            area: 'Jiangning',
            postcode: this.state.address.city.postcode + 100
        }
    }
})
```

但是这样的写法看上去很让人很难受，写的时候也比较繁琐，有点恶心，后来我们用深拷贝把原有的对象拷贝一份，再去修改，就像这样：

```js
import { deepClone } from 'utils'
const copyUserInfo = deepClone(this.state);
copyUserInfo.address.city.postcode = copyUserInfo.address.city.postcode + 100;
this.setState({
    ...copyUserInfo
})
```

这样确实看起来清爽了很多，但还是存在了一个问题，那就是性能，如果对象过于复杂，深拷贝的性能堪忧，那么有没有什么办法能取代深拷贝的方法呢？答案是肯定的，`immer`由此而来，上图：

![image-20210412173552667](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20210412173552667.png)

*当我们调用了`immer`中的`produce`方法时，`immer`会将当前的对象存储到`immer`内部，然后通过暴露出来一个草稿对象（`draft`），我们后续的所有的改动全部都在草稿上进行，等到我们修改结束之后再将`current`按照草稿的修改再次返回一个新的对象。*

在我们当前做的项目中，因为`reducer`中我们不能更改当前的状态值是一个纯函数，这在上面我们也提到过，在`reducer`中采用了`immer`中的`produce`方法：

![image-20210412172142541](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20210412172142541.png)

`produce`方法接受两个参数，一个当前的状态值`state`，另一个是一个回调函数，回调函数中的`draft`就是`immer`内部暴露出来的草稿，后续所有的修改全部基于草稿修改即可。

例如上面的是一个注册功能的`reducer`，在`reducer`内部将`produce`方法返回出来，这样就不需要我们用先拷贝一份对象的方式处理数据了，也极大的降低了性能的消耗。

## 总结

虽然说了挺多，但是这些只是`redux`的一些基础用法，当然知道这些是远远不够的，后续我将会阅读部分源码再来将这些补充上去，不得不感慨一下，`react`的生态圈实在是太强大了。其实有些东西我这边说的可能不是特别准确，我也会继续更新，争取把每个小点都解释地通俗易懂。这篇文章也花了我不少时间去弄，虽然有些描述地不是很到位。但是我觉得对于入门这篇文章以及足够了。