---
layout: post
title: "springBoot介绍与入门"
date: 2020-07-20 20:17:55 +0800
categories: notes
tags: springBoot idea
img: https://s1.ax1x.com/2020/07/20/U4h0Gq.md.png
---
springBoot介绍，创建方式，单元测试及热部署，运行原理


## springBoot介绍

随着互联网的兴起，Spring势如破竹，占据着Java领域轻量级开发的王者地位。随着Java语言的发展以及市场开发的需求，Spring推陈出新，推出了全新的Spring Boot框架。Spring Boot是Spring家族的一个子项目，其设计初衷是为了简化Spring配置，从而可以轻松构建独立运行的程序，并极大提高开发效率。

### 简介

* Spring Boot是基于Spring框架开发的全新框架，其设计目的是简化新Spring应用的初始化搭建和开发过程。
* Spring Boot整合了许多框架和第三方库配置，几乎可以达到“开箱即用”。


从最根本上来讲，Spring Boot 就是一些库的集合，它能够被任意项目的构建系统所使用。它使用 “习惯优于配置” （项目中存在大量的配置，此外还内置一个习惯性的配置）的理念让你的项目快速运行起来。用大佬的话来理解，就是 spring boot 其实不是什么新的框架，它默认配置了很多框架的使用方式，就像 maven 整合了所有的 jar 包，spring boot 整合了所有的框架，总结一下及几点：

1. 为所有Spring开发提供一个更快更广泛的入门体验
2. 零配置，无冗余代码的生成和XML强制配置，遵循“约定大于配置”
3. 集成了大量常用的第三方库的配置，SpringBoot应用为这些第三方库提供了几乎可以零配置的开箱即用的功能
4. 提供了一系列大型项目常用的非功能性的特征，如嵌入式服务器、安全性、度量、运行状况检查、外部化配置等
5. springBoot不是Spring的替代者，Spring框架是通过IOC机制来管理Bean的。SpringBoot以来Spring框架来管理对象的依赖，SpringBoot并不是Spring的精简版本，而是为使用spring做好各种产品级准备

### 优势

* 可快速构建独立的Spring应用	
* 直接嵌入Tomcat、Jetty和Undertow服务器（无需部署WAR文件）
* 提供依赖启动器简化构建配置
* 极大程度的自动化配置Spring和第三方库
* 提供生产就绪功能
* 极少的代码生成和XML配置

### springBoot在应用中的角色

* Spring Boot 是基于 Spring Framework 来构建的，Spring Framework 是一种 J2EE 的框架
* Spring Boot 是一种快速构建 Spring 应用
* Spring Cloud 是构建 Spring Boot 分布式环境，也就是常说的云应用
* Spring Boot 中流砥柱，承上启下



## 新建SpringBoot项目

### 环境准备

* JDK 1.8.0_201（及以上版本）
* Apache Maven 3.6.0（ Maven 管理工具 3.2.5 及以上版本）
* IntelliJ IDEA Ultimate旗舰版


### 使用Maven创建Spring Boot项目

#### 搭建步骤

1. 创建Maven项目
2. 在pom.xml中添加Spring Boot相关依赖
3. 编写主程序启动类
4. 创建一个用于Web访问的Controller
5. 运行项目

#### 在pom.xml中添加Spring Boot相关依赖

	<!-- 引入Spring Boot依赖 -->
	<parent>
		<groupId>org.springframework.boot</groupId>
		//统一父类管理
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.3.RELEASE</version>
	</parent>
	<dependencies>
		<!-- 引入Web场景依赖启动器 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			//Web依赖启动器
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
	</dependencies>

#### 编写主程序启动类
	
	//标记该类为主程序启动类
	@SpringBootApplication 
	public class ManualChapter01Application {
	public static void main(String[] args){
			//SpringApplication.run()方法启动主程序类       
			SpringApplication.run(ManualChapter01Application.class,args);
	    }
	 }

#### 创建一个用于Web访问的Controller

	//该注解为组合注解，等同于Spring中的@Controller+@ResponseBody
	@RestController   
	public class HelloController {
	//等同于Spring框架中@RequestMapping(RequestMethod.GET)注解
	@GetMapping("/hello")
	   public String hello(){
	        return "hello Spring Boot";
	    }
	}

#### 启动项目

在浏览器上访问 http://localhost:8080/hello

