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

### 前进后退

基于索引值操作[推荐]

    git reset --hard [局部索引值]
    
    git reset --hard a6ace91

使用^符号：只能后退

    git reset --hard HEAD^
    
    注：一个^表示后退一步，n 个表示后退 n 步

使用~符号：只能后退

    git reset --hard HEAD~n
    
    注：表示后退 n 步


### reset 命令的三个参数对比

--soft 参数
    
    仅仅在本地库移动 HEAD 指针

--mixed 参数

    在本地库移动 HEAD 指针
    
    重置暂存区

--hard 参数

    在本地库移动 HEAD 指针

    重置暂存区

    重置工作区

### 删除文件并找回

前提：删除前，文件存在时的状态提交到了本地库。

操作：git reset --hard [指针位置]

删除操作已经提交到本地库：指针位置指向历史记录

删除操作尚未提交到本地库：指针位置使用 HEAD


### 比较文件差异

* git diff [文件名]

将工作区中的文件和暂存区进行比较

* git diff [本地库中历史版本] [文件名]

将工作区中的文件和本地库历史记录比较

不带文件名比较多个文件

## 分支管理

在版本控制过程中，使用多条线同时推进多个任务。

### 好处

同时并行推进多个功能开发，提高开发效率

各个分支在开发过程中，如果某一个分支开发失败，不会对其他分支有任 何影响。失败的分支删除重新开始即可。

### 分支操作

* 创建分支：git branch [分支名]
* 查看分支：git branch -v
* 切换分支：git checkout [分支名]

* 合并分支：

    第一步：切换到接受修改的分支（被合并，增加新内容）上
    
    git checkout [被合并分支名]
    
    第二步：执行 merge 命令
    
    git merge [有新内容分支名]

### 解决冲突

    第一步：编辑文件，删除特殊符号
    
    第二步：把文件修改到满意的程度，保存退出
    
    第三步：git add [文件名]
    
    第四步：git commit -m "日志信息"
    
    注意：此时 commit 一定不能带具体文件名