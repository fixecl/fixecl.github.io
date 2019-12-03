---  
layout: post  
title:  "Javascript笔记"  
date:  2019-12-3 8:31:12  
categories: 前端  
tags: JS  
---  

* content  
{:toc}  

### 加入方式
- 直接嵌入  
```js
<Script type="text/javascript"> 
//代码
</Script>
```
- 文件导入  
```js
<script src="script.js"></script>
```
- JS位置
```<head>```部分：页面打开时执行  
```<body>```部分：解析到对应位置时执行  
直接执行：直接写语句  
事件执行：加入```function```，对位置无要求  

### 基本语法
- 注释：```/**/``` ```//``` 同C
- JS区分大小写
- 运算法则同Java
- 定义变量var
需以```字母``` ```_``` 或 ```$``` 开头  
- 判断：
```js
if(条件)
{}
else
{}
```
- 函数
```js
function 函数名()
{}
function name(x,y,z)
{
  return z;
}
```

### 数组
定义：```var myarray=new Array();```   
或```var myarray=new Array(length);```  
JS中数组会自动扩充  
赋值：   
```js
myarr[i]=num(二维数组：myarr[i][j]);
var myarray = new Array(66,80,90,77,59);
//创建数组同时赋值
var myarray = [66,80,90,77,59];
//直接输入一个数组（称 “字面量数组”）
myarray.length可以查看和修改数组长度
```

### 事件
```html
<input name="button" type="button" value="点击提交" onclick="add2()" />
<a href="http://www.xxx.com" onmouseout="message()">点击我</a>
<textarea name="summary" cols="60" rows="5" onchange="message()">改变</textarea>
<body onload="message()">  加载时运行    </body>
```
```js
//对应js处理：关闭或刷新网页
    window.onunload = onunload_message;   
     function onunload_message(){   
        alert("您确定离开该网页吗？");   
    } 
```
事件 | 说明
-|-
onclick | 鼠标单击
onmouseover | 鼠标经过
onmouseout | 鼠标移开
onchange | 文本框内容改变
onselect | 文本框内容被选中
onfocus | 光标聚焦
onblur | 光标离开
onload | 网页加载
onunload | 关闭网页

### 窗口相关
输出网页内容```document.write```  回车```"<br>"```  
警告消息对话框```alert```  
```confirm(str)```确认消息对话框 返回boolean  
```prompt(display_str,default_str)```提问消息对话框  
点确定返回文本内容，点取消返回null  
```window.open([URL],[窗口名称],[参数])``` 返回窗口对象  
```"_top"、"_blank"、"_self"```具有特殊意义的窗口名称:  
-  _blank：在新窗口显示目标网页
-  _self：在当前窗口显示目标网页
-  _top：框架网页中在上部窗口中显示目标网页

```window.close()```关闭本窗口  
```窗口对象.close()```关闭对应窗口  

### JS DOM
#### DOM节点：
1.元素节点 即标签
2.文本节点 向用户展示的内容 即```<>内容</>```
3.属性节点 元素属性 即```<xx="xx">```

```js
obj=document.getElementById(id)//获取元素对象
obj.innerHTML//元素内容
obj.style.属性=新属性
obj.style.display='none'/'block' //隐藏/显示元素
obj.className="类名" //修改元素显示类

getElementsByName("name") //返回元素数组，不唯一
getElementsByTagName(Tagname)//返回标签名元素数组
elementObject.getAttribute(name)//获取属性值
elementNode.setAttribute(name,value)//设置属性
```

#### DOM 节点有三个重要的属性
- nodeName : 节点的名称
- nodeValue ：节点的值
- nodeType ：节点的类型
##### nodeName 属性: 节点的名称，是只读的。
1. 元素节点的 nodeName 与标签名相同
2. 属性节点的 nodeName 是属性的名称
3. 文本节点的 nodeName 永远是 #text
4. 文档节点的 nodeName 永远是 #document
##### nodeValue 属性：节点的值
1. 元素节点的 nodeValue 是 undefined 或 null
2. 文本节点的 nodeValue 是文本自身
3. 属性节点的 nodeValue 是属性的值
##### nodeType 属性: 节点的类型，是只读的
以下常用的几种结点类型:
元素类型 | 节点类型
-|-
  元素    |     1
  属性    |     2
  文本    |     3
  注释    |     8
  文档    |     9

注意: 浏览器兼容问题，chrome、firefox等浏览器标签之间的空白也算是一个文本节点。

#### 节点属性
方法 | 说明
- | -
nodeName | 返回一个字符串，给定节点的名字
nodeType | 返回一个整数，代表节点的类型
nodeValue | 返回给定节点的当前值

访问节点树
方法 | 说明
- | -
childNodes | 返回一个子节点的数组
firstChild | 返回第一个子节点
lastChild | 返回最后一个子节点
parentNode | 返回父节点
nestSibling | 返回给定节点的下一个子节点
previousSibling | 返回给定节点的上一个子节点

DOM操作
方法 | 说明
- | -
createElement(element) | 创建一个新元素节点
createTextNode() | 创建一个包含着给定文本的新文本节点
appendChild() | 给定节点的子节点后面添加一个新节点
insertBefore() | 将一个给定节点插入到给定元素的给定子节点的前面
removeChild() |从给定元素中删除一个节点
replaceChild() | 把一个给定父元素里面的一个子节点替换为另一个节点


