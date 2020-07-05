---
layout: post
title: "Spring 学习笔记（7）——Spring之数据访问"
date: 2020-07-04 13:22:18 +0800
categories: notes
tags: spring
img: https://s1.ax1x.com/2020/07/02/NqzFiV.png
---
Spring数据访问工程环境搭建;Spring之JdbcTemplate使用;通过继承JdbcDaoSupport创建JdbcTemplate的Dao


# Spring之数据访问

## Spring数据访问工程环境搭建

#### 环境搭建

导入需要的jar包

    commons-logging-1.1.3.jar
    druid-1.1.9.jar
    mysql-connector-java-5.1.37-bin.jar
    spring-aop-4.3.18.RELEASE.jar
    spring-beans-4.3.18.RELEASE.jar
    spring-context-4.3.18.RELEASE.jar
    spring-core-4.3.18.RELEASE.jar
    spring-expression-4.3.18.RELEASE.jar
    spring-jdbc-4.3.18.RELEASE.jar
    spring-orm-4.3.18.RELEASE.jar
    spring-test-4.3.18.RELEASE.jar
    spring-tx-4.3.18.RELEASE.jar

      <build>  
       <plugins>  
       <plugin>  
       <groupId>org.apache.maven.plugins</groupId>  
       <artifactId>maven-compiler-plugin</artifactId>  
       <configuration>  
       <source>1.8</source>  
       <target>1.8</target>  
       </configuration>  
       </plugin>  
       </plugins>  
  
pom.xml
     
    </build>
     <dependencies>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-beans</artifactId>
      <version>5.1.16.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.1.6.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>5.1.16.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-expression</artifactId>
      <version>5.1.16.RELEASE</version>
    </dependency>
    <dependency>
    	<groupId>org.springframework</groupId>
    	<artifactId>spring-aop</artifactId>
    	<version>5.1.16.RELEASE</version>
    </dependency>
    <dependency>
    	<groupId>org.springframework</groupId>
    	<artifactId>spring-jdbc</artifactId>
    	<version>5.1.16.RELEASE</version>
    </dependency>
    <dependency>
    	<groupId>org.springframework</groupId>
    	<artifactId>spring-orm</artifactId>
    	<version>5.1.16.RELEASE</version>
    </dependency>
    <dependency>
    	<groupId>org.springframework</groupId>
    	<artifactId>spring-test</artifactId>
    	<version>5.1.16.RELEASE</version>
    </dependency>
     	<dependency>
     		<groupId>commons-logging</groupId>
     		<artifactId>commons-logging</artifactId>
     		<version>1.2</version>
     	</dependency>
     	
    	<dependency>
    		<groupId>mysql</groupId>
    		<artifactId>mysql-connector-java</artifactId>
    		<version>5.1.26</version>
    	</dependency>
    	<dependency>
    	  <groupId>com.alibaba</groupId>
    	  <artifactId>druid</artifactId>
    	  <version>1.1.23</version>
    	</dependency>
     </dependencies>

配置jdbc.properties

    user=root
    password=root
    driverClassName=com.mysql.jdbc.Driver
    url=jdbc:mysql://localhost:3306/test
    initialSize=5
    maxActive=10

applicationContext.xml配置文件：

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    	xmlns:context="http://www.springframework.org/schema/context"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd">
    	
    	<!-- 包扫描 -->
    	<context:component-scan base-package="com.atguigu"></context:component-scan>
    	<!-- 加载jdbc.properties属性配置文件 -->
    	<context:property-placeholder location="classpath:jdbc.properties"/>
    	<!-- 配置数据源 -->
    	<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    		<property name="username" value="${user}" />
    		<property name="password" value="${password}" />
    		<property name="driverClassName" value="${driverClassName}" />
    		<property name="url" value="${url}" />
    		<property name="initialSize" value="${initialSize}" />
    		<property name="maxActive" value="${maxActive}" />
    	</bean>
    
    </beans>

测试的代码：

    @ContextConfiguration(locations="classpath:applicationContext.xml")
    @RunWith(SpringJUnit4ClassRunner.class)
    public class SpringTest {
    
    	@Autowired
    	DataSource dataSource;
    	
    	@Test
    	public void testDataSource() throws Exception {
    		System.out.println( dataSource.getConnection() );
    	}
    	
    }

测试结果：

