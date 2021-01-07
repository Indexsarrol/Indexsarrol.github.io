---
title: TypeScript从入门到精（fang）通（qi）
author: Indexsarrol
date: 2020-12-30 19:11:58
categories: 
- TypeScript
tags:
- TypeScript
---

## 前言

​	TypeScript在这几年前端开发者可能是必会的一项技术了，它是`javascript`的一个超集。就类似我们之前学过的`css`，这个`css`就相当于是我们原生的`js`，但是当我们真正开发项目时，我们可能不会用原生的`css`，我们可能会使用`less`、`sass`、`stylus` 等语言实现我们想要的`css`，因为使用这些解释器就可以将我们的代码解析成`css`。这个过程同样适用于`js`。我们都知道，`js`是一门弱类型的语言，什么是弱类型语言呢？就是说，它内部的一些类型，都是很松散的。学过C、C++ 、以及Java 的都知道，当我们声明了一个变量，这个变量就会被赋予一个类型（整形，浮点型等）,然而弱类型并没有这种优势。js：不管你来什么数据，小爷我都是照单全收~~，此时TS就是为了解决这个问题而被创作出来的。现在就让我们一起看看吧！
<!-- more -->
## 安装

在学习之前，我们还是来一起看一下如何安装typescript，如果连开发环境都运行不起来，那一切都还有什么意义呢？

1. 通过 `npm` 进行全局安装：

   ```shell
   $ npm install  -g typescript
   ```

2. 安装完成后，我们新建一个名为 `test.ts` 的文件：

   ```typescript
   function greeting(sth: string) {
       return `I am learning ${sth}`
   }
   let results = greeting('TypeScript');
   console.log(results);
   ```

3. 打开控制台，然后用编译命令 `tsc` + 我们的文件名：

   ```
   $ tsc test.ts
   ```

​	这个时候我们会看到一个文件名为 `test.js` 的文件，这个就是将我们 `test.ts` 通过 `tsc` 命令解析出来后的js文件，里面是原生js的代码

4. 然后我们利用命令行去执行 `test.js` 的代码，在这之前请确保你的机器上安装了`node`：

   ```shell
   $ node test.js  // 'I am learning TypeScript'
   ```

## 开始

### 基础类型

​		无论什么语言，都会有基础类型，我们这次就直接拿`js`与 `ts`做个比较学习，首先我们复习一下`js`的基础类型有哪些：`String`、`Number`、`Boolean`、`Null`、`Undefined`、`Symbol`。那么在`ts`中，我们也有自己的基础类型，并且在`js`的基础上，新加了几种基础类型，`TS`的基础类型有：`string`、`number`、`boolean`、`any`、`undefined`、`null`、数组类型、元组类型、枚举类型、`void`类型、`Never`类型、`object`类型、类型断言。接下来我们一个一个的看：

#### String类型

```typescript
let str: string = 'indexsarrol';
str = 123; // 会报错，因为声明了str为string类型的变量
```

​		由上述代码我们可以知道，如果我们想为一个变量去设置某一种类型，可以在变量后面添加 ’：类型名称‘。在这里我们声明了有一个变量`str`并给了 `str` 一个类型 `string` ，这就表示这个变量只能接受字符串类型的变量，如果我们给它设置了`str = 123` ，此时就会报错 `Type '123' is not assignable to type 'string'` 。

#### Number类型

```typescript
let num: number = 123;
num = '123'; // 会报错，因为声明了num为number类型的变量
```

在赋予了一个变量为`number`类型后，该变量就和`js`中的数字没有任何区别，所以方法以及属性完全使用。

#### Boolean类型

```typescript
let bool: boolean = false;
bool = '123'; // 会报错，因为声明了bool为boolean类型的变量
```

#### Any类型

话说刚开始写`ts`的时候，各种不习惯，然后各种写`any`，很显然这个确实不太合理。

```typescript
let anyVar: any;
anyVar = 'str';
anyVar = 123;
anyVar = false;
// 以上方式都是合法的
```

any类型我们可以单纯的理解为就是原生js中的变量，是一种非常松散的类型，如果声明了该类型，那么这个变量可以赋值为任意值。

#### 数组类型

在TS中，数组的表示方法有两种：

