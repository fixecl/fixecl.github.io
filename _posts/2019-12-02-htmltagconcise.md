---  
layout: post  
title:  "HTML标签"  
date:  2019-12-2 16:59:13  
categories: 前端  
tags: HTML  
---  

* content  
{:toc}  


```<html>``` 网页总体,底下有```<head>```和```<body>```两个标签  
```<!--注释-->```  
```<head>```里面的```<title>```是网页标题  位于浏览器标题栏上  
```<p>```段落，前后有空隙  
```<hx>``` 1-6级标题  
强调：```<em>```斜体  ```<strong>```黑体  
```<span>```对某些字体进行格式设置（要有```<style>```定义span）  
```<q>```引用其他内容（短文本），自动添加双引号  
```<blockquote>```引用其他内容（长文本），两端自动缩进  
```<br />```等于word里面的回车，另起一行，单个使用（空标签）  
```&nbsp;```等于空格，注意```&```和```;```两个标点符号  
```<hr />```水平横线（空标签）  
```<address>```个人地址等信息  
```<code>```显示单行程序代码  
```<pre>```显示多行文本，预格式化代码，识别回车空格等（多用于多行代码显示）  
```<ul> <li>1</li> …  </ul>``` 无序信息，每个```<li>```里面是一个信息，默认前面加圆点  
```<ol> <li>1</li> …  </ol>``` 有序信息，每个```<li>```里面是一个信息，默认前面加序号  
```<div>```容器，划分独立的逻辑部分  
```<div  id="版块名称">…</div>```带名字的容器  
```<table>``` 表格 ```<tbody>```指示表格下载完成再显示  ```<tr>```行  ```<th>``` 表头列  ```<td>```表列  
```<table>```里的```<caption>```指定表格标题，于表格正上方  
```<a>超链接 <a  href="目标网址"  title="鼠标滑过显示的文本">链接显示的文本</a>```  
```<a>```里面添加属性```target="_blank"``` 则改为新窗口打开（默认为当前窗口）  
```<a>```的```href```可以发邮件```mailto:xx@xx.com?cc=抄送&bcc=密抄&subject=主题&body=正文```  
```<img src="图片地址" alt="下载失败显示文本" title = "鼠标划过提示文本">```  
```<form>```表单，属性有```method="get/post"``` ```action="传送的目标"```  
输入框```<input type="text/password" name="名称(供php/asp使用)" value="默认文本" />```  
提交/重置 ```input```的属性```type```改成```submit/reset```  
文本框```<textarea  rows="行数" cols="列数">默认文本</textarea>```  
单选多选```<input type="radio/checkbox" value="值" name="名称(相同则为同组)" checked="checked"/>```  
下拉列表```<select>``` 子项目```<option value="提交值" (selected="selected")>显示文本</option>```  
多选列表```<select multiple="multiple">```  
标签```<label (for="控件id")>```用于关联标签文本与控件，自动焦点转移，控件属性要有```id="控件id"```  