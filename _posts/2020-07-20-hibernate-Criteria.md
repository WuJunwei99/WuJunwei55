---
layout: post
title: "hibernate中Criteria查询"
date: 2020-07-20 11:26:37 +0800
categories: notes
tags: hibernate
img: https://s1.ax1x.com/2020/07/17/UsU08H.png
---
Criteria查询介绍;使用；查询排序;分页

## Criteria查询介绍

### 问题

用到特定于数据库的SQL 语句，程序本身会依赖于特定的数据库

不了解SQL 语句，恐怕对使用HQL带来困难

Hibernate提供的Criteria查询帮助我们解决了这种问题

### 特点

* Criteria 查询采用面向对象方式封装查询条件，又称为对象查询
* 对SQL 语句进行封装
* 采用对象的方式来组合各种查询条件
* 由Hibernate 自动产生SQL 查询语句
* Criteria由Hibernate Session进行创建

## 使用

### 简单查询

	SessionFactory sessionFactory = new Configuration().configure()
		        .buildSessionFactory();
		Session session = sessionFactory.openSession();
		//创建Criteria对象
		Criteria criteria = session.createCriteria(User.class);
		//使用Criteria 的list()方法获得数据，list()方法返回List 实例
		List result = criteria.list();
		Iterator it = result.iterator();
		while (it.hasNext()) {
			User user = (User) it.next();
			System.out.println("用户名：" + user.getName());
		}
		session.close();
		sessionFactory.close();

![](https://s1.ax1x.com/2020/07/18/UcfxJg.png)

### 按照条件查询

	 SessionFactory sessionFactory = 
	            new Configuration().configure() .buildSessionFactory();
	    Session session = sessionFactory.openSession();
	    Criteria criteria = session.createCriteria(User.class);
		//使用add()添加查询条件
		//查询条件：name='admin'
		//返回条件实例
	    criteria.add(Restrictions.eq("name", "bob"));
	    List result = criteria.list();
	    Iterator it = result.iterator();
	    while (it.hasNext()) {
	        User user = (User) it.next();
	        System.out.println("用户名：" + user.getName());
	    }
	    session.close();
	    sessionFactory.close();

![](https://s1.ax1x.com/2020/07/18/UcfzWQ.png)

## Restrictions常用限定查询方法

![](https://s1.ax1x.com/2020/07/18/UcfXo8.png)

### 使用Restrictions

如果属性条件很多，使用Restrictions 也不方便

	SessionFactory sessionFactory = 
	        new Configuration().configure().buildSessionFactory();
	    Session session = sessionFactory.openSession();
	    Criteria criteria = session.createCriteria(House.class);
	    criteria.add(Restrictions.or(
	            Restrictions.eq("price", new Double(2300)),
	            Restrictions.like("title", "%地铁%")));
	    List result = criteria.list();
	    Iterator it = result.iterator();
	    while (it.hasNext()) {
	        House house = (House) it.next();
	        System.out.println("标题：" + house.getTitle());
	    }
	    session.close();
	    sessionFactory.close();

### 使用Example

Hibernate提供Example 的create()方法来建立Example 实例，Example 实现了Criteria 接口

Hibernate 在自动生成SQL 语句时将自动过滤掉对象的空属性，根据有非空属性值的属性生成查询条件

	House house = new House();
	house.setPrice(new Double(2000));
	house.setFloorage(new Integer(40));
	SessionFactory sessionFactory = new Configuration().configure().buildSessionFactory();
	Session session = sessionFactory.openSession();
	Criteria criteria = session.createCriteria(House.class);
	criteria.add(Example.create(house));
	List results = criteria.list();
	Iterator it = results.iterator();
	while(it.hasNext()){
	    House h= (House)it.next();
	    System.out.println("标题："+h.getTitle()+"  价格"+h.getPrice());
	}
	session.close();
	sessionFactory.close();


![](https://s1.ax1x.com/2020/07/18/UcfvFS.png)

## Criteria查询排序

Criteria 查询不仅能组合出SQL中where子句的功能，还可以组合出排序查询功能
使用org.hibernate.criterion.Order对结果进行排序

排序的方法为：

* asc()
* desc()


	SessionFactory sessionFactory = new Configuration().configure()
	        .buildSessionFactory();
	Session session = sessionFactory.openSession();
	Criteria criteria = session.createCriteria(House.class);
	//加入Order 条件并以价格降序的方式排列
	criteria.addOrder(Order.desc("price"));
	List result = criteria.list();
	Iterator it = result.iterator();
	while (it.hasNext()) {
	    House house = (House) it.next();
	    System.out.println("标题：" + house.getTitle() + "  价格"
	            + house.getPrice());
	}
	session.close();
	sessionFactory.close();

## Criteria查询实现分页

	SessionFactory sessionFactory = new Configuration().configure().buildSessionFactory();
	Session session = sessionFactory.openSession();
	Criteria criteria = session.createCriteria(House.class);
	criteria.setFirstResult(3);
	criteria.setMaxResults(2);
	List results = criteria.list();
	Iterator it = results.iterator();
	while(it.hasNext()){
		House h = (House)it.next();
		System.out.println("标题："+h.getTitle()+ "  价格："+h.getPrice());			
	}
	session.close(); 
	sessionFactory.close();

![](https://s1.ax1x.com/2020/07/18/UcfOdf.png)
	
	Hibernate: select * from ( select row_.*, rownum rownum_ from ( select this_.id as id4_3_, 
	this_.user_id as user2_4_3_, this_.type_id as type3_4_3_, this_.street_id as street4_4_3_, 
	this_.title as title4_3_, this_.price as price4_3_, this_.pubdate as pubdate4_3_, this_.floorage as
	 floorage4_3_, this_.contact as contact4_3_, user2_.id as id0_0_, user2_.name as name0_0_, 
	user2_.password as password0_0_, user2_.telephone as telephone0_0_, user2_.username as 
	username0_0_, user2_.isadmin as isadmin0_0_, type3_.id as id3_1_, type3_.type_name as type2_3_1_, 
	street4_.id as id2_2_, street4_.street_name as street2_2_2_, street4_.district_id as district3_2_2_
	 from jbit.house this_ inner join jbit.users user2_ on this_.user_id=user2_.id inner join housetype 
	type3_ on this_.type_id=type3_.id inner join street street4_ on this_.street_id=street4_.id ) row_ 
	where rownum <= ?) where rownum_ > ?

### 使用Criteria进行分页查询

实现思路：

* 创建HibernateUtil类
* 创建DAO 类
	* 使用Restrictions.like()完成SQL中like 子句的功能
	* 使用Restrictions.between()完成SQL中between关键字的功能
	* criteria.setFirstResult((pageIndex-1) * 5);
	* criteria.setMaxResults(5);
	* criteria.addOrder(Order.desc("price"));
* 创建业务类
* 创建测试类，在main方法中获取控制台输入数据，调用相应的业务类，获得数据
