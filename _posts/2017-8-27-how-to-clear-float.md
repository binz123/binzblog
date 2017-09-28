---
layout: post
title: float浮动与清除全面剖析
categories: css
description: float全面剖析
keywords: css 小技巧
---
## 前言
最近来了一个实习的小伙，由于项目也比较紧，不可能把现有的业务交给他一个新手，于是老大突发奇想弄了一个罗列bug list的小项目给他做，上手挺快的，基本雏形都做的七七八八了，突然有天跑过来跟我说出问题了，说样式崩了，有个容器怎么也撑不开，缩进去了？看了一眼代码，跟我想的一样，float惹的祸。

float这东西，刚开始用的时候觉得挺好用的，布局很方便呀，但我刚入门的时候就有人提醒我说少用float，坑很多。后来为了彻底弄懂这些坑也做了不少研究，今天就拿出来分享分享。

## 问题
我们来看下面这段代码：
```html
<div class="parent">
  <div class="child-left"></div>
  <div class="child-right"></div>
</div>
```
```css
.parent{
  width:200px;
  margin:40px auto;
  padding:10px;
  background:blue;
}
[class^='child']{
  width:40px;
  height:40px;
  background:red;
  opacity:0.6;
}
.child-left{
  float:left;
}
.child-right{
  float:right;
}
```
最终显示：
![float1](http://binzhome.com/assets/images/floattest/float1.jpg)

我们蓝色背景的父级容器，里面有两个子元素，按照盒子模型的原理，块级子元素应当可以撑开父级元素的高度，但是这里并没有。造成这个现象的原因是因为我们在子元素中定义了float浮动的样式，导致父元素无法被撑起，也就是我们常说的父元素塌陷。

### float定义
float 属性定义元素在哪个方向浮动。以往这个属性总应用于图像，使文本围绕在图像周围，不过在 CSS 中，任何元素都可以浮动。浮动元素会生成一个块级框，而不论它本身是何种元素。

如果浮动非替换元素，则要指定一个明确的宽度；否则，它们会尽可能地窄。

**注释**：假如在一行之上只有极少的空间可供浮动元素，那么这个元素会跳至下一行，这个过程会持续到某一行拥有足够的空间为止。

我认为这里改为**float属性定义元素尽可能向哪个方向浮动**更为合适。这里的尽可能指的是元素会一直平移到所处容器的边框，或者是碰到另一个浮动元素。浮动元素最远的浮动距离是其所处容器的边框，所以我们可以看到上文代码中红色方框设置浮动后并没有超过蓝色的左右边框。（你可以将padding设为：padding:5px 0;试试哦）

我们再来看另一种说法：**float元素会脱离文档流**。很多书籍上都是这么写的，但这样的定义其实并不严谨，给许多刚开始的前端程序员造成了不少困扰，个人认为正确说法是**float元素会相对脱离文档流**，因为它仍然保持部分的流动性，那么有绝对脱离文档流的元素么？有，position设置为absolute，fixed的元素就是绝对脱离文档流的。其相对指的就是父元素。

文档流，文本流傻傻分不清？<a href="http://www.cnblogs.com/gvip-cyl/p/6258119.html" title="flowtips">点这里</a>

## 原因

既然问题与float元素的定义都给了出来了，那么就要开始分析造成上面父元素塌陷的原因了，其实上面已经解释了。

其实就是因为float元素（相对）脱离了文档流造成的。html文档加载解析时是从上到下，从左到右的，浮动元素脱离文档流后就不受文档流的管辖了，这就导致它存在父元素的高度随着浮动被抹去了，父元素也默认无视了这个浮动元素，上文代码中除浮动元素外没有其他元素，所以就会出现上面的父元素塌陷的问题。

## clear属性造成的困扰

clear属性的官方定义是：规定元素的哪一侧不允许其他浮动元素。注意这里清除并不是清除float元素的效果，也不是清除float元素造成塌陷的结果，我见过有人将clear写在父元素上结果喊着说clear欺骗了他（没错，就是我）。

clear属性只是规定了拥有该属性的元素两侧的浮动元素不会对它造成影响，什么意思？再看段代码

```html
<div class="parent">
  <div class="child-first"></div>
  <div class="child-second"></div>
</div>
```

```css
.parent{
  width:200px;
  margin:40px auto;
  padding:10px;
  background:blue;
}
[class^='child']{
  float:left;
  width:40px;
  height:40px;
  background:red;
  opacity:0.6;
}

.child-second{
  clear:both;
}
```
![float2](http://binzhome.com/assets/images/floattest/floattest2.jpg)
我们在第二子元素child-second上添加了clear:both;属性,这时第二个子元素并没有挨着第一个浮动的子元素排列，而是像块级元素一样(float清除了div中块级元素的特性)展示到了下一行。但是它并没有撑开父元素，也没有改变其他子元素的排列。

clear属性**只影响使用这个属性的元素本身，不影响其他元素。** 所以在浮动元素或者父元素上利用clear属性来解决塌陷问题不成立的，因为无论你是加在父元素还是子元素上，都对其他元素没有任何影响。(但clear确实可以解决塌陷问题，下面就给出正确答案)


## 解决
我们想要达成的结果是，既保持浮动的效果，又能解决浮动导致的塌陷问题。

分别先给出解决方案，再解释原因。我们还是分别以第一个例子的基础上做实验。

1. **添加额外标签**

```html
<div class="parent">
  <div class="child-left"></div>
  <div class="child-right"></div>
  <div style="clear:both;"></div>
</div>
```

我们发现此时parent元素已经被撑起来了，撑起来的原因是我们在两个设置浮动的元素下添加了一个额外的标签，并且将clear属性设置为both，clear属性是让自身不能和前面的浮动元素相邻，这样父级中就有了一个块级元素，从而被动撑起。

但是这种方法会添加额外的无意义的标签，使得结构和表现分离，让后期维护变成噩梦，实际项目中还是不要使用这种方法。

2. br标签

```html
<div class="parent">
  <div class="child-left"></div>
  <div class="child-right"></div>
  <br clear="all"/>
</div>
```

br标签自带clear属性，其实现原理跟上面相同，所以缺点也是使得结构和表现分离了，所以同样不推荐这种方法。

3. 父元素设置overflow:hidden

```css
.parent{
  overflow:hidden;
  width:200px;
  margin:40px auto;
  padding:10px;
  background:blue;
}
```

html元素与一开始的demo一样不做任何变动，只是再parent类元素中添加overflow:hidden;的样式。一开始我对这种清除浮动的方式很不理解，后来查了不少资料才找到比较合理的解释。overflow:hidden 的意思是超出的部分要裁切隐藏掉，如果float元素不占用普通流位置，则普通流的包含块需要根据内容进行裁切隐藏，如果高度的默认值是auto，那么不计算float的元素就进行裁切，这样就会把float元素也裁切掉，显然是违背布局常识的。

所以在没有明确容器高度的情况下，必须要把浮动元素的高度也计算进来，顺理成章的也达到了清除浮动的目标（许多人用BFC来解释这个问题，但我觉得这样理解其实更好，感谢知乎的@貘吃馍香）。

这种方式的缺点也一目了然，我们无法再显示溢出的元素，且不会自动换行了，所以最好还是不要使用这种方式。

4. 父元素设置 overflow：auto 属性

这个方法与上面原理差不多，但是在firefox中会出现focus的效果，并且某些情况下还会出现内容全选。所以也不推荐这种方式。

5. 父元素设置为浮动

这种方式会使得与父元素相邻的元素的布局会受到影响，不可能一直浮动到body，不推荐使用。

6. 父元素设置display:table

```css
.parent{
  display:table;
  width:200px;
  margin:40px auto;
  padding:10px;
  background:blue;
}
```
我们先把解决方案放在这里，至于原理，下面将结合Block formatting contexts （块级格式化上下文）,简称BFC。一同解释。这种方案会导致盒模型属性发生变化，从而引起一系列的问题。同样不推荐使用。

7. after伪元素

```html
<div class="parent clearfix">
  <div class="child-left"></div>
  <div class="child-right"></div>
</div>
```

```css
.parent{
  width:200px;
  margin:40px auto;
  padding:10px;
  background:blue;
}
.clearfix:after 
 {
   content:"."; 
   display:block;
   height:0; 
   visibility:hidden; 
   clear:both; 
}
.clearfix { 
  *zoom:1; 
}
[class^='child']{
  width:40px;
  height:40px;
  background:red;
  opacity:0.6;
}
.child-left{
  float:left;
}
.child-right{
  float:right;
}
```
这段代码里，我们并没有再html代码中添加新的标签，只是在父级元素中新增了一个clearfix的类名。在css代码中，我们定义了clearfix的:after伪元素，并相应的做了一些样式上的改变，于是发现原来塌陷的父元素被撑起了。

这里将:after伪元素设置为display:block;的原因是使生成的元素以块级元素显示，从而占满剩余空间。   height:0;是为了避免生成的内容破坏了原有布局的高度。visibility:hidden 使生成的内容不可见，并允许可能被生成内容盖住的内容可以进行点击和交互。最后clear就不言而喻了。

最后提一点，可能有人会问clearfix类中的*zoom:1;是什么意思？这是为了兼容IE6~7的原因，IE6~7不支持:after伪元素，我们使用zoom:1;触发 hasLayout。

这是比较推荐的方式。
## BFC

为了彻底了解display:table;能够解决塌陷问题的原因，我们就必须先弄懂BFC到底是个什么东西。


一个**块格式化上下文**由以下之一创建：

- 根元素或其它包含它的元素

- 浮动元素 (元素的 float 不是 none)

- 绝对定位元素 (元素具有 position 为 absolute 或 fixed)

- 内联块 (元素具有 display: inline-block)

- 表格单元格 (元素具有 display: table-cell，HTML表格单元格默认属性)

- 表格标题 (元素具有 display: table-caption, HTML表格标题默认属性)

- 具有overflow 且值不是 visible 的块元素，

- display: flow-root

- column-span: all 应当总是会创建一个新的格式化上下文，即便具有 column-span: all 的元素并不被包裹在一个多列容器中。


一个**块格式化上下文（block formatting context）** 是Web页面的可视化CSS渲染出的一部分。它是块级盒布局出现的区域，也是浮动层元素进行交互的区域。

上面是MDN与w3c给出的定义，说实话，我第一次看到这玩意的时候，表情基本是问号问号问号的，它到底表示的是什么呢？

换句说法，BFC是一个决定了块盒子布局与浮动相互影响的范围。首先，它是一个范围，一个BFC包含创建该上下文元素的所有子元素，但不包括创建了新BFC的子元素的内部元素。这看上去还是有点拗口，我们再来看一段代码。

```html
<div id="father" class="BFC">
  <div id="son1">
     <div id="grandson1"></div>
     <div id="grandson2"></div>
  </div>
  <div id="son2" class="BFC">
     <div id="grandson3"></div>
     <div id="grandson4"></div>
  </div>
</div>
```
这里BFC类表示这个元素创建了新的块格式化上下文（BFC）。

我们的#father创建了一个BFC，它的范围包括了#son1,#grandson1,#grandson2,#son2，在这里#son1的子元素也是属于#father的BFC的。但是#son2中的#grandson3,#grandson4就不属于#father的BFC了，这是因为#son2创建了一个新的BFC，它所包含的子元素此时应当属于#son2的BFC，不难得出**一个元素在任何时候只能属于一个BFC**。

当我们在父元素上使用display:table;时，它的子元素会产生匿名框(anonymous boxes) 而匿名框中的display:table-cell可以创建新的BFC，这样就解决了塌陷的问题。同理，overflow:hidden;也可以用这种方式解释。

关于BFC的跟多解释，<a href="https://segmentfault.com/a/1190000004246731" title="BFC">请看这里</a>

## 总结
到这里，关于浮动跟清除浮动已经解释完了，前端水很深坑很多，黑科技更是层出不穷，但沉下心去学习会发现很多之前没有探索过的东西。保持一颗好奇的心吧。

推荐阅读：

<a href="https://developer.mozilla.org/zh-CN/docs/CSS/float">
 MDN float</a>

 <a href="http://www.iyunlu.com/view/css-xhtml/55.html">那些年我们一起清除过的浮动</a>

 <a href="https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context">MDN BFC</a>

 
