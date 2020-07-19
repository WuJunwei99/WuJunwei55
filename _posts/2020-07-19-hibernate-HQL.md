---
layout: post
title: "hibernate的查询方式——HQL"
date: 2020-07-19 18:26:37 +0800
categories: notes
tags: hibernate
img: https://s1.ax1x.com/2020/07/17/UsU08H.png
---
使用HQL基本步骤；条件查询；参数绑定；排序；HQL优化；链接查询；命名查询

## 使用HQL基本步骤

### 使用HQL原因

Hibernate 支持两种主要的查询方式

HQL（Hibernate Query Languge，Hibernate 查询语言）查询

* 是一种面向对象的查询语言，其中没有表和字段的概念，只有类、对象和属性的概念
* HQL 是应用较为广泛的方式

Criteria 查询

* 又称为“对象查询”，它用面向对象的方式将构造查询的过程做了封装


#### 使用HQL 可以避免使用JDBC 查询的一些弊端

1. 不需要再编写繁复的SQL 语句，针对实体类及其属性进行查询
2. 查询结果是直接存放在List 中的对象，不需要再次封装
3. 独立于数据库，对不同的数据库根据Hibernate dialect 属性的配置自动生成不同的SQL 语句执行


### 语法

    [select/update/delete……] from Entity [where……] 
	[group by……] [having……] [order by……]

### 使用步骤

1. 得到Session
2. 编写HQL语句
3. 创建Query对象（Query接口是HQL 查询接口。它提供了各种的查询功能）
4. 执行查询，得到结果

#### 设置别名（alias）

s 是Street 的别名，通过as 关键字指定，关键字as 是可选的

