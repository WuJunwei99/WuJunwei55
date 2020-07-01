---
layout: post
title: "mybatis 学习笔记（4）——mybatis的参数传递和自定义结果集"
date: 2020-06-29 18:22:15 +0800
categories: notes
tags: mybatis
img: https://s1.ax1x.com/2020/06/29/Nfjd8f.png
---
MyBatis的注解使用方式；mybatis的参数传递；自定义结果集<resultMap></resultMap> 


# MyBatis的注解使用方式（了解，主要使用xml）

还是使用t_user表，来实现CRUD操作。

    public interface UserMapper {
    	
    	@Select("select id,last_name lastName,sex from t_user where id = #{id}")
    	public User queryUserById(Integer id);
    
    	@Select("select id,last_name lastName,sex from t_user")
    	public List<User> queryUsers();
    
    	@Update(value="update t_user set last_name=#{lastName},sex=#{sex} where id = #{id}")
    	public int updateUser(User user);
    
    	@SelectKey(statement="select last_insert_id()",before=false,keyProperty="id",resultType=Integer.class)
    	@Insert(value="insert into t_user(`last_name`,`sex`) values(#{lastName},#{sex})")
    	public int saveUser(User user);
    
    	@Delete("delete from t_user where id = #{id}")
    	public int deleteUserById(Integer id);
    
    }

因为有时项目的sql语句会比较多，而写在注解中不现实，我们将可以采用一种注解和xml方式共用配置sql语句，在注解中书写简单的sql语句，将复杂的sql语句写在mapper中。

UserMapper接口修改：

    //	@Select("select id,last_name lastName,sex from t_user where id = #{id}")
    	public User queryUserById(Integer id);

UserMapper.xml配置文件内容： 
   
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.atguigu.mapper.UserMapper">
    
    <!-- 	public User queryUserById(Integer id); -->
      <select id="queryUserById" resultType="com.atguigu.pojo.User" >
       	 select id,last_name lastName,sex from t_user where id = #{id}
      </select>
    </mapper>

需要注意的是，对于同一个函数只能出现一次，要么使用注解方式要么就使用mapper接口的方式。

# mybatis的参数传递

## 一个普通数据类型

    public interface UserMapper {
    	
    	public User queryUserById(Integer id);
    
    }

UserMapper.xml配置文件：

	<!-- 
		public User queryUserById(Integer id);
		当方法的参数类型是一个普通数据类型的时候，
		那么sql语句中配置的占位符里，可以写上参数名：#{id}
	 -->
	<select id="queryUserById" resultType="com.atguigu.pojo.User">
		select id,last_name lastName,sex from t_user where id = #{id}
	</select>

## 多个普通数据类型

UserMapper接口：

	/**
	 * 	根据用户名和性别查询用户信息
	 */
	public List<User> queryUsersByNameOrSex(String name, Integer sex);

UserMapper.xml配置文件：

    <!-- 	public List<User> queryUsersByNameOrSex(String name, Integer sex);
    		当方法参数是多个普通类型的时候。我们需要在占位符中写入的可用值是：0,1，param1，param2
    		
    		0 			表示第一个参数（不推荐使用）
    		1 			表示第二个参数 （不推荐使用）
    		param1		表示第一个参数（推荐使用）
    		param2  	表示第二个参数（推荐使用）
    		paramN		表示第n个参数（推荐使用）
     -->
    	<select id="queryUsersByNameOrSex" resultType="com.atguigu.pojo.User">
    		select id,last_name lastName,sex from t_user where last_name = #{param1} or sex = #{param2}
    	</select>

## @Param注解命名参数

UserMapper接口

	/**
	 * 根据用户名和性别查询用户信息
	 */
	public List<User> queryUsersByNameOrSex(@Param("name") String name,
			@Param("sex") Integer sex);

UserMapper.xml配置文件：

	<!-- 
			public List<User> queryUsersByNameOrSex(@Param("name") String name,
			@Param("sex") Integer sex);
			当方法有多个参数的时候，我们可以使用mybatis提供的注解@Param来对方法的参数进行命名。
			全名之后的使用。如下：
			@Param("name") String name			====使用>>>>				#{name}
			@Param("sex") Integer sex			====使用>>>>				#{sex}
			
			使用了@Param之后，原来的0,1就不能再使用了。
			但是Param1，和param2，可以使用。
	 -->
	<select id="queryUsersByNameOrSex" resultType="com.atguigu.pojo.User">
		select id,last_name lastName,sex from t_user where last_name = #{name} or sex = #{sex}
	</select>