![](https://s1.ax1x.com/2020/07/05/USzQO0.md.png)

## Spring之JdbcTemplate使用

在Spring中提供了对jdbc的封装类叫JdbcTemplate。它可以很方便的帮我们执行sql语句，操作数据库。

先准备单表的数据库数据
    
    drop database if exists jdbctemplate;
    create database jdbctemplate;
    use jdbctemplate;
    
    CREATE TABLE `employee` (
      `id` int(11) primary key AUTO_INCREMENT,
      `name` varchar(100) DEFAULT NULL,
      `salary` decimal(11,2) DEFAULT NULL
    );
    
    insert  into `employee`(`id`,`name`,`salary`) 
    values (1,'李三',5000.23),(2,'李四',4234.77),(3,'王五',9034.51),
    (4,'赵六',8054.33),(5,'孔七',6039.11),(6,'曹八',7714.11);
    
    select * from employee;


JdbcTemplate的使用需要在applicationContext.xml中进行配置

	<!-- jdbcTemplate -->
	<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
		<property name="dataSource"  ref="dataSource"/>
	</bean>

![](https://s1.ax1x.com/2020/07/05/USzMyq.md.png)

#### 实验2：将id=5的记录的salary字段更新为1300.00

	@Test
	public void test2() throws Exception {
		String sql = "update employee set salary = ? where id = ?";
		// update方法执行insret、update、delete语句
		jdbcTemplate.update(sql,new BigDecimal(1300),5);
	}

![](https://s1.ax1x.com/2020/07/05/USzuSs.md.png)

#### 批量插入

	@Test
	public void test3() throws Exception {
		String sql = "insert into employee(`name`,`salary`) values(?,?)";
		/**
		 * 一条sql语句参数是一个一维数组，那么多条sql语句，它的参数是多个一维数组
		 */
		List<Object[]> batchArgs = new ArrayList<Object[]>();
		
		batchArgs.add(new Object[]{"mjc",new BigDecimal(1600)});
		batchArgs.add(new Object[]{"zzp",new BigDecimal(1900)});
		jdbcTemplate.batchUpdate(sql, batchArgs);
	}

![](https://s1.ax1x.com/2020/07/05/USzKln.md.png)

#### 查询id=5的数据库记录，封装为一个Java对象返回


创建Employee对象

    public class Employee {
    	private Integer id;
    	private String name;
    	private BigDecimal salary;


测试代码：

	@Test
	public void test4() throws Exception {
		String sql = "select id,name,salary from employee where id = ?";
		/**
		 * 第一个参数是sql语句<br/>
		 * RowMapper接口，它的实现类可以帮我们把查询到的resultSet每一行记录封装成为javaBean返回
		 */
		Employee employee = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<Employee>(Employee.class), 5);
		System.out.println(employee);
	}

![](https://s1.ax1x.com/2020/07/05/USzmWj.md.png)

#### 实验5：查询salary>4000的数据库记录，封装为List集合返回

	@Test
	public void test5() throws Exception {
		String sql = "select id,name,salary from employee where salary > ?";
		/**
		 * 查询一行记录使用queryForObject。<br/>
		 * 查询多行记录，使用query方法
		 */
		jdbcTemplate.query(sql, new BeanPropertyRowMapper<Employee>(Employee.class), new BigDecimal(4000))
				.forEach(System.out::println);
	}

![](https://s1.ax1x.com/2020/07/05/USz3wT.md.png)

#### 查询最大salary

	// 实验6：查询最大salary
	@Test
	public void test6() throws Exception {
		String sql = "select max(salary) from employee";
		BigDecimal salary = jdbcTemplate.queryForObject(sql, BigDecimal.class);
		System.out.println( salary );
	}

![](https://s1.ax1x.com/2020/07/05/USz1mV.md.png)

#### 使用带有具名参数的SQL语句插入一条员工记录，并以Map形式传入参数值

配置applicationContext.xml配置文件：

	<!-- 配置可以执行命名参数sql语句的 NamedParameterJdbcTemplate -->
	<bean id="namedParameterJdbcTemplate" 
		class="org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate">
		<constructor-arg name="dataSource" ref="dataSource" />
	</bean>

测试代码：

	@Test
	public void test7() throws Exception {
		/**
		 * :name 相当于 ? 占位符，name就是这个参数的名称
		 */
		String sql = "insert into employee(`name`,`salary`) values(:name,:salary)";
		
		Map<String, Object> paramMap = new HashMap<String, Object>();
		paramMap.put("name", "我是命名（具名）参数的");
		paramMap.put("salary", new BigDecimal(1234));
		
		namedParameterJdbcTemplate.update(sql, paramMap);
	}

#### 重复实验7，以SqlParameterSource形式传入参数值

	// 实验8：重复实验7，以SqlParameterSource形式传入参数值
	@Test
	public void test8() throws Exception {
		/**
		 * :name 相当于 ? 占位符，name就是这个参数的名称
		 */
		String sql = "insert into employee(`name`,`salary`) values(:name,:salary)";
		
		Employee employee = new Employee(null, "我是具名参数插入的", new BigDecimal(30000));
		
		/**
		 * SqlParameterSource给sql语句传入需要的参数值
		 */
		namedParameterJdbcTemplate.update(sql, new BeanPropertySqlParameterSource(employee));
	}

## 创建Dao，自动装配JdbcTemplate对象

创建EmployeeDao ：

    @Repository
    public class EmployeeDao {
    
    	@Autowired
    	JdbcTemplate jdbcTemplate;
    
    	public int saveEmployee(Employee employee) {
    		return jdbcTemplate.update("insert into employee(`name`,`salary`) values(?,?)", employee.getName(),
    				employee.getSalary());
    	}
    }


测试代码：

	@Test
	public void test9() throws Exception {
		Employee employee = new Employee(null, "我是Dao插入的", new BigDecimal(1234));
		employeeDao.saveEmployee(employee);
	}

![](https://s1.ax1x.com/2020/07/05/USz8TU.md.png)

## 通过继承JdbcDaoSupport创建JdbcTemplate的Dao

    @Repository
    public class EmployeeDao extends JdbcDaoSupport{
    
    //	@Autowired
    //	JdbcTemplate jdbcTemplate;
    
    	@Autowired
    	public void initJdbcTemplate(DataSource dataSource) {
    		setDataSource(dataSource);
    	}
    
    	public int saveEmployee(Employee employee) {
    		return getJdbcTemplate().update("insert into employee(`name`,`salary`) values(?,?)", employee.getName(),
    				employee.getSalary());
    	}
    }

测试代码：

	@Test
	public void test10() throws Exception {
		Employee employee = new Employee(null, "我是JdbcDaoSupport插入的", new BigDecimal(1234));
		employeeDao.saveEmployee(employee);
	}

![](https://s1.ax1x.com/2020/07/05/USzJkF.md.png)


