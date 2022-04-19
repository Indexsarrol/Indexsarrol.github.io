---
title: Vue源码之虚拟dom和diff算法（下）
author: Indexsarrol
date: 2022-04-17
categories: 
- Vue
tags:
- Vue
- Diff算法
---

![image-20220415220203679](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220415220203679.png)

<!-- more -->

上篇：[Vue源码之虚拟dom和diff算法（下）](https://indexsarrol.cn/2022/04/15/virtualDom-and-diff-top/)

## Diff算法——patch函数

当我们去研究`snabbdom`这个库的时候，发现DIff算法是在`patch`函数种完成的。`patch`函数则是通过`init`函数返回出来的，由于`init`函数跟`Diff`算法没有多大关系，我们在这里就不做过多解释。那么具体在`patch`函数中，Diff到底做了些什么呢？我们一步一步去看，首先，`patch`函数有两个参数`oldVnode`和 `newVnode`，然后针对新旧节点进行比较。流程图如下：

![image-20220415214849569](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220415214849569.png)

由流程图，我们可以得知，当我们去调用`patch`函数时，内部会执行以下几个步骤。

> 1. 判断旧节点是否为虚拟节点，如果不是虚拟节点，则先将该节点转换成虚拟节点。其实这里不难理解，只有大家都是虚拟节点的时候才方便比较，毕竟js对象的比较相较于真实DOM可好比较太多了；
> 2. 如果新旧节点都为虚拟节点，则判断新旧虚拟节点是否为同一个节点，如果为同一个节点，则进行深层次的比较。如果不是，则直接把旧节点删除，将新节点直接插入。

接下来，我们来一起简单的手写一下`patch`函数。新建`patch.js`文件，首先导入我们之前写好的`vnode.js`：

```js
import vnode from './vnode.js';
function isSameVnode (oldVnode, newVnode) {
    return oldVnode.key === newVnode.key && oldVnode.sel === newVnode.sel;
}

export default function patch(oldVnode, newVnode) {
    // 1. 判断当前传入的旧节点是否为虚拟节点
    if (!oldVnode.sel) {
        // 2. 将真实节点转换成虚拟节点
        oldVnode = vnode(oldVnode.tagName.toLowerCase(), {}, [], undefined, oldVnode);
    }
   	// 3. 判断新旧节点是否为相同节点
    if (isSameVnode(oldVnode, newVnode)) {
        // TODO: 进行细节化的比较
    } else {
        // 4. 暴力拆除旧的，插入新的节点
    }
}
```

### createElement方法

当我们写到这里时，我们需要去拆除旧的，插入新的，这里的拆除和插入都是针对真实节点来的，所以我们需要先去封装一下常用的创建方法。在当前目录下新建`createElement.js`文件，用于将虚拟节点创建成真实节点。以便上树使用，那么这里的`createElement.js`内部做了以下的事：

1. 将传入的vnode转换成真实DOM节点；
2. 判断子节点类型；
3. 如果子节点不是文本类型，则遍历子节点的数组，递归调用`createElement`方法，生成真实DOM再依次填充到上一次生成的DOM节点上；
4. 如果子节点为文本类型，则将`vnode.text`直接赋值给创建出来的真实DOM的`innerText`；
5. 将真实DOM挂载到`vnode`的`elm`属性中；
6. 返回真实 DOM。

具体流程图如下：

![image-20220417204508747](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220417204508747.png)

具体代码如下：

```js
// createElement.js
// 将vnode创建为真实节点，插入到pivot元素之前
export default function createElement(vnode) {
    // 1. 将传入的虚拟节点创建为真实节点
    const realDom = document.createElement(vnode.sel);
    // 2. 判断虚拟节点的子节点类型
    if (vnode.text !== '' && (vnode.children === undefined || vnode.children.length === 0)) {
        // 3. 如果为文本类型，则直接将vnode.text直接赋值给创建出来的真实DOM的innerText
        realDom.innerText = vnode.text;
    } else if (Array.isArray(vnode.children) && vnode.children.length > 0) {
        // 4. 如果子节点不是文本类型，则遍历子节点的数组，递归调用createElement方法
        for (let i = 0; i < vnode.children.length; i++) {
            const child = vnode.children[i];
            let childDom = createElement(child);
            realDom.appendChild(childDom);
        }
    }
    // 5. 将真实DOM挂载到`vnode`的`elm`属性中
    vnode.elm = realDom;
    // 6. 将真实的DOM返回出去
    return realDom;
}
```

那么我们这个时候再回到`patch.js`文件中，引入刚刚写好的`createElement.js`文件。

```JS
...
import createElement from './createElement.js';

export default function patch(oldVnode, newVnode) {
    ...other code
   	// 3. 判断新旧节点是否为相同节点
    if (isSameVnode(oldVnode, newVnode)) {
        // TODO: 进行细节化的比较
    } else {
        // 4. 暴力拆除旧的，插入新的节点
        let newVnodeElm = createElement(newVnode);
        // 插入到老节点之前
        if (oldVnode.elm.parentNode && newVnode) {
            oldVnode.elm.parentNode.insertBefore(newVnodeElm, oldVnode.elm);
        }
        // 删除老节点
        oldVnode.elm.parentNode.removeChild(oldVnode.elm);
    }
}
```

### patchVnode方法

由上述我们可知，当新旧节点在进行比较的时候，如果`sel`属性和`key`属性相同，则被认为是同一个节点（不是内存地址的相同），进而调用`patchVnode`方法，`patchVnode`方法无非做了这几件事：

- 新节点有`text`属性，旧节点有`text`属性   ===>   将新`text`覆盖旧`text`;
- 新节点有`text`属性，旧节点有`children`属性   ===>  将新`text`覆盖旧`children`;
- 新节点有`children`属性，旧节点有`text`属性   ===>  将旧节点`text`置空，将新`children`追加；
- 新节点有`children`属性，旧节点有`children`属性   ===>   调用`updateChildren`方法，优化更新策略。

所以话不多说，直接上流程图：

![image-20220417213449350](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220417213449350.png)

通过上述流程图我们可以清楚的看到这四种的比较情况；针对这四种情况分别如何处理，流程图看的很清楚，这里就一一解释了，以下贴出代码（非官网，自己手写）：

```js
function patchVnode(oldVnode, newVnode) {
    // 1. 如果新旧节点是内存地址中的相等，则直接return
    if (oldVnode === newVnode) return;
    // 2. 判断新节点是否存在text属性
    if (newVnode.text && !newVnode.children?.length) {
        // 3. 判断旧节点是否存在text属性，如果存在则直接替换，不存在的话，则代表有children，一律使用innerText强			制替换
        if (newVnode.text !== oldVnode.text) {
            oldVnode.elm.innerText = newVnode.text;
        }
    } else { //  新节点存在children属性
        // 4. 旧节点存在children属性
        if (oldVnode?.children?.length > 0) {
            // TODO 调用updateChildren方法
            // updateChildren(oldVnode, newVnode);
        } else {
            //5. 旧接点存在text属性, 将旧节点的text设置为空字符串，然后将新节点的children处插入到就节点的子节点
            oldVnode.elm.innerText = '';
            // 6. 遍历新节点的children，生成真实dom，然后appendChild
            for (let i = 0; i < newVnode.children.length; i++ ) {
                let chDOM = createElement(newVnode.children[i]);
                oldVnode.elm.appendChild(chDOM);
            }
        }
    }
}
```

## updateChildren方法

整个diff算法中，最核心的估计就是这个方法了，他是用来比较新旧节点都存在子节点的情况的，做到最小量更新，那么这个方法中做了一个很核心的命中规则，而且引入了4个比较指针，分别为新前、旧前、新后、旧后，那么这是什么意思呢？

- 新前：指的是新的虚拟节点最开始的节点；
- 新后：指的是新的虚拟节点中最后的节点；
- 旧前：指的是旧的虚拟节点最开始的节点；
- 旧后：指的是旧的虚拟节点中最后的节点；

而这里同时也有一个命中规则，分别是：

1. 新前节点与旧前节点比较；
2. 新后节点与旧前节点比较；
3. 新后节点与旧前节点比较；
4. 旧后节点与新前节点比较；

比较逻辑如下：

当命中以上某个规则时，即节点比较之后相等，则让新节点与后节点的指针发生变化，举个例子：

当新前节点与旧前节点相等时，则让新前节点的指针向下移动一位，同时让旧前的节点对应的指针也向下移动一位；

当新后节点与旧后节点相等时，则让新后节点的指针向上移动一位，同时让旧后的节点对应的指针也向上移动一位；

如果没有命中的话，则依次向下执行命中规则。在这里其实是有个循坏的，循坏条件是：

```js
while(新前 <= 新后 && 旧前 <= 旧后)
```

也就是说，只要当我们的后节点的指针大于前节点的指针，循环就会停止。循环停止后，又分为两种情况：

1. 如果是旧节点先循环完毕，则说明新节点中有要插入的节点（即新前与新后之间的节点）
2. 如果是新节点先循环完毕，如果老节点中还有剩余节点，则删除剩余节点（即旧前与旧后之前的节点）；

### 新增节点

具体的细节我们直接放入例子中做解释即可，要不然太枯燥了。假设我们现在有如下节点，现在开始进入比对：

![image-20220418215956405](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220418215956405.png)

- 第一步，判断新前和旧前节点是否相同，由上图发现是相同的，则旧前和新前的指针都向下移动一位；

![image-20220418220247743](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220418220247743.png)

- 第二步，还是判断新前和旧前节点是否命中，此时还是命中，则着旧前和新前的指针向下移动一位；

![image-20220418220447483](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220418220447483.png)

- 第三步，还是判断新前和旧前节点是否命中，此时还是命中，则着旧前和新前的指针向下移动一位；

![image-20220418220759823](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220418220759823.png)

这是我们发现，旧节点的旧前已经小于旧后了，那么循环终止。这时我们可以得出，是旧节点先循坏结束，则将新前与新后之间的节点，这里是D、E直接插入到真实节点中（其实是插入到旧节点中，再返回旧节点之后，通过`createElement`方法创建出真实节点）。

### 删除节点

前面我们说了新增节点，这一节我们来看一下删除节点的例子，删除和新增一样，都是先按照4中命中规则来判断并且加上我们的条件。我们还是先来看图：

![image-20220419201804391](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220419201804391.png)

这里我们可以分析得出，在此次更新节点中，我们发现，新节点中将D这个节点删除了，那么其内部如何去做对比的呢？我们还是按照前面所提到的四种命中方法来做对比：

- 第一步，旧前和新前比较，结果：命中，则前指针都向下移动一位；

![image-20220419201733783](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220419201733783.png)

- 第二步，接着比较旧前和新前，结果：命中，指针接着向下移动一位；

![image-20220419201650816](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220419201650816.png)

- 第三步，接着比较旧前和新前，结果：不命中，则采用第二种命中方式，结果，命中，则后节点指针向上移动一位；

![image-20220419202045816](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220419202045816.png)

这里我们发现，新后节点的指针已经大于新前节点的指针，所以循环终止。这次是新节点首先遍历完成，则将旧前和旧后指针之间对应的节点删除，在这里就是C节点，对应的真实DOM节点也会被删除。

![image-20220419202345669](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220419202345669.png)

### 复杂情况

前面我们看了一个新增节点和删除节点，这里我们来个反转节点，这一步呢，就是为了让我们能够知道后两种命中方式，也就是新后与旧前和新前与旧后的对比。来看例子：

![image-20220419203232662](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220419203232662.png)

这里我们可以看到，新节点和旧节点的顺序已经完全颠倒，这时候我们就可以命中后两条规则了。

- 第一步，旧前新前比较，没有命中；
- 第二步，旧后与新后比较，没有命中；
- 第三步，新后与旧前比较命中，涉及到移动节点。将旧前指针对应的节点设置为`undefined`，并且在旧后对应指针的后面新增一个节点，节点为与旧前匹配到的节点，这里即为A节点。然后新后指针向上移动一位，旧前指针向下移动一位。我们直接用图表示，设为`undefined`:

![image-20220419204335207](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220419204335207.png)

`并且在旧后对应指针的后面新增一个节点，节点为与旧前匹配到的节点，这里即为A节点`：

![image-20220419204851244](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220419204851244.png)

`然后新后指针向上移动一位，旧前指针向下移动一位。`：

![image-20220419204927958](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220419204927958.png)

在这里由于篇幅的关系，其余的节点小伙伴们可以自己试着按照这种方法推导，这里就不过多的去写了。

### 四种命中规则都未命中的情况

由上可以清楚的知道，命中方式总共有四种，但是还是覆盖不了所有情况，这个时候就需要我们单独进行处理，处理逻辑如下：

- 当新前或新旧节点的指针指向新节点的某个节点时，这时去遍历旧节点，去查找新节点指针所对应的节点；
- 如果找到了，将旧节点的那一项设置为`undefiend`，然后将新节点插入到`undefined`那一项之前即可。
- 如果没有找到，说明新节点指针对应的那一项是新增的节点，然后插入到旧前节点之前即可。

这么说还是太过于枯燥，直接上案例：

![image-20220419214551589](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220419214551589.png)

- 第一步，新前与旧前比较，命中，指针下移；

![image-20220419214647626](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220419214647626.png)

- 第二步，旧前新前，不命中。旧后新后，不命中。新后旧前，命中，
  - ①则设置旧前对应节点为`undefined`；
  - ②将新后节点插入到旧后之后；
  - ③同时让新后节点对应指针上移，旧前节点对应指针下移；

![image-20220419215510561](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220419215510561.png)

- 第三步，旧前新前，不命中。旧后新后，不命中。新后旧前，不命中。新前旧后，命中，
  - ① 设置旧后对应指针节点为`undefined`；
  - ② 将当前新前节点指针对应的节点，插入到旧前节点的前面（旧前之前）；
  - ③ 让新前节点指针下移，旧后节点指针上移动；

![image-20220419220303384](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220419220303384.png)

- 第四步，旧前新前，不命中。旧后新后，不命中。新后旧前，不命中。新前旧后，不命中，则进入都未命中的阶段；
  - ① 遍历旧节点，是否能在旧节点中找到新节点指针对应的节点；
  - ② 如果找到，将旧节点的那一项设置为`undefiend`, 将新节点指针对应的节点插入到设置`undefined`节点之前；
  - ③如果未找到，说明新节点指针对应的那一项是新增的节点，然后插入到旧前节点之前即可。
  - ④ 旧后节点指针向上移，新前节点指针下移；循环结束，此时为新节点先循环结束，则删除旧前和旧后之前的节点；

![image-20220419221842789](https://cdn.jsdelivr.net/gh/Indexsarrol/image/blogs/image-20220419221842789.png)

这是我们将真实节点与新节点进行对比，发现结果相同，所以算法执行完毕，比较结束。

## 总结

以上就是真的diff算法的核心思想，当前，在这里我们由于篇幅的问题，我们并没有将所有的情况都列举在上了，但是只要通过这几种命中方法加未命中判断原则，就能解决一切节点更新思路。不得不说，率先相出这个算法的人是多么的牛叉。
