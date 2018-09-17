---
layout: post
title: float浮动与清除(闭合)全面剖析
categories: css
description: float全面剖析
keywords: css 小技巧
---

## 前言
关于圣杯布局跟双飞翼布局其实网上有许多详细的教程了，虽然很久以前就会这两种布局，但最近跟同事交流的时候发现自己在表述为什么以及怎么做的问题上有一个很大的欠缺，于是就想着把两个布局写成一篇自己看的博文吧，看看这两个布局的代码为什么要这么些

二者都说是为了实现下面图中**左右两边宽度固定，中间自适应**的情形产生的  

![floatleftright](http://binzhome.com/assets/images/shuangfeiyisshengbei/1537191491910.jpg)

### 圣杯布局
```html
<div id="container">
  <div class="middle">这里是中间自适应的部分</div>
  <div class="left"></div>
  <div class="right"></div>
</div>
<style type="text/css">
*{
    margin:0;
    padding:0;
}
#container{
    width:100%;
}
div{
    position:relative;
    float:left;
    min-height:100px;
}
.middle{
    width:100%;
    padding:0 200px 0 200px;
    background-color:yellow;
}
.left{
    width:200px;
    margin-left:-100%;
    background-color:blue;
    left:-200px;
}
.right{
    width:200px;
    margin-left:-200px;
    background-color:red;
}
<style>
```
这里根据思路一步步整理一下思路：  

1.我们首先在容器里创建分别代表三个位置的div，为了撑开高度，将最小高度设置为了200px，左右侧边栏宽度固定为200px。问题：block元素各自占据一行  

2.将div全部设置为float:left;使其脱离文档流，目的是为了**让元素能在同一行中排列**。问题：中间元素由于没有设置宽度被挤压成content的宽度了

3.将middle元素的宽度设置为100%，占据整个屏幕。问题：left,right元素被置于middle元素下方了，因为middle元素已经占据整个宽度了

4.将left元素的margin-left设置为-100%（margin-left:-100%;）,将left元素设置为它自身的宽度（margin-left:-200px）问题：由于middle元素宽度是100%，middle元素中content内容被left元素遮挡住了

5.将middle元素设置为padding:0 200px 0 200px;至此整个圣杯布局就出来了

这里有3点我觉得是比较需要注意的：
- 我们将margin-left的值设为**负值**来左右侧边栏的位置，至于margin为负数的详细讲解下面这篇文章写的非常详细了，这里就不多概述了
CSS布局奇淫巧计 [强大的负边距](http://www.cnblogs.com/2050/archive/2012/08/13/2636467.html#2457812).

- 将position设置为relative是因为当我们设置设置了padding:0 200px 0 200px后，左侧元素也跟着右移了200px，而将**position设置为relative后再设置left则会使其相对于自身向左移动200px**（不设为absolute的原因是因为absolute相对的父级元素，若我们将设置div上的relative去除，单独将left设置为absolute，此时在正常开发过程，left相对的元素就不确定了）

- 该布局在ie7,8,9中会有高度不一的问题


### 双飞翼布局
```html
<div id="container">
  <div class="middle">
    <div class="inner">
      内部内容
    </div>
  </div>
  <div class="left"></div>
  <div class="right"></div>
</div>

<style type="text/css">
*{
  margin:0;
  padding:0;
}

#container{ 
  width:100%;
}
div{
  min-height:200px;
  float:left;
}
.middle{
  width:100%;
  background-color:yellow;
  
}
.inner{
  margin:0 200px;
}
.left{
  margin-left:-100%;
  background-color:red;
  width:200px;
  
}
.right{
  margin-left:-200px;
  background-color:blue;
  width:200px;
}
</style>
```

我们发现，在双飞翼布局中Dom结构上在middle中多了一个inner的div元素，前面关于浮动，排列的处理都与圣杯布局相同，其不同点在于如何处理middle设置为100%以后左右侧被left,right元素遮挡的问题，圣杯布局用的是通过padding使得content的居中显示。  
而双飞翼布局采用的策略则是，我另起炉灶，middle作为父元素继续保持其content为100%，而我们在中间写一个子元素inner，**将它的左右外边距宽度设置为左右侧边栏的宽度**，这样就能做到中间内容自适应的处理了。  


## 一点废话
其实圣杯布局跟双飞翼布局都是为了解决左右侧边栏固定，中间自适应的问题，只是二者实现思路上稍微有点不一样。目前flex的出现也是的这种布局的实现更为简单。也许这些东西很基础，但细细的品味一下，会觉得这些思路跟想法也让人赞叹不已。