## 传递一个Map对象作为参数

UserMapper接口：

	/**
	 * 希望Map中传入姓名和性别信息，以做为查询条件。
	 */
	public List<User> queryUsersByMap(Map<String, Object> paramMap);

UserMapper.xml配置文件：

    <!-- 	public List<User> queryUsersByMap(Map<String, Object> paramMap); 
    		当我们方法的参数类型是Map类型的时候，注意。
    		在配置的sql语句的占位符中写的参数名一定要和Map的key一致对应。
    		
    		last_name = #{name} 			<<<<========>>>		paramMap.put("name","bbb");	
    		sex = #{sex}					<<<<========>>>		paramMap.put("sex",1);	
    -->
    	<select id="queryUsersByMap" resultType="com.atguigu.pojo.User">
    		select id,last_name lastName,sex from t_user where last_name = #{name} or sex = #{sex}
    	</select>
UserMapperTest.java中添加:

	@Test
	public void queryUserByMap() {
		SqlSession session = sqlSessionFactory.openSession();
		try{
			UserMapper mapper = session.getMapper(UserMapper.class);
			Map<String,Object> paraMap = new HashMap<String,Object>();
			paraMap.put("name", "wjw");
			paraMap.put("sex", 2);
			mapper.queryUserByMap(paraMap).forEach(System.out::println);
		}finally{
			session.close();
		}
		fail("Not yet implemented");
	}

## 一个javaBean数据类型

UserMapper接口

	public int updateUser(User user);

UserMapper.xml配置文件：

    <!-- 	public int updateUser(User user)
    		如果传入的参数是一个javaBean的时候，占位符中，要写上javaBean的属性名。
    		
    		JavaBean属性								sql中的占位符
    	private Integer id;								#{id}
    	private String lastName;						#{lastName}
    	private Integer sex;							#{sex}
    		
     -->
	<update id="updateUser" parameterType="com.atguigu.pojo.User">
		update 
			t_user 
		set 
			last_name=#{lastName},
			sex=#{sex}
		where 
			id=#{id}
	</update>

## 多个Pojo数据类型

UserMapper接口

	/**
	 * 	要求使用第一个User对象的lastName属性，和第二个User对象的sex属性来查询用户信息。
	 */
	public List<User> queryUsersByTwoUsers(User name,User sex);

UserMapper.xml配置文件：

    <!-- 	public List<User> queryUsersByTwoUsers(User name,User sex);
    
    		如果你是多个JavaBean类型的时候，第一个参数是param1，第二个参数是param2.以此类推第n个参数就是paramN
    		当然你也可以使用@Param来规定参数名。
    		
    		如果你想要的只是参数对象中的属性，而需要写成为如下格式：
    			#{参数.属性名} 

示例:
		last_name ======== #{param1.lastName}  
		sex ========= #{param2.sex} 
 -->
	<select id="queryUsersByTwoUsers" resultType="com.atguigu.pojo.User">
		select id,last_name lastName,sex from t_user where last_name = #{param1.lastName} or sex = #{param2.sex} 
	</select>

## 模糊查询

需求：现在要根据用户名查询用户对象。 也就是希望查询如下：	select * from t_user where user_name like '%张%'

UserMapper接口：

	/**
	 * 根据给定的名称做用户名的模糊查询
	 */
	public List<User> queryUsersLikeName(String name);

UserMapper.xml配置文件：
    
    <!-- 		/** -->
    <!-- 	 * 根据给定的名称做用户名的模糊查询 -->
    <!-- 	 */ -->
    <!-- 	public List<User> queryUsersLikeName(String name); -->
    	<select id="queryUsersLikeName" resultType="com.atguigu.pojo.User">
    		select id,last_name lastName,sex from t_user where last_name like #{name} 
    	</select>

测试的代码是：

	@Test
	public void testQueryUsersLikeName() {
		SqlSession session = sqlSessionFactory.openSession();
		try {
			
			UserMapper mapper = session.getMapper(UserMapper.class);
			
			String name = "bb";
			
			String temp = "%"+name+"%";
			
			mapper.queryUsersLikeName(temp).forEach(System.out::println);
			
		} finally {
			session.close();
		}
	}

