---
layout: post
title: "mybatis 学习笔记（3）——mybatis的核心配置"
date: 2020-06-29 18:22:15 +0800
categories: notes
tags: mybatis
img: https://s1.ax1x.com/2020/06/29/Nfjd8f.png
---
mybatis的核心配置：properties，settings，typeAliases，environments，databaseIdProvider，Mappers


## mybatis的核心配置之properties

它可以用来定义键值对的属性 

resource属性引入属性配置文件

当properties标签内定义了键值对，而resource引入的属性配置文件，也有相同的key的时候。以外部的属性配置文件值为准

外部引入的属性配置文件会覆盖properties标签内定义的值

    	<properties resource="jdbc.properties">
    <!-- 		<property name="username" value="root"/> -->
    <!-- 		<property name="password" value="root"/> -->
    		<property name="url" value="jdbc:mysql://localhost:3306/test"/>
    <!-- 		<property name="driver" value="com.mysql.jdbc.Driver"/> -->
    	</properties>

在jdbc.properties中进行配置：

    username=root
    password=root
    driver=com.mysql.jdbc.Driver
    url=jdbc:mysql://localhost:3306/mybatis

在mybatis-config.xml中配置：

    <configuration>
    <properties resource="jdbc.properties">
    
    </properties>
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
    <mapper resource="com/ncu/zte/mybatis6_29/UserMapper.xml"/>
      </mappers>
    </configuration>

## mybatis的核心配置之settings

这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。下表描述了设置中各项的意图、默认值等。

### 所有mybatis的settings设置选项


