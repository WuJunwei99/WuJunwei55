---
layout: post
title: "Spring 学习笔记（3）——IOC方法创建Bean"
date: 2020-07-02 13:22:53 +0800
categories: notes
tags: Spring
img: https://s1.ax1x.com/2020/07/02/NqzFiV.png
---
=内部Bean的使用，工厂实例方法创建Bean，工厂实例方法创建Bean，自动注入，Bean的单例和多例


## IOC之静态工厂方法创建Bean

实验15：配置通过静态工厂方法创建的bean

创建Person的工厂类

    public class PersonFactory {
    
    	public static Person createPerson() {
    		return new Person(15, "静态工厂方法创建的bean对象", null);
    	}
    	
    }

在applicationContext.xml中的配置：

* 配置调用静态工厂方法创建Bean对象
* class 配置工厂类的全类名
* factory-method 调用工厂类的哪个方法

	<bean id="p16" class="com.ncu.zte.factory.PersonFactory" factory-method="creatPerson">
	</bean>

![](https://s1.ax1x.com/2020/07/03/NLxo0s.md.png)

## IOC之工厂实例方法创建Bean

    public class PersonFactory {
    	
    	public Person createPerson2() {
    		return new Person(16, "工厂实例方法创建的bean对象", null);
    	}
    	
    }

applicationContext.xml中的配置


1. 在spring容器中创建一个bean对象
2. 配置调用工厂实例的方法


    <bean id="personFactory" class="com.ncu.zte.factory.PersonFactory"></bean>
	<!-- 
		class属性和factory-method组合是静态工厂方法
		factory-bean 和 factory-method 组合是工厂实例方法
		
		
		factory-bean 使用哪个bean对象做为工厂实例
		factory-method 调用工厂类实例的哪个方法
	 -->
	<bean id="p16" factory-bean="personFactory" factory-method="createPerson2"></bean>

![](https://s1.ax1x.com/2020/07/03/NLzGjg.md.png)

## IOC之FactoryBean接口方式创建对象

实验17：配置FactoryBean接口创建Bean对象

实现FactoryBean接口

![](https://s1.ax1x.com/2020/07/03/NLxqhV.md.png)

    public class PersonFacotryBean implements FactoryBean<Person> {
    	/**
    	 * 创建bean对象的方法
    	 */
    	@Override
    	public Person getObject() throws Exception {
    		return new Person(17, "这是FactoryBean接口创建的bean", null);
    	}
    	/**
    	 * 获取bean的具体类型的方法
    	 */
    	@Override
    	public Class<?> getObjectType() {
    		return Person.class;
    	}
    	/**
    	 * 判断是否是单例的方法
    	 */
    	@Override
    	public boolean isSingleton() {
    		return true;
    	}
    
    }

applicationContext.xml配置文件

如果指定的Class是实现了Spring的FactoryBean接口，
Spring容器本身会自动的判断，如果有实现这个FactoryBean接口，
创建对象的时候，就会调用 getObject()返回对象

	<bean id="p17" class="com.atguigu.factory.PersonFacotryBean"></bean>

![](https://s1.ax1x.com/2020/07/03/NLxHkq.md.png)


## IOC之继承Bean配置

	<bean id="parent" class="com.ncu.zte.Person">
		<property name="id" value="100" />
		<property name="name" value="我是父亲" />
		<property name="list">
			<list>
				<value>list1</value>
				<value>list2</value>
				<value>list3</value>
			</list>
		</property>
	</bean>
	<!-- parent属性设置你要继承哪个id的配置 -->
	<bean id="p19" class="com.ncu.zte.Person" parent="parent">
		<property name="name" value="我是子"></property>
	</bean>

![](https://s1.ax1x.com/2020/07/03/NLxT7n.md.png)

## IOC之abstract抽象Bean

abstract的bean配置，就是为了让其他的bean做继承使用。而不能实例化

实验19：通过abstract属性创建一个模板bean

abstract="true"表示这个bean，不能被实例化，而只能被继承使用

	<bean id="parent" class="com.ncu.zte.Person" abstract="true">
		<property name="id" value="100" />
		<property name="name" value="我是父亲" />
		<property name="list">
			<list>
				<value>list1</value>s
				<value>list2</value>
				<value>list3</value>
			</list>
		</property>
	</bean>

![](https://s1.ax1x.com/2020/07/03/NLxImj.md.png)

## IOC之组件创建顺序

实验20：bean之间的依赖  depends-on 属性


bean对象

    public class A {
    	public A() {
    		System.out.println("A 被创建了");
    	}
    }
    public class B {
    	public B() {
    		System.out.println("B 被创建了");
    	}
    }
    public class C {
    	public C() {
    		System.out.println("C 被创建了");
    	}
    }

applicationContext.xml配置：

	<!-- 默认情况下。bean对象创建的顺序，是从上到下
			depends-on 可以设定依赖
	 -->

	<bean id="a" class="com.ncu.zte.A" depends-on="b,c"></bean>
	<bean id="b" class="com.ncu.zte.B"></bean>
	<bean id="c" class="com.ncu.zte.C"></bean>


![](https://s1.ax1x.com/2020/07/03/NLxbt0.md.png)


## 基于xml配置文件的自动注入

先创建Person类和Car类

    public class Car {
    	private String carNo;
    	private String name;
    
    public class Person {
    
    	private Car car;
    
    	public Person(Car car) {
    		this.car = car;
    	}



applicationContext.xml配置文件：

    <!-- 
    		autowire 属性设置是否自动查找bean对象并给子对象赋值
    		
    		default 和 no 表示不自动查找并注入（你不赋值，它就null）
    		byName 	是指通过属性名做为id来查找bean对象，并注入
    					1、找到就注入
    					2、找不到就为null
    		byType  是指按属性的类型进行查找并注入
    					1、找到一个就注入
    					2、找到多个就报错
    					3、没有找到就为null
    		constructor 是指按构造器参数进行查找并注入。
    					1、先按照构造器参数类型进行查找并注入
    					2、如果按类型查找到多个，接着按参数名做为id继续查找并注入。
    					3、按id查找不到，就不赋值。
    	 -->
    
    <bean id="p20" class="com.ncu.zte.Person" autowire="constructor">
    <property name="name" value="p20"></property>
    
    </bean>


![](https://s1.ax1x.com/2020/07/03/NLxOpT.md.png)


![](https://s1.ax1x.com/2020/07/03/NLxX1U.md.png)



![](https://s1.ax1x.com/2020/07/03/NLxjcF.md.png)

## IOC之Bean的单例和多例（重点）

实验21：测试bean的作用域，分别创建单实例和多实例的bean★

applicationContext.xml配置文件：

    <!-- 
    		scope 属性设置对象的域
    			singleton			表示单例（默认）
    								1、Spring容器在创建的时候，就会创建Bean对象
    								2、每次调用getBean都返回spring容器中的唯一一个对象
    								
    			prototype			表示多例
    								1、多例在Spring容器被创建的时候，不会跟着一起被创建。
    								2、每次调用getBean都会创建一个新对象返回
    								
    			request				在一次请求中，多次调用getBean方法都是返回同一个实例。
    			
    								getBean("p20"); 底层大概的实现原理
    									
    								Object bean = request.getAttribute("p20");
    								if (bean == null) {
    									bean = new 对象();
    									request.setAttribute("p20",bean);
    								}
    								return bean;
    								
    								
    			session				在一个会话中，多次调用getBean方法都是返回同一个实例。
    			
    								getBean("p20"); 底层大概的实现原理
    									
    								Object bean = session.getAttribute("p20");
    								if (bean == null) {
    									bean = new 对象();
    									session.setAttribute("p20",bean);
    								}
    								return bean;
    	 -->

![](https://s1.ax1x.com/2020/07/03/NLxvX4.md.png)

![](https://s1.ax1x.com/2020/07/03/NLxznJ.md.png)
