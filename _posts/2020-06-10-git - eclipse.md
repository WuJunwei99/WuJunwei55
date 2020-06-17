---
layout: post
title: "git 学习笔记（6）"
date: 2020-06-10 15:44:25 +0800
categories: notes
tags: git eclipse
img: https://www.z4a.net/images/2020/06/10/u4248224721330618640fm26gp0.jpg
---
git图形化界面操作——eclipse


## 工程初始化为本地库

工程→右键→Team→Share   
 

Project→Git

![](https://www.z4a.net/images/2020/06/17/git06_01.md.png)



![](https://www.z4a.net/images/2020/06/17/git06_02.md.png)



### 设置本地签名


![](https://www.z4a.net/images/2020/06/17/git06_03.md.png)


![](https://www.z4a.net/images/2020/06/17/git06_04.md.png)


![](https://www.z4a.net/images/2020/06/17/git06_05.md.png)


### 图标

![](https://www.z4a.net/images/2020/06/17/git06_06.md.png)


带圆柱体：未被提交的文件

untracked：未被追踪的文件

带*:添加到暂存区的文件

带+：由未被追踪的状态转变为加入到版本控制体系（对其进行追踪）

## Eclipse 中忽略文件

概念：Eclipse 特定文件

    这些都是 Eclipse 为了管理我们创建的工程而维护的文件，和开发的代码没有 直接关系。最好不要在 Git 中进行追踪，也就是把它们忽略。
    .classpath 文件
    .project 文件
    .settings 目录下所有文件

为什么要忽略 Eclipse 特定文件呢？

    同一个团队中很难保证大家使用相同的 IDE 工具，而 IDE 工具不同时，相关工 程特定文件就有可能不同。如果这些文件加入版本控制，那么开发时很可能需要为了这些文件解决冲突。




[GitHub 官网样例文件](https://github.com/github/gitignore)


[gitignore](https://github.com/github/gitignore/blob/master/Java.gitignore)

    Java.gitignore

    # Compiled class file
    *.class
    
    # Log file
    *.log
    
    # BlueJ files
    *.ctxt
    
    # Mobile Tools for Java (J2ME)
    .mtj.tmp/
    
    # Package Files #
    *.jar
    *.war
    *.nar
    *.ear
    *.zip
    *.tar.gz
    *.rar
    
    # virtual machine crash logs, see http://www.java.com/en/download/help/error_hotspot.xml
    hs_err_pid*
    
    .classpath
    .project
    .settings 
    target

在~/.gitconfig 文件中引入上述文件

[core]

excludesfile = C:/Users/Lenovo/Java.gitignore [注意：这里路径中一定要使用“/”，不能使用“\”]

![](https://www.z4a.net/images/2020/06/17/git06_07.png)

![](https://www.z4a.net/images/2020/06/17/git06_08.png)


## 推送到远程库

![](https://www.z4a.net/images/2020/06/17/git06_09.png)

![](https://www.z4a.net/images/2020/06/17/git06_10.png)

填写用户和密码信息：

![](https://www.z4a.net/images/2020/06/17/git06_11.png)


![](https://www.z4a.net/images/2020/06/17/git06_12.png)

push成功，并且新建了分支：

![](https://www.z4a.net/images/2020/06/17/git06_14.png)

## Oxygen Eclipse 克隆工程操作

* Import...导入工程

![](https://www.z4a.net/images/2020/06/17/git06_15.png)

![](https://www.z4a.net/images/2020/06/17/git06_16.png)

![](https://www.z4a.net/images/2020/06/17/git06_17.png)

* 指定工程导入方式，这里只能用：Import as general project

![](https://www.z4a.net/images/2020/06/17/git06_18.png)

* 转换工程类型

![](https://www.z4a.net/images/2020/06/17/git06_19.png)


低版本的eclipse在下载到本地时要指定另外一个目录，不可以用原先的eclipse工作目录


## 解决冲突

冲突文件→右键→Team→Merge Tool
修改完成后正常执行 add/commit 操作即可