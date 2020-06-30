---
layout: post
title: "mybatis 学习笔记（2）——mybatis的增，删，改，查实现"
date: 2020-06-28 20:10:53 +0800
categories: notes
tags: mybatis
img: https://s1.ax1x.com/2020/06/29/Nfjd8f.png
---
传统方式mybatis的增，删，改，查实现与


## 传统方式mybatis的增，删，改，查实现

### 编写UserDao接口

编写一个接口，里面方法包括了增删改查的数据操作函数；

    public interface UserDao {
    	
    	public int addUser(User user);
    	
    	public int deleteUserById(Integer id);
    	
    	public int updateUser(User user);
    	
    	public User queryUserById(Integer id);
    	
    	public List<User> queryUser();
    	
    	
    }

### 编写UserMapper.xml配置文件

UserMapper的配置文件中需要标明命名空间namespace，并通过此进行调用映射语句，调用方式为命名空间名字.函数名

在书写select标签时，如果返回类型为javabean，需要（最好）写上resultType="com.ncu.zte.mybatis6_29.User"（双引号内为路径.类名）

当然，如果传入的参数类型为javabean时，也需要（最好）进行标识，写上parameterType="com.ncu.zte.mybatis6_29.User"（双引号内为路径.类名）

    <mapper namespace="com.ncu.zte.mybatis6_29.User">
    
    <!-- 
    select 是sql语句
    id是当前的这个语句配置的一个唯一标识符
    resultType执行了select的查询语句后，每行记录对应的javabean对象全类名
     -->
    
      <select id="selectUserById" resultType="com.ncu.zte.mybatis6_29.User">
       select id,last_name lastName,sex from t_user where id = #{id}
      </select>
      
    <!--   	public int addUser(User user);
    
     -->
      <insert  id="addUser"  parameterType="com.ncu.zte.mybatis6_29.User" useGeneratedKeys="true" keyProperty="id">
       insert into t_user(last_name,sex)values(#{lastName},#{sex})
      </insert >
    
      <delete id="deleteUserById" >
       delete from t_user where id = #{id}
      </delete>
      
      <update  id="updateUser" parameterType="com.ncu.zte.mybatis6_29.User">
       update t_user set last_name = #{lastName},sex=#{sex} where id = #{id}
      </update >
      
      	<select id="queryUsers" resultType="com.ncu.zte.mybatis6_29.User">
    		select id,last_name lastName,sex from t_user
    	</select>
    </mapper>

### 编写UserDaoImpl实现类

准备一个sqlsessionfactory对象（相当于连接connection）

使用mybatis开发，一个数据库只能有一个sqlsessionfactory对象

	private SqlSessionFactory sqlSessionFactory;

因为sqlsessionFactory是参数传入，所以要写构造方法

	public UserDaoImpl(SqlSessionFactory sqlSessionFactory) {
		super();
		this.sqlSessionFactory = sqlSessionFactory;
	}

编写实现类时除了查询语句外，要为别的语句添加手动提交事务的命令

session.commit();

	public int addUser(User user) {
		// TODO Auto-generated method stub
		int result = -1;
		SqlSession session = sqlSessionFactory.openSession();
		try{
			result = session.insert("com.ncu.zte.mybatis6_29.User.addUser",user);
			session.commit();//手动提交事务

		}finally{
			session.close();
		}
		
		return result;
	}

### 编写mybatis-config.xml核心配置文件

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
      PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
      <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
    <property name="driver" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mybatis"/>
    <property name="username" value="root"/>
    <property name="password" value="root"/>
      </dataSource>
    </environment>
      </environments>
      <mappers>
    <mapper resource="com/atguigu/pojo/UserMapper.xml"/>
      </mappers>
    </configuration>

### 编写UserDao的测试

右键userDao.java，创建所有函数的测试类，在类中首先创建SqlSessionFactory实例并进行连接，创建userDao用于测试

	static SqlSessionFactory sqlSessionFactory;
	static UserDao userDao;
	
	@BeforeClass
	public static void setUpBeforeClass() throws Exception {
		sqlSessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("mybatis-config.xml"));
		userDao = new UserDaoImpl(sqlSessionFactory);
	}


再编写相关测试代码，

	@Test
	public void testAddUser() {
		userDao.addUser(new User(null, "admin", 1));
		fail("Not yet implemented");
	}

	@Test
	public void testQueryUser() {
		userDao.queryUser().forEach(System.out::println);
		fail("Not yet implemented");
	}

### 插入记录并返回主键

