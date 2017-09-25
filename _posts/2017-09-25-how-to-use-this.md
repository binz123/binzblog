---
layout: post
title: JS基础解析：this关键字详解
categories: javascript
description: 详解this关键字
keywords: javascript 基础 入门
---

## 说明

这两天项目在招人，面试了不少比我工作年限要长的前端程序员，但总的来说不是很满意，问的基础问题虽然都能答上来，但感觉死记硬背的那种，给我的感觉就是一知半解的，仅仅是为面试而突击的，越来越觉得程序员真正重要的不是写代码，而是解决问题的能力。最近决定把js基础部分重新写个系列文章，也当是自己复习巩固了吧。

这些文章主要围绕的是JS入门时的三个难点，this关键字，闭包，原型链，虽然网上相关的文章很多，但应该算是我学习时的思路吧，我也跟列出ES5跟ES6不同的地方，供后人参考。

## this关键字
this关键字提供了一种优雅的传值方式，它是在运行时自动定义到所有函数的作用域中的。

运行时是指this的绑定是在函数运行时进行的，与一般对象绑定是与函数声明的位置相关相比，this的绑定并不取决于函数声明的位置。


__this的绑定与函数声明的位置无关，只取决于函数在哪里调用__

网上很多文章写的总结又是window对象下，又是apply方法里，又是call方法里，其实不需要看得这么头疼，许多优秀的JS入门书籍里都已经指出
绑定规则主要包含三种：
1. **默认绑定**
2. **隐式绑定**
3. **显式绑定**

## 函数调用位置
顾名思义，函数调用位置就是一个函数被调用的位置。也许你会认为函数的调用为止如同a()这样表达式，但是有时候真正的调用为止是会被隐藏起来的，我们要做的就是拨开层层迷雾（调用栈），找到当前正在执行的函数的前一个调用。来段代码吧(真正工作的时候千万不要像我下面这样随便命名喔)。
```javascript
function a(){
    console.log('a');//当前执行函数是a，其调用位置为全局函数中
    b();            //调用栈:a
    
}
function b(){
    console.log('b'); //当前执行函数是b，其调用位置为a中
    c();              //调用栈 a-->b
}
function c(){
    //当前执行函数是c，其调用位置为b中
    //调用栈a-->b-->c
    console.log('c'); 
}
a(); 
```

## 默认绑定
默认绑定既在其他规则无法应用时默认采取的规则。
```javascript
function myfun(){
    console.log(this.a);
}
var a = 2;
myfun();        //2
```

我们在全局作用域里定义了一个函数myfun与一个变量a，当我们调用test()时，this.a被解析为全局作用域中的a，这是因为myfun与变量a都被定义成了全局对象中的属性，既window.a = a;

且由于调用myfun是作为单独变量出现的（其实这里我觉得这么表述有问题，它其实是作为window对象的属性出现的），所以应用默认规则后，this指向了全局对象window，从而此时this.a输出的就是window.a中的值。

**在其他规则无法应用时且调用函数作为单独变量出现，this指向全局对象window**

