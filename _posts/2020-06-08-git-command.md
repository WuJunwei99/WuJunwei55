---
layout: post
title: "git 学习笔记（3）"
date: 2020-06-08 16:42:45 +0800
categories: notes
tags: git
img: https://www.z4a.net/images/2020/06/10/u4248224721330618640fm26gp0.jpg
---
git的命令行命令


## 本地库初始化

* git init

注意：.git目录中存放的是本地库相关的子目录和文件，不要删除，也不要胡乱修改。

## 设置签名

* 形式：

	用户名：WuJunwei
	
	Email 地址：1159958707@qq.com
* 作用：区分不同开发人员的身份
* 辨析：这里设置的签名和登录远程库(代码托管中心)的账号、密码没有任何关系。
* 命令 
    * 项目级别/仓库级别：仅在当前本地库范围内有效

    git config user.name tom_pro

    git config user.email goodMorning_pro@atguigu.com

    信息保存位置：./.git/config 文件

    * 系统用户级别：登录当前操作系统的用户范围
    
    git config --global user.name tom_glb
    
    git config --global user.email goodMorning_pro@atguigu.com
    
    信息保存位置：~/.gitconfig 文件

    * 级别优先级

    就近原则：项目级别优先于系统用户级别，二者都有时采用项目级别的签名
    
    如果只有系统用户级别的签名，就以系统用户级别的签名为准
    
    二者都没有不允许

* 设置并查看系统用户

    ![设置系统用户](https://www.z4a.net/images/2020/06/09/config01.md.jpg)
    
    
    ![查看系统用户](https://www.z4a.net/images/2020/06/09/config02.md.jpg)

## 基本操作

### 状态查看

git status
查看工作区、暂存区状态

### 添加
git add [file name]

将工作区的“新建/修改”添加到暂存区

### 提交
git commit -m "commit message" [file name]


将暂存区的内容提交到本地库

### 查看历史记录

* git log
    

    ![查看历史](https://www.z4a.net/images/2020/06/09/04.md.jpg)

    多屏显示控制方式：

    空格向下翻页

    b 向上翻页
    
	q 退出
    
git log --pretty=oneline
	
* 每一条日志只显示一行

    ![查看历史](https://www.z4a.net/images/2020/06/09/02dbe1ce442d70a12b.md.jpg)

* git log --oneline

	每条日志头部只显示一部分

     ![查看历史](https://www.z4a.net/images/2020/06/09/03.md.jpg)

* git reflog

     ![查看历史](https://www.z4a.net/images/2020/06/09/03.md.jpg)

HEAD@{移动到当前版本需要多少步}