![](https://s1.ax1x.com/2020/07/18/UctvPU.png)

#### 代码
	
	// 省略部分代码
	try {
	    sessionFactory =
	        new Configuration().configure().buildSessionFactory();
	    session = sessionFactory.openSession();
		//Street 并非表名，而是实体类名，也可以写成“from cn.jbit.houserent.bean.Street”
	    String hql = “from Street”; 
	    Query query = session.createQuery(hql);//获得Query对象
	    List<Street> list = query.list();//执行查询
	    for(Street street:list){
	        System.out.println("街道名称 " +
	        street.getDistrict().getName()+"区" +street.getName());
	    }
	} catch (HibernateException e) {
	    e.printStackTrace();
	} finally{
	    // 省略部分代码
	}


### where子句

	//省略代码
	String hql ="from Street as s where s.name='中关村大街'";
	Query query = session.createQuery(hql);
	List userList = query.list();
	//省略代码


#### where 子句指定限定条件

* 通过与SQL 相同的比较操作符指定条件
* 如： 
	* ==、<>、<、>、>=、<=
	* between、not between
	* in、not in
	* is、like
* 通过and、or 等逻辑连接符组合各个逻辑表达式


## 条件查询

###属性查询

#### 查询实体对象的某个属性（数据库表中的某个字段信息）
	
	String hql ="select u.password from User as u where u.name='admin'";

#### 获取实体的多个属性
	
	String hql ="select u.id,u.password from User as u where u.name='admin'";


#### 获取属性类型

	//省略代码
	String hql ="select u.id from User as u where u.name='admin'";
	Query query = session.createQuery(hql);
	List list = query.list();
	Iterator it = list.iterator();
	if(it.hasNext()){
	    System.out.println("id 的类型为 :"+ it.next().getClass());
	}
	//省略代码


#### 获取实体多个属性，返回为Object数组

	//省略代码
	String hql =“select u.id,u.name from User as u where u.name='admin'";
	Query query = session.createQuery(hql);
	List list = query.list();
	Iterator it = list.iterator();
	while(it.hasNext()){
		Object[] obj=(Object[])it.next();
		for(int i=0;i<obj.length;i++){
	                     System.out.println(obj[i]);
	               }
	}
	//省略代码

#### 获取实体的多个属性组合成一个实体

需要对应的实体类的构造方法

	//省略代码
	String hql =“select new User(u.id,u.name) from User as u where u.name='admin'";
	Query query = session.createQuery(hql);
	List<User> list = query.list();
	Iterator<User> it = list.iterator();
	while(it.hasNext()){
		User user=it.next();
		System.out.println(“ID:”+user.getId());
		System.out.println(“Name:”+user.getName());
	}
	//省略代码

## 参数绑定

### “?”占位符

* 使用“?”作占位符，可以先设定查询参数
* 通过setType()方法设置指定的参数
* 必须保证每个占位符都设置了参数值
* 必须依照“？”所设定顺序设定
* 下标从0开始，而不是使用PreparedStatement 对象时的从1开始

	//省略代码
	String hql ="select u.password from User as u where u.name = ?";
	Query query = session.createQuery(hql);
	query.setString(0, "admin"); 
	//省略代码

### 命名参数

* :name 即命名参数
* 标识了一个名为“name”的查询参数
* 根据此参数名进行参数值设定
* 不需要依照特定的顺序

	//省略代码
	String hql ="select u.password from User as u where u.name= :name";
	Query query = session.createQuery(hql);
	query.setString("name", "admin");
	//省略代码

### 封装参数

* 动态设置查询参数
* 将参数封装为一个bean
* 通过Query对象的setProperties(Object bean)实现参数的设定

#### 封装参数的bean
	
	public class QueryProperties {
	    private String title;                     //标题
	    private Double high_price;       //价格最高值
	    private Double low_price;         //价格最低值
	    private String type_id;               //房屋类型编号
	    private String street_id;             //街道编号
	    private Integer small_floorage; //面积最小值
	    private Integer big_floorage;     //面积最大值
	    //省略setter 和getter 方法
	}


#### HQL语句

	StringBuffer queryString = new StringBuffer();
	queryString.append("from House where ");
	queryString.append("(title like :title) ");
	queryString.append("and (street_id like :street_id) ");
	queryString.append("and (type_id like :type_id) ");
	queryString.append("and (price between :low_price and :high_price) ");
	queryString.append("and(floorage between :small_floorage and :big_floorage) ");

#### 执行查询
	
	// 省略部分代码
	Query query = session.createQuery(queryString.toString());
	query.setProperties(qp);
	List<House> list = query.list();
	// 省略部分代码


## 聚合函数和排序

### count( )

统计函数

	select count(house) from House h where h.user_id = '1010'

### max( )和min( )

最大值和最小值函数

    select max(h.price),min(h.price) from House h

### avg( )和sum( )

平均值和求和函数
	
	select avg(h.price),sum(h.floorage) from House h where h.user_id= '1000'

### 排序

与SQL类似，HQL 通过order by 子句实现对查询结果的排序

默认情况下按升序顺序排序

排序策略（asc 升序、desc 降序）
	
	from House house order by house.price
	
	from House house order by house.price desc
	
	from House house order by house.price , house.floorage

## 分组、分页、子查询

### 分组

通过group by 子句实现

并使用having 子句对group by 返回的结果集进行筛选
	
	select sum(house.floorage) from House house group by house.street_id having sum(house.floorage) > 1000

### 分页

Query对象提供了简便的分页方法

* setFirstResult(int firstResult)方法
	* 设置第一条记录的位置
* setMaxResults(int maxResults)方法
	* 设置最大返回的记录条数

	// 省略部分代码
	query.setFirstResult((pageIndex-1)*pageSize);
	query.setMaxResults(pageSize);
	List result=query.list();

### 子查询

子查询是SQL 中非常重要的功能

HQL同样支持此机制

Hibernate子查询必须用圆括号包围且必须出现在where子句中

怎样查询价格大于street_id = ‘1000’的房屋的平均价格的房屋信息？
	
	select * from House as h1 where h1.price > ( select avg(h2.price) from House h2 where h2.street_id = '1000')

## HQL性能优化

### HQL优化

#### 避免or操作

* where 子句包含or 操作，执行时不使用索引
* 可以使用in条件来替换

![](https://s1.ax1x.com/2020/07/18/Uctz24.png)

#### 避免使用not

* where 子句包含not 关键字，执行时该字段的索引失效
* 使用比较运算符替换not

![](https://s1.ax1x.com/2020/07/18/UctxGF.png)

#### 避免like的特殊形式

查询时，尽可能少使用like

#### 避免having子句

尽可能在where 子句中指定条件

#### 避免使用distinct

在不要求或允许冗余时，应避免使用distinct


### 使用延时加载

该部分内容可参考我的笔记《hibernate的懒加载与抓取策略》

### list()方法和iterate()方法

使用list()方法获取查询结果，每次发出一条查询语句，获取全部数据

使用iterate()方法获取查询结果，先发出一条SQL 语句用来查询满足条件数据的id，然后依次按这些id 查询记录，也就是要执行N+1 条SQL 语句（N 为符合条件的记录数）

	sessionFactory = new Configuration().configure().buildSessionFactory();
	session = sessionFactory.openSession();
	Query query = session.createQuery("from House as h ");
	System.out.println("使用iterate()方法查询数据");
	Iterator<House> it = query.iterate();
	while(it.hasNext()){
		House house = it.next();
		System.out.println("标题:"+house.getTitle());
	}
	System.out.println("-------------------");	
	System.out.println("使用list()方法查询数据");		
	List<House> result = query.list();
	for(House house:result){ 
		System.out.println("标题："+house.getTitle());
	}


![](https://s1.ax1x.com/2020/07/18/UcNAIK.png)

list()方法将不会在缓存中读取数据，它总是一次性的从数据库中直接查询所有符合条件的数据，同时将获取的数据写入缓存

iterate()方法则是获取了符合条件的数据的id 后，首先根据id 在缓存中寻找符合条件的数据，若缓存中无符合条件的数据，再到数据库中查询


#### 演示示例：即时加载iterate()方法

	sessionFactory = new Configuration().configure().buildSessionFactory();
	session = sessionFactory.openSession();
	Query query = session.createQuery("from House as h ");
	System.out.println("使用iterate()方法查询数据");
	Iterator it1 = query.iterate();
	while(it1.hasNext()){
	    House house = (House)it1.next();
	    System.out.println("标题:"+house.getTitle());
	}
	System.out.println("----------------------");
	System.out.println("再一次执行iterate()方法");
	Iterator it2 = query.iterate();
	while(it2.hasNext()){
	    House house = (House)it2.next();
	    System.out.println("标题:"+house.getTitle());
	}

![](https://s1.ax1x.com/2020/07/18/UcNZGD.png)

#### 即时加载list()方法

	sessionFactory = new Configuration().configure().buildSessionFactory();
	session = sessionFactory.openSession();
	Query query = session.createQuery("from House as h ");
	System.out.println("使用list()方法查询数据");
	List result1 = query.list();
	for(int i=0;i<result1.size();i++){ 
	    House house = (House)result1.get(i); 
	    System.out.println("标题："+house.getTitle());
	}
	System.out.println("----------------------");
	System.out.println("再一次执行list()方法");
	List result2 = query.list();
	for(int i=0;i<result2.size();i++){ 
	    House house = (House)result2.get(i); 
	    System.out.println("标题："+house.getTitle());
	}

![](https://s1.ax1x.com/2020/07/18/UcNVPO.png)

## HQL 链接查询

**内联接：inner join**

* 最典型、最常用的联接查询
* 两个表存在主外键关系时通常会使用内联接查询

**外联接**

* 左外联接：left join 或 left outer join
* 右外联接：right join 或right outer join
* 完整外联接：full join或 full outer join

#### HQL支持的联接类型

![](https://s1.ax1x.com/2020/07/18/UcNka6.png)

### 内联接

Hibernate的内联接语法如下

    from Entity inner join [fetch] Entity.property

忽略fetch 关键字，我们得到的结果集中，每行数据都是一个Object 数组

	sessionFactory = new Configuration().configure().buildSessionFactory();
	session = sessionFactory.openSession();
	Query query = session.createQuery("from District d inner join d.streets s");
	List result = query.list();
	Iterator it = result.iterator();
	while(it.hasNext()){
	    Object[] results = (Object[]) it.next() ;
	    System.out.println("数据的类型：");
	    for (int i=0;i<results.length;i++){
	        System.out.println(results[i]);
	    }
	}

![](https://s1.ax1x.com/2020/07/18/UcNeRe.png)


### 左外联接

Hibernate的左外联接语法如下

    from Entity left join [fetch] Entity.property


	sessionFactory = new Configuration().configure().buildSessionFactory();
	session = sessionFactory.openSession();
	Query query =
	    session.createQuery("from District d left join fetch d.streets s");
	List result = query.list();

等价的sql语句：

	Hibernate: select district0_.id as id1_0_, streets1_.id as id2_1_, district0_.district_name as district2_1_0_, streets1_.street_name as street2_2_1_, streets1_.district_id as district3_2_1_, treets1_.district_id as district3_0__, streets1_.id as id0__ from district district0_ left outer join street streets1_ on district0_.id=streets1_.district_id

### 右外联接

Hibernate的右外联接语法如下

    from Entity right join [fetch] Entity.property

	sessionFactory = new Configuration().configure().buildSessionFactory();
	session = sessionFactory.openSession();
	Query query =
	    session.createQuery("from District d right join fetch d.streets s");
	List result = query.list();

等价的sql语句：

	Hibernate: select district0_.id as id1_0_, streets1_.id as id2_1_, district0_.district_name as district2_1_0_, streets1_.street_name as street2_2_1_, streets1_.district_id as district3_2_1_, treets1_.district_id as district3_0__, streets1_.id as id0__ from district district0_ right outer join street streets1_ on district0_.id=streets1_.district_id

### fetch

* 使用fetch关键字表明Street对象属性读出后立即填充到对应的District对象（setStreet集合属性）中 
* 如果不使用fetch 结果集中，每个条目都是一个Object数组 
* 假若使用iterate()来调用查询，请注意fetch构造是不能使用的 
* fetch也不应该与setMaxResults() 或setFirstResult()共用
* fetch还不能与独立的 with条件一起使用 

## 命名查询
	
* <query>元素用于定义一个HQL 查询语句，它和<class>元素并列
* 以<![CDATA[HQL]]>方式保存HQL 语句,标明是纯文本的，没有这个的话 < > & 字符是不能直接存入XML的，需要转义，而用这个标记则不需要转义而将这些符号存入XML文档。可以避免未预料的特殊符号导致XML解析出错 
* 在程序中通过Session 对象的getNamedQuery()方法获取该查询语句


	<hibernate-mapping>
	    <class name="cn.jbit.houserent.bean.User" table="users">
	        <!--省略其他配置-->
	    </class>
	    <query name="loginUser">
	        <![CDATA[
	            from User u where u.name =:name and u.password =:password
	        ]]>
	    </query>
	</hibernate-mapping>

### 本地SQL查询

Hibernate对本地SQL查询提供了内置的支持

* Session的createSQLQuery()方法返回SQLQuery 对象
* SQLQuery接口继承了Query接口
* SQLQuery接口的addEntity()方法将查询结果集中的关系数据映射为对象
* 通过命名查询实现本地SQL查询
	* 使用<sql-query>元素定义本地SQL 查询语句
	* 与<class>元素并列
	* 以<![CDATA[SQL]]>方式保存SQL 语句
	* getNamedQuery()方法获取该查询语句

### 命名查询的使用

实现思路：

* 封装查询参数
* 修改映射文件
* 使用session对象的getNamedQuery()方法

####  SQL查询

		   //编写SQL语句
		    String sqlString = "select {s.*} from student s where s.name like '马军'";
		    //以SQL语句创建SQLQuery对象
		    List list = session.createSQLQuery(sqlString)
		                 //将查询到的记录与特定实体关联起来
		                 .addEntity("s",Student.class)
		                 //返回全部的记录集
		                 .list();

		//SQL查询转换成标量值可以使用addScalar方法，
		      String sqlString = "select max(cat.weight) as maxWeight from cats cat";
		      Double max = (Double) session.createSQLQuery(sqlString)
		        .addScalar("maxWeight", Hibernate.DOUBLE);
		        .uniqueResult();
		// uniqueResult()唯一结果

####  命名SQL查询

	<sql-query name="sqlquery">
	<!-- 将p别名转换为Person实体 -->
	<return alias="p" class="Person" />
	<!-- 将e别名转换成Event实体 -->
	<return alias="e" class="MyEvent" />
	<!-- 指定将person_inf表的name属性列作为标量值返回-->
	<return-scalar column="p.name" type="string"/> 
	select p.*,e.* from person_inf as p,event_inf as e where p.person_id = e.person_id and p.age=:age
	</sql-query> 


 调用命名查询，直接返回结果 

	List list = session.getNamedQuery("sqlquery") .setInteger("age", 30).list();
	  
