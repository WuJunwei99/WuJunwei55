---
layout: post
title: "maven 学习笔记（1）——maven概述"
date: 2020-06-22 15:22:14 +0800
categories: notes
tags: maven
img: https://s1.ax1x.com/2020/06/23/NUss58.png
---
maven概述 配置本地仓库 构建helloworld项目(命令方式) 操作项目工程（命令方式）


##  maven概述

### 现实问题

1. 如果jar包都到各个官网网站下载，会浪费很多时间，而且可能不全。
2. 一个jar包依赖的其他jar包，可能没导入到项目，而导致项目跑不起来。而有些时候，根本搞不清楚一个jar包依赖了那些jar包。
3. 项目的jar包需要复制和粘贴到WEB-INF/lib下。同样的jar包重复出现在不同的工程中，一方面浪费空间，同时也让工程臃肿
4. 平时我们开发项目时，一般都是一个项目就是一个工程。我们划分模块时，都是使用package来进行划分。但是，当项目很大时，有很多子模块时，即使是package来进行划分，也是让人眼花缭乱。

### maven介绍

Maven 翻译为"专家"、"内行"，是 Apache 下的一个纯 Java 开发的开源项目。基于项目对象模型（缩写：POM）概念，Maven利用一个中央信息片断能管理一个项目的构建、报告和文档等步骤。

Maven 是一个项目管理工具，可以对 Java 项目进行构建、依赖管理。

Maven 也可被用于构建和管理各种项目，例如 C#，Ruby，Scala 和其他语言编写的项目。Maven 曾是 Jakarta 项目的子项目，现为由 Apache 软件基金会主持的独立 Apache 项目。

### maven作用

Maven是面向技术层面，针对java开发的项目管理工具，它提供了构建工具所提供的功能，除了构建功能之外，maven还可以**管理项目结构**、**管理依赖关系**、生成报告、生成Web站点、有助于团队成员之间的协作。

1. 项目构建

项目构建过程包括【清理项目】→【编译项目】→【测试项目】→【生成测试报告】→【打包项目】→【部署项目】这几个步骤，这六个步骤就是一个项目的完整构建过程。

java 

* 清理[clean]：删除上一次编译得到的class字节码文件
* 编译[compile]：将Java源程序编译为class字节码文件

    主体程序编译

    测试程序编译

* 测试[test]：运行以前准备好的测试程序，验证代码是否正确
* 报告：测试报告
* 打包[package]：

    Java工程打jar包

    Web工程打war包

* 安装[install]：将jar包或war包存入Maven仓库
* 部署[deploy]：将war包部署到服务器运行

理想的项目构建是高度自动化，跨平台，可重用的组件，标准化的，使用maven就可以帮我们完成上述所说的项目构建过程。

Maven将构建环节定义为有标准顺序的生命周期，执行构建命令时，每一次都是从生命周期最开始的位置，一直执行到命令指定位置。


2. 依赖管理

开发项目工程需要依赖其他的项目（jar包），通过配置即可引入这些依赖jar包。

maven的配置文件看似很复杂，其实只需要根据项目的实际背景，设置个别的几个配置项而已。maven有自己的一套默认配置，使用者除非必要，并不需要去修改那些约定内容。这就是所谓的“**约定优于配置**”。

## 配置本地仓库

### 仓库的概述

仓库就是一个目录，这个目录被用来存储我们项目的所有依赖（插件的jar包还有一些其他的文件），简单的说，当你build一个Maven项目的时候，所有的依赖文件都会放在本地仓库里，仓库供所有项目使用。

