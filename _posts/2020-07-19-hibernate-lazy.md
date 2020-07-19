---
layout: post
title: "hibernate的懒加载与抓取策略 "
date: 2020-07-19 13:54:51 +0800
categories: notes
tags: hibernate
img: https://s1.ax1x.com/2020/07/17/UsU08H.png
---
数据检索策略；懒加载;抓取策略

## Hibernate数据检索策略

Hibernate提供多种数据检索策略，常用的有：

* 立即检索
* 延迟检索（懒加载）

不同检索策略的作用域及默认检索策略

![](https://s1.ax1x.com/2020/07/18/UctX5T.png)

### 检索策略的运行机制

![](https://s1.ax1x.com/2020/07/18/UctOaV.png)

### 设定检索策略的常用属性

![](https://s1.ax1x.com/2020/07/18/UcNSxJ.png)


## 即时加载

当实体加载完成后，立即加载其关联数据

在配置文件（xx.hbm.xml）中设置lazy=false表示采用即时加载策略

	<hibernate-mapping>
	    <class name="cn.jbit.houserent.bean.District" table="district">
	        <id name="id" type="java.lang.Integer">
	            <column name="id" />
	            <generator class="native" />
	        </id>
	        <property name="name" type="java.lang.String">
	            <column name="district_name" length="50" not-null="true" />
	        </property>
	        <!--设置lazy属性为false，立即加载关联的对象-->
	        <set name="streets" table="street" cascade="all" inverse="true" lazy="false">
	            <key>
	                <column name="district_id" not-null="true" />
	            </key>
	            <one-to-many class="cn.jbit.houserent.bean.Street" />
	        </set>
	    </class>
	</hibernate-mapping>

![](https://s1.ax1x.com/2020/07/18/UcN9M9.png)

## 懒加载

懒加载(Load On Demand)是一种独特而又强大的数据获取方法,它能够在用户滚动页面的时候自动获取更多的数据,而新得到的数据不会影响原有数据的显示,同时最大程度上减少服务器端的资源耗用。总结一句话：什么时候需要数据，什么时候加载。

### 何处使用

Lazy(懒加载)在 hibernate 何处使用：

* <class>标签上，可以取值：true/false,(默认值为：true)
* <property>标签上，可以取值：true/false，需要类增强工具
* <set>、<list>集合上，可以取值：true/false/extra,(默认值为：true)
* <one-to-one>、<many-to-one>单端关联上，可以取值：false/proxy/noproxy
* Session.load()方法支持lazy，而session.get()不支持lazy;
* hibernate支持lazy策略只有在session打开状态下有效。


### 类的懒加载

![](https://s1.ax1x.com/2020/07/17/UyTFaV.png)

由javassist产生的代理类与Classes类是继承关系，

session.load()方法产生的是代理对象，该代理类是持久化类的子类

	/**
		 * 类的懒加载
		 */
		@Test
		public void testClass_lazy(){
			session = HibernateUtils.openSession();
			transaction = session.beginTransaction();
			Classes classes = (Classes) session.load(Classes.class, 1L);
			//classes.setName("asdj");
			System.out.println(classes.getName());//发出sql
			transaction.commit();
			session.close();
		}

#### 关联映射文件中集合标签中的lazy(默认为true)

	try {
		config=new Configuration().configure();
		sf=config.buildSessionFactory();
		session= sf.openSession();
		ts=session.beginTransaction();
		user=(UserBean)session.load(UserBean.class, 1);
		System.out.println("**"+user.getUserId());
		System.out.println("**"+user.getUsername());
		ts.commit();
	} catch (Exception e) {
		e.printStackTrace();
		ts.rollback();
	}
	finally{
		session.close();
	}
	
	  //不会发出sql语句
	  District district=(District)session.load(District.class, 83);
	  System.out.println("**");
	  //调用对象发出sql语句
	  System.out.println(district.getDisName());
	  //不会发sql语句
	  Set<Street> set=district.getSetStreet();
	  System.out.println("**");
	  //调用对象发出sql语句
	  Iterator<Street> it=set.iterator();
	  while(it.hasNext()){
	     Street street=it.next();
	     System.out.println(street.getStrName());
	}


#### 演示示例：实体对象延迟加载

为实体设置延时加载，即设置<class>元素的lazy属性为true

	<class name="cn.jbit.houserent.bean.User" table="users" schema="jbit" lazy="true">
	…
	</class>

测试：
	
	// 省略部分代码
	User user = (User)session.load(User.class, new Integer("281"));
	System.out.println("获取用户ID为1000的用户");
	user.getName();
	// 省略部分代码

![](https://s1.ax1x.com/2020/07/18/UcNPq1.png)

### 集合的懒加载

	/**
	 * 集合的懒加载
	 */
	@Test
	public void testCollect_lazy(){
		session = HibernateUtils.openSession();
		transaction = session.beginTransaction();
		
		Classes classes = (Classes) session.get(Classes.class, 1L);
		Set<Student> students = classes.getStudents();//不发出
		for (Student student : students) {//迭代的时候发出sql
			System.out.println(student.getName());
		}
		transaction.commit();
		session.close();
	}
	/**
	 * 更进一步的懒加载,设置extra,对于查询行数,最大值,最小值等
	 */
	@Test
	public void testlazy_extra(){
		session = HibernateUtils.openSession();
		transaction = session.beginTransaction();
		
		Classes classes = (Classes) session.get(Classes.class, 1L);
		Set<Student> students = classes.getStudents();
		System.out.println(students.size());
		transaction.commit();
		session.close();
	}

#### 演示示例：集合的延时加载

集合类型的延迟加载

* 意义最为重大，可使性能大为提高
* 修改映射文件的关联部分

	<hibernate-mapping>
	    <class name="cn.jbit.houserent.bean.District" table="district">
	        <id name="id" type="java.lang.Integer">
	            <column name="id" />
	            <generator class="native" />
	        </id>
	        <property name="name" type="java.lang.String">
	            <column name="district_name" length="50" not-null="true" />
	        </property>
	        <!--Lazy属性的默认值为true，故可以省略-->
	        <set name="streets" table="street" cascade="all" inverse="true"  lazy="true">
	            <key>
	                <column name="district_id" not-null="true" />
	            </key>
	            <one-to-many class="cn.jbit.houserent.bean.Street" />
	        </set>
	    </class>
	</hibernate-mapping>

![](https://s1.ax1x.com/2020/07/18/UcNCrR.png)

### 属性的延迟加载（懒加载）

Hibernate 3 中，引入的新特性——属性的延迟加载

适用于二进制大对象、字符串大对象以及大容量组件类型的属性

	<class name="cn.jbit.houserent.bean.House" table="house">
	    <!--省略其他配置-->
	    <property name="description" type="text" lazy="true">
	        <column name="description"/>
	    </property>
	</class>

测试：

	Query query = session.createQuery("from House as h where h.price = 2000");
	System.out.println("获取价格为2000 元的房屋信息");
	List<House> result = query.list();
	for(House house:result){
	    System.out.println("标题:"+house.getTitle());
	    System.out.println("描述:"+house.getDescription());
	}

![](https://s1.ax1x.com/2020/07/18/UcNFVx.md.png)

### 单端关联的懒加载(many2one的懒加载)

![](https://s1.ax1x.com/2020/07/17/UyTiV0.png)

No-proxy 延迟加载 默认值

Proxy是加强版的延迟加载

因为是通过多的一方加载一的一方，所以对效率影响不大，所以一般情况下用默认值即可。


根据多的一端加载一的一端，事实上只是多查了一条数据，对性能几乎没有影响，比如根据学生来查班级

### 总结

懒加载是Hibernate提供的一种优化方式。

1、延迟加载在映射文件设置，而映射文件一旦确定，不能修改了。

2、延迟加载是通过控制sql语句的发出时间来提高效率的。

所以在开发中一般是不会去设置懒加载的配置。

## 抓取策略

抓取策略解决的是如何加载Set集合中数据的问题

![](https://s1.ax1x.com/2020/07/17/UyTCbq.png)

	public class FetchTest {
	 
		private Session session;
		private Transaction transaction;
		private SessionFactory sessionFactory;
	 
		/**
		 * fetch : select情况:
		 * 先查找Classes的id,在根据id查Student
		 * 	n+1条数据
		 * 		n:classes表中的条数
		 * 		1:classes表本身
		 */
		@Test
		public void testQueryAllStudent(){
			session = HibernateUtils.openSession();
			List<Classes> classes = session.createQuery("from Classes").list();
			for (Classes classes2 : classes) {
				System.out.println(classes2.getName());
				Set<Student> students = classes2.getStudents();
				for (Student student : students) {
					System.out.println(student.getName());
				}
			}
			session.close();
			
		}
		/**
		 * fetch : subselect情况:
		 * 因为该需求含有子查询,所以fetch 使用 subselect
		 * 2条sql
		 */
		@Test
		public void testQueryAllStudent2(){
			session = HibernateUtils.openSession();
			List<Classes> classes = session.createQuery("from Classes").list();
			for (Classes classes2 : classes) {
				System.out.println(classes2.getName());
				Set<Student> students = classes2.getStudents();
				for (Student student : students) {
					System.out.println(student.getName());
				}
			}
			session.close();
			
		}
		/**
		 * select
		 *    先查询一的一方的所有的对象(Classes)，再根据每一个对象的id值查询其关联对象(Student)
		 */
		@Test
		public void testQueryClassesByCidAndStudents_Select(){
			Session session = HibernateUtils.openSession();
			Classes classes = (Classes)session.get(Classes.class, 1L);
			System.out.println(classes.getName());
			Set<Student> students = classes.getStudents();
			for (Student student : students) {
				System.out.println(student.getName());
			}
			session.close();
		}
		
		/**
		 * join  利用左外连接一条SQL语句把classes和student表全部查询出来了
		 * 
		 * 在含有子查询的需求分析中，利用join的抓取策略是不取的
		 * 1条sql
		 */
		@Test
		public void testQueryClassesByCidAndStudents_Join(){
			Session session = HibernateUtils.openSession();
			Classes classes = (Classes)session.get(Classes.class, 1L);//发出sql,故fetch为join的抓取,会导致懒加载的失效
			System.out.println(classes.getName());
			Set<Student> students = classes.getStudents();
			
			for (Student student : students) {
				System.out.println(student.getName());
			}
			session.close();
		}
	}

### 总结：

因为抓取策略的设置在映射文件中，所以一旦映射文件生成就不能改变了。

通过发出SQL语句的不同的形式加载集合，从而优化效率的。

fetch为join的抓取策略会导致懒加载的失效

在设为join时，他会直接将从表信息以join方式查询到而不是再次使用select查询，这样导致了懒加载的失效。


### 抓取策略和延迟加载的结合

#### Set集合

1、 使用fetch关键字表明Street对象属性读出后立即填充到对应的District对象（setStreet集合属性）中

* 当fetch为join时，lazy失效

2、 当fetch为select时

* 如果lazy为true/extra:当遍历集合的时候，发出加载集合的sql语句
* 如果lazy为false:当获取班级的时候，发出加载集合的sql语句

3、 当fetch为subselect时和上面的情况一致。
