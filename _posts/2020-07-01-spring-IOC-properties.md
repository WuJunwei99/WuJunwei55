---
layout: post
title: "Spring 学习笔记（2+）——IOC的赋值和创建Bean"
date: 2020-07-01 20:17:53 +0800
categories: notes
tags: Spring
img: https://s1.ax1x.com/2020/07/02/NqzFiV.png
---
IOC之 P名称空间，IOC之子对象的赋值测试，内部Bean的使用，工厂实例方法创建Bean，工厂实例方法创建Bean，自动注入，Bean的单例和多例

## IOC之 P名称空间

p名称空间，是一种通过简短属性方式调用setXxx方法赋值属性的技术


实验6：通过p名称空间为bean赋值


加入p名称空间：

![](https://s1.ax1x.com/2020/07/02/NLlAMQ.png)

![](https://s1.ax1x.com/2020/07/02/NLlErj.md.png)

applicationContext.xml配置文件：

	<bean id="p6" class="com.ncu.zte.beans.Person"  p:id="6" p:name="p名称空间" p:age="18" p:phone="13600889999"></bean>


测试代码：

	@Test
	public void test6() throws Exception {
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicaionContext.xml");
		
		System.out.println( applicationContext.getBean("p6") );
	}

![](https://s1.ax1x.com/2020/07/02/NLliRS.md.png)

## 测试null值的使用

实验7：测试使用null值

<property name="name" value="null">表示的是name值为“null”字符串，若想设置name为null，应该写：

		<property name="name">
			<!-- null子标签表示赋null值 -->
			<null></null>
		</property>

applicationContext.xml配置文件：

	<bean id="p7" class="com.ncu.zte.beans.Person">
		<property name="id" value="3" />
		<property name="name">
			<!-- null子标签表示赋null值 -->
			<null></null>
		</property>
		<property name="phone" value="15270868888" />
		<property name="age" value="18" />
	</bean>

![](https://s1.ax1x.com/2020/07/02/NLlVqs.md.png)

## IOC之子对象的赋值测试（重点）

实验8：引用其他bean★

创建个新的工程。测试Spring的开发环境。此不重复。请参阅前面，环境搭建。

JavaBean对象

    public class Car {
    	private String carNo;
    	private String name;
    
    public class Person {
    	private Integer id;
    	private String name;
    	private Car car;
    
applicationContext.xml配置文件

ref属性通过设置id值，将指定的bean对象赋值给属性

applicationContext.xml配置文件
    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xmlns:p="http://www.springframework.org/schema/p"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    	
    	<bean id="car" class="com.ncu.zte.Car">
    		<property name="carNo" value="晋k801"></property>
    		<property name="name" value="宝马"></property>
    	</bean>
    	<bean id="p8" class="com.ncu.zte.Person">
    		<property name="id" value="8"></property>
    		<property name="name" value="武俊伟"></property>
    		<property name="car" ref="car"></property>
    	
    	</bean>
    </beans>
    
测试代码：

	@Test
	public void test1() throws Exception {
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
		
		System.out.println( applicationContext.getBean("p8") );
	}

![](https://s1.ax1x.com/2020/07/02/NLlFxg.md.png)

![](https://s1.ax1x.com/2020/07/02/NLlmaq.png)

## IOC之内部Bean的使用

实验9：引用内部bean

applicationContext.xml配置文件：

<!-- 内部bean，内部bean只能作为 赋值使用。不能通过容器获取 -->

applicationContext.xml配置文件：

	<bean id="p9" class="com.ncu.zte.Person">
		<property name="id" value="9"></property>
		<property name="name" value="p9"></property>
		<property name="car">
			<bean name="car02" class="com.ncu.zte.Car">
				<property name="carNo" value="晋k801"></property>
				<property name="name" value="宝马"></property>
			</bean>
		</property>
	
	</bean>

![](https://s1.ax1x.com/2020/07/02/NLleZn.png)

常见错误：内部的Bean不能被外部使用

![](https://s1.ax1x.com/2020/07/02/NLlnI0.md.png)

## IOC之List属性的赋值

实验10：使用list子元素为List类型的属性赋值

list标签表示配置一个集合赋值属性

给Person对象添加Map集合属性：

	<bean id="p10" class="com.ncu.zte.Person">
		<property name="id" value="10"></property>
		<property name="name" value="武俊伟"></property>
		<property name="list">
		<list>
			<value>item1</value>
			<value>item2</value>
			<value>item3</value>
		</list>
		</property>
		<property name="car">
			<bean name="car02" class="com.ncu.zte.Car">
				<property name="carNo" value="晋k802"></property>
				<property name="name" value="宝马"></property>
			</bean>
		</property>
	</bean>

![](https://s1.ax1x.com/2020/07/02/NLlKiV.pngg)

## IOC之Map属性的赋值

实验11：使用map子元素为Map类型的属性赋值

* map标签表示配置一个map集合给属性赋值
* entry是map集合中的每一项

给Person对象添加Map集合属性：

    public class Person {
    
    	private Integer id;
    	private String name;
    	private Car car;
    	private List<Object> list;
    	private Map<String, Object> map;
    
    	public Map<String, Object> getMap() {
    		return map;
    	}
    	public void setMap(Map<String, Object> map) {
    		this.map = map;
    	}

applicationContext.xml配置文件：

	<bean id="p11" class="com.ncu.zte.Person">
		<property name="id" value="10"></property>
		<property name="name" value="武俊伟"></property>
		<property name="map">
			<map>
				<entry key="key1" value="value1"></entry>
				<entry key="key2" value="value2"></entry>
				<entry key="key3" value-ref="car"></entry>
			</map>
		</property>
	</bean>

![](https://s1.ax1x.com/2020/07/02/NLlMGT.md.png)


## IOC之Properties属性的赋值

实验12：使用prop子元素为Properties类型的属性赋值

给Person对象，添加Properties类型的属性

    public class Person {
    
    	private Integer id;
    	private String name;
    	private Car car;
    	private List<Object> list;
    	private Map<String, Object> map;
    	private Properties props;
    
    	public Properties getProps() {
    		return props;
    	}
    	public void setProps(Properties props) {
    		this.props = props;
    	}

pplicationContext.xml配置文件

props标签表示配置Properties类型给属性赋值

	<bean id="p12" class="com.ncu.zte.Person">
		<property name="id" value="12"></property>
		<property name="props">
			<props>
				<prop key="user">root</prop>
				<prop key="password">root</prop>
				<prop key="driver">com.mysql.jdbc.Driver</prop>
				<prop key="url">jdbc:mysql://localhost:3306/test</prop>
			</props>
		</property>
	</bean>

![](https://s1.ax1x.com/2020/07/02/NLlQRU.md.png)

## IOC之Set类型属性赋值

给Person添加set类型的属性

    public class Person {
    	private Integer id;
    	private String name;
    	private Car car;
    	private List<Object> list;
    	private Map<String, Object> map;
    	private Properties props;
    	private Set<Object> set;
    
    	public Set<Object> getSet() {
    		return set;
    	}
    	public void setSet(Set<Object> set) {
    		this.set = set;
    	}

applicationContext.xml配置文件：

set标签配置一个Set集合赋值给属性

value 表示集合中一个字符串值

ref 表示引用一个bean对象

	<bean id="p13" class="com.ncu.zte.Person">
	<property name="set">
		<set>
			<value>value1</value>
			<value>value2</value>
			<ref bean="car"/>
		</set>
	</property>
	</bean>

![](https://s1.ax1x.com/2020/07/02/NLllzF.md.png)

## IOC之util 名称空间

util名称空间，可以定义公共的集合类型数据。供外部和内部引用使用。

实验13：通过util名称空间创建集合类型的bean

添加util名称空间：

![](https://s1.ax1x.com/2020/07/02/NLl8sJ.png)

applicationContext.xml配置文件


使用util名称空间定义一个公共的，可供别人引用的list集合

	<util:list id="list01">
		<value>list1</value>
		<value>list2</value>
		<value>list3</value>
		<value>list4</value>
	</util:list>
	<!-- 把list01的id对象。赋值给list属性 -->
	<bean id="p14" class="com.ncu.zte.Person">
		<property name="id" value="14"></property>
		<property name="list" ref="list01"></property>
	</bean>

![](https://s1.ax1x.com/2020/07/02/NLl3M4.md.png)

## IOC之级联属性赋值

spring中可以使用级联属性方式赋值，但赋值前这个子对象一定要先有值

	<bean id="car" class="com.atguigu.pojo.Car">
		<property name="carNo" value="京B531212" />
		<property name="name" value="冰利"/>
	</bean>

	<bean id="p15" class="com.ncu.zte.Person">
		<property name="id" value="14"></property>
		<property name="car" ref="car" />
		<property name="car.carNo" value="京A123412">
		</property>
	</bean>

![](https://s1.ax1x.com/2020/07/02/NLlGL9.md.png)

常见错误：

级联属性一定要先注入对象。再注入对象的属性

![](https://s1.ax1x.com/2020/07/02/NLlYZR.md.png)

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
