---
layout: post
title: "Spring 学习笔记（5）——注解功能"
date: 2020-07-03 18:21:55 +0800
categories: notes
tags: spring
img: https://s1.ax1x.com/2020/07/02/NqzFiV.png
---
注解配置Dao、Service、Controller组件，指定扫描包时的过滤内容，@Qualifier注解，@Autowired注解，多个同类型的bean如何自动装配，Junit测试


## 注解配置Dao、Service、Controller组件

实验32：通过注解分别创建Dao、Service、Controller★

Spring配置bean的常用注解有

    @Controller	 		配置控制器
    @Service				service业务层组件
    @Repository			持久层dao组件
    @Component			除了以上三种都可以用component
    @Scope				配置作用域（单例，多例）

bean对象

    /**
     * @Component 注解表示<br/>
     * <bean id="book" class="com.atguigu.pojo.Book" />
     */
    @Component
    public class Book {
    }
    
    /**
     * @Repository 注解表示<br/>
     * <bean id="bookDao" class="com.atguigu.dao.BookDao" /><br/>
     * @Repository("abc") 表示
     * <bean id="abc" class="com.atguigu.dao.BookDao" /><br/>
     */
    @Repository
    @Scope("prototype")
    public class BookDao {
    }
    /**
     * @Service 注解表示<br/>
     * <bean id="bookService" class="com.atguigu.service.BookService" />
     */
    @Service
    public class BookService {
    }
    /**
     * @Controller 注解表示<br/>
     * <bean id="bookController" class="com.atguigu.controller.BookController" />
     */
    @Controller
    public class BookController {
    }

applicationContext.xml配置文件;

	<!-- 配置包扫描
			base-package	设置需要扫描的包名（它的子包也会被扫描）
	 -->
	<context:component-scan base-package="com.atguigu"></context:component-scan>

测试代码：

	@Test
	public void test1() throws Exception {

		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");

		System.out.println(applicationContext.getBean("bookDao"));
		System.out.println(applicationContext.getBean("bookDao"));
		System.out.println(applicationContext.getBean("bookDao"));
		System.out.println(applicationContext.getBean("book"));
		System.out.println(applicationContext.getBean("bookService"));
		System.out.println(applicationContext.getBean("bookController"));

	}

