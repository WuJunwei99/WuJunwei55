---
layout: post
title: "mybatis 学习笔记（1）——概述及简单使用"
date: 2020-06-28 15:20:12 +0800
categories: notes
tags: mybatis
img: https://s1.ax1x.com/2020/06/29/Nfjd8f.png
---
mybatis简介；使用mybatis的简单示例程序；log4j配置日记功能


## 概述


### mybatis简介

MyBatis 是支持定制化 SQL、存储过程以及高级映射的优秀的持久层框架。

MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。

MyBatis可以使用简单的XML或注解用于配置和原始映射，将接口和Java的POJO（Plain Old Java Objects，普通的Java对象）映射成数据库中的记录。

### mybatis历史

原是apache的一个开源项目iBatis, 2010年6月这个项目由apache software foundation 迁移到了google code，随着开发团队转投Google Code旗下，ibatis3.x正式更名为Mybatis ，代码于2013年11月迁移到Github（下载地址见后）。
iBATIS一词来源于“internet”和“abatis”的组合，是一个基于Java的持久层框架。iBATIS提供的持久层框架包括SQL Maps和Data Access Objects（DAO）

### 为什么要使用mybatis

MyBatis是一个半自动化的持久化层框架。

jdbc编程---当我们使用jdbc持久化的时候，sql语句被硬编码到java代码中。这样耦合度太高。代码不易于维护。在实际项目开发中会经常添加sql或者修改sql，这样我们就只能到java代码中去修改。

Hibernate和JPA
长难复杂SQL，对于Hibernate而言处理也不容易
内部自动生产的SQL，不容易做特殊优化。
基于全映射的全自动框架，javaBean存在大量字段时无法只映射部分字段。导致数据库性能下降。

对开发人员而言，核心sql还是需要自己优化
sql和java编码分开，功能边界清晰，一个专注业务、一个专注数据。
可以使用简单的XML或注解用于配置和原始映射，将接口和Java的POJO映射成数据库中的记录。成为业务代码+底层数据库的媒介


## mybatis的简单示例程序

### 创建一个数据库和一个单表


    drop database if exists mybatis;
    
    create database mybatis;
    
    use mybatis;
    
    create table t_user(
    	`id` int primary key auto_increment,
    	`last_name`	varchar(50),
    	`sex` int
    );
    
    insert into t_user(`last_name`,`sex`) values('wzg168',1);
    
    select * from t_user;
    
### 搭建mybatis开发环境

创建一个Java工程,最后项目完成时，最终的目录结构如下：

![](https://s1.ax1x.com/2020/06/29/NfjN5t.png)


### 创建mybatis-config.xml核心配置文件

每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为中心的。SqlSessionFactory 的实例可以通过 SqlSessionFactoryBuilder 获得。而 SqlSessionFactoryBuilder 则可以从 XML 配置文件或一个预先定制的 Configuration 的实例构建出 SqlSessionFactory 的实例。


从 XML 文件中构建 SqlSessionFactory 的实例非常简单，建议使用类路径下的资源文件进行配置。但是也可以使用任意的输入流(InputStream)实例，包括字符串形式的文件路径或者 file:// 的 URL 形式的文件路径来配置。MyBatis 包含一个名叫 Resources 的工具类，它包含一些实用方法，可使从 classpath 或其他位置加载资源文件更加容易。

    String resource = "org/mybatis/example/mybatis-config.xml";
    InputStream inputStream = Resources.getResourceAsStream(resource);
    sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);

XML 配置文件（configuration XML）中包含了对 MyBatis 系统的核心设置，包含获取数据库连接实例的数据源（DataSource）和决定事务范围和控制方式的事务管理器（TransactionManager）。XML 配置文件的详细内容后面再探讨，这里先给出一个简单的示例：

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
      PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
    	
      <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <!-- 配置数据源
      		需要配置数据库的四个连接属性
       -->
      <dataSource type="POOLED">
    <property name="driver" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
    <property name="username" value="root"/>
    <property name="password" value="root"/>
      </dataSource>
    </environment>
      </environments>
      <!-- 
      	mappers标签用来配置 sql 的 mapper配置文件
       -->
      <mappers>
      	<!-- mapper引入一个sql语句的配置文件
      			resource属性配置你要引入的配置文件的路径
      	 -->
    <mapper resource="com/ncu/zte/mybatis6_29/UserMapper.xml"/>
      </mappers>
    </configuration>


当然，还有很多可以在XML 文件中进行配置，上面的示例指出的则是最关键的部分。要注意 XML 头部的声明，用来验证 XML 文档正确性。environment 元素体中包含了事务管理和连接池的配置。mappers 元素则是包含一组 mapper 映射器（这些 mapper 的 XML 文件包含了 SQL 代码和映射定义信息）。

    
### 在User对象的包下，创建UserMapper.xml配置文件

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
      <!-- 
      namespace 名称空间（一般有两种）
      1.对应的javabean的全类名
      2.对应的mapper接口的全类名
       -->
    <mapper namespace="com.ncu.zte.mybtis6_29.User">
    
    <!-- 
    select 是sql语句
    id是当前的这个语句配置的一个唯一标识符
    resultType执行了select的查询语句后，每行记录对应的javabean对象全类名
    // #{id}在mybatis中是占位符
     -->
    
      <select id="selectUserById" resultType="com.ncu.zte.mybatis6_29.User">
       select id,last_name,sex from t_user where id = #{id}
      </select>
    </mapper>

### 从 SqlSessionFactory 中获取 SqlSession

既然有了 SqlSessionFactory ，顾名思义，我们就可以从中获得 SqlSession 的实例了。SqlSession 完全包含了面向数据库执行 SQL 命令所需的所有方法。你可以通过 SqlSession 实例来直接执行已映射的 SQL 语句。例如：

    SqlSession session = sqlSessionFactory.openSession();
    try {
      Blog blog = (Blog) session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);
    } finally {
      session.close();
    }

诚然这种方式能够正常工作，并且对于使用旧版本 MyBatis 的用户来说也比较熟悉，不过现在有了一种更直白的方式。使用对于给定语句能够合理描述参数和返回值的接口（比如说BlogMapper.class），你现在不但可以执行更清晰和类型安全的代码，而且还不用担心易错的字符串字面值以及强制类型转换。


    SqlSession session = sqlSessionFactory.openSession();
    try {
      BlogMapper mapper = session.getMapper(BlogMapper.class);
      Blog blog = mapper.selectBlog(101);
    } finally {
      session.close();
    }
    
### 测试生成SqlSessionFactory对象

    @Test
    public void test2() throws IOException{
    		InputStream is = Resources.getResourceAsStream("com/ncu/zte/mybatis6_29/mybatis-config.xml");
    		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(is);
    		SqlSession session = sqlSessionFactory.openSession();
    		try{
    			User user = session.selectOne("com.ncu.zte.mybtis6_29.User.selectUserById", 1);
    			System.out.println(user);
    		}finally{
    			session.close();
    		}
    	}


## 给mybatis配置日记功能
    
    \# Global logging configuration
    log4j.rootLogger=DEBUG, stdout
    \# Console output...
    log4j.appender.stdout=org.apache.log4j.ConsoleAppender
    log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
    log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n

上述测试的日志信息如下：

![](https://s1.ax1x.com/2020/06/29/NfjaPP.md.png)

### mybatis配置文件的提示

![](https://s1.ax1x.com/2020/06/29/Nfjw28.md.png)

![](https://s1.ax1x.com/2020/06/29/NfjtUI.md.png)






