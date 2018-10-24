---
layout: post
title: React中setState中对于数组，对象操作的问题
categories: React
description: React setState的坑
keywords: React  JS
---
## 前言
从两个月前开始维护一个React的项目，基本都是一些功能上的迭代，说实话这对于用习惯了Vue的人来说，转React还是有不少思想上的转变的。首先遇到的一个坑就是setState的问题，对于setState很多人褒贬不一，但说实话从个人角度来说如果某个API在使用的过程中，让使用者产生困扰，这个API就是有缺陷的。虽然话是这么说的，但坑还是要填的。

## 正文
对于有setState使用的困扰总结了一下主要集中在下面三个方面  

- **setState是异步更新的**  
这个问题应该是setState争议最大的问题，首先我们要明白当我们使用this.setState()的时候，它并不是实时对状态进行了修改，这是一个异步过程，也就是我们需要等待JS执行队列完成后才能真正得到修改后对的状态。
``` javascript
//初始化设置count为1
state = {count:1};

doClick=()=>{
    this.setState({count:2});
    console.log(this.state.count);
}

render(){
    return <button onClick={this.doClick}>Click Me</button>
}
```
这里我们定义了一个doClick()方法，当点击按钮时，对state中对的count值进行改变，我们会发现第一次点击按钮后，控制台输出的并不是修改后的值2，而仍然是1。很显然，这是因为setState是异步执行对的原因。  
  
这也就引发了一个问题，既然setState是异步的，它的真正运行时间我们又无法保证，那如果我们需求就是要在setState之后里吗进行某些操作怎么办呢？还好，React开发团队给我添加了第二个参数，这是一个回调函数，它会在setState状态发生变化后执行 ，这样我们就能精确对的执行我们的代码了,我们将刚才的doClick()方法修改一下。
```javascript
doClick=()=>{
    this.setState({count:2},()=>console.log(this.state.count));
    
}
```
这时候我们发现输出count的值已经是修改后的2了。

- **setState会合并**  
在说状态合并前我们思考一下，为什么setState要作为异步方法来修改状态呢？按照正常思路来说，同步不应该更加符合命令式编程么(虽然命令式编程一直被人诟病，但人家确实直观啊)？  
我们对状态修改，最终都要反应到视图上，state是作为视图的一个数据抽象，既然要反映到视图上，我们就需要对视图进行渲染，如果每次轻微的改变都对视图进行渲染显然会造成极大的性能损耗，这时候我们就考虑是不是应该将几次相同状态变更合并，以此减少渲染的次数？并且如果一个状态变更需要导致父组件的状态的变更，同步就显得很不合适了。  
要记住 ，React 模型更愿意保证内部的一致性和状态提升的安全性，而不总是追求代码的简洁性。

```javascript
this.setState({count:this.state.count+1});
this.setState({count:this.state.count+1});
this.setState({count:this.state.count+1});
//此时state中count的值是多少呢？
```
很可惜，最终count的值是2，并不是我们想象的4，这是因为React将三次setState合并为了一次。那如果我们就是要将它变为4怎么办呢？React给我们提供了一个函数setState
```javascript
this.setState((preState,props)=>({count:preState+1}))
this.setState((preState,props)=>({count:preState+1}))
this.setState((preState,props)=>({count:preState+1}))
```
或者我们也可以用上面的回调函数方法，但这样显然跟合适一些。

- **突变数据的坑**  
在快速上手react的过程中，我碰到最花时间的问题并不是上面提到的两种，而是突变数据导致的无法更新视图的问题。因为快速上手，当时文档看的也不是很详细，后来仔细看到的时候才恍然大悟，原来说的就是这么点破事啊！

首先我们要知道什么是突变数据，什么是不可突变数据。可突变数据指的是对数据的操作是作用在自己本身上，导致了自身数据结构的变化。而不可突变数据则表示对数据的操作会产生一个新的数据，它并不作用在自己身上。我们来举个🌰：  
```javascript
var list = [1];
list.push(2);
//数组就是一个突变的数据类型，push方法改变的是list自身，此时的list中的数据相对之前的list已经发生了变化，但是它们依然还是同一个引用
```
虽然list中实际的数据发生了变化，但它并没有生成一个新的对象，这也就是导致并不会发生渲染的原因。我们修改一下:
```javascript
var list = [1];
var newList = list.concat(2);
```
concat()方法会返回一个新的数组，这样就会重新渲染了。  

原生JS中对不同数据的操作：  
1.**不可突变数据**  
数字，字符，布尔值，Null,undefined  (基本数据类型)  
2.**数组**
```javascript
//方法1
var list = this.state.list;
this.setState({
    list:list.concat('otherData')
})

//方法2
this.setState({
    list:[...list,'otherData']
})
//数组中对数组操作返回新对象的方法包括：
//concat,slice
```
3.**对象**
```javascript
//方法1 利用Object.assign方法
// user = {name:'BZ'}
var user = this.state.user;
this.setState({
    user:Object.assign({},user,{age:18})
})
//方法2 使用对象扩展语法
var user = this.state.user;
this.setState({
    user:{...user,age:18}
})
```

以上方法总结来说就是返回了一个新的对象，并不是修改原有数据。当然了，也可以按照React官方的推荐Immutable.js进行,但秉承能用原生就用原生的精神，上面的方法还是有很大用处。  

这里只是对React中setState开发遇到一些问题进行了一些解释跟方案，只有setState的合并是如何实现的，以及为什么React推荐组件的状态是不可改变的原因过段时间再写文章慢慢填坑吧。