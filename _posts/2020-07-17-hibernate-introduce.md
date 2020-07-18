---
layout: post
title: "hibernate的介绍以及使用hibernate操作数据库"
date: 2020-07-17 13:22:38 +0800
categories: notes
tags: hibernate
img: https://s1.ax1x.com/2020/07/17/UsU08H.png
---
初识Hibernate;ORM;配置文件;使用Hibernate 操作数据库


## 初识Hibernate

#### JDBC的缺点

1).编写代码的时候过于繁琐：try和catch比较多；pstmt的setXX方法；方法参数冗余的getXX方法

2).没有做数据缓存 

3).不是面向对象编程 

4).sql语句固定，可移植性差


### Hibernate简介

#### Hibernate作者——Gavin King

* Hibernate创始人
* 《 Hibernate in action 》作者
* EJB 3.0的Entity bean specification的实际领导人（sun任命的领导人是Linda DeMichiel）
* 参加了XDoclet和Middlegen的开发
* 2003年9月加入JBoss，全职进行Hibernate开发

#### Hibernate

* 一个开放源代码的对象关系映射框架
* 对JDBC进行了非常轻量级的对象封装
* 将JavaBean对象和数据库的表建立对应关系

#### Hibernate优势

* Hibernate 是一个优秀的Java 持久化层解决方案
* 是当今主流的对象—关系映射工具
* Hibernate 简化了JDBC 繁琐的编码
* Hibernate 将数据库的连接信息都存放在配置文件中
* hibernate的缓存很牛的，一级缓存，二级缓存，查询缓存
* 跨平台性强
* 使用场合多应用于企业内部的系统

#### Hibernate缺点

* 效率低
* 表中的数据如果在千万级别，则hibernate不适合
* 如果表与表之间的关系特别复杂，则hibernate也不适合

## ORM

学习Hibernate，首先要先了解ORM(Object/Relation Mapping),对象关系映射，主要思想是：将关系数据库中表中的记录映射成为对象，以对象的形式展现，程序员可以把对数据库的操作转化为对对象的操作

ORM 采用元数据来描述对象-关系映射细节,元数据通常采用XML格式,并且存放在专门的对象-关系映射文件中

