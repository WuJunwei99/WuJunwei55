---
layout: post
title: "mybatis 学习笔记（6）——mybatis缓存及逆向工程"
date: 2020-06-30 21:45:24 +0800
categories: notes
tags: mybatis
img: https://s1.ax1x.com/2020/06/29/Nfjd8f.png
---
mybatis缓存，缓存的使用顺序说明，mybatis 逆向工程

#

缓存：

1. 缓存是指把经常需要读取的数据保存到一个高速的缓冲区中，这个行为叫缓存。
2. 缓存也可以是指被保存到高速缓冲区中的数据，也叫缓存。

一级缓存：	是指把数据保存到SqlSession中

二级缓存：	是指把数据保存到SqlSessionFactory中

## mybatis的一级缓存

mybatis每次查询的时候，都会执行以下几个步骤的工作：

1. 到一级缓存SqlSession中查询是否有搜索的数据，有就返回
2. 发送sql语句到数据库中去查询
3. 把查询到的结果保存到SqlSession一级缓存中

![](https://s1.ax1x.com/2020/07/02/NbCzHs.md.png)


#### User对象

    public class User {
    
    	private Integer id;
    	private String lastName;
    	private Integer sex;

#### UserMapper接口

    public interface UserMapper {
    	public User queryUserById(Integer id);
    }

#### UserMapper.xml配置

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.ncu.zte.mybatis6_29.mapper.UserMapper">
    
    <select id="queryUserById" resultType="com.ncu.zte.mybatis6_29.User">
    	select id,last_name lsatName,sex from t_user where id = #{id}
    </select>
    
    </mapper>

#### 测试代码

	@Test
	public void testQueryUserById() {
		SqlSession session = sqlSessionFactory.openSession();
		try{
			UserMapper mapper = session.getMapper(UserMapper.class);
			System.out.println(mapper.queryUserById(2));
			System.out.println(mapper.queryUserById(2));
		}finally{
			session.close();
		}
	}

![](https://s1.ax1x.com/2020/07/02/NbCL38.md.png)

### 一级缓存的管理

缓存失效的四种情况：

* 不在同一个SqlSession对象中

    	public void testCacheFail1() throws IOException {
    //		1.不在同一个SqlSession对象中
    		testQueryUserById();
    		
    		testQueryUserById();
    		
    		testQueryUserById();
    		
    	}

![](https://s1.ax1x.com/2020/07/02/NbCXjg.md.png)

* 执行语句的参数不同。缓存中也不存在数据。

	@Test
	public void testCacheFail2() throws IOException {

		// 2.执行语句的参数不同。缓存中也不存在数据。

		SqlSession session = sqlSessionFactory.openSession();
		try {
			UserMapper mapper = session.getMapper(UserMapper.class);

			System.out.println( mapper.queryUserById(1) );
			System.out.println( mapper.queryUserById(5) );

		} finally {
			session.close();
		}

	}

![](https://s1.ax1x.com/2020/07/02/NbCH4P.md.png)

* 执行增，删，改，语句，会清空掉缓存

	@Test
	public void testCacheFail3() throws IOException {

		SqlSession session = sqlSessionFactory.openSession();
		try {
			UserMapper mapper = session.getMapper(UserMapper.class);

			System.out.println(mapper.queryUserById(1));

			3.执行增，删，改，语句，会清空掉缓存
			mapper.updateUser(new User(2,"cc",1));
			

			System.out.println(mapper.queryUserById(1));

		} finally {
			session.close();
		}

	}

![](https://s1.ax1x.com/2020/07/02/NbCq9f.md.png)

* 手动清空缓存数据
    
    @Test
    	public void testCacheFail4() throws IOException {
    
    		SqlSession session = sqlSessionFactory.openSession();
    		try {
    			UserMapper mapper = session.getMapper(UserMapper.class);
    
    			System.out.println(mapper.queryUserById(1));
    
    			// 4.手动清空缓存数据
    			session.clearCache();
    
    			System.out.println(mapper.queryUserById(1));
    
    		} finally {
    			session.close();
    		}
    
    	}

![](https://s1.ax1x.com/2020/07/02/NbCvuQ.md.png)

## mybatis的二级缓存

二级缓存的图解示意

![](https://s1.ax1x.com/2020/07/02/NbPbZ9.md.png)

二级缓存的使用：

myBatis的二级缓存默认是不开启的。

1、我们需要在mybatis的核心配置文件中配置setting选项 

		<!-- 开启二级缓存 -->
		<setting name="cacheEnabled" value="true"/>

2、在Mapper的配置文件中加入cache标签。
    <cache></cache>



3、并且需要被二级缓存的对象必须要实现java的序列化接口。

![](https://s1.ax1x.com/2020/07/02/NbCxBj.png)

### 二级缓存的演示

	@Test
	public void testCacheFail1() throws IOException {
		1.不在同一个SqlSession对象中
		testQueryUserById();
		
		testQueryUserById();
		
		testQueryUserById();
		
	}

![](https://s1.ax1x.com/2020/07/02/NbPpEn.md.png)

### useCache="false"的演示和说明

在select标签中，useCache属性表示是否使用二缓存。默认是true，表示使用二级缓存（每次查询完之后。也会把这个查询的数据，放到二级缓存中）

    <select id="queryUserById" resultType="com.ncu.zte.mybatis6_29.User"  useCache="true">
    		select id,last_name lastName,sex from t_user where id = #{id}
    	</select>

### flushCache="false"的演示和说明

在insert、delete、update标签中，都有flushCache属性，这个属性决定着执行完增，删，改的语句之后要不要清空二级缓存。
默认值都是true。

	<update id="updateUser" parameterType="com.ncu.zte.mybatis6_29.User" flushCache="false">
		update t_user set last_name = #{lastName},sex=#{sex} where id = #{id}
	</update>

### <cache\></cache\>标签的介绍和说明

MyBatis 包含一个非常强大的查询缓存特性,它可以非常方便地配置和定制。MyBatis 3 中的缓存实现的很多改进都已经实现了,使得它更加强大而且易于配置。

默认情况下是没有开启缓存的,除了局部的 session 缓存,可以增强变现而且处理循环 依赖也是必须的。要开启二级缓存,你需要在你的 SQL 映射文件中添加一行:

<cache/>

默认的<cache/>标签的作用:

1. 映射语句文件中的所有 select 语句将会被缓存。
2. 射语句文件中的所有 insert,update 和 delete 语句会刷新缓存。
3. 缓存会使用 Least Recently Used(LRU,最近最少使用的)算法来收回。
4. 根据时间表(比如 no Flush Interval,没有刷新间隔), 缓存不会以任何时间顺序 来刷新。
5. 缓存会存储列表集合或对象(无论查询方法返回什么)的 1024 个引用。
6. 缓存会被视为是 read/write(可读/可写)的缓存,意味着对象检索不是共享的,而 且可以安全地被调用者修改,而不干扰其他调用者或线程所做的潜在修改。

所有的这些属性都可以通过缓存元素的属性来修改。比如:

<cache
  eviction="FIFO"
  flushInterval="60000"
  size="512"
  readOnly="true"/>

这个更高级的配置创建了一个 FIFO 缓存,并每隔 60 秒刷新,存数结果对象或列表的 512 个引用,而且返回的对象被认为是只读的,因此在不同线程中的调用者之间修改它们会 导致冲突。

* eviction可用的收回策略有:
	* LRU – 最近最少使用的:移除最长时间不被使用的对象。
	* FIFO – 先进先出:按对象进入缓存的顺序来移除它们。
	* SOFT – 软引用:移除基于垃圾回收器状态和软引用规则的对象。
	* WEAK – 弱引用:更积极地移除基于垃圾收集器状态和弱引用规则的对象。

* flushInterval(刷新间隔)可以被设置为任意的正整数,而且它们代表一个合理的毫秒 形式的时间段。默认情况是不设置,也就是没有刷新间隔,缓存仅仅调用语句时刷新。
* size(引用数目)可以被设置为任意正整数,要记住你缓存的对象数目和你运行环境的 可用内存资源数目。默认值是 1024。
* readOnly(只读)属性可以被设置为 true 或 false。只读的缓存会给所有调用者返回缓 存对象的相同实例。因此这些对象不能被修改。这提供了很重要的性能优势。可读写的缓存 会返回缓存对象的拷贝(通过序列化) 。这会慢一些,但是安全,因此默认是 false。

### 使用自定义缓存

除了这些自定义缓存的方式, 你也可以通过实现你自己的缓存或为其他第三方缓存方案 创建适配器来完全覆盖缓存行为。

<cache type="com.domain.something.MyCustomCache"/>

这个示 例展 示了 如何 使用 一个 自定义 的缓 存实 现。type 属 性指 定的 类必 须实现 org.mybatis.cache.Cache 接口。这个接口是 MyBatis 框架中很多复杂的接口之一,但是简单 给定它做什么就行。

    public interface Cache {
      String getId();
      int getSize();
      void putObject(Object key, Object value);
      Object getObject(Object key);
      boolean hasKey(Object key);
      Object removeObject(Object key);
      void clear();
    }

右键创建cache接口的实现类，再按Ctrl+T查看cache类的继承结构，选择PerpetualCache，复制相关内容到自己的实现类，进行相关修改。

![](https://s1.ax1x.com/2020/07/02/NbPyrQ.md.png)

![](https://s1.ax1x.com/2020/07/02/NbideJ.md.png)

## 缓存的使用顺序说明

1. 当我们执行一个查询语句的时候。mybatis会先去二级缓存中查询数据。如果二级缓存中没有。就到一级缓存中查找。
2. 如果二级缓存和一级缓存都没有。就发sql语句到数据库中去查询。
3. 查询出来之后马上把数据保存到一级缓存中。
4. 当SqlSession关闭的时候，会把一级缓存中的数据保存到二级缓存中。

## mybatis 逆向工程

MyBatis逆向工程，简称MBG。是一个专门为MyBatis框架使用者定制的代码生成器。可以快速的根据表生成对应的映射文件，接口，以及Bean类对象。

在Mybatis中，有一个可以自动对单表生成的增，删，改，查代码的插件。

叫 mybatis-generator-core-1.3.2。

它可以帮我们对比数据库表之后，生成大量的这个基础代码。
这些基础代码有：

1. 数据库表对应的javaBean对象
2. 这些javaBean对象对应的Mapper接口
3. 这些Mapper接口对应的配置文件
    
   <!-- 去掉全部的注释 -->
    <commentGenerator>
    <property name="suppressAllComments" value="true" />
    </commentGenerator>

#### 准备数据库表
    
    create database mbg;
    
    use mbg;
    
    create table t_user(
    	`id` int primary key auto_increment,
    	`username` varchar(30) not null unique,
    	`password` varchar(40) not null,
    	`email` varchar(50)
    );
    
    insert into t_user(`username`,`password`,`email`) values('admin','admin','admin@atguigu.com');
    insert into t_user(`username`,`password`,`email`) values('wzg168','123456','admin@atguigu.com');
    insert into t_user(`username`,`password`,`email`) values('admin168','123456','admin@atguigu.com');
    insert into t_user(`username`,`password`,`email`) values('lisi','123456','admin@atguigu.com');
    insert into t_user(`username`,`password`,`email`) values('wangwu','123456','admin@atguigu.com');
    
    create table t_book(
    	`id` int primary key auto_increment,
    	`name` varchar(50),
    	`author` varchar(50),
    	`price`	decimal(11,2),
    	`sales`	int,
    	`stock` int
    );
    
    
    ## 插入初始化测试数据
    insert into t_book(`id` , `name` , `author` , `price` , `sales` , `stock` ) 
    values(null , 'java从入门到放弃' , '国哥' , 80 , 9999 , 9);
    
    insert into t_book(`id` , `name` , `author` , `price` , `sales` , `stock` ) 
    values(null , '数据结构与算法' , '严敏君' , 78.5 , 6 , 13);
    
    insert into t_book(`id` , `name` , `author` , `price` , `sales` , `stock` ) 
    values(null , '怎样拐跑别人的媳妇' , '龙伍' , 68, 99999 , 52);
    
    insert into t_book(`id` , `name` , `author` , `price` , `sales` , `stock` ) 
    values(null , '木虚肉盖饭' , '小胖' , 16, 1000 , 50);
    
    insert into t_book(`id` , `name` , `author` , `price` , `sales` , `stock` ) 
    values(null , 'C++编程思想' , '刚哥' , 45.5 , 14 , 95);
    
    insert into t_book(`id` , `name` , `author` , `price` , `sales` , `stock` ) 
    values(null , '蛋炒饭' , '周星星' , 9.9, 12 , 53);
     
    insert into t_book(`id` , `name` , `author` , `price` , `sales` , `stock` ) 
    values(null , '赌神' , '龙伍' , 66.5, 125 , 535);
    
    select * from t_user;
    select * from t_book;
    


#### 导入jar包

新建pom.xml

    <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
      <groupId>com.ncu.mybatis</groupId>
      <artifactId>mybatis6_29</artifactId>
      <version>0.0.1-SNAPSHOT</version>
    <dependencies>
    
      <dependency>
      	<groupId>mysql</groupId>
      	<artifactId>mysql-connector-java</artifactId>
      	<version>5.1.26</version>
      </dependency>
      <dependency>
      	<groupId>log4j</groupId>
      	<artifactId>log4j</artifactId>
      	<version>1.2.12</version>
      </dependency>
      <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.5</version>
    </dependency>
    <dependency>
      <groupId>org.mybatis.generator</groupId>
      <artifactId>mybatis-generator-core</artifactId>
      <version>1.4.0</version>
    </dependency>
    </dependencies>
    </project>
    
#### 准备配置文件

mbg.xml 逆向工程的配置文件，在配置时要修改目录配置等：

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE generatorConfiguration
      PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
      "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
    
    <generatorConfiguration>
    	<!-- 
    		targetRuntime 表示你要生成的版本
    			MyBatis3				豪华版本
    			MyBatis3Simple			CRUD标准版
    	 -->
      <context id="DB2Tables" targetRuntime="MyBatis3Simple">
    
      	<!-- 去掉全部的注释 -->
    	<commentGenerator>
    <property name="suppressAllComments" value="true" />
    </commentGenerator>
    
      
    <jdbcConnection driverClass="com.mysql.jdbc.Driver"
    connectionURL="jdbc:mysql://localhost:3306/mbg"
    userId="root"
    password="root">
    </jdbcConnection>
    
    
    
    <javaTypeResolver >
      <property name="forceBigDecimals" value="false" />
    </javaTypeResolver>
    
    	<!-- javaModelGenerator配置生成模型JavaBean
    			targetPackage生成的javaBean的包名
    			targetProject生成之后在哪个工程目录下
    	 -->
    <javaModelGenerator targetPackage="com.atguigu.pojo" targetProject=".\src\main\java">
      <property name="enableSubPackages" value="true" />
      <property name="trimStrings" value="true" />
    </javaModelGenerator>
    	
    	<!-- 
    		sqlMapGenerator生成sql的mapper.xml配置文件
    			targetPackage生成mapper.xml配置文件放的包名
    	 -->
    <sqlMapGenerator targetPackage="com.atguigu.mapper"  targetProject=".\src\main\java">
      <property name="enableSubPackages" value="true" />
    </sqlMapGenerator>
    
    	<!-- 
    		javaClientGenerator配置生成的Mapper接口
    			
    	 -->
    <javaClientGenerator type="XMLMAPPER" targetPackage="com.atguigu.mapper"  targetProject=".\src\main\java">
      <property name="enableSubPackages" value="true" />
    </javaClientGenerator>
    
    	<!-- 一个table标签，表示一个表 -->
    <table tableName="t_user" domainObjectName="User" ></table>
    <table tableName="t_book" domainObjectName="Book" ></table>
    
    
      </context>
    </generatorConfiguration>
    
#### log4j.properties

    # Global logging configuration
    log4j.rootLogger=DEBUG, stdout
    # Console output...
    log4j.appender.stdout=org.apache.log4j.ConsoleAppender
    log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
    log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n

#### mybatis-config.xml
    
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" 
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
    <properties resource="jdbc.properties">
    
    </properties>
    
      <settings>
      <setting name="mapUnderscoreToCamelCase" value="true"/>
      </settings>
      
      	<typeAliases>
    		<package name="com.ncu.zte.mybatis6_29"/>
    	</typeAliases>
      
      <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <!-- 配置数据源
      		需要配置数据库的四个连接属性
       -->
      <dataSource type="POOLED">
    <property name="driver" value="${driver}"/>
    <property name="url" value="${url}"/>
    <property name="username" value="${username}"/>
    <property name="password" value="${password}"/>
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
      	 <package name="com.atguigu.mapper" />
      </mappers>
    
    </configuration>

##### 数据库配置jdbc.properties

username=root
password=root
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis

#### 生成mbyatis的代码

注意导入包时选择正确

    import java.io.File;
    import java.util.ArrayList;
    import java.util.List;
    
    import org.mybatis.generator.api.MyBatisGenerator;
    import org.mybatis.generator.config.Configuration;
    import org.mybatis.generator.config.xml.ConfigurationParser;
    import org.mybatis.generator.internal.DefaultShellCallback;
    
    public class Runner {
    
    	public static void main(String[] args) throws Exception {
    		List<String> warnings = new ArrayList<String>();
    		boolean overwrite = true;
    		File configFile = new File("mbg.xml");
    		ConfigurationParser cp = new ConfigurationParser(warnings);
    		Configuration config = cp.parseConfiguration(configFile);
    		DefaultShellCallback callback = new DefaultShellCallback(overwrite);
    		MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config,
    				callback, warnings);
    		myBatisGenerator.generate(null);
    	}
    
    }

生成的项目结构如图：

![](https://s1.ax1x.com/2020/07/02/NbihTA.png)
 
#### 测试

测试插入之前，要在Book类中生成有参构造，无参构造方法。

	@Test
	public void testInsert() {
		SqlSession session = sqlSessionFactory.openSession();
		try{
			BookMapper mapper = session.getMapper(BookMapper.class);
			mapper.insert(new Book(null,"wjw","zzp",new BigDecimal("15000.48"),22,22));
			session.commit();
		}finally{
			session.close();
		}
	}

	@Test
	public void testSelectAll() {
		SqlSession session = sqlSessionFactory.openSession();
		try{
			BookMapper mapper = session.getMapper(BookMapper.class);
			mapper.selectAll().forEach(System.out::println);
		}finally{
			session.close();
		}
		fail("Not yet implemented");
	}

![](https://s1.ax1x.com/2020/07/02/NbFP6U.png)