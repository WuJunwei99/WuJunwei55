---
layout: post
title: "git 学习笔记（2）"
date: 2020-06-08 15:56:17 +0800
categories: notes
tags: git
img: https://www.z4a.net/images/2020/06/10/u4248224721330618640fm26gp0.jpg
---
git 简史 优势 安装 结构 本地库 远程库

## git简史

![](https://www.z4a.net/images/2020/06/09/git.md.jpg)


## git的优势
* 大部分操作在本地完成，不需要联网
* 完整性保证
* 尽可能添加数据而不是删除或修改数据
* 分支操作非常快捷流畅
* 与 Linux 命令全面兼容

## git的安装

到[git官网]("https://git-scm.com/downloads")根据自己电脑的操作系统而选择不同的下载版本。

打开.exe文件，然后就不断点确定，直到install，就可以了,一般全部默认。


## git结构

![](https://www.z4a.net/images/2020/06/09/git01.jpg)

1. 工作区：就是你在电脑里能看到的目录。
2. 暂存区：英文叫stage, 或index。一般存放在 ".git目录下" 下的index文件（.git/index）中，所以我们把暂存区有时也叫作索引（index）。
3. 版本库：工作区有一个隐藏目录.git，这个不算工作区，而是Git的版本库。

## git和git托管中心
代码托管中心作用：维护远程库
局域网环境下：
* GitLab 服务器

外网环境下：
* GitHub
* 码云

## 本地库和远程库
### 团队内协作
![](https://www.z4a.net/images/2020/06/09/git02.md.jpg)

通过clone将项目克隆到本地，对文件进行修改之后再上传push到远程库中。以此来达到团队协作的目的。

### 跨团队协作
![](https://www.z4a.net/images/2020/06/09/git03.md.jpg)

团队外的成员可以通过fork操作将项目复制到自己的远程库中，内容完全一致，然后再clone到本地做一些修改之后push到自己的远程库中；发起一个pull request请求，岳不群对提交的文件进行审核，如果没问题可以进行merge操作，合并到自己的远程库中。