![](https://s1.ax1x.com/2020/06/30/NIEpEq.md.png)

![](https://s1.ax1x.com/2020/06/30/NIE9U0.md.png)

![](https://s1.ax1x.com/2020/06/30/NIAzbn.md.png)


例如：

**mapUnderscoreToCamelCase**是否开启自动驼峰命名规则（camel case）映射，即从经典数据库列名 A_COLUMN 到经典 Java 属性名 aColumn 的类似映射。

    <select id="queryUsers" resultType="com.ncu.zte.mybatis6_29.User">
    	select id,last_name lsatName,sex from t_user 
    </select>

可以删除别名，并在mybatis-config.xml中对settings进行配置：
    
      <settings>
      <setting name="mapUnderscoreToCamelCase" value="true"/>
      </settings>

运行结果仍然可以看到last_name:

![](https://s1.ax1x.com/2020/06/30/NIAxDs.png)

## mybatis的核心配置之typeAliases

类型别名是为 Java 类型设置一个短的名字。它只和 XML 配置有关，存在的意义仅在于用来减少类完全限定名的冗余。

	<typeAliases>
		<!-- 给一个具体的类型配置别名
				type 具体的类型
				alias 别名
		<typeAlias type="com.atguigu.pojo.User" alias="user"/> -->
		<!-- 
			package标签设置通过包名来扫描，所有的类。自动的配置上别名
				默认的别名，是类名，而且首字母小写
		 -->
		<package name="com.atguigu.pojo"/>
		<package name="com.atguigu.domain"/>
	</typeAliases>

### 系统提示的预定义别名

在mybatis-config.xml进行配置，给一个具体的类型配置别名；type 具体的类型；alias 别名

    	<typeAliases>
    		<typeAlias type="com.atguigu.pojo.User" alias="user"/>
    	</typeAliases>

因为配置的javabean不止一个，所以我们可以将typeAliases的内容修改如下，只写出包名：
		<!-- 
			package标签设置通过包名来扫描，所有的类。自动的配置上别名
				默认的别名，是类名，而且首字母小写
		 -->
		<package name="com.atguigu.pojo"/>
		<package name="com.atguigu.domain"/>
    	</typeAliases>

而别名就是类名的小写字母，比如user

    <select id="queryUsers"  resultType="user">
    	select id,last_name,sex from t_user 
    </select>

若引用的不同包下有相同的类，则修改其中一个的别名：

![](https://s1.ax1x.com/2020/06/30/NIAXvQ.png)


### 系统提示的预定义别名

已经为许多常见的 Java 类型内建了相应的类型别名。它们都是大小写不敏感的，需要注意的是由基本类型名称重复导致的特殊处理。

![](https://s1.ax1x.com/2020/06/30/NIAvuj.png)

## mybatis的核心配置之typeHandlers

无论是 MyBatis 在预处理语句（PreparedStatement）中设置一个参数时，还是从结果集中取出一个值时， 都会用类型处理器将获取的值以合适的方式转换成 Java 类型。下表描述了一些默认的类型处理器。

![](https://s1.ax1x.com/2020/06/30/NIEC5V.md.png)

## mybatis的核心配置之environments

MyBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中， 现实情况下有多种理由需要这么做。例如，开发、测试和生产环境需要有不同的配置；或者共享相同 Schema 的多个生产数据库， 想使用相同的 SQL 映射。许多类似的用例。

不过要记住：尽管可以配置多个环境，每个 SqlSessionFactory 实例只能选择其一。

所以，如果你想连接两个数据库，就需要创建两个 SqlSessionFactory 实例，每个数据库对应一个。而如果是三个数据库，就需要三个实例。

### 事务管理器（transactionManager）

在 MyBatis 中有两种类型的事务管理器（也就是 type=”[JDBC|MANAGED]”）：

JDBC – 这个配置就是直接使用了 JDBC 的提交和回滚设置，它依赖于从数据源得到的连接来管理事务范围。

MANAGED – 这个配置几乎没做什么。它从来不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）。 默认情况下它会关闭连接，然而一些容器并不希望这样，因此需要将 closeConnection 属性设置为 false 来阻止它默认的关闭行为。

### 数据源（dataSource）

dataSource 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象的资源。

许多 MyBatis 的应用程序将会按示例中的例子来配置数据源。然而它并不是必须的。要知道为了方便使用延迟加载，数据源才是必须的。
有三种内建的数据源类型（也就是 type=”[UNPOOLED|POOLED|JNDI]”）：


**UNPOOLED**– 这个数据源的实现只是每次被请求时打开和关闭连接。虽然一点慢，它对在及时可用连接方面没有性能要求的简单应用程序是一个很好的选择。 不同的数据库在这方面表现也是不一样的，所以对某些数据库来说使用连接池并不重要，这个配置也是理想的。UNPOOLED 类型的数据源仅仅需要配置以下 5 种属性：

* driver – 这是 JDBC 驱动的 Java 类的完全限定名（并不是JDBC驱动中可能包含的数据源类）。
* url – 这是数据库的 JDBC URL 地址。
* username – 登录数据库的用户名。
* password – 登录数据库的密码。
* defaultTransactionIsolationLevel – 默认的连接事务隔离级别。

作为可选项，你也可以传递属性给数据库驱动。要这样做，属性的前缀为“driver.”，例如：

driver.encoding=UTF8

这将通过DriverManager.getConnection(url,driverProperties)方法传递值为 UTF8 的 encoding 属性给数据库驱动。

**POOLED**– 这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。 这是一种使得并发 Web 应用快速响应请求的流行处理方式。

除了上述提到 UNPOOLED 下的属性外，会有更多属性用来配置 POOLED 的数据源：

* poolMaximumActiveConnections – 在任意时间可以存在的活动（也就是正在使用）连接数量，默认值：10
* poolMaximumIdleConnections – 任意时间可能存在的空闲连接数。
* poolMaximumCheckoutTime – 在被强制返回之前，池中连接被检出（checked out）时间，默认值：20000 毫秒（即 20 秒）
* poolTimeToWait – 这是一个底层设置，如果获取连接花费的相当长的时间，它会给连接池打印状态日志并重新尝试获取一个连接（避免在误配置的情况下一直安静的失败），默认值：20000 毫秒（即 20 秒）。
* poolPingQuery – 发送到数据库的侦测查询，用来检验连接是否处在正常工作秩序中并准备接受请求。默认是“NO PING QUERY SET”，这会导致多数数据库驱动失败时带有一个恰当的错误消息。
* poolPingEnabled – 是否启用侦测查询。若开启，也必须使用一个可执行的 SQL 语句设置 poolPingQuery 属性（最好是一个非常快的 SQL），默认值：false。
* poolPingConnectionsNotUsedFor – 配置 poolPingQuery 的使用频度。这可以被设置成匹配具体的数据库连接超时时间，来避免不必要的侦测，默认值：0（即所有连接每一时刻都被侦测 — 当然仅当poolPingEnabled 为 true 时适用）。

**JNDI**– 这个数据源的实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的引用。这种数据源配置只需要两个属性：

* initial_context – 这个属性用来在 InitialContext 中寻找上下文（即，initialContext.lookup(initial_context)）。这是个可选属性，如果忽略，那么 data_source 属性将会直接从 InitialContext 中寻找。
* data_source – 这是引用数据源实例位置的上下文的路径。提供了 initial_context 配置时会在其返回的上下文中进行查找，没有提供时则直接在 InitialContext 中查找。

    <!-- mybatis中的环境，就是数据库连接信息
    		environments 可以配置多个数据库连接环境
    	 -->
      <environments default="test">
      	<!-- environment 标签配置一个数据库环境 -->
    <environment id="development">
    	<!-- transactionManager配置事务管理
    		jdbc 有提交，和回滚
    		managed 没有提交和回滚
    	 -->
      <transactionManager type="JDBC"/>
      <!-- 
      	dataSource 配置是否使用数据库连接池
      		POOLED 表示使用数据库连接池
      		UNPOOLED 表示不使用数据库连接池
       -->
      <dataSource type="POOLED">
    <property name="driver" value="${driver}"/>
    <property name="url" value="${url}"/>
    <property name="username" value="${username}"/>
    <property name="password" value="${password}"/>
      </dataSource>

## mybatis的核心配置之databaseIdProvider

MyBatis 可以根据不同的数据库厂商执行不同的语句，这种多厂商的支持是基于映射语句中的 databaseId 属性。 MyBatis 会加载不带 databaseId 属性和带有匹配当前数据库 databaseId 属性的所有语句。

    <databaseIdProvider type="DB_VENDOR">
    		<property name="SQL Server" value="sqlserver" />
    		<property name="MySQL" value="mysql" />
    		<property name="DB2" value="db2" />
    		<property name="Oracle" value="oracle" />
    </databaseIdProvider>

mybatis提供了一个类VendorDatabaseIdProvider，中的getDatabaseId() 方法用于获取数据库的标识。

property 标签name属性是获取数据库ID标识。

property 标签value属性是我们给mybatis定义的一个简短的标识。


### databaseId测试

    <!-- 	如果没有databaseId这属性，则找不到匹配的情况下。就默认执行它 -->
    	<select id="queryUserById" resultType="com.atguigu.pojo.User" >
    		select id,last_name lastName,sex from t_user where id = #{id}
    	</select>
    	<!-- 
    	databaseId="mysql" 表示只要你的数据库是mysql，执行queryUserById方法时就会执行当前sql语句
    	 -->
    	<select id="queryUserById" resultType="com.atguigu.pojo.User" databaseId="mysql">
    		select id,last_name lastName,sex from t_user where id = #{id} or 1 = 1 
    	</select>
    		<!-- 
    	databaseId="oracle" 表示只要你的数据库是oracle，执行queryUserById方法时就会执行当前sql语句
    	 -->
    	<select id="queryUserById" resultType="com.atguigu.pojo.User" databaseId="oracle">
    		select id,last_name lastName,sex from t_user where id = #{id} or 1 = 2
    	</select>

## mybatis的核心配置之Mappers

把mapper配置文件注入到mybatis-config.xml核心配置文件中有三种常用方式。

1. 在classpath路径下引入
2. 使用mapper接口的形式导入配置
3. 使用包扫描的方式引入配置文件

    <!-- 从classpath路径下导入指定的配置文件 -->
    	<mappers>
    		<mapper resource="org/mybatis/builder/AuthorMapper.xml" />
    		<mapper resource="org/mybatis/builder/BlogMapper.xml" />
    		<mapper resource="org/mybatis/builder/PostMapper.xml" />
    	</mappers>
    	<!-- 使用mapper接口类导入配置文件 -->
    	<mappers>
    		<mapper class="org.mybatis.builder.AuthorMapper" />
    		<mapper class="org.mybatis.builder.BlogMapper" />
    		<mapper class="org.mybatis.builder.PostMapper" />
    	</mappers>
    	<!-- 扫描包下所有的配置文件
	    1、接口名和Mapper配置文件名必须相同
	    2、接口文件和Mapper配置文件必须在同一个包下
	     	-->
	    	<mappers>
	    		<package name="org.mybatis.builder" />
	    	</mappers>