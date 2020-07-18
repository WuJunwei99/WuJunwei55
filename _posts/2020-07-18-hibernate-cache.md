---
layout: post
title: "hibernate的缓存机制"
date: 2020-07-18 12:28:36 +0800
categories: notes
tags: hibernate
img: https://s1.ax1x.com/2020/07/17/UsU08H.png
---
缓存；hibernate的一级缓存；二级缓存；查询缓存


## 缓存

缓存说白了，就是应用程序向数据库要数据，然后把一些数据，临时的放在了内存的区域中，第二次再要数据的时候，直接从内存中拿即可。

### 缓存需要解决的事情

1. 能够把数据放入缓存
2. 能够把数据从缓存中取出来
3. 如果缓存中的数据发生变化，需要同步到数据库中
4. 把数据库的数据同步到缓存
5. hits命中率低的对象应该及时从缓存中移除

### 分布式缓存

应用程序运行在服务器上，并发访问时，服务器压力过大，分布式缓存就是来分担服务器压力的。

分布式缓存之间的数据是同步的。(比如购物车中的数据都是存在session中)一旦某个服务器挂了，那么操作的数据在其他的服务器的缓存中还可以继续取出来。所以在Tomcat集群的时候，session是先存在了分布式缓存中，在 tomcat 内部集成了memory cache的分布式缓存，就能自动的把 session 同步。

面试，集群时解决 session 问题：就是利用分布式缓存来处理，tomcat可以与memory cache无缝的集成，在tomcat中配置即可。程序员不需要任何干涉。


比较流行的缓存

小型应用：oscache，**ehcache**

分布式：memory cache，**redis**，hbase

## hibernate的一级缓存

Hibernate的一级缓存,也称为 session 级别的缓存，其生命周期与 session 的生命周期保持一致