![](https://s1.ax1x.com/2020/06/23/NUGhYq.md.png)

maven会优先从本地仓库中寻找，如果没有则上网（中央仓库）下载。而且会将互联网中的jar缓存到本地仓库。

### 配置本地仓库

本地仓库的位置是通过maven的核心配置文件（conf/settings.xml）来配置的。

![](https://s1.ax1x.com/2020/06/23/NUGolT.md.png)

* 指定本地仓库：

![](https://s1.ax1x.com/2020/06/23/NUGfkn.md.png)

因为mvn会通过互联网的中央仓库自动下载jar包，但是比较慢，这里不做下载

注意：注意解压文件的目录层次！


扩展：里面目录是很多可以依赖项目（jar）和一些插件。


### 配置阿里私服仓库

Maven默认是从中央仓库下载jar包或插件，而中央仓库在国内并没有服务器。所以速度让人着急

国内很多公司或者组织为了提高下载速度，搭建了一些私服。目前国内比较流行的私服是：开源中国、阿里私服

在mirrors镜像标签下配置（所以私服又称镜像），默认被注释掉：

![](https://s1.ax1x.com/2020/06/23/NUG4f0.md.png)

添加阿里私服镜像：

![](https://s1.ax1x.com/2020/06/23/NUGIpV.md.png)
    
    <mirror>
    		<id>nexus-aliyun</id>
    		<mirrorOf>central</mirrorOf>
    		<name>Nexus aliyun</name>
    		<url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>

## 构建helloworld项目(命令方式)


我们可以使用Maven Archetype插件的create命令创建maven项目，执行：

mvn archetype:generate -DgroupId=com.atguigu.myapp -DartifactId=myapp -DarchetypeArtifactId=maven-archetype-quickstart

![](https://s1.ax1x.com/2020/06/23/NUGL79.md.png)


mvn：mvn的命令

archetype: generate：构建工程

-DgroupId：组id（包名）

com.atguigu.myapp：自定义包名

-DartifactId=myapp：工程的名字为myapp

-DarchetypeArtifactId=maven-archetype-quickstart：javase的项目的骨架（maven的插件）

提示：只要使用相同的骨架，则生成的项目的结构都是一样的。

![](https://s1.ax1x.com/2020/06/23/NUG7XF.md.png)

Maven项目结构是标准的：

* pom.xml 位于工程根目录，对项目进行配置，任何一个maven项目都有。
* src/main/java 存放项目源码 （源码和测试代码是分开的）
* src/test/java 存放项目测试代码 


## 操作项目工程（命令方式）

常用的Maven命令行指令：

| 命令作用 | 命令格式 |
| :----: | :----: |
| 清理 | mvn clean |
| 编译 | mvn compile |
| 测试 | mvn test |
| 打包 | mvn package |
| 安装 | mvn install |
| 发布 | mvn deploy |

**注：**

想要执行mvn命令，需要进入工程所在目录 （即进入到pom.xml 所在目录）

注意：所有的mvn命令都必须针对pom.xml运行 ！

* 编译命令 mvn compile 
* 
作用：在工程目录生成target目录，将源码（\src\main）编译为class文件，存放在target/classes目录。

![](https://s1.ax1x.com/2020/06/23/NUGT6U.md.png)

![](https://s1.ax1x.com/2020/06/23/NUGbm4.md.png)

* 清除命令 mvn clean

作用：清除编译后的结果，会删除target目录及相关文件。

* 测试命令 mvn test

作用：运行测试，会先对代码自动编译，生成target目录和测试报告。

提示：Maven会自动**先编译再自动运行测试**。

* 打包命令 mvn package

作用：Java项目**自动**打成 jar包；Web项目**自动**打成war包。

![](https://s1.ax1x.com/2020/06/23/NUGq0J.md.png)

![](https://s1.ax1x.com/2020/06/23/NUGvfx.md.png)

提示：该文件名的名字为：工程名字（其实是artifactID）-版本-版本性质.jar，SNAPSHOT表示开发版本。

另外：

如果打包的时候不想执行测试（跳过测试），可以执行：mvn package -Dmaven.test.skip=true

* 安装命令 mvn install

作用：将项目打包后，安装到仓库（repository）中

根据日志找到安装目录所在位置

![](https://s1.ax1x.com/2020/06/23/NUGjt1.md.png)

查看pom.xml文件：

![](https://s1.ax1x.com/2020/06/23/NUGXkR.md.png)

看出：安装到的仓库目录是有一定规律的，是固定的，格式如下：


	${仓库根目录}/groupId/artifactId/version/artifactId-version.jar
