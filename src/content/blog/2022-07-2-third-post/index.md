---
title: '声明式or命令式？编译时or运行时or编译时+运行时？'
slug: 'third-post'
description: 'Lorem ipsum dolor sit amet'
tags: ['Dummy Tag']
pubDate: 'Jul 22 2022'
coverImage: './blog-placeholder-3.jpg'
---

为了学习如何开发一款框架，我开始了对Vue源码的深入学习

作为一个框架的开发者，首先要考虑的一个东西——权衡！！

那么开发一款框架，有哪些东西是需要权衡利弊的？

比如：

范式上来看

`声明式or命令式`？

运行方式来看

`编译时or运行时or编译时+运行时`？

对于每一种方式都有自己的利弊，如何选择才能让框架使用效果最佳呢？

## 对于声明式和命令式：

### 了解

首先让我们来了解一下声明式和命令式的特点和区别。

`命令式`注重过程（自然语言描述能够与代码产生一一对应的关系，代码本身描述的是“做事的过程）

```javascript
const div = document.querySelector('#app'); // 获取 div
div.innerText = 'hello world'; // 设置文本内容
div.addEventListener('click', () => {
	alert('ok');
}); // 绑定点击事件
```

`声明式`更注重结果

结合这段Vue.js代码

```javascript
 <div @click="() => alert('ok')">hello world</div>
```

两段代码实现了相同的功能，显然相较于命令式更加注重结果，至于如何实现这个结果，并不关心，Vue帮我们封装了过程。

### 对比

在性能与可维护性两方面对两个范式进行对比。

#### 性能

`声明式<命令式`

**一个结论：声明式代码的性能不优于命令式代码的性能。**

原因就在于，声明式的内部实现一定是命令式。

对于命令式，修改内容时，明确知道要修改什么直接进行修改，

对于声明式，修改内容的开销是：差异比较找到要修改的内容+进行修改

如果我们把直接修改的性能消耗定义为 A，把找出差异的性能消耗定义为 B，那么有：

● 命令式代码的更新性能消耗 = A

● 声明式代码的更新性能消耗 = B + A

#### 可维护性

`声明式>命令式`

在可维护方面声明式代码可维护性更强，在采用命令式代码开发的时候，我们需要维护实现目标的整个过程，包括要手动完成 DOM 元素的创建、更新、删除等工作。而声明式代码展示的就是我们要的结果，看上去更加直观，至于做事儿的过程，并不需要我们关心，Vue.js 都为我们封装好了。

**框架开发者要做到的是，在保持可维护性的同时，尽可能地让性能损失最小化**

### 权衡解决

这里我们引入虚拟DOM

#### 虚拟DOM

由上面我们可以知道声明式代码比命令式代码在性能方面多了一个找出差异的性能消耗，也就是`B`

那么为了利用声明式的可维护性强的优点，`如果我们能够最小化找出差异的性能消耗，就可以让声明式代码的性能无限接近命令式代码的性能。`

而所谓的`虚拟 DOM`，就是为了最小化找出差异这一步的性能消耗而出现的。

**虚拟DOM与innerHTML对比**

innerHTML 创建页面的性能：HTML 字符串拼接的计算量 + innerHTML 的DOM 计算量。

虚拟DOM的性能消耗：创建 JavaScript 对象的计算量 + 创建真实 DOM 的计算量。

二者在创建页面时性能差异不大，可以认为没有差异

**但是虚拟DOM的优势展现在页面更新时**

使用 innerHTML 更新页面的过程是重新构建 HTML 字符串，再重新设置 DOM 元素的 innerHTML 属性，这其实是在说，哪怕我们只更改了一个文字，也要重新设置innerHTML 属性。而重新设置 innerHTML 属性就等价于销毁所有旧的 DOM 元素，再全量创建新的 DOM 元素。

再来看虚拟 DOM 是如何更新页面的。它需要重新创建JavaScript 对象（虚拟 DOM 树），然后比较新旧虚拟DOM，找到变化的元素并更新它。

![](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/fde727f987d5460d831d7254b55aa544~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgRW5kZWF2b3VyX1Q=:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiMjc0MjU5OTU3MzA0ODYwMSJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1755496698&x-orig-sign=tUf4Odpbu%2BY4BIPrulcWb0gH%2Fc0%3D)

表格来自《Vue.js设计与实现》

**从心智负担、可维护性和性能。三个维度进行对比**

原生DOM操作心智负担最大，但是性能高，维护性差

innerHTML 拼接HTML字符串 心智负担中等 性能差尤其是页面大时 可维护性一般

虚拟DOM 声明式的心智负担最小 性能不错 可维护性强

## 编译时和运行时

### 运行时：（不可优化）

用户提供树形结构数据对象

```javascript
const obj = {
	tag: 'div',
	children: [{ tag: 'span', children: 'hello world' }]
};
```

Render函数根据该数据对象渲染成DOM元素

```javascript
function Render(obj, root) {
	const el = document.createElement(obj.tag);
	if (typeof obj.children === 'string') {
		const text = document.createTextNode(obj.children);
		el.appendChild(text);
	} else if (obj.children) {
		// 数组，递归调用 Render，使用 el 作为 root 参数
		obj.children.forEach((child) => Render(child, el));
	}

	// 将元素添加到 root
	root.appendChild(el);
}
```

通过编写render函数将树形结构的数据对象递归渲染成DOM元素

### 编译时+运行时：（推荐）

将html标签编译成树形结构的数据对象，然后再用Render函数进行递归渲染

### 编译时：（不灵活）

直接将html文件编译成命令式代码，纯编译舍弃Render函数

### 权衡选择：

首先是`纯运行时`的框架。由于它没有编译的过程，因此我们没办法分析用户提供的内容，但是如果加入编译步骤，可能就大不一样了，我们可以分析用户提供的内容，看看哪些内容未来可能会改变，哪些内容永远不会改变，这样我们就可以在编译的时候提取这些信息，然后将其传递给 Render 函数，Render 函数得到这些信息之后，就可以做进一步的优化了。

然而，假如我们设计的框架是`纯编译时的`，那么它也可以分析用户提供的内容。由于不需要任何运行时，而是直接编译成可执行的 JavaScript 代码，因此性能可能会更好，但是这种做法有损灵活性，即用户提供的内容必须编译后才能用。

**Vue.js是声明式，编译时+运行时的框架，它在保持灵活性的基础上，还能够通过编译手段分析用户提供的内容，从而进一步提升更新性能。**

这是本栏目《从 0 到 1 读 Vue 源码：设计原理与实现细节》的第一篇文章，后续会持续更新，大家可以持续关注哦！
（本文参考文献《Vue.js设计与实现》）
