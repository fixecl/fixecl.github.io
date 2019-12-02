---  
layout: post  
title:  "css笔记"  
date:  2019-12-2 18:01:43  
categories: 前端  
tags: css  
---  

* content  
{:toc}  

### 定义css
在```<head>```中：
```css
<style type="text/css">
name{
font-size:20px;
color:red;
font-weight:bold; /*bold字体加粗  italic斜体*/
font-family:"Microsoft Yahei";
}
</style>
```
在```<body>```中使用：
```html
<name>内容</name>
```

### css实现
内联式，嵌入式，外部式(优先级：就近原则)
内联式：
```css
<p style="color:red"> 内容 </p>
```
嵌入式：
```css
<style type="text/css">
p{……}
</style>（一般在head标签中）
```
外部式：
```css
<link href="style.css" rel="stylesheet" type="text/css" />
```
（一般在head标签中）

### 选择器
标签选择器： ```p{}```  ```span{}``` ….
类选择器：  ```.classname{…}``` 使用：```<span class="class name">```…
ID选择器： ```#idname{…}``` 使用：```<span id="idname">```
#### ID和类选择器的区别：
- ID选择器只能在文档中使用一次
- 可以使用类选择器词列表方法为一个元素同时设置多个样式
  ```.a{..} ```   ```.b{..}```
  ```<span class="a b">…```
#### 子选择器：
为标签的某一子元素设置格式（直接后代）：
```.first>span{border:1px solid red/*红色边框*/}```
```<p class="first"> 内容<span>此处内容有红框</span> …</span>```
#### 后代选择器：
为标签的所有子后代元素设置格式：
```.first span{…}```（>改成空格）
#### 通用选择器：
```*{…}```
html所有元素
选择器之间可以分组，用“,”分隔，每个组项可以用以上各选择器

#### 伪类选择符
```
a:hover
{…}
```
鼠标划过时的样式

#### 继承
某些样式可以继承，比如设置字体
某些不可以，比如设置边框

权值，优先级：
标签：1
类选择符：10
ID：100
权值相加，使用最高的权值对应属性
相同权值采用就近原则

属性后加```!important```表明权值最高，覆盖所有
（浏览器默认样式<网页样式<用户设置样式<```!important```）

### 排版
```css
font-family:"Microsoft Yahei" 字体
font-weight:italic  /*斜体*/
text-decoration:underline /*下划线*/
text-decoration:line-through/*删除线*/
text-indent:2em /*段前缩进 2em为2个文字大小*/
line-height:2em /*行间距*/
letter-spacing:50px /*中文文字/英文字母间隔*/
word-spacing:50px /*英文单次间隔*/
text-align:center/left/right /*居中 居左 居右*/
```

### 元素分类
块状元素(另起一行开始；宽高行高顶底边距可设定；宽度默认=父元素)：
```<div>、<p>、<h1>...<h6>、<ol>、<ul>、<dl>、<table>、<address>、<blockquote> 、<form>…```
内联元素(一行；宽高顶底边距不可设；宽为其包涵的内容的宽度，不可改)：
```<a>、<span>、<br>、<i>、<em>、<strong>、<label>、<q>、<var>、<cite>、<code>…```
内联块状元素(一行；宽高行高顶底边距可设定)：
```<img>、<input>…```
```display:inline```：转换为内联元素
```display:block```：转换为块状元素
```display:inline-block```：转换为内联块状元素

### 盒模型
#### 边框
```css
border-width:2px; 
border-style:solid; /*dashed（虚线）| dotted（点线）| solid（实线）*/
border-color:red;   /* #xxx x为16进制数*/
```
简写：```border:2px  solid  red;```
如需只设置单边，则使用```border-bottom/top/right/left: 1px solid red;```

#### 宽高
![cssbox]({{site.baseurl}}/images/cssnote/box.png)  

#### 填充padding
```padding-top:20px …```
```padding:10px``` 四个方向都是相同的

#### 边界margin
```margin:20px 10px 15px 30px``` 上、右、下、左

### CSS布局模型
1、流动模型（Flow）
2、浮动模型 (Float)
3、层模型（Layer）

- 流动模型（默认模型）
块状元素自上而下按顺序垂直分布，宽100%
内联元素从左至右水平分别
- 浮动模型
```float:left/right``` 左右浮动
- 层模型

#### 绝对定位(position: absolute)
```css
    position:absolute;
    left:100px; /*距离左边100*/
    top:50px;
```
#### 相对定位(position: relative)
相对于本来在原来位置的偏移，不影响后面的元素
```css
    position:relative;
    left:100px;
    top:50px;
```
#### 固定定位(position: fixed)
不会随浏览器滚动条滚动而移动
```css
    position:fixed;
    left:100px;
    top:50px;
```
参照元素定位：父元素须为```relative```，子元素为```absolute```

### 缩写
#### 数值缩写
```margin```，```padding```等
```top，right，bottom，left```（顺时针）
四值相同：只写一个
上下和左右分别相等：写两个-上-右
只有左右相等：写三个-上-右-下
#### 颜色缩写
```Color:#336699``` 等价于 ```#369```
#### 字体缩写
至少要指定 ```font-size``` 和 ```font-family``` 属性
缩写时 ```font-size``` 与 ```line-height``` 中间要加入```/```斜扛
```font:12px/1.5em  "宋体",sans-serif;```

### 视觉
#### 颜色
```css
color:red
color:rgb(133,144,155) /*(0-255)*/
color:rgb(20%,30%,40%)
color:#00ffcc
```
#### 单位
```px```：像素
```em```：字体大小倍数
如果```font-size```为```em```，则表示为父元素字体大小的倍数
如果```line-height```为```%```，则表示行间距为字体高的百分数
#### 显示隐藏
```display```：元素的位置不被占用
```visibility```：元素的位置仍被占用

### 其他
#### 单行（内联元素）水平居中
在父类元素中使用```text-align:center```
定宽（```width```固定！！）块状元素水平居中：
设置左右```margin```为```auto```：```margin:20px auto```
不定宽块状元素水平居中：
1.加入 ```table``` 标签，```table```标签长度自适应，可看成定宽
2.设置 ```display: inline``` 方法：显示类型设为行内元素（```display:inline```），再设置```text-align```为```center```
3.设置父元素```float```，``` position:relative``` 和 ```left:50%```：子元素设置 ```position:relative``` 和 ```left: -50%``` 实现水平居中

#### 父元素高度确定的单行文本
设置```height```和```line-height```一致，但当字太多超出块宽时不行
父元素高度确定的多行文本：
1.插入```table```，设置```vertical-align:middle```
2.设置元素```display:table-cell```表格单元显示，再使用```vertical-align:middle```；兼容性较差

#### 隐式改变display类型
设置元素
- ```position : absolute ```
- ```float : left``` 或 ```float:right``` 
元素会变成inline-block，可设置元素宽高