![](https://s1.ax1x.com/2020/07/17/Uyec8O.md.png)

持久化状态的对象，就是进入了一级缓存中，换句话说，如果一个对象是一个持久化对象，那么这个对象一定在一级缓存中。

### Session的操作

session.get():可以把对象放入到一级缓存中，也可以从一级缓存中把对象提取出来(第一次调用放，以后取)

session.save():可以把一个对象放入到一级缓存中

session.evit():可以把一个对象从缓存中清空

session.update():可以把一个对象放入到一级缓存中

session.clear():清空一级缓存中所有的数据

session.close():一级缓存的生命周期结束


### 测试
	
	private Session session;
		private Transaction transaction;
		
		@Before
		public void init(){
			session = HibernateUtils.openSession();
			transaction = session.beginTransaction();
		}
		
		@Test
		public void testGet(){
			User user = (User) session.get(User.class, 1);//发sql,把对象放入到一级缓存中
			User user2 = (User) session.get(User.class, 1);//从一级缓存中直接取,不发sql
			//hibernate提供一个统计机制。获取缓存中实体的个数
			int count = session.getStatistics().getEntityCount();
			System.out.println(count);//1,所以get方法把对象放入到缓存
			transaction.commit();
			session.close();
		}
		
		
		@Test
		public void testSave(){
			User user = new User();
			user.setAge(250);
			user.setName("heh");
			session.save(user);
			int count = session.getStatistics().getEntityCount();
			System.out.println(count);//1,所以save方法把对象放入到缓存
			transaction.commit();
			session.close();
		}
		
		@Test
		public void testEvict(){
			User user = (User) session.get(User.class, 1);
			session.evict(user);
			int count = session.getStatistics().getEntityCount();
			System.out.println(count);//0,evit方法把get放入到缓存中的对象,清空了
			transaction.commit();
			session.close();
		}
		
		@Test
		public void testUpdate(){
			User user = (User) session.get(User.class, 1);
			session.evict(user);
			/**
			 * 此处注意,evict方法之后,执行update会发送sql,即使对象不做任何修改
			 * 此处,可以加深session.flush(),进行的对照副本的操作的理解
			 * 注释掉evict,就不会发送update语句
			 */
			session.update(user);
			int count = session.getStatistics().getEntityCount();
			
			System.out.println(count);//1,update方法把对象放入到缓存
			transaction.commit();
			session.close();
		}
		
	 
		@Test
		public void testClear(){
			User user = (User) session.get(User.class, 1);
			session.clear();
			int count = session.getStatistics().getEntityCount();
			System.out.println(count);//0,清空所有
			transaction.commit();
			session.close();
		}

![](https://s1.ax1x.com/2020/07/17/Uyet8U.md.png)

### 一级缓存的真正意义


从数据库中取出一个班级的所有学生信息，对学生信息进行修改,所有改的操作，都只是针对一级缓存中的数据，只有在 session 的 flush 后，才会与数据库交互。所以不论改多少人的信息，都只是 session.flush之后才会与数据库交互一次。这样就可以提供效率。

而不是get一次,发送一次sql语句，再次get就不发送sql语句，取数据get一次就行。

![](https://s1.ax1x.com/2020/07/17/Uyelbn.md.png)

### 一级缓存的内存结构

![](https://s1.ax1x.com/2020/07/17/Uye8U0.png)

### 内存结构图

![](https://s1.ax1x.com/2020/07/17/Uye3Eq.md.png)

## 二级缓存

### 应用场合

比如，在12306购票时，需要选择出发地与目的地，如果每点一次都与数据库交互一次，这就很不合适，这些地点数据在相当长的一段时间内是不会发生变化的(山东省在相当长的时间内还叫山东省)，所以应该缓存起来,没必要每次都与数据库交互,而且该类数据安全性也不是很高。

#### 适合二级缓存的数据

在现代软件开发中，确实存在一类数据没有什么私有性，为公开的数据，数据基本上不发生变化，该数据保密性不是很强，但又会经常被用到(比如火车票上的出发地与目的地数据)。

注意:如果一个数据一直在改变，不适合用缓存。

流式数据：数据时时刻刻在变的数据，比如手机应用获取手机所在地,后台的推送系统，发出一些信息，比如短信会提示你某天夜间流量超过多少，建议购买夜间流量。而这类数据适合使用strom来处理 。

### 生命周期

二级缓存为sessionFactory级别的缓存，其生命周期和sessionFactory是一致的,Hibernate启动后就有了。

#### 二级缓存在Hibernate中的位置

![](https://s1.ax1x.com/2020/07/17/UyeYCT.png)

所以Hibernate内部并没有实现二级缓存,而是应用第三方插件来实现二级缓存的。

### 二级缓存的设置

利用的是ehcache实现的二级缓存

1. 添加ehcache所需的jar包
2. 在hibernate的配置文件中进行配置

![](https://s1.ax1x.com/2020/07/17/UyeN2F.md.png)

![](https://s1.ax1x.com/2020/07/17/UyedKJ.png)

### 二级缓存的操作

#### 哪些方法可以把对象放入到二级缓存中？

get方法,list方法可以把一个或者一些对象放入到二级缓存中

#### 哪些方法可以把对象从二级缓存中提取出来

get方法,iterator方法可以提取

注意：用的时候一定要小心使用各方法，取数据时，如果把list用成iterate会造成效率的极其降低


### 二级缓存的存储策略

![](https://s1.ax1x.com/2020/07/17/Uyewr9.png)

read-only：对象只要加载到二级缓存以后，就只能读取，不能修改。

read-write：对二级缓存中的对象能够进行读和写的操作

注意:一般都设置为read-only

#### 缓存得到磁盘

如果一个系统的权限特别大，这就不适合长时间的放入到二级缓存中，会导致占用内存逐渐变大，查询效率逐渐降低，这种情况可以二级缓存移到磁盘上，但是在现代开发如果需要缓存较多，一般都是使用分布式缓存。


### 代码及测试

#### hibernate.cfg.xml
	
	<?xml version='1.0' encoding='utf-8'?>
	<!DOCTYPE hibernate-configuration PUBLIC
	        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
	        "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
	<hibernate-configuration>
	 
	<session-factory>
	 
		<property name="hibernate.dialect"><![CDATA[org.hibernate.dialect.MySQLDialect]]></property>
		<property name="hibernate.connection.driver_class"><![CDATA[com.mysql.jdbc.Driver]]></property>
		<property name="hibernate.connection.url"><![CDATA[jdbc:mysql:///hibernate1]]></property>
		<property name="hibernate.connection.username"><![CDATA[root]]></property>
		<property name="hibernate.connection.password"><![CDATA[qiaolezi]]></property>
	 
		<property name="hibernate.c3p0.max_size">20</property>
		<property name="hibernate.c3p0.min_size">10</property>
		<property name="hibernate.c3p0.max_statements">10</property>
		<property name="hibernate.c3p0.acquire_increment">5</property>
		<property name="hibernate.c3p0.timeout">3000</property>
		
		<!-- 二级缓存供应商 -->
		<property name="cache.provider_class">org.hibernate.cache.EhCacheProvider</property>
		<!-- 开启二级缓存 -->
		<property name="cache.use_second_level_cache">true</property>
		<!-- 开启二级缓存的统计机制 -->
		<property name="generate_statistics">true</property>
		
		<property name="hbm2ddl.auto">update</property>
		<property name="show_sql">true</property>
		<property name="current_session_context_class">thread</property>
		<property name="format_sql">true</property>
		
		
		<mapping resource="cn/cil/domain/Classes.hbm.xml" />
		<mapping resource="cn/cil/domain/Student.hbm.xml" />
	 
	</session-factory>
	</hibernate-configuration>

#### Classes

	public class Classes implements Serializable{
	 
		private Long cid;
		private String name;
		private Set<Student> students;
	}

#### Classes.hbm.xml

	<class name="Classes" table="CLASSES">
			<cache usage="read-write"/><!-- 只读 -->
			<id name="cid">
				<generator class="native"></generator>
			</id>
			<property name="name"></property>
			<set name="students" cascade="save-update" inverse="true">
				<cache usage="read-only"/><!-- 设置集合的二级缓存 -->
			<!-- key:外键,告诉hibernate通过cid来建立classes与student的关系 -->
				<key column="cid"></key>
				<one-to-many class="Student"/>
			</set>
		</class>

#### student

	public class Student implements Serializable{
	 
		private Long sid;
		private String name;
		private Classes classes;
	}

#### Student.hbm.xml

	<class name="Student" table="STUDENT">
			<id name="sid">
				<generator class="native"></generator>
			</id>
			<property name="name"></property>
		
			<many-to-one cascade="save-update" name="classes" column="cid" class="Classes">
			</many-to-one>
		</class>

#### 测试类

	public class SessionFactoryCacheTest {
	 
		private Session session;
		private Transaction transaction;
		private SessionFactory sessionFactory;
		
		/**
		 * 测试get,在session关闭后,再次请求,不发出sql
		 * 该方法在取数据时,先从一级缓存中找,如果二级缓存开启了,接下来就会从二级缓存中,都没有,则查数据库
		 * 在DefaultLoadEventListener.class中可以清晰的看到
		 * 440行: 从一级缓存,459 行: 从二级缓存,477行: 从数据库
		 * 
		 * get可以把对象放入二级缓存,也可以从二级缓存中取数据
		 */
		@Test
		public void testGet(){
			
			sessionFactory = HibernateUtils.getSessionFactory();
			 session = HibernateUtils.openSession();
			Classes classes = (Classes) session.get(Classes.class, 1L);
			System.out.println(sessionFactory.getStatistics()
					.getEntityLoadCount());//1
			session.close();
			session = HibernateUtils.openSession();
		    classes = (Classes) session.get(Classes.class, 1L);//不发出sql
			session.close();
		}
		/**
		 * Hibernate提供了二级缓存的统计机制
		 * save不能把对象存入二级缓存
		 */
		@Test
		public void testSave(){
			sessionFactory = HibernateUtils.getSessionFactory();
			session = HibernateUtils.openSession();
			transaction = session.beginTransaction();
			Classes classes = new Classes();
			classes.setName("a");
			session.save(classes);
			System.out.println(sessionFactory.getStatistics()
					.getEntityLoadCount());//0
			transaction.commit();
			session.close();
		}
		
		/**
		 * session.update不操作二级缓存
		 */
		@Test
		public void testUpdate(){
			sessionFactory = HibernateUtils.getSessionFactory();
			session = sessionFactory.openSession();
			Transaction transaction = session.beginTransaction();
			Classes classes = new Classes();
			classes.setCid(3L);
			classes.setName("afds");
			session.update(classes);
			System.out.println(sessionFactory.getStatistics()
					.getEntityInsertCount());//0
			transaction.commit();//会报错,但是不影响,设置为<cache usage="read-write">,如果集合也设置了二级缓存,那集合也要设置为同样
			session.close();
		}
		
		/**
		 * HQL中的list,可以把对象放入到二级缓存,但是不能从二级缓存取数据
		 */
		@Test
		public void testQueryList(){
			sessionFactory = HibernateUtils.getSessionFactory();
			session = sessionFactory.openSession();
			
			session.createQuery("from Classes").list();
			System.out.println(sessionFactory.getStatistics()
					.getEntityLoadCount());//不是0
			session.close();
			session = sessionFactory.openSession();
			session.createQuery("from Classes").list();//发出sql,因为getEntityLoadCount不为0,所以list()可以把对象放入二级缓存,但是不能从二级缓存中取对象
			session.close();
		}
		
		/**
		 *  iterator,可以从二级缓存中取数据
		 *  iterate方法的查询策略:
		 *  	先查找该表中所有的id值
		 *  	再根据id值从二级缓存中查找对象,如果有,则利用二级缓存,
		 *  						     如果没有,则根据id查询所有属性的值
		 */
		
		@Test
		public void testQueryIterate(){
			sessionFactory = HibernateUtils.getSessionFactory();
			session = sessionFactory.openSession();
			session.createQuery("from Classes").list();
			
			System.out.println(sessionFactory.getStatistics()
					.getEntityLoadCount());//不是0
			session.close();
			session = sessionFactory.openSession();
			Iterator<Classes> iterator = session.createQuery("from Classes").iterate();
			while (iterator.hasNext()) {
				Classes classes = (Classes) iterator.next();
				System.out.println(classes.getName());
			}
			
			session.close();
		}
		
		/**
		 * 集合的二级缓存,需要在hbm.xml文件中进行相应的设置
		 * 集合有一级缓存也有二级缓存
		 */
		@Test
		public void testCollection(){
			sessionFactory = HibernateUtils.getSessionFactory();
			session = sessionFactory.openSession();
			
			Classes classes = (Classes) session.get(Classes.class, 1L);
			Set<Student> students = classes.getStudents();
			for(Student student : students){
				System.out.println(student.getName());
			}
			System.out.println(sessionFactory.getStatistics()
					.getCollectionLoadCount());//1
			session.close();
		}
		
		
		/**
		 * 二级缓存 到 磁盘
		 */
		@Test
		public void testCacheDisk(){
			sessionFactory = HibernateUtils.getSessionFactory();
			session = sessionFactory.openSession();
			
			session.createQuery("from Classes").list();
			try {
				Thread.sleep(2000L);//停一会,否则写不进去数据
			} catch (Exception e) {
				
				e.printStackTrace();
			}
			session.close();
		}
	}

## 查询缓存

一级缓存解决在一次查询中，只与数据库交互一次

二级缓存解决一些常用的、公开的数据存放起来,方便使用，那查询缓存呢？

### 查询缓存的原理

一级缓存和二级缓存都是对象的缓存。对象缓存就是把该对象对应的数据库表中的所有的字段全部查询出来，而这种查询通常会让小效率降低，比如，表中字段很多，但程序中只需要几个字段，而这样的情况，使用一级缓存或二级缓存会让效率降低。

查询缓存也叫数据缓存，也就是说内存中需要多少数据，就把多少数据放入到查询缓存，这样就大大提高了查询效率。所以查询缓存解决了一张表中部分字段查询的问题。

#### 生命周期

只要数据放入到查询缓存中，该缓存会一直存在，直到缓存中的数据被修改了，该缓存的生命周期结束.

![](https://s1.ax1x.com/2020/07/17/Uymyes.jpg)

两者构成了查询缓存。


#### 操作

![](https://s1.ax1x.com/2020/07/17/UymrLj.png)

#### 思考：

Hibernate是如判断查询缓存中的值与数据库中的值是否一样呢？

一样就保存，不一样就干掉。实际上 Hibernate 利用的时间戳缓存来判断数据的有效性，时间戳中记录了查询缓存从创建到清除的变化日志，Hibernate 利用该日志来判断缓存中的数据是否更新

![](https://s1.ax1x.com/2020/07/17/UymDyQ.md.png)

### 查询缓存的应用

**开启二级缓存**

	public class QueryCacheTest {
	 
		private Session session;
		private Transaction transaction;
		private SessionFactory sessionFactory;
		
		@Test
		public void testList1(){
			
			session = HibernateUtils.openSession();
	 
			Query query = session.createQuery("from Classes");
			query.setCacheable(true);//query要使用查询缓存
			query.list();//把数据放入查询缓存
			session.close();
			
			session = HibernateUtils.openSession();
			query = session.createQuery("from Classes");
			query.setCacheable(true);
			query.list();//不发出sql
			session.close();
		} 
		
		
		/**
		 * "select name from Classes" 查name
		 * 		查询出来的数据能够放入到查询缓存中
		 *      但是不能放入到二级缓存中，因为不是对象
		 */
		@Test
		public void testList_2(){
			session = HibernateUtils.openSession();
			Query query = session.createQuery("select name from Classes");
			query.setCacheable(true);
			query.list();
			System.out.println(HibernateUtils.getSessionFactory()
					.getStatistics().getEntityLoadCount());//0,因为name就是一个数据,不是对象,所以不能放到二级缓存中
			session.close();
			
			session = HibernateUtils.openSession();
			query = session.createQuery("select name from Classes");
			query.setCacheable(true);
			query.list();
			session.close();
		}
		
		/**
		 * 如果两个hql一样，则可以利用查询缓存,
		 * 如果不一样，哪怕有一点不一样，就不能够利用了。
		 */
		@Test
		public void testList_3(){
			session = HibernateUtils.openSession();
			Query query = session.createQuery("select name from Classes");
			query.setCacheable(true);
			query.list();
			session.close();
			
			session = HibernateUtils.openSession();
			query = session.createQuery("select name from Classes where cid=1");
			query.setCacheable(true);
			query.list();//发出sql语句,按理说第二条hql可以从第一条中取,但是查询缓存不行,其命中率特别低
			session.close();
		}
		
		
		
		/**
		 * 先把一些数据放入到查询缓存中，修改一些数据，看生命周期
		 */
		@Test
		public void testActiveTime(){
			/**
			 * 把name放入到了查询缓存中
			 */
			Session session = HibernateUtils.openSession();
			Query query = session.createQuery("select name from Classes");
			query.setCacheable(true);
			query.list();
			session.close();
			
			/**
			 * 修改name属性的值
			 *   修改了查询缓存的时间戳缓存，从而知道了该数据已经被修改了,查询缓存中的数据就被清空了
			 */
			session = HibernateUtils.openSession();
			transaction = session.beginTransaction();
			Classes classes = (Classes)session.get(Classes.class, 1L);
			classes.setName("123213456");
			transaction.commit();
			session.close();
			
			/**
			 * 再次查询name属性的值,查询缓存中name属性的值清空了
			 */
			session = HibernateUtils.openSession();
			query = session.createQuery("select name from Classes");
			query.setCacheable(true);
			query.list();//发出sql
			session.close();
		}
	}