```typescript
// 方法1
let arr: Array<number> = [1, 2, 3] // 尖括号内部写的是该数组内部的每一项是什么类型，在这里，内部都是number类型
arr = [1, '2', 3'] // 报错， 因为内部定义的是number类型
// 方法2
let arr number[] = [1, 2, 3] // 推荐使用，因为可能在jsx语法中，尖括号这种可能会被识别成jsx语法
arr = [1, '2', 3'] // 报错， 因为内部定义的是number类型
```

这里我们可能会有点疑惑？数组既然这样的话，那就没有办法实现内部每一项不同类型的情况了，别急，我们接着往下看。

#### 元组类型

我们上述说到了数组类型，但是我们如果想设置数组内部为不同类型的话，我们就可能需要用到元组类型了。元组类型允许表示一个已知元素数量和类型的数组，各元素的类型不必相同：

```typescript
let arr: [number, boolean, string];
arr = [1, false, '1']; // 正确
arr = ['2', true, '234']; //报错
arr = [1, false, '1', undefined] // 报错 
```

这个时候我们可以看到声明了arr数组之后，如果内部的类型不对的话或者length不对的情况下，则会报错。

#### 枚举类型

枚举类型，通常表示一些常量的组，例如：人的性别，一年四季等；我们用关键字 `enum` 声明：

```typescript
enum Sex {
    Male,
    Female
}
let indexsarrol: Sex = Sex.Female;
// 这里indexsarrol这个变量 只能取Male/Female
console.log(Sex.Male); // 0
console.log(Sex.Female); // 1
Sex.Male = 10;
Sex.Female // 11
Sex[0] = 'Male';
Sex[1] = 'Female';
```

这里我们可以看到，如果我们打印`Sex.Male` 是为 0 ；这个就是`Sex`中`Male`的值；如果我们设置了`Male = 10` 的话，则`Female` 则为11，由此我们可以知道，如果前一个类型被赋值了，则下列类型的值则递增。

如果我们直接访问`Sex[0]`的话，我们会拿到`Male`；这让我们联想到了对象，那么这个`enum`是怎么实现的呢？我们可以看看：

```typescript
var Sex;
(function(Sex){
    Sex[Sex['Male'] = 0] = 'Male';
    Sex[Sex['Female'] = 1] = 'Female';
}(Sex || {}))
```

这里其实我们可以看到 首先声明了一个对象，这里的写法可能有点同学未必能看懂，我们可以转换一下：

```typescript
var Sex = {
    Male: 0,
    Female: 1,
    0: 'Male',
    1: 'Female'
}
```

这样我们就可以看清楚了，其实就是一个对象，其内部放入了4个属性，这样也就不难理解我们之前些的一些代码了。

#### Void类型

`Void`类型可以说是与`Any`相反的一个类型用来表示没有任何类型，一般我们在日常开发中，一般用于函数是否存在返回值：

```typescript
let arr: number[] = [1, 2, 4];
let num: number = 10;
function test(arr: number[], num: number): void {
    console.log(arr.length + num);
    return arr.length + num; // 报错，test函数设置了没有返回值void，所以return语句就会报错。
}
test();
```

#### Never类型

`Never`类型表示那些永远不存在的值，`never`类型是那些总是会抛出异常或根本就不会有返回值的的函数表达式：

```typescript
function test(): never {
    throw new Error('报错了');
}
test();

function test2():never {
    while(true) {}
}
test2();
```

#### Object类型

`Object`类型就是指定一个变量为`object`类型但是这个地方需要我们注意的是，数组也是一个对象，所以当我们声明了变量为`object`，给他赋值为一个数组是没有问题的：

```typescript
let obj: object = {};  // 可以
let arr: object = []; // 可以
let str: object = '2345'; // 报错
```

#### 类型断言

有时候你会遇到这样的情况，你会比`TypeScript`更了解某个值的详细信息。 通常这会发生在你清楚地知道一个实体具有比它现有类型更确切的类型。

通过**类型断言**这种方式可以告诉编译器，“相信我，我知道自己在干什么”。 类型断言好比其它语言里的类型转换，但是不进行特殊的数据检查和解构。 它没有运行时的影响，只是在编译阶段起作用。 `TypeScript`会假设你，程序员，已经进行了必须的检查。

类型断言有两种形式：

```typescript
let str: any = 'Indexsarrol';
// 方式1
let len: number = (<string>str).length;
// 方式2
let len:number = (str as string).length; // 推荐使用
// 其实类型断言的本质就是强制改变一个变量本身存在的类型
```

未完待续...