UserMapper.xml中另一种写法：

    <!-- 		/** -->
    <!-- 	 * 根据给定的名称做用户名的模糊查询 -->
    <!-- 	 */ -->
    <!-- 	public List<User> queryUsersLikeName(String name);
    			\#{}	是占位符
    			${} 是把参数的值原样输出到sql语句中，然后做字符串的拼接操作
     -->
    	<select id="queryUsersLikeName" resultType="com.atguigu.pojo.User">
    		select id,last_name lastName,sex from t_user where last_name like '%${value}%'
    	</select>

## {}和${}的区别

\#{} 它是占位符	 ===>>>	?

${} 它是把表示的参数值原样输出，然后和sql语句的字符串做拼接操作。

### concat函数

![](https://s1.ax1x.com/2020/07/01/NozvHf.png)

可以使用concat很好的解决 模糊查询的问题

	<select id="queryUsersLikeName" resultType="com.atguigu.pojo.User">
		select id,last_name lastName,sex from t_user where last_name like concat('%',#{name},'%')
	</select>

## 自定义结果集<resultMap></resultMap>

resultMap标签，是自定义结果集标签。

resultMap标签可以把查询回来的结果集封装为复杂的javaBean对象。

原来的resultType属性它只能把查询到的结果集转换成为简单的JavaBean对象。

所谓简单的JavaBean对象，是指它的属性里没有JavaBean或集合类型的对象，反之亦然。

复杂的JavaBean对象，属性里有JavaBean类型或集合类型的属性的对象叫复杂的javaBean。

### 创建一对一数据库表

    ## 一对一数据表
    ## 创建锁表
    create table t_lock(
    	`id` int primary key auto_increment,
    	`name` varchar(50)
    );
    
    
    ## 创建钥匙表
    create table t_key(
    	`id` int primary key auto_increment,
    	`name` varchar(50),
    	`lock_id` int ,
    	foreign key(`lock_id`) references t_lock(`id`)
    );
    
    
    ## 插入初始化数据
    insert into t_lock(`name`) values('阿里巴巴');
    insert into t_lock(`name`) values('华为');
    insert into t_lock(`name`) values('联想');
    
    insert into t_key(`name`,`lock_id`) values('马云',1);
    insert into t_key(`name`,`lock_id`) values('任正非',2);
    insert into t_key(`name`,`lock_id`) values('柳传志',3);

### 创建实体对象

锁对象
    
    public class Lock {
    
    	private Integer id;
    	private String name;

钥匙对象

    public class Key {
    
    	private Integer id;
    	private String name;
    	private Lock lock;

### 一对一级联属性使用

#### KeyMapper接口

    public interface KeyMapper {
    
    	public Key queryByIdSampler(Integer id);
    
    }

#### 测试的代码：

    public class KeyMapperTest {
    
    	@Test
	public void testQueryByIdSampler() throws IOException {
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("com/ncu/zte/mybatis6_29/mybatis-config.xml"));
		SqlSession session = sqlSessionFactory.openSession();
		
		try{
			KeyMapper mapper = session.getMapper(KeyMapper.class);
			System.out.println(mapper.queryByIdSampler(2));
		}finally{
			session.close();
		}
		fail("Not yet implemented");
	}

#### KeyMapper.xml配置文件：

**普通查询：**
	<select id="queryByIdSampler" resultType="key">
		select 
			t_key.*,t_lock.name lock_name
		from 
			t_key left join t_lock
		on 
			t_key.lock_id = t_lock.id
		where 
			t_key.id = #{id}
	</select>


**运行结果：**

![](https://s1.ax1x.com/2020/07/01/NozXut.png)

**修改后的配置文件**：

    	<!-- 
    		resultMap 标签可以把查询到的resultSet结果集转换成为复杂的JavaBean对象
    			type 属性 设置resultMap需要转换出来的Bean对象的全类名<br/>
    			id 属性 设置一个唯一的标识，给别人引用
    	 -->
    		<resultMap type="key" id="queryKeyByIdForSample_resultMap">
    		<!-- id标签负责将主键列转换到bean对象的属性
    				column 设置将哪个主键列转换到指定的对象属性
    				property 属性设置将值注入到哪个对象的属性
    		 -->
    		<id column="id" property="id"/>
    		<!-- result标签负责将非主键列转换到bean对象的属性 -->
    		<result column="name" property="name"/>
    		<!-- 将lock_id列注入到lock对象的id属性中
    			子对象.属性名  这种写法叫级联属性映射
    		 -->
    		<result column="lock_id" property="lock.id"/>
    		<result column="lock_name" property="lock.name"/>
    	</resultMap>
    	
    <!-- 		public Key queryKeyByIdForSample(Integer id); -->
    	<select id="queryByIdSampler" resultMap="queryKeyByIdForSample_resultMap">
    		select 
    			t_key.*,t_lock.name lock_name
    		from 
    			t_key left join t_lock
    		on 
    			t_key.lock_id = t_lock.id
    		where 
    			t_key.id = #{id}
    	</select>

**运行结果：**

![](https://s1.ax1x.com/2020/07/01/NozLjI.png)

### <association /> 嵌套结果集映射配置

<association /> 可以配置映射一个子对象

	<resultMap type="key" id="queryKeyByIdForSample_resultMap">
		<!-- id标签负责将主键列转换到bean对象的属性
				column 设置将哪个主键列转换到指定的对象属性
				property 属性设置将值注入到哪个对象的属性
		 -->
		<id column="id" property="id"/>
		<!-- result标签负责将非主键列转换到bean对象的属性 -->
		<result column="name" property="name"/>
		
		<!-- 
			association 标签映射子对象
				property 属性设置association配置哪个子对象
				javaType 属性设置lock具体的全类名
		 -->
		<association property="lock" javaType="lock">
			<id column="lock_id" property="id"/>
			<result column="lock_name" property="name"/>
		</association>
		
		<!-- 将lock_id列注入到lock对象的id属性中
			子对象.属性名  这种写法叫级联属性映射
		
		<result column="lock_id" property="lock.id"/>
		<result column="lock_name" property="lock.name"/>
	 -->
	</resultMap>

### <association /> 定义分步查询

association标签还可以通过调用一个查询，得到子对象

#### KeyMapper接口

	/**
	 * 这个方法只查key的信息
	 */
	public Key queryKeyByIdForTwoStep(Integer id);

#### LockMapper接口

    public interface LockMapper {
    	/**
    	 * 只查lock
    	 * @param id
    	 * @return
    	 */
    	public Lock queryLockById(Integer id);
    	
    }

#### LockMapper.xml配置文件：

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
      PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
      "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="com.ncu.zte.mybatis6_29.mapper.LockMapper">
    	
    	<select id="queryLockById" resultType="lock">
    		select id,name from t_lock where id = #{id}
    	</select>
    
    </mapper>

#### KeyMapper.xml配置文件：

	<resultMap type="key" id="queryKeyByIdForTwoStep_resultMap">
		<id column="id" property="id"/>
		<result column="name" property="name"/>
		<!-- 
			association 用来配置子对象
				property属性设置你要配置哪个子对象
				select 属性配置你要执行哪个查询得到这个子对象
				column 属性配置你要将哪个列做为参数传递给查询用
		 -->
		<association property="lock" column="lock_id"
			select="com.ncu.zte.mybatis6_29.mapper.LockMapper.queryLockById" />
	</resultMap>


### 延迟加载（懒加载）

延迟加载在一定程序上可以减少很多没有必要的查询。给数据库服务器提升性能上的优化。
要启用延迟加载，需要在mybatis-config.xml配置文件中，添加如下两个全局的settings配置。

		<!-- 打开延迟加载的开关 -->  
       <setting name="lazyLoadingEnabled" value="true" />  
       <!-- 将积极加载改为消极加载  按需加载 -->  
    <setting name="aggressiveLazyLoading" value="false"/>  

测试代码：

	@Test
	public void testQueryByIdSampler() throws IOException, InterruptedException {
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("com/ncu/zte/mybatis6_29/mybatis-config.xml"));
		SqlSession session = sqlSessionFactory.openSession();
		
		try{
			KeyMapper mapper = session.getMapper(KeyMapper.class);
			Key key = mapper.queryKeyByIdForTwoStep(1);
			
			System.out.println( key.getName() );
			

		}finally{
			session.close();
		}
		fail("Not yet implemented");
	}

运行结果：

![](https://s1.ax1x.com/2020/07/01/NozjDP.md.png)

测试代码：

	@Test
	public void testQueryByIdSampler() throws IOException, InterruptedException {
		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsStream("com/ncu/zte/mybatis6_29/mybatis-config.xml"));
		SqlSession session = sqlSessionFactory.openSession();
		
		try{
			KeyMapper mapper = session.getMapper(KeyMapper.class);
			Key key = mapper.queryKeyByIdForTwoStep(1);
			
			System.out.println( key.getName() );
			
			Thread.sleep(5000);
			
			System.out.println( key.getLock() );
		}finally{
			session.close();
		}
		fail("Not yet implemented");
	}

运行结果：

![](https://s1.ax1x.com/2020/07/01/NTSp4g.md.png)