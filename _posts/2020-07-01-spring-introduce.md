---
layout: post
title: "Spring 学习笔记（1）——Spring概述和IOC示例程序"
date: 2020-07-01 13:24:36 +0800
categories: notes
tags: spring
img: https://s1.ax1x.com/2020/07/02/NqzFiV.png
---
Spring概述，IOC的几种示例程序：通过id获取对象；通过类型获取对象；通过构造方法参数名；index属性指定；根据参数类型注入；


## Spring概述

1. Spring是一个开源框架
2. Spring为简化企业级开发而生，使用Spring开发可以将Bean对象，Dao组件对象，Service组件对象等交给Spring容器来管理，这样使得很多复杂的代码在Spring中开发却变得非常的优雅和简洁，有效的降低代码的耦合度，极大的方便项目的后期维护、升级和扩展。
3. Spring是一个IOC(DI)和AOP容器框架。
4. Spring的优良特性
	* 非侵入式：基于Spring开发的应用中的对象可以不依赖于Spring的API
	* 控制反转：IOC——Inversion of Control，指的是将对象的创建权交给Spring去创建。使用Spring之前，对象的创建都是由我们自己在代码中new创建。而使用Spring之后。对象的创建都是由给了Spring框架。
	* 依赖注入：DI——Dependency Injection，是指依赖的对象不需要手动调用setXX方法去设置，而是通过配置赋值。
	* 面向切面编程：Aspect Oriented Programming——AOP
	* 容器：Spring是一个容器，因为它包含并且管理应用对象的生命周期
	* 组件化：Spring实现了使用简单的组件配置组合成一个复杂的应用。在 Spring 中可以使用XML和Java注解组合这些对象。
	* 一站式：在IOC和AOP的基础上可以整合各种企业应用的开源框架和优秀的第三方类库（实际上Spring 自身也提供了表述层的SpringMVC和持久层的Spring JDBC）。

## Spring的模块介绍

Spring框架分为四大模块：