![](https://s1.ax1x.com/2020/07/20/U4W0Ts.png)

### 使用Spring Initializr创建SpringBoot项目

#### 安装springBoot插件

plugins里边搜Spring Assistant，然后点击安装

![](https://s1.ax1x.com/2020/07/20/U4Wf0J.md.png)

#### 创建项目

创建Spring Assistant

![](https://s1.ax1x.com/2020/07/20/U4WrYq.md.png)

直接next

![](https://s1.ax1x.com/2020/07/20/U4WDkn.md.png)

勾选Web依赖

![](https://s1.ax1x.com/2020/07/20/U4Wsf0.md.png)


选择项目位置，finish，创建成功，记得配置本地maven，不然下载速度怀疑人生，还容易超时

![](https://s1.ax1x.com/2020/07/20/U4W2XF.md.png)

项目结构还是看上去挺清爽的，少了很多配置文件，我们来了解一下默认生成的有什么：

* SpringbootApplication： 一个带有 main() 方法的类，用于启动应用程序
* SpringbootApplicationTests：一个空的 Junit 测试了，它加载了一个使用Spring Boot 字典配置功能的 Spring 应用程序上下文
* application.properties：一个空的 properties 文件，可以根据需要添加配置属性
* pom.xml： Maven 构建说明文件

#### 创建HelloController

	package com.xpwi.springboot;
	
	import org.springframework.web.bind.annotation.RequestMapping;
	import org.springframework.web.bind.annotation.RestController;
	
	@RestController
	public class HelloController {
	
	    @RequestMapping("/hello")
	    public String hello() {
	        return "Hello Spring Boot!";
	    }
	}

#### 运行项目

启动项目，在浏览器上访问 http://localhost:8080/hello

![](https://s1.ax1x.com/2020/07/20/U4W6pV.md.png)

## 单元测试

### 步骤

1. 在pom文件中添加spring-boot-starter-test测试启动器
2. 编写单元测试类
3. 编写单元测试方法
4. 运行结果

### 在pom文件中添加spring-boot-starter-test测试启动器

	
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>


### 编写单元测试类

	//加载Spring Boot测试注解
	@RunWith(SpringRunner.class) 
	@SpringBootTest  //加载项目的ApplicationContext上下文环境
	public class Chapter01ApplicationTests {
		@Test
		public void contextLoads() {
		}
	}

### 编写单元测试方法
	
	import org.junit.jupiter.api.Test;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.boot.test.context.SpringBootTest;
	
	@SpringBootTest
	class DemoApplicationTests {
		@Autowired
		private HelloController helloController;
	
		@Test
		void contextLoads() {
			String hello = helloController.hello();
			System.out.println(hello);
	
		}
	
	}

## 热部署

在应用运行的时升级软件，无需重新启动的方式有两种，热部署和热加载。

对于Java应用程序来说，热部署就是在服务器运行时重新部署项目，热加载即在在运行时重新加载class，从而升级应用。

### 搭建步骤

1. 在pom文件中添加spring-boot-devtools热部署依赖
2. IDEA中热部署设置
3. 热部署测试

### 添加依赖

在pom文件中添加spring-boot-devtools热部署依赖

	<dependency>                
	   <groupId>org.springframework.boot</groupId>
	   <artifactId>spring-boot-devtools</artifactId>
	</dependency>

### Settings设置

 选择【File】→【Settings】选项，打开Compiler面板设置页。

![](https://s1.ax1x.com/2020/07/20/U4Wofx.md.png)

### Registry

使用快捷键“Ctrl+Shift+Alt+/”打开Maintenance选项框，选中并打开Registry页面。

指定IDEA工具在程序运行过程中自动编译

![](https://s1.ax1x.com/2020/07/20/U4WclT.md.png)

### 热部署测试

修改类HelloController中的请求处理方法hello()的返回值，刷新浏览器。

![](https://s1.ax1x.com/2020/07/20/U4W7p6.png)

## Spring Boot 原理分析

### 依赖管理

#### spring-boot-starter-parent依赖
	
	<!-- Spring Boot父项目依赖管理 -->
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.3.RELEASE</version>
		<relativePath/>
	</parent>

spring-boot-starter-parent是通过<properties>标签对一些常用技术框架的依赖文件进行了统一版本号管理。

#### spring-boot-starter-web依赖

	
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>

spring-boot-starter-web依赖启动器的主要作用是提供Web开发场景所需的底层所有依赖文件，它对Web开发场景所需的依赖文件进行了统一管理。

### 自动配置

* Spring Boot应用的启动入口是@SpringBootApplication注解标注类中的main()方法；
* @SpringBootApplication能够扫描Spring组件并自动配置Spring Boot。
* @SpringBootApplication注解是一个组合注解，包含@SpringBootConfiguration、@EnableAutoConfiguration、@ComponentScan三个核心注解

![](https://s1.ax1x.com/2020/07/20/U4WH1K.png)

### 执行流程

Spring Boot 的执行流程主要分为两步：

1. 初始化Spring Application实例
2. 初始化Spring Boot 项目启动

![](https://s1.ax1x.com/2020/07/20/U4Wb6O.png)

#### 初始化Spring Application实例

![](https://s1.ax1x.com/2020/07/20/U4WqXD.md.png)

#### 初始化Spring Boot项目启动

![](https://s1.ax1x.com/2020/07/20/U4WOne.md.png)