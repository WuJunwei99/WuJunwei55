---
layout: post
title: "微信开发者工具的git版本控制"
date: 2020-07-03 18:21:55 +0800
categories: technology
tags: 微信小程序 git
img: https://s1.ax1x.com/2020/07/04/NxZEr9.png
---
Git 版本管理；Git 状态展示；微信开发工具使用git；


在上线“大学生学科竞赛”version2.4之后，已经有大半个月没有登录微信开发者平台，今天登陆的时候，无法正常登陆，并且无法进行更新，我以为是太久没有更新导致版本太低无法支持这些功能，而且自己的代码从最开始只备份过一次，而且数据库的数据存在云数据库中，如果数据库崩掉，我将重新录一遍数据，所以当时慌得一批，考虑到可能是校园网络卡顿的原因，我之后又换了网络，这次终于可以正常打开。经过此次教训，我就想着将数据库文件还有代码进行备份。而微信开发者工具刚好集成了 Git 版本管理面板，这将大大简化开发者的操作流程。接下来我将结合微信开发文档以及自己的备份经历讲解微信开发者工具中的git版本控制。


## Git 版本管理

为了方便开发者更简单快捷地进行代码版本管理，简化一些常用的 Git 操作，以及降低代码版本管理使用的学习成本，开发者工具集成了 Git 版本管理面板。

开发者可以在打开的项目窗口里，点击工具栏上的 “版本管理” 按钮进入 Git 版本管理界面。

### 提交工作区更改

在 “工作区” 可以查看到目前工作目录的变更及对比，并直接通过勾选文件前面的复选框将其添加到暂存区。右键点击工作区或者此文件，可以丢弃修改。输入提交标题和详情，点击提交按钮即可以提交本次的变更。在标题栏上点击右键可以使用常用的 Gitmoji 符号。

![](https://s1.ax1x.com/2020/07/04/NxZAKJ.png)

### 查看文件修改历史

在提交记录的目录树文件上右键点击，可以查看到某个文件截至该提交的所有变更记录，并可直接查看文件内容，方便排查问题。

![](https://s1.ax1x.com/2020/07/04/NxZVbR.md.png)


### 检出和创建分支

要检出某分支，直接在分支上点击右键选择 “检出” 即可。要创建分支，可以在要创建的提交记录或者分支名上右键，选择 “创建分支” 即可。

![](https://s1.ax1x.com/2020/07/04/NxZFv4.md.png)

### 拉取，推送和抓取

通过工具栏上的拉取，推送和抓取按钮，可以很方便地对远程仓库执行各种操作。某些远程仓库可能需要身份验证或者网络代理配置，可以在 “设置” 页中 “网络和认证” 中配置这些信息。

![](https://s1.ax1x.com/2020/07/04/NxZKPK.md.png)

### 网络和认证设置

如果连接远程仓库需要代理或者用户身份验证的设置，可以在设置 “网络和认证” 中配置。

![](https://s1.ax1x.com/2020/07/04/NxZn56.md.png)

### 用户设置

在设置页面可以对用户名进行配置。配置完成后，下次提交时，将会使用此用户名和邮箱进行提交。

![](https://s1.ax1x.com/2020/07/04/NxZeV1.md.png)

### 子模块

如果项目包含子模块 (Submodule)，可以在子模块列表下查看到子模块的信息。目前不支持对子模块进行更多的操作。

![](https://s1.ax1x.com/2020/07/04/NxZmUx.md.jpg)

### 初始化 Git 仓库

如果所在的项目文件夹下没有找到 Git 仓库，可以根据提示初始化一个仓库，并可选择是否立即提交所有文件，以及自动生成一个 .gitignore 文件模板。

![](https://s1.ax1x.com/2020/07/04/NxZQ2D.md.jpg)



## Git状态展示


如果所在的小程序工程目录（project.config.json 所在目录）存在 Git 仓库，编辑器可以展示目前的 Git 状态。

目录树
如图所示，当某些文件存在变动时，目录树的文件右侧将展示相应的图标来表明这一状态。当某一处于收起状态的目录下存在有变动的文件时，此目录的右侧亦会展示一个圆点图标表明此情况。

文件图标状态的含义如下：


![](https://s1.ax1x.com/2020/07/04/NxZM8O.jpg)

![](https://s1.ax1x.com/2020/07/04/NxZlxe.md.png)

![](https://s1.ax1x.com/2020/07/04/NxZ8rd.png)

如果某一文件存在修改（Modified），可以右键点击此文件，并选择 “与上一版本比较”，则可以查看当前工作区文件与 HEAD 版本的比较。


![](https://s1.ax1x.com/2020/07/04/NxZtat.md.jpg)

## 微信开发工具使用git

当然开发者也可以使用工具自带的图形化操作界面，也可以使用命令行命令进行相关的提交、拉取等操作，具体的语句可以参照我的[git学习笔记(3)](https://wujunwei99.github.io/notes/2020/06/08/git-command.html)

如果需要将项目提交到github等的仓库，需要进行一些配置：

### github创建仓库

关于创建仓库以及Github操作可以参照我的[git学习笔记(5)](https://wujunwei99.github.io/notes/2020/06/09/github.html)

### 配置仓库信息

初始化完成后，依次点击「工作空间」->「设置」->「通用」->「编辑」，编辑在Git中使用的用户名和邮箱。这一步相当于git config命令中的配置操作。

![](https://s1.ax1x.com/2020/07/04/NxZGqA.md.png)

### 添加远程库

点击微信号开发工具的项目管理---设置--远程--添加

设置在github上创建的项目的名称和克隆地址，点击确定即可


![](https://s1.ax1x.com/2020/07/04/NxZYVI.md.png)

### 网络认证

 选择网络认证，认证方式为用户名、密码认证，填写在github上使用的用户名密码即可。


![](https://s1.ax1x.com/2020/07/04/NxZNIP.md.png)

### 添加工作区文件到暂存区

因为我的开发者工具的版本问题，我并没有找到如何将工作区文件添加到暂存区，所以使用的是git命令的方式，当然，二者在功能实现上并没有什么实质的区别，最后的效果是一样的。


![](https://s1.ax1x.com/2020/07/04/NxZi2F.md.png)

除此之外，也并没有什么需要特殊说明的内容，相关的github操作按照图形化界面提示或者git命令行即可实现。