![](https://s1.ax1x.com/2020/07/02/NqzARU.md.png)

#### Core核心模块。

负责管理组件的Bean对象
    spring-beans-4.0.0.RELEASE.jar
    spring-context-4.0.0.RELEASE.jar
    spring-core-4.0.0.RELEASE.jar
    spring-expression-4.0.0.RELEASE.jar

#### 面向切面编程
    spring-aop-4.0.0.RELEASE.jar
    spring-aspects-4.0.0.RELEASE.jar

#### 数据库操作
    spring-jdbc-4.0.0.RELEASE.jar
    spring-orm-4.0.0.RELEASE.jar
    spring-oxm-4.0.0.RELEASE.jar
    spring-tx-4.0.0.RELEASE.jar
    spring-jms-4.0.0.RELEASE.jar

#### Web模块
    spring-web-4.0.0.RELEASE.jar
    spring-webmvc-4.0.0.RELEASE.jar
    spring-websocket-4.0.0.RELEASE.jar
    spring-webmvc-portlet-4.0.0.RELEASE.jar

### 安装Spring的插件

网上有比较多的eclipse添加spring插件的方式，读者可自行从网络上获取下载经验。

## IOC依赖注入

### IOC定义

IOC	全称指的是 Inverse Of Control 控制反转。 

控制反转指的是对象的创建权力被反转了。

使用Spring以前，对象都是自己new去创建。

使用Spring之后。对象都通过配置Spring的配置文件，然后由Spring容器负责创建。

### DI的定义

DI 指的是Dependency	Injection 。是依赖注入的意思。

没有使用Spring的时候对依赖对象的赋值。
    
    public class BookService {
    private BookDao bookDao;
    
    public void setBookDao( BookDao bookDao ){
    this.bookDao = bookDao;
    }
    
    }
    
使用了Spring之后，就不需要再通过编码来实现对依赖对象的赋值。而是通过配置。

## 第一个IOC示例程序 -- 通过id获取对象

实验1：通过IOC容器创建对象，并为属性赋值

第一步，创建一个Java工程


并导入jar包

创建config源码目录

#### 准备JavaBean对象

    public class Person {
    	private Integer id;
    	private String name;
    	private String phone;
    	private Integer age;

#### 创建applicationContext.xml配置文件，并配置bean对象

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    	<!-- 
    		bean标签用来配置一个Bean（bean就是对象）
    			class属性设置你要配置的Bean的全类名
    			id属性设置一个唯一的标识
    	 -->
    	<bean id="p1" class="com.ncu.zte.beans.Person">
    		<!-- property标签配置属性值
    				name设置属性名
    				value属性设置值
    		 -->
    		<property name="id" value="1" />
    		<property name="name" value="国哥又有机会帅了" />
    		<property name="age" value="18" />
    		<property name="phone" value="18610541354" />
    	</bean>
    
    </beans>

#### 测试的代码：

    public class test {
    
    	@Test
    	public void test1() throws Exception {
    		// applicationContext.xml是Spring的配置文件，
    		// 我们需要先有一个Spring容器（Spring IOC 容器），再从容器中获取配置的bean对象
    		//ApplicationContext接口表示Spring IOC容器
    		// Spring容器（Spring IOC容器） 在初始化的时候需要一个配置文件
    		// ClassPathXmlApplicationContext表示从Classpath类路径下加载你指定的配置文件名，生成Spring容器
    		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicaionContext.xml");
    		// getBean是从Spring容器中获取指定id值的bean对象
    		Person person = (Person) applicationContext.getBean("p1");
    	
    		System.out.println(person);
    	}
    }

#### 一些问题

* FileSystemXmlApplicationContext怎么用?

答：跟使用JavaSE的相对路径一样

// 从文件系统路径中加载指定的xml配置文件生成Spring容器

		ApplicationContext applicationContext = new FileSystemXmlApplicationContext("config/applicaionContext.xml");


* Bean是在什么时候被创建的?

在创建ApplicatiocnContext容器对象的时候创建（默认）

* 如果调用getBean多次，会创建几个?

默认创建同一个

#### 常见的错误

指定的id不存在。找不到bean对象。

## IOC示例程序 -- 通过类型获取对象（重点）

实验2：根据bean的类型从IOC容器中获取bean的实例★

applicationContext.xml配置文件：

    <!-- 
    		bean标签用来配置一个Bean（bean就是对象）
    			class属性设置你要配置的Bean的全类名
    			id属性设置一个唯一的标识
    	 -->
    	<bean id="p1" class="com.atguigu.pojo.Person">
    		<!-- property标签配置属性值
    				name设置属性名
    				value属性设置值
    		 -->
    		<property name="id" value="1" />
    		<property name="name" value="国哥又有机会帅了" />
    		<property name="age" value="18" />
    		<property name="phone" value="18610541354" />
    	</bean>
    	<bean id="p2" class="com.atguigu.pojo.Person">
    		<!-- property标签配置属性值
    				name设置属性名
    				value属性设置值
    		 -->
    		<property name="id" value="2" />
    		<property name="name" value="国哥帅2次" />
    		<property name="age" value="18" />
    		<property name="phone" value="18610541354" />
    	</bean>

#### 测试代码：

	@Test
	public void test2() throws Exception {
		// 先创建Spring容器对象
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicaionContext.xml");
		// 通过类型获取Spring容器中的对象
		/**
		 * 如果通过类型查找bean对象。<br/>
		 * 	1、找到一个就直接返回<br/>
		 *  2、找到两个就报错<br/>
		 *  3、没有找到也报错
		 */
		Person person = applicationContext.getBean(Person.class);
		
		System.out.println( person );
	}

常见错误说明：

当在applicationContext.xml配置文件中。有多个同Person.class类型实现的时候。

![](https://s1.ax1x.com/2020/07/02/NqzCaq.md.png)

## IOC示例程序 -- 通过构造方法参数名注入值

实验3：通过构造器为bean的属性赋值

#### applicationContext.xml配置文件：

* constructor-arg 标签是指通过构造器赋值
* name设置构造器的参数名
* value设置构造器对应的参数值

	<bean id="p3" class="com.ncu.zte.beans.Person">
		<constructor-arg name="id" value="3"></constructor-arg>
		<constructor-arg name="name" value="武俊伟"></constructor-arg>
		<constructor-arg name="phone" value="15270868888"></constructor-arg>
		<constructor-arg name="age" value="18"></constructor-arg>
	</bean>

#### 测试代码：

	@Test
	public void test3() throws Exception {
		// 先创建Spring容器对象
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicaionContext.xml");
		
		Person p3 = (Person) applicationContext.getBean("p3");
		
		System.out.println(p3);
		
	}

因为p1,p2都是调用了无参构造器进行赋值，p3是通过有参构造器进行的赋值，所以运行结果如下：

![](https://s1.ax1x.com/2020/07/02/NqzPI0.md.png)

## IOC示例程序 -- index属性指定参数的位置

实验4：通过index属性指定参数的位置

#### applicationContext.xml配置文件：

		<!-- Person(Integer id, String name, String phone, Integer age)  -->
		<!-- idnex属性设置参数的索引 
				0 第一个参数
				1 第二个参数
				…… 以此类推
				n 第n+1个参数
		 -->

#### applicationContext.xml配置文件：

	<bean id="p4" class="com.ncu.zte.beans.Person">
		<constructor-arg index="0" value="3"></constructor-arg>
		<constructor-arg index="1" value="武俊伟"></constructor-arg>
		<constructor-arg index="2" value="15270868888"></constructor-arg>
		<constructor-arg index="3" value="18"></constructor-arg>
	</bean>

#### 测试代码：

	@Test
	public void test4() throws Exception {
		// 先创建Spring容器对象
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicaionContext.xml");
		
		System.out.println( applicationContext.getBean("p4") );
		
	}

![](https://s1.ax1x.com/2020/07/02/Nqz9Zn.md.png)

## IOC示例程序 -- 根据参数类型注入

实验5：根据参数类型注入

#### 有两上Person的有参构造器：
    
    public class Person {
    
    	private Integer id;
    	private String name;
    	private String phone;
    	private Integer age;
    
    	public Person(Integer id, String name, String phone, Integer age) {
    		super();
    		System.out.println("有参构造器");
    		this.id = id;
    		this.name = name;
    		this.phone = phone;
    		this.age = age;
    	}
    	
    	public Person(Integer id, String name, Integer age, String phone) {
    		super();
    		System.out.println("有参构造器");
    		this.id = id;
    		this.name = name;
    		this.phone = phone;
    		this.age = age;
    	}
    
#### applicationContext.xml配置文件

	<bean id="p5" class="com.ncu.zte.beans.Person">
		<constructor-arg index="0" value="3"></constructor-arg>
		<constructor-arg index="1" value="武俊伟" ></constructor-arg>
		<constructor-arg index="2" value="15" ></constructor-arg>
		<constructor-arg index="3" value="18" ></constructor-arg>
	</bean>

测试结果：

![](https://s1.ax1x.com/2020/07/02/NqzkGT.md.png)

	<bean id="p5" class="com.ncu.zte.beans.Person">
		<constructor-arg index="0" value="3" type="java.lang.Integer"></constructor-arg>
		<constructor-arg index="1" value="武俊伟" type="java.lang.String"></constructor-arg>
		<constructor-arg index="2" value="15" type="java.lang.String"></constructor-arg>
		<constructor-arg index="3" value="18" type="java.lang.Integer"></constructor-arg>
	</bean>

测试结果：

![](https://s1.ax1x.com/2020/07/02/NqzEzF.md.png)



