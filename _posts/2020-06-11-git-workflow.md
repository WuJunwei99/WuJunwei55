---
layout: post
title: "git 学习笔记（7）"
date: 2020-06-11 18:26:45 +0800
categories: living
tags: git
img: https://www.z4a.net/images/2020/06/10/u4248224721330618640fm26gp0.jpg
---
Git 工作流概念，分类，详解，操作


## 概念

在项目开发过程中使用 Git 的方式

## 分类

### 集中式工作流

像SVN一样，集中式工作流以中央仓库作为项目所有修改的单点实体。所有修改都提交到Master这个分支上。

这种方式与SVN的主要区别就是开发人员有本地库。Git很多特性并没有用到。

![](https://s1.ax1x.com/2020/06/18/NexEHP.png)

### GitFlow 工作流

Gitflow 工作流通过为功能开发、发布准备和维护设立了独立的分支，让发布迭代过程更流畅。严格的分支模型也为大型项目提供了一些非常必要的结构

![](https://s1.ax1x.com/2020/06/18/NexABt.png)


### Forking 工作流

Forking 工作流是在 GitFlow 基础上，充分利用了 Git 的 Fork 和 pull request 的功能以达到代码审核的目的。更适合安全可靠地管理大团队的开发者，而且能接受不信任贡献者的提交。

![](https://s1.ax1x.com/2020/06/18/NexZAf.png)

## GitFlow 工作流详解

### 分支种类

* 主干分支 master：主要负责管理正在运行的生产环境代码。永远保持与正在运行的生产环境完全一致。
* 开发分支 develop：主要负责管理正在开发过程中的代码。一般情况下应该是最新的代码。
* bug 修理分支 hotfix：主要负责管理生产环境下出现的紧急修复的代码。 从主干分支分出，修理完毕并测试上线后，并回主干分支。并回后，视情况可以删除该分支。
* 准生产分支（预发布分支）	release：较大的版本上线前，会从开发分支中分出准生产分支，进行最后阶段的集成测试。该版本上线后，会合并到主干分支。生产环境运行一段阶段较稳定后可以视情况删除。
* 功能分支 feature：为了不影响较短周期的开发工作，一般把中长期开发模块，会从开发分支中独立出来。开发完成后会合并到开发分支。

### GitFlow 工作流举例

![](https://s1.ax1x.com/2020/06/18/NmCw0s.png)


### 分支实战

![](https://s1.ax1x.com/2020/06/18/NmC07n.png)

### 具体操作

创建分支

![](https://s1.ax1x.com/2020/06/18/Nm9vWT.png)

![](https://s1.ax1x.com/2020/06/18/Nm9jYV.png)

切换分支审查代码

![](https://s1.ax1x.com/2020/06/18/Nm9Loq.png)

![](https://s1.ax1x.com/2020/06/18/Nm9XF0.png)

![](https://s1.ax1x.com/2020/06/18/Nm9qwn.png)

检出远程新分支

![](https://s1.ax1x.com/2020/06/18/Nm9ISS.png)

切换回 master

![](https://s1.ax1x.com/2020/06/18/Nm9TyQ.png)

合并分支

![](https://s1.ax1x.com/2020/06/18/Nm9oQg.png)

![](https://s1.ax1x.com/2020/06/18/Nm9bes.png)

合并结果

![](https://s1.ax1x.com/2020/06/18/Nm97Lj.png)