![](https://s1.ax1x.com/2020/07/17/UsU6qP.jpg)

#### 持久化

将程序中数据在瞬时状态和持久状态间转换的机制

![](https://s1.ax1x.com/2020/07/17/UsU2a8.md.png)

#### 持久化层

JDBC 就是一种持久化机制

将程序数据直接保存成文本文件也是持久化机制的一种实现

在分层结构中，DAO 层（数据访问层）也被称为持久化层


#### 持久化完成的操作

将对象保存到关系型数据库中

将关系型数据库中的数据读取出来

以对象的形式封装

![](https://s1.ax1x.com/2020/07/17/UsUB2d.png)

![](https://s1.ax1x.com/2020/07/17/UsUgVf.md.png)

## 配置文件

#### hibernate.cfg.xml文件

	<hibernate-configuration>
		<session-factory name="foo">
	 
			<property name="hibernate.dialect"><![CDATA[org.hibernate.dialect.MySQLDialect]]></property>
			<property name="hibernate.connection.driver_class"><![CDATA[com.mysql.jdbc.Driver]]></property>
			<property name="hibernate.connection.url"><![CDATA[jdbc:mysql:///hibernate1]]></property>
			<property name="hibernate.connection.username"><![CDATA[root]]></property>
			<property name="hibernate.connection.password"><![CDATA[root]]></property>
			<!-- 是否显示sql语句 -->
			<property name="show_sql">true</property>
			<!--是否格式化sql语句-->
			<property name="format_sql">true</property>
			<!--生成表的策略,通常是update-->
			<property name="hbm2ddl.auto">update</property>
			<!--添加映射文件-->
			<mapping resource="cn/xxx/User.hbm.xml"/>
	 
		</session-factory>
	</hibernate-configuration>
		</property>

#### User.hbm.xml文件
	
	<hibernate-mapping>
	    <class name="cn.jbit.houserent.bean.User" table="users" schema="jbit">
			<!--id 主键,column是数据表中对应的列名 -->
	        <id name="id" type="java.lang.Long">
	            <column name="id" />
	            <generator class="native" /><!-- 主键的增长策略 -->
	        </id>
			<!--type:该属性对应的类型,该类型可分为Hibernate类型与java类型-->
	        <property name="password" type="java.lang.String" lazy="false">
	            <column name="password" length="50" not-null="true" />
	        </property>
	        <property name="telephone" type="java.lang.String" lazy="false">
	            <column name="telephone" length="12" />
	        </property>
	        <property name="username" type="java.lang.String" lazy="false">
	            <column name="username" length="50" />
	        </property>
	    </class>
	</hibernate-mapping>

#### 常见主键生成策略


主键生成策略：Hibernate中,<id>标签下的可选<generator>子元素是一个Java类的名字，用来为该持久化类的实例生成惟一标示，所有的生成器都实现net.sf.hibernate.id.IdentifierGenerator接口 

* assigned ：主键由外部程序负责生成，无需Hibernate参与。 
* identity ：采用数据库提供的主键生成机制。如DB2、SQL Server、MySQL中主键生成机制。 
* sequence：采用数据库提供的sequence 机制生成主键。如Oralce 中的Sequence。 
* native ：由Hibernate根据底层数据库自行判断采用identity、hilo、sequence其中一种作为主键生成方式。 

### 类型对应表

在hibernate内部，有一张类型对应表，这张表中有如下的映射关系：

![](https://s1.ax1x.com/2020/07/17/UsUNVK.png)

## 使用Hibernate 操作数据库

### 创建实体类和实体映射文件

定义实体类（也称持久化类），实现java.io.Serializable 接口，添加默认构造方法

配置映射文件（*.hbm.xml）

向hibernate.cfg.xml文件中配置映射文件

#### 定义实体类

	public class User implements java.io.Serializable {
	    //字段
	    private Integer id;
	    private String name;
	    private String password;
	    private String telephone;
	    private String username;
	    private String isadmin;
	    public User(){
	    }
	    //省略getter&setter 方法
	}



#### hibernate.cfg.xml的加载

	<hibernate-mapping>
	    <class name="com.hibernate.bean.UserBean" table="users">
	        <id name="id" type="java.lang.Integer">
	            <column name="id" />
	            <generator class="sequence" >
	                <param name="sequence">SEQ_ID</param>
	            </generator>
	        </id>
	        <property name="name" type="java.lang.String">
	            <column name="name" length="50" />
	        </property>
	        <property name="password" type="java.lang.String">
	            <column name="password" length="50" />
	        </property>
	        <!--省略其他配置-->
	    </class>
	</hibernate-mapping>

#### 配置映射文件

	<session-factory>
	    <!--省略其他配置-->
	    <!--注意配置文件名必须包含其相对于classpath 的全路径-->
	    <mapping resource="com/hibernate/bean/User.hbm.xml" />
	</session-factory>


### 使用Hibernate 操作数据库

#### hibernate.cfg.xml的加载

	public class HibernateUtils {
	 
		private static SessionFactory factory;
		static{
			factory = new Configuration()//
			//这种方式需要注意,hibernate的配置文件名必须是hibernate.hbm.xml,
			//且必须放classpath目录下
					.configure()//
					.buildSessionFactory();
			//new Configuration().configure(xxx)可以指定配置文件的路径，配置文件可以随意放
		}
		public static SessionFactory getSessionFactory(){
			return factory;
		}
		public static Session openSession(){
			return factory.openSession();
		}
	 
	}

#### SessionFactory接口

1. hibernate中的配置文件、映射文件、持久化类的信息都在sessionFactory中
2. sessionFactory中存放的信息都是共享的信息
3. sessionFactory是线程安全的
4. 一个hibernate框架sessionFactory只有一个
5. sessionFactory是一个重量级别的类,很消耗资源

#### Session接口

Session是应用程序与数据库之间交互操作的一个单线程对象，是Hibernate运作的中心，所有持久化对象必须在session的管理下才可以进行持久化操作。此对象的生命周期很短。Session对象有一个一级缓存，显式执行flush之前，所有的持久层操作的数据都缓存在session对象处。相当于JDBC中的 Connection。

1. 得到了一个session，相当于打开了一次数据库的连接
2. 在hibernate中，对数据的crud操作都是由session来完成的

#### 生成数据表

	public class SessionfatoryTest {
	 
		@Before
		public void init(){
			SessionFactory factory =  HibernateUtils.getSessionFactory();
		}
		@Test
		public void test(){
			
		}
	}

sessionFactory初始化后,对应的数据表也就生成了


#### Transaction(事务)

代表一次原子操作，它具有数据库事务的概念。所有持久层都应该在事务管理下进行，且hibernate中的事务默认不是自动提交的

![](https://s1.ax1x.com/2020/07/17/UsUwPe.png)

设置了connection的setAutoCommit为false， 只有产生了连接，才能进行事务的操作。所以只有有了session以后，才能有transaction
	
	public class SessionfatoryTest {
		SessionFactory factory;
		Session session ;
		Transaction transaction;
		@Before
		public void init(){
			factory =  HibernateUtils.getSessionFactory();
			session = factory.openSession();
			transaction = session.beginTransaction();
		}
		@Test
		public void test(){
			User user = new User();
			user.setAge(10);
			user.setName("A");
			session.save(user);
		}
		
		@After
		public void destory(){
			transaction.commit();
			session.close();
			
		}
	}

## 执行流程

### Hibernate的执行流程

![](https://s1.ax1x.com/2020/07/17/UsUsKI.png)

（1）读取并解析配置文件

Configuration conf = newConfiguration().configure();

（2）读取并解析映射信息，创建SessionFactory

SessionFactory sf = conf.buildSessionFactory();

（3）打开Session

Session session = sf.openSession();

（4）开始一个事务（增删改操作必须，查询操作可选）

Transaction tx = session.beginTransaction();

（5）数据库操作

session.save(user);//或其它操作

（6）提交事务（回滚事务）

tx.commit();(tx.rollback();)

（7）关闭session

session.close();

### 内部执行原理图

![](https://s1.ax1x.com/2020/07/17/UsUyrt.md.png)