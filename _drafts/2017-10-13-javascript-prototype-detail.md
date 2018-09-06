---
layout: post
title: 对Javascript中原型的理解
categories: javascript
description: 个人对js中原型及原型链的理解
keywords: javascript 基础 理解
---

## 写在前面
本来开始写博客的时候第一篇就像以原型开始的，但是考虑再三还是酝酿一阵吧，并不是理解的不透彻，而是总想着用一种最通俗易懂的方式来阐述，关于原型的文章网上跟书本上都有很多，而且阐述的也很透彻，但不经过知识的串联看起来很容易头晕，所以除了对原型的讲解以外最后我还会对知识点进行串连，让看的小伙伴能够有个系统的理解。

## 面向对象
说到原型就不得不说面向对象的思想，虽然javascript是一种基于对象的编程语言，但它并不想其他语言一样有类（class）这个概念，虽然ES6有了class关键字，但这只是一个语法糖而已，并没有从概念上有所改变。

面向对象的核心思想就是把万物都抽象成一个个对象，在抽象的基础上我们可以不在乎数据原来的内容与类型，更多关注于数据的本身与行为，说白了就是关注对象是什么，对象能干什么，也就说我们所说的属性与方法。因为我之前是做java后台的，所以我会用java对比javascript。

我们来看一下java中的对象：

```java
public class Person{
    String hand;          //定义这个类具有hand属性
    String foot;           //定义这个类具有foot属性
    String face;            //定义这个类具有face属性
    String body;              //定义这个类具有body属性

    public String handsup(){
        //吧啦吧啦吧啦
    }

    public String say(){
        //吧啦吧啦吧
    }
}

Person person1 = new Person();
person1.handsup();
person1.say();
```

这里我们不看具体代码是如何实现的，只需要从代码中看出来，我们定义了一个名为Person的类，这个类具有hand,foot,face,body这四个属性，两个方法handsup(),say()，这样我们实例化该类的所有对象就具有了这四个属性。例子中我们new了一个person1对象，并且使用它的handsup()与say()方法。

通过这段代码我们可以初步了解类的功能，它像是约定了它所生成所有对象的公有属性与方法一样。那么，在js中我们应该如何实现（仿照）这一概念呢？

## 工厂模式
创建一个对象可以说非常简单，只要有一点js基础都可以用许多种方法duangduangduang的敲出一个对象来。如：
```javascript
var person2 = new Object();
    person2.name ='xiaoming';
    person2.age = 12;
    person2.say = function(){
    console.log('balabalabalabala');
};
var person3 = {
    name:'lilei',
    age:13,
    say:function(){
       console.log('balabalabalabala');
    },
};
```
上面一段代码中，我们定义了两个不同的对象，但是这两个对象明明有相同的属性名与方法，我们需要将它抽象成类似于类的东西。首先我们想到的是使用工厂模式。
```javascript
function createPerson(name,age){
    var person = new Object();
    person.name = name;
    person.age = age;
    person.say = function (){
      console.log('balabalabalabala');  
    }
    return person;
}
    var person4 = createPerson('xiaohong',15);
```

工厂模式方法十分简单明了，我们创建了一个方法，在这个方法里我创建了person对象并且丰富了它属性与方法,最后返回该对象。每次调用这个对象时，只要传入相应的参数就能获得一个新的对象。

但是利用工厂模式创建对象还是有很大缺陷：

- 不能识别对象到底是什么类型（instanceof）

- 滥用会导致代码臃肿

- 需要创建对象且在函数最后返回对象

最后总是想偷懒的程序员我们当然不愿意这么干了，于是就有了另一种创建对象的方式：构造函数。

## 构造函数
构造函数的概念用的明显的应该是C++了，因为与其他普通的函数的区别还是很明显的。那么在js中我们是如何做的呢？
```javascript
function Person(name,age){
    this.name = name;
    this.age = age;
    this.say = function(){
        console.log('balabalabalabala');
    }
}
var person5 = new Person('xiaolei',20);
```
乍一看，这个好像跟工厂模式没什么的区别