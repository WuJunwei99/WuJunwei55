---
layout: post
title: "markdown 学习笔记（4）"
date: 2020-06-08 09:47:36 +0800
categories: notes
tags: markdown
img: https://www.z4a.net/images/2020/06/10/timgimagequality80sizeb9999_10000sec1591785942235di54047a86dee077803d9f6652c3045adaimgtype0srchttp3A2F2Fimg.mp.itc.cn2Fupload2F201706052F6ca2601297784da1a49c6b1b9986b05b_th.md.jpg
---
markdown 链接、图片、嵌入html、转义符、快捷键


## 链接

### 内联链接

使用内联链接可以链接到给定网址的网页，语法是[链接描述](链接地址)。比如链链接到百度：
百度.(语法是[百度](http://www.baidu.com))

### 引用链接

引用链接也是可以链接到外部的网址，只是使用方法不同。语法结构是[文本内容][id](id可以任意写),然后在任意位置给该[id]配上地址。比如链接到百度的语法是：


    [百度][hello]这里的hello就是定义的id
    [hello]: http://www.baidu.com 这里是给id配上地址，可以在任意位置书写，而且冒号后面有空格

使用连接变量：`[链接名称][链接变量名]`，然后在别的地方，如文档最下面，写上`[链接变量名]:链接地址`。

### 直接使用地址

也可以直接使用网址，语法格式是:<网址>，比如链接到简书语法是：

    <https://www.jianshu.com>



## 图片

插入链接与插入图片的语法很像，区别在一个 !号

    图片为:![markdown](https://p1.ssl.qhimg.com/dr/270_500_/t018c3c3460b1132cce.png?size=512x512)
    
    链接为:[]()

一般：`![alt 属性文本](图片地址)` 

或者可以加title属性：`![alt 属性文本](图片地址 "可选标题")`

一样可以使用连接变量：`[图片名称][图片变量名]`，然后在别的地方，如文档最下面，写上`[图片变量名]:图片地址`。

## 嵌入html

Markdown无法对字体进行设置，经过查找学习，可以使用内嵌html进行解决。

### 设置字号、字体、颜色

* 折叠语法
    
	    <details>
	      <summary>点击时的区域标题：点击查看详细内容</summary>
	      <p> - 测试 测试测试</p>
	      <pre><code>title，value，callBack可以缺省  </code>  </pre>
	    </details>

	    summary：折叠语法展示的摘要
	    
	    details：折叠语法标签
	    
	    pre：以原有格式显示元素内的文字是已经格式化的文本。
	    
	    blockcode：表示程序的代码块。
	    
	    code：指定代码范例。
    

* 其他HTML语法

	    <span style='color:red'>This is red</span>   //字体颜色
	    <ruby> 漢 <rt> ㄏㄢˋ </rt> </ruby> // 特殊字
	    <kbd>Ctrl</kbd>+<kbd>F9</kbd>  // 按键标识
	    <span style="font-size:2rem; background:yellow;">**Bigger**</span> //字体大小和背景
	    
	    <font face="微软雅黑" color="red" size="6">字体及字体颜色和大小</font>
	    <font color="#0000ff">字体颜色</font>
	    
	    <p align="left">居左文本</p>
	    <p align="center">居中文本</p>
	    <p align="right">居右文本</p>

## 转义符

    \\ 反斜杠
    
    \` 反引号
    
    \* 星号
    
    \_ 下划线
    
    \{} 大括号
    
    \[] 中括号
    
    \() 小括号
    
    \# 井号
    
    \ 加号
    
    \- 减号
    
    \. 英文句号
    
    \! 感叹号
    

## 快捷键

    ctrl 1 一级标题
    
    ctrl 2 二级标题
    
    ····
    
    ctrl shift o 有序列表
    
    ctrl u 无序列表
    
    ctrl g 插入图片
    
    ctrl l 插入超链接
    
    Ctrl B 粗体
    
    Ctrl I 斜体
    
    Ctrl Q 引用
    
    Ctrl K 代码块