![](https://s1.ax1x.com/2020/07/04/Nvt7D0.md.png)

![](https://s1.ax1x.com/2020/07/04/NvtTuq.md.png)

![](https://s1.ax1x.com/2020/07/04/NvtIvn.md.png)

## 指定扫描包时的过滤内容

实验33：使用context:include-filter指定扫描包时要包含的类

实验34：使用context:exclude-filter指定扫描包时不包含的类

	<context:include-filter />	设置包含的内容

注意：通常需要与use-default-filters属性配合使用才能够达到“仅包含某些组件”这样的效果。即：通过将use-default-filters属性设置为false，

		<context:exclude-filter />	设置排除的内容

![](https://s1.ax1x.com/2020/07/04/Nvt5gs.md.png)

applicationContext.xml	中配置的内容如下

     	<!-- use-default-filters="false" 设置取消默认包含规则 -->
    	<context:component-scan base-package="com.atguigu" use-default-filters="false">
    		<!-- context:include-filter 设置包含的内容 -->
    		<context:include-filter type="annotation" expression="org.springframework.stereotype.Service"/>
    		<!-- context:exclude-filter 设置排除的内容 -->
    		<context:exclude-filter type="assignable" expression="com.atguigu.service.BookService"/>
    	</context:component-scan>
    
以上配置会包含所有@Service注解的类。排除com.atguigu.service.BookService类


	<!-- 配置包扫描
			base-package	设置需要扫描的包名（它的子包也会被扫描）
			use-default-filters="false" 去掉包扫描时默认包含规则
	 -->
	<context:component-scan base-package="com.atguigu" use-default-filters="false">
		<!-- 指定排除的内容
				type="annotation" 按注解进行过滤
				expression 注解的表达式（什么注解或注解的全类名）
		<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
		<context:exclude-filter type="assignable" expression="com.atguigu.controller.BookController"/> -->
		 <!-- 
		 		type="annotation" 按注解进行过滤
		  -->
		 <context:include-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
		 <context:include-filter type="assignable" expression="com.atguigu.controller.BookController"/>
	</context:component-scan>

![](https://s1.ax1x.com/2020/07/04/NvNFUO.md.png)

![](https://s1.ax1x.com/2020/07/04/NvtqET.md.png)


## 使用注解@Autowired自动装配

实验35：使用@Autowired注解实现根据类型实现自动装配★

@Autowired 注解 会自动的根据标注的对象类型在Spring容器中查找相对应的类。如果找到，就自动装配。

使用@Autowired注解，不需要get/set方法

    @Service
    public class BookService {
    	/**
    	 * @Autowired 实现自动注入<br/>
    	 * 	1、先按类型查找并注入<br/>
    	 */
    	@Autowired
    	private BookDao bookDao;
    
    	@Override
    	public String toString() {
    		return "BookService [bookDao=" + bookDao + "]";
    	}
    
    }

![](https://s1.ax1x.com/2020/07/04/NvtHbV.md.png)


## 多个同类型的bean如何自动装配

实验36：如果资源类型的bean不止一个，默认根据@Autowired注解标记的成员变量名作为id查找bean，进行装配★


bean对象

    @Repository
    @Scope("prototype")
    public class BookDao {
    }
    @Repository
    public class BookDaoExt extends BookDao{
    }
    @Service
    public class BookService {
    	/**
    	 * @Autowired 实现自动注入<br/>
    	 * 	1、先按类型查找并注入<br/>
    	 *  2、如果找到多个，就接着按属性名做为id继续查找并注入<br/>
    	 */
    	@Autowired
    	private BookDao bookDao;
    
    	@Override
    	public String toString() {
    		return "BookService [bookDao=" + bookDao + "]";
    	}
    
    }
    

测试的代码：

	@Test
	public void test1() throws Exception {
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
		System.out.println(applicationContext.getBean("bookService"));
	}

![](https://s1.ax1x.com/2020/07/04/NvtO5F.md.png)


## 使用@Qualifier装配指定id的bean对象

实验37：如果根据成员变量名作为id还是找不到bean，可以使用@Qualifier注解明确指定目标bean的id★


    @Service
    public class BookService {
    	/**
    	 * @Autowired 实现自动注入<br/>
    	 * 	1、先按类型查找并注入<br/>
    	 *  2、如果找到多个，就接着按属性名做为id继续查找并注入<br/>
    	 *  3、如果找到多个。但是属性名做为id找不到，可以使用@Qualifier("bookDao")注解指定id查找并注入<br/>
    	 */
    	@Autowired
    	@Qualifier("bookDaoExt")
    	private BookDao bookDao1;
    
    	@Override
    	public String toString() {
    		return "BookService [bookDao=" + bookDao1 + "]";
    	}
    
    }

![](https://s1.ax1x.com/2020/07/04/NvtLUU.md.png)

## @Autowired注解的required属性作用

实验39：@Autowired注解的required属性指定某个属性允许不被设置

    @Service
    public class BookService {
    	/**
    	 * @Autowired 实现自动注入<br/>
    	 * 	1、先按类型查找并注入<br/>
    	 *  2、如果找到多个，就接着按属性名做为id继续查找并注入<br/>
    	 *  3、如果找到多个。但是属性名做为id找不到，可以使用@Qualifier("bookDao")注解指定id查找并注入<br/>
    	 *  4、可以通过修改@Autowired(required=false)允许字段值为null
    	 */
    	@Autowired(required=false)
    	@Qualifier("bookDaoExt1")
    	private BookDao bookDao1;
    
    	@Override
    	public String toString() {
    		return "BookService [bookDao=" + bookDao1 + "]";
    	}
    
    }


![](https://s1.ax1x.com/2020/07/04/NvtjC4.md.png)

![](https://s1.ax1x.com/2020/07/04/Nvtv8J.md.png)


## @Autowired和@Qualifier在方法上的使用

实验38：在方法的形参位置使用@Qualifier注解


	/**
	 * @Autowired标注在方法上，那么此方法会在对象创建之后调用。
	 * 		1、先按类型查找参数并调用方法传递<br/>
	 * 		2、如果找到多个，就接着按参数名做为id继续查找并注入<br/>
	 *		3、如果找到多个。但是参数名做为id找不到，可以使用@Qualifier("bookDao")注解指定id查找并调用<br/>
	 *     4、可以通过修改@Autowired(required=false)允许不调用此方法也不报错
	 */
	@Autowired(required=false)
	public void setBookDao(@Qualifier("bookDaoExt1")BookDao abc) {
		System.out.println("BookDao进来啦 --->>> " + abc);

	}


![](https://s1.ax1x.com/2020/07/04/Nvad4U.md.png)

![](https://s1.ax1x.com/2020/07/04/Nva0CF.md.png)

![](https://s1.ax1x.com/2020/07/04/NvaaNT.md.png)

## 泛型注入（了解内容）

实验40：测试泛型依赖注入★


![](https://s1.ax1x.com/2020/07/04/NvN9Dx.md.png)

![](https://s1.ax1x.com/2020/07/04/NvNiVK.md.png)

## Spring的扩展的Junit测试

@ContextConfiguration
@RunWith

    /**
     * Spring扩展了Junit测试。测试的上下文环境中自带Spring容器。<br/>
     * 我们要获取Spring容器中的bean对象。就跟写一个属性一样方便。
     */
    // @ContextConfiguration配置Spring容器
    @ContextConfiguration(locations="classpath:applicationContext.xml")
    // @RunWith配置使用Spring扩展之后的Junit测试运行器
    @RunWith(SpringJUnit4ClassRunner.class)
    public class SpringJunitTest {
    
    	@Autowired
    	UserService userService;
    	
    	@Autowired
    	BookService bookService;
    	
    	@Test
    	public void test1() throws Exception {
    		bookService.save(new Book());
    		System.out.println("===========================================");
    		userService.save(new User());
    	}
    }


![](https://s1.ax1x.com/2020/07/04/NvNCb6.md.png)