insert标签配置insert语句
id属性配置唯一的标识
parameterType 设置方法的参数类型（可以省略，一般如果是JavaBean，不推荐省略）
useGeneratedKeys="true" 表示使用数据库生所的主键
keyProperty="id" 属性设置将数据库中返回的自增id值交给哪个属性

	<insert id="saveUser" parameterType="com.atguigu.pojo.User" useGeneratedKeys="true" keyProperty="id">
		insert into t_user(`last_name`,`sex`) values(#{lastName},#{sex})
	</insert>

### selectKey标签的使用

order属性设置selectKey里配置的sql语句的执行顺序:

1. AFTER		在insert语句之后执行
2. BEFORE		在insert语句之前执行

keyProperty="id" 属性设置将数据库中返回的自增id值交给哪个属性

resultType="int" 属性表示查询之后返回的类型

int	表示Integer类型


    <insert id="saveUser" parameterType="com.atguigu.pojo.User">
    		<selectKey order="AFTER" keyProperty="id" resultType="int">
    			select last_insert_id()
    		</selectKey>
    		insert into t_user(`last_name`,`sex`) values(#{lastName},#{sex})
    </insert>
    
<selectKey\> 是一个子标签，可以设置一个sql语句去执行。

selectKey	返回Oracle的序列自增主键

    <selectKey order="BEFORE" resultType="int" keyProperty="id"> 
     select 序列名.nextval as id from dual 
    </selectKey> 
    
## Mapper接口方式的mybatis的增，删，改，查实现

### Mapper接口编程的命名习惯

JavaBean			====>>>>			User

sql配置文件     ====>>>>			UserMapper.xml			

Mapper接口		====>>>>			UserMapper接口

Mapper接口实现CRUD，不需要编写接口的实现类，只要有mapper.xml（sql配置文件）就可以使用。

### Mapper接口开发有四个开发规范**必须遵守**

1. Mapper.xml配置文件的名称空间值必须是Mapper接口的全类名
2. mapper.xml配置文件中id值必须是方法名
3. Mapper.xml配置文件中的parameterType参数类型必须和接口的方法参数类型一致（如果写的话）
4. Mapper.xml配置文件中的resultType返回值类型必须和接口的方法返回值类型一致（javaBean才有需要）

### Mapper接口

    public interface UserMapper {
    
    	public int saveUser(User user);
    	
    	public int deleteUserById(Integer id);
    	
    	public int updateUser(User user);
    
    	public User queryUserById(Integer id);
    	
    	public List<User> queryUsers();
    	
    }

### UserMapper.xml配置文件

    <mapper namespace="com.ncu.zte.mybatis6_29.mapper.UserMapper">
    <insert id="addUser" parameterType="com.ncu.zte.mybatis6_29.User">
    	insert into t_user(last_name,sex)values (#{lastName},#{sex})
    </insert>
    
    <delete id = "deleteUserById">
    	delete from t_user where id=#{id}
    </delete>
    
    <update id="updateUser" parameterType="com.ncu.zte.mybatis6_29.User">
    	update t_user set last_name=#{lastName},sex = #{sex} where id = #{id}
    </update>
    
    <select id="queryUserById" resultType="com.ncu.zte.mybatis6_29.User">
    	select id,last_name lsatName,sex from t_user where id = #{id}
    </select>
    <select id="queryUsers" resultType="com.ncu.zte.mybatis6_29.User">
    	select id,last_name lsatName,sex from t_user 
    </select>
    </mapper>

### Mapper接口的测试

右键UserMapper的接口类，创建测试类，在测试类中创建sqlsessionfactory实体并进行连接，之后开始编写具体的测试函数，测试函数下应该创建session实体，调用opensession（），之后记得close（）；在try中应该创建usermapper对象，调用相关的mapper方法进行测试。

而且它不是基于字符串常量的，就会更安全

	static SqlSessionFactory sqlSessionFactory;
	
	@BeforeClass
	public static void setUpBeforeClass() throws Exception {
		sqlSessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("com/ncu/zte/mybatis6_29/mybatis-config.xml"));
	}

	@Test
	public void testAddUser() {
		SqlSession session = sqlSessionFactory.openSession();
		try{
			UserMapper mapper = session.getMapper(UserMapper.class);
			mapper.addUser(new User(null,"zzp",1));
			session.commit();
		}finally{
			session.close();
		}
		
		fail("Not yet implemented");
	}

	@Test
	public void testQueryUsers() {
		SqlSession session = sqlSessionFactory.openSession();
		try{
			UserMapper mapper = session.getMapper(UserMapper.class);
			mapper.queryUsers().forEach(System.out::println);
		}finally{
			session.close();
		}
		fail("Not yet implemented");
	}

相关测试均运行成功，部分运行结果如下：

![](https://s1.ax1x.com/2020/06/30/N4LWxU.md.png)