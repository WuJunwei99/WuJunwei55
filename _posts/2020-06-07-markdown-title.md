---
layout: post
title: "markdown 学习笔记（1）"
date: 2020-06-07 18:25:13 +0800
categories: notes
tags: markdown
img: https://www.z4a.net/images/2020/06/10/timgimagequality80sizeb9999_10000sec1591785942235di54047a86dee077803d9f6652c3045adaimgtype0srchttp3A2F2Fimg.mp.itc.cn2Fupload2F201706052F6ca2601297784da1a49c6b1b9986b05b_th.md.jpg
---
markdown 简介 基础 标题


## 简介

Markdown是一种可以使用普通文本编辑器编写的标记语言，通过简单的标记语法，它可以使普通文本内容具有一定的格式。

有的读者自然会诧异，编辑文档可以使用word，那为什么要使用Markdown呢，其实，选择使用Markdown的原因很简单， 它是一种轻量级的标记语言，他的优势就在于用户在通过键盘把内容输入的同时，可以搞定文字的排版，甚至在整个编辑过程中，用户 都不会使用到鼠标，这样，用户就可以把更多的精力放在文档内容的编辑上，而不是去设计排版等的方面。

它的用途很广泛，可以用来写博客日志，记录笔记，编写代码；在github上，它也是作为主流的编辑方式，它的语法也很简单，文本内容也可轻松地转换为html等的形式 ，而且排版样式不会发生变化，不会出现像word版本不兼容时出现的各种问题。

Markdown的编辑器下载可以参照网上的一些经验，这里我就不做过多的介绍，好了，接下来就让我们一起开始Markdown的学习。

## 标题
### 使用 \#

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Markdown语法中标题都是在文字前面使用#进行标题。可以到六级标题，文字前面有几个#就是几级标题。(文字与#之间需要有空格才行)，在大多数markdown的编辑器中可以使用快捷键ctrtl+1~6来创建标题。即ctrl+1是一级标题，以此类推crtl+6是六级标题。

<span style="color:aqua">格式：</span>

```markdown
# 这是一级标题 #
## 这是二级标题 ##
### 333 ###
#### 444 ####
#### 555 ####
#### 666 ####
```
&emsp;

### 使用---和===

<span style="color:aqua">格式：</span>

```markdown
一级标题
===

二级标题
---
```


<span style="color:orange">注意：</span>标题和===/---之间不能空行，不然会被转换成分割线。