但该规则仅限在非严格模式，在严格模式(use strict mode），全局对象将不用于默认绑定，此时this将绑定到undefined上。
```javascript
function myfun(){
    "use strict";
    console.log(this.a);
}
var a = 2;
console.log(a);    //TypeError:this is undefined
```
如果我们在函数中使用了严格模式，则此时默认规则将不会生效，只有在非严格模式下，this才会默认绑定到全局对象上。

## 隐式绑定
我们在给默认规则下定义时，有一条作为单独变量出现，这里指的单独变量说的就是是否被某个对象所拥有或者包，换句话说就是其调用位置是否存在上下文。当函数引用上下文对象时，this将会被绑定到这个上下文对象。
```javascript
var a =2;
function myfun(){
    console.log(this.a);
}
var oneperson = {
    a:100,
    saySth:myfun,
}
onperson.saySth();
```

在这段代码中，我们定一个全局变量a，全局对象oneperson以及一个函数myfun，在oneperson中，我们将函数myfun作为对象oneperson的saySth属性，并调用oneperson的saySth属性，此时发现像上面默认规则一样输出2，而是输出oneperson属性中a的值100。

由此可发现this在myfun()运行时并没有绑定到全局对象上，而是绑定到它的引用oneperson。此时的this.a既为oneperson.a

**当函数引用上下文对象时，隐式绑定规则会将函数调用中this绑定到这个上下文对象中**

既然弄懂了隐式绑定，那就不得不提一下隐式绑定需要注意事项：

1.this只会绑定到对象属性引用链的最后一层
```javascript
function basefun(){
    console.log(this.a);
}
var obj1 ={
    a:2,
    basefun:basefun,
}
var obj2 = {
    a:4,
    obj1:obj1,
}
obj2.obj1.basefun();   //2,最后一层是obj1
```

2.隐式丢失。当隐式绑定丢失绑定对象时，将应用默认绑定规则，既this将绑定到全局对象或者undefined上（严格模式）

那么什么时候会出现隐式丢失，或者说隐式绑定丢失绑定对象到底是怎么回事呢？
```javascript
function myfun(){
    console.log(this.a);
}
var obj = {
    a:100,
    myfun:myfun,
}
var b = obj.myfun;     //此时发生了丢失
b();   //undefined
```

我们会发现上面的代码输出的并不是obj.a的值，这证明myfun函数中this并没有绑定到obj对象上，之所以会出现这种现象，是因为我们在执行var b = obj.myfun;时，其实是将myfun的引用地址复制给了b，也就是说b与obj.myfun指向的是同一个地址，b()其实就是在不带任何修饰的调用myfun。（用个广告词就是：没有中间商赚差价）。

至于为什么会发生隐式丢失，这就涉及到js中的深拷贝与浅拷贝了，var b = obj.myfun实际上仅仅只复制了引用的地址而已，复制之后原来的变量与现在的变量指向的是同一个东西。

## 显示绑定
刚开始学习this关键字的时候，对显示绑定挺害怕的，其实哪个时候是因为对call(),apply()方法并不熟悉，后来才发现显示绑定才是三个默认规则里最简单的。首先来复习一下call()与apply()函数的定义。

a. call()

定义：调用一个对象的一个方法，以另一个对象替换当前对象。

解释：call方法可以将一个函数的对象上下文改变为指定对象的上下文，如果没有提供另一个对象的参数，将默认指向全局对象。

b. apply()

定义：应用某一对象的一个方法，用另一个对象替换当前对象。

解释：其实apply()方法与call()方法的作用是**完全一样**的，只是二者的参数有所不同而已，同理当没有提供另一个对象时，将默认指向全局对象。

call()方法与apply()方法的出现，就是为了改变函数运行时的上下文，也可以说就是为了改变函数内部的this指向而存在的。（因此我个人更喜欢把显示绑定称为强制绑定）

```javascript
function myfun(){
    console.log(this.a);
}
var oneperson = {
    a:2
}
myfun.call(oneperson);   //2
```

在这段代码中,myfun并没有任何前面的修饰符，也就是说它并不适用与隐式绑定，但是通过call()方法，我们指定了它的绑定对象，也就是oneperson对象，此时myfun函数运行的上下文也被强制改变为oneperson了，其this对象也指向为oneperson对象，所以this.a输出的就是oneperson的值。

如果我们在call()与apply()方法中使用的参数是null或者undefined又会发生什么神奇的效果呢？
```javascript
var a = 100;
function myfun(){
    console.log(this.a);
}
myfun.call(null);//2 or undefined
myfun.call(undefined); //2 or undefined
```
我们会发现此时myfun函数的执行上下文被改变为全局对象了，此时应用了的是默认绑定，所以根据默认规则分别在非严格模式与严格模式下输出的是2或者undefined。

## setTimeout与setInterval
我们来看一下下面的代码：
```javascript
var a = 2;
function myfun(){
    setTimeout(function(){
        console.log(this.a);
    },1000);
     setInterval(function(){
        console.log(this.a);
    },1000);
}
var obj1 = {
    a:100,
    test:myfun,
}
obj1.test();
//2 or undefined
//2 or undefined
```
这里我们采用和隐式绑定相同的方式，将test()方法绑定到obj1对象上，只是在myfun()函数中封装了一个setTimeout()方法与setInterval()方法，如果按照隐式绑定的规则，此时obj1.test()应当输出100，但结果却并不是。造成这个结果的原因的是因为**setTimeout和setInterval调用的代码运行在所在函数完全分离的执行环境上**，当前代码下如果是在非严格模式下就会this就会绑定到window对象上，严格模式下则为undefined，这和所预期的this值是不一样的。详情请见：<a href="https://developer.mozilla.org/zh-CN/docs/Web/API/Window/setTimeout" title="MDNsetTimeout">MDN中对setTimeout的解释</a>

## 箭头函数
既然this这么调皮，总是根据执行环境而发生变化，在多次调用后难免会让人头疼，于是在ES6中给我们提供了箭头函数解决这个问题。先把结论抛出来，再看代码是如何的。

**箭头函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象**

结合上文中setTimeout和setInterval来解释这个问题
```javascript
var a = 2;
function myfun(){
    setTimeout(()=>{
        console.log(this.a);
    },1000);
    setInterval(()=>{
        console.log(this.a);
    },1000);
}
var obj1 = {
    a:100,
    test:myfun,
}
obj1.test();
//100
//100
```
这段代码与上面setTimeout与setInterval中代码一模一样，唯一不同在于setTimeout中我们使用的是ES6的箭头函数。如果是普通的函数，执行时setTimeout中函数的this应当指向window对象或者undefined，但是箭头函数使得this总是指向**函数定义生效时所在的对象**(本列中是obj1)，所以输出是100。

箭头函数固定了this的指向，这使得非常有利于封装回调函数。

```javascript
var handler = {
    id:'123456',
    init:function(){
        document.addEventListner('click',
        event=>this.doSomething(event.tyoe),false);
    },
     doSomething: function(type) {
    console.log('Handling ' + type  + ' for ' + this.id);
  }
}
```
如果我们不用箭头函数来实现这个函数的回调，但运行init()方法时，this指向的应该是document对象，这样就会报错。但是由于箭头函数在定义时就已经绑定了对象，此时this的指向是handler对象。

关于更详细的箭头函数解析，请参见阮一峰的<a href="http://es6.ruanyifeng.com/#docs/function#严格模式" title="sorrowfunction">《ES6函数扩展》</a>。

会不会有点乱呢，来最后整理一下，在你下次判断this指向的时候可以按照下面的顺序进行判断：

箭头函数-->setTimeout,setInterval-->显示绑定-->隐式绑定-->默认绑定

