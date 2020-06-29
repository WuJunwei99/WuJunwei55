---
layout: post
title: "maven 学习笔记（3）——聚合和继承"
date: 2020-06-23 19:15:24 +0800
categories: notes
tags: maven
img: https://s1.ax1x.com/2020/06/23/NUss58.png
---
创建父工程;创建web工程;创建service工程;创建mapper工程;项目结构;
依赖的统一管理

多模块（聚合）： 将maven项目分解，**为了统一开发和代码复用**；

继承： 使用pom配置，**为了统一管理并消除重复**；

聚合主要为了快速构建项目，继承主要为了消除重复

提示：两者虽然概念不同，但在企业中会一起使用。

示例：构建一个父工程，多个子工程，让父工程管理子工程。

![](https://s1.ax1x.com/2020/06/25/NDClpd.md.png)

## 创建父工程（usermanage-parent）

跳过骨架创建maven工程：

1. new maven project
2. 勾选使用简单骨架
3. 打包方式为**pom**
4. 点击“finish”

![](https://s1.ax1x.com/2020/06/25/NDCmTO.png)


## 创建web工程（usermanage-web）

1. 右键usermanage-parent工程，创建maven module模块
2. 勾选使用简单骨架，填写模块名称
3. 注意打包方式选择**war**
4. 点击“finish”

![](https://s1.ax1x.com/2020/06/25/NDCKte.png)

不要忘了maven刷新工程，并添加web.xml

把usermanage工程中的controller层的代码放入usermanage-web工程：

![](https://s1.ax1x.com/2020/06/25/NDCMfH.png)

![](https://s1.ax1x.com/2020/06/25/NDC11A.png)

## 创建service工程（usermanage-service）


1. 创建usermanage-service模块
2. 勾选使用简单骨架
3. 打包方式选jar
4. 点击“finish”

![](https://s1.ax1x.com/2020/06/25/NDCukD.png)

把usermanage工程中的service层的代码放入usermanage-web工程：

![](https://s1.ax1x.com/2020/06/25/NDC36I.png)

## 创建mapper工程（usermanage-mapper）

同service工程

把usermanage工程中的mapper层的代码放入usermanage-web工程：

![](https://s1.ax1x.com/2020/06/25/NDC8Xt.png)

## 项目结构

![](https://s1.ax1x.com/2020/06/25/NDCJnP.png)

打开usermanage-parent的pom.xml会发现有段儿配置：

![](https://s1.ax1x.com/2020/06/25/NDCY0f.md.png)

打开工作空间,发现只有父工程目录，没有子模块目录

打开工作空间中的usermanage-parent工程，查看目录结构：子模块对应的目录都在父工程中。

## 依赖的统一管理

在usermanage-parent父工程的pom.xml中配置：

    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    	<modelVersion>4.0.0</modelVersion>
    	<groupId>com.atguigu.usermanage</groupId>
    	<artifactId>usermanage-parent</artifactId>
    	<version>0.0.1-SNAPSHOT</version>
    	<packaging>pom</packaging>
    
    	<!-- 集中定义依赖版本号 -->
    	<properties>
    		<junit.version>4.10</junit.version>
    		<spring.version>4.3.22.RELEASE</spring.version>
    		<mybatis.version>3.4.1</mybatis.version>
    		<mybatis.spring.version>1.3.2</mybatis.spring.version>
    		<mysql.version>5.1.37</mysql.version>
    		<slf4j.version>1.6.4</slf4j.version>
    		<jackson.version>2.9.5</jackson.version>
    		<c3p0.version>0.9.1.2</c3p0.version>
    		<hibernate.version>5.1.3.Final</hibernate.version>
    		<jstl.version>1.2</jstl.version>
    		<servlet-api.version>2.5</servlet-api.version>
    		<jsp-api.version>2.0</jsp-api.version>
    	</properties>
    
		<!--只是起到管理版本号的作用-->
    	<dependencyManagement>
    		<dependencies>
    			<!-- 单元测试 -->
    			<dependency>
    				<groupId>junit</groupId>
    				<artifactId>junit</artifactId>
    				<version>${junit.version}</version>
    				<scope>test</scope>
    			</dependency>
    
    			<!-- Spring -->
    			<dependency>
    				<groupId>org.springframework</groupId>
    				<artifactId>spring-context</artifactId>
    				<version>${spring.version}</version>
    			</dependency>
    			<dependency>
    				<groupId>org.springframework</groupId>
    				<artifactId>spring-beans</artifactId>
    				<version>${spring.version}</version>
    			</dependency>
    			<dependency>
    				<groupId>org.springframework</groupId>
    				<artifactId>spring-webmvc</artifactId>
    				<version>${spring.version}</version>
    			</dependency>
    			<dependency>
    				<groupId>org.springframework</groupId>
    				<artifactId>spring-jdbc</artifactId>
    				<version>${spring.version}</version>
    			</dependency>
    			<dependency>
    				<groupId>org.springframework</groupId>
    				<artifactId>spring-aspects</artifactId>
    				<version>${spring.version}</version>
    			</dependency>
    
    			<!-- Mybatis -->
    			<dependency>
    				<groupId>org.mybatis</groupId>
    				<artifactId>mybatis</artifactId>
    				<version>${mybatis.version}</version>
    			</dependency>
    			<dependency>
    				<groupId>org.mybatis</groupId>
    				<artifactId>mybatis-spring</artifactId>
    				<version>${mybatis.spring.version}</version>
    			</dependency>
    
    			<!-- MySql -->
    			<dependency>
    				<groupId>mysql</groupId>
    				<artifactId>mysql-connector-java</artifactId>
    				<version>${mysql.version}</version>
    			</dependency>
    
    			<dependency>
    				<groupId>org.hibernate</groupId>
    				<artifactId>hibernate-validator</artifactId>
    				<version>${hibernate.version}</version>
    			</dependency>
    
    			<dependency>
    				<groupId>org.slf4j</groupId>
    				<artifactId>slf4j-log4j12</artifactId>
    				<version>${slf4j.version}</version>
    			</dependency>
    
    			<!-- Jackson Json处理工具包 -->
    			<dependency>
    				<groupId>com.fasterxml.jackson.core</groupId>
    				<artifactId>jackson-databind</artifactId>
    				<version>${jackson.version}</version>
    			</dependency>
    
    			<!-- 连接池 -->
    			<dependency>
    				<groupId>c3p0</groupId>
    				<artifactId>c3p0</artifactId>
    				<version>${c3p0.version}</version>
    			</dependency>
    
    			<!-- JSP相关 -->
    			<dependency>
    				<groupId>jstl</groupId>
    				<artifactId>jstl</artifactId>
    				<version>${jstl.version}</version>
    			</dependency>
    			<dependency>
    				<groupId>javax.servlet</groupId>
    				<artifactId>servlet-api</artifactId>
    				<version>${servlet-api.version}</version>
    				<scope>provided</scope>
    			</dependency>
    			<dependency>
    				<groupId>javax.servlet</groupId>
    				<artifactId>jsp-api</artifactId>
    				<version>${jsp-api.version}</version>
    				<scope>provided</scope>
    			</dependency>
    
    		</dependencies>
    	</dependencyManagement>
    	
    	<build>
    		<pluginManagement>
    			<plugins>
    				<!-- 配置Tomcat插件 -->
    				<plugin>
    					<groupId>org.apache.tomcat.maven</groupId>
    					<artifactId>tomcat7-maven-plugin</artifactId>
    					<version>2.2</version>
    				</plugin>
    			</plugins>
    		</pluginManagement>
    	</build>
    
    	<modules>
    		<module>usermanage-web</module>
    		<module>usermanage-service</module>
    		<module>usermanage-mapper</module>
    	</modules>
    
    </project>

## 引入依赖

子工程之间的依赖关系：

usermanage-web  usermanageservice  usermanage-mapper

解决代码报错，需要引入相关依赖。引入依赖的原则是：

1. 所有的工程都需要的依赖应该在聚合工程（usermanage-parent）中导入。
2. 在使用依赖的最底层导入。
3. 运行时所需要的依赖在web工程中加入。


usermanage-web工程的pom.xml:

    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    	<modelVersion>4.0.0</modelVersion>
    	<parent>
    		<groupId>com.atguigu.usermanage</groupId>
    		<artifactId>usermanage-parent</artifactId>
    		<version>0.0.1-SNAPSHOT</version>
    	</parent>
    	<artifactId>usermanage-web</artifactId>
    	<packaging>war</packaging>
    	<dependencies>
    		<dependency>
    			<groupId>com.atguigu.usermanage</groupId>
    			<artifactId>usermanage-service</artifactId>
    			<version>0.0.1-SNAPSHOT</version>
    		</dependency>
    	
    		<dependency>
    			<groupId>junit</groupId>
    			<artifactId>junit</artifactId>
    			<scope>test</scope>
    		</dependency>
    		
    		<dependency>
    			<groupId>org.springframework</groupId>
    			<artifactId>spring-webmvc</artifactId>
    		</dependency>
    
    		<!-- 运行时依赖 -->
    		<dependency>
    			<groupId>org.springframework</groupId>
    			<artifactId>spring-jdbc</artifactId>
    		</dependency>
    
    		<!-- 运行时依赖 -->
    		<dependency>
    			<groupId>org.springframework</groupId>
    			<artifactId>spring-aspects</artifactId>
    		</dependency>
    
    		<!-- 运行时依赖 -->
    		<dependency>
    			<groupId>org.mybatis</groupId>
    			<artifactId>mybatis-spring</artifactId>
    		</dependency>
    		
    		<!-- 运行时依赖 -->
    		<dependency>
    			<groupId>mysql</groupId>
    			<artifactId>mysql-connector-java</artifactId>
    		</dependency>
    
    		<!-- 运行时依赖 -->
    		<dependency>
    			<groupId>org.slf4j</groupId>
    			<artifactId>slf4j-log4j12</artifactId>
    		</dependency>
    		
    		<!-- 运行时依赖 -->
    		<dependency>
    			<groupId>c3p0</groupId>
    			<artifactId>c3p0</artifactId>
    		</dependency>
    
    		<!-- JSP相关 -->
    		<dependency>
    			<groupId>jstl</groupId>
    			<artifactId>jstl</artifactId>
    		</dependency>
    		<dependency>
    			<groupId>javax.servlet</groupId>
    			<artifactId>servlet-api</artifactId>
    			<scope>provided</scope>
    		</dependency>
    		<dependency>
    			<groupId>javax.servlet</groupId>
    			<artifactId>jsp-api</artifactId>
    			<scope>provided</scope>
    		</dependency>
    
    	</dependencies>
    	<build>
    		<plugins>
    			<!-- 配置Tomcat插件 -->
    			<plugin>
    				<groupId>org.apache.tomcat.maven</groupId>
    				<artifactId>tomcat7-maven-plugin</artifactId>
    				<configuration>
    					<!-- 配置它之后，不需要项目名就可以访问 -->
    					<path>/</path>
    					<!-- 配置端口 -->
    					<port>8080</port>
    				</configuration>
    			</plugin>
    		</plugins>
    	</build>
    </project>

Usermanage-service的pom.xml：

    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    	<modelVersion>4.0.0</modelVersion>
    	<parent>
    		<groupId>com.atguigu.usermanage</groupId>
    		<artifactId>usermanage-parent</artifactId>
    		<version>0.0.1-SNAPSHOT</version>
    	</parent>
    	<artifactId>usermanage-service</artifactId>
    
    	<dependencies>
    
    		<dependency>
    			<groupId>com.atguigu.usermanage</groupId>
    			<artifactId>usermanage-mapper</artifactId>
    			<version>0.0.1-SNAPSHOT</version>
    		</dependency>
    
    	</dependencies>
    
    </project>

Usermanage-mapper的pom.xml：

    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    	<modelVersion>4.0.0</modelVersion>
    	<parent>
    		<groupId>com.atguigu.usermanage</groupId>
    		<artifactId>usermanage-parent</artifactId>
    		<version>0.0.1-SNAPSHOT</version>
    	</parent>
    	<artifactId>usermanage-mapper</artifactId>
    	<dependencies>
    		<dependency>
    			<groupId>org.mybatis</groupId>
    			<artifactId>mybatis</artifactId>
    		</dependency>
    		<dependency>
    			<groupId>org.hibernate</groupId>
    			<artifactId>hibernate-validator</artifactId>
    		</dependency>
    		<dependency>
    			<groupId>com.fasterxml.jackson.core</groupId>
    			<artifactId>jackson-databind</artifactId>
    		</dependency>
    		<dependency>
    			<groupId>org.springframework</groupId>
    			<artifactId>spring-context</artifactId>
    		</dependency>
    	</dependencies>
    </project>

## 运行测试

1. 右键父工程
2. tomcat7:run
