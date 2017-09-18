---
layout: post
title: 如何封装JS工具库
categories: javascript
description: 如何封装自己的js工具库
keywords: javascript 库
---
## 描述

当我们不断写一些重复的js代码的时候，这样不仅枯燥而且浪费时间，这时候我们就会想到将常用的代码写成方法，但是如果将方法直接暴露，这样就会触犯许多js书籍中提到大忌：污染全局变量。也就是说，如果有人使用了跟你定义的方法相同的变量名，这样你的方法就可能会被覆盖或者报错（非多态）。
这个时候我们就需要进一步的将自己用到的方法封装成一个js库，在需要用到的地方调用对应的文件及方法即可。

## 封装方法
新建一个js文件，在文件中写入创建一个全局变量，将方法作为对象的属性写入：
###1.一般封装方法###
```javascript
  var mylib = {
    changeCase:function(){},
    replaceAll:function(){},
    checkPwd:function(){},
  }
  //如果有其他的库，就算名字相同，这样也不会互相影响
  var otherlib = {
    changeCase:function(){},
    replaceAll:function(){},
    checkPwd:function(){},
  }
  //在调用的时候加上对象名称即可，如：mylib.changeCase(),otherlib.changeCase()
```

###2.ES6中模块化的封装方法###
ES6自带了模块化，这也是js第一次支持module，这样我们就可以通过import和export在文件导入和导出各个模块了。前段时间详细的看了一下RequireJS与CommonJS,其实知识点非常少，但是概念很多，现在感觉又忘的差不多了，过段时间在博客上把模块化这个坑再补上吧。现在先将实现的代码列出：

```javascript
//如果是es6的模块化开发，大家也可以
let myJS={
    //去除字符串空格
    trim:function(obj){..},
    //字母大小写切换
    changeCase:function(obj){..},
    //字符串循环复制
    repeatStr:function(str){..},
    .....
}
//暴露模块，里面的方式大家也可以用es6方式实现，代码至少会少一点！
export {myJS}

//在需要引入的文件中加入即可
import(myJS)
```

###3.利用JS原型实现
利用js原型也可以进行封装，但是并不是很推荐这么做，这样做虽然不污染全局变量，但是却造成了另一种污染：污染原生对象。
当其他人使用该被污染的原生对象时，就会造成不必要的开销。
另外一点，如果自己创造的方法与原生方法重名，这样就会覆盖原生的方法，毕竟搞不好什么时候你的方法也被W3C例为通用方法名了呢。
并不推荐这种方法。
```javascript
String.prototype.trim=function(type){
    switch (type){
        case 1:return this.replace(/\s+/g,"");
        case 2:return this.replace(/(^\s*)|(\s*$)/g, "");
        case 3:return this.replace(/(^\s*)/g, "");
        case 4:return this.replace(/(\s*$)/g, "");
        default:return this;
    }
}
//调用方法:'  55345 6 8 96  '.trim(1)
//553456896
```
