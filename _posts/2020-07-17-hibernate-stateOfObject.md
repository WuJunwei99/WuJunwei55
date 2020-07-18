---
layout: post
title: "hibernate中对象的状态"
date: 2020-07-17 19:41:25 +0800
categories: notes
tags: hibernate
img: https://s1.ax1x.com/2020/07/17/UsU08H.png
---
三种对象的状态;相互转换;Hibernate对象的副本(快照)

## 三种对象的状态

临时、游离和持久化，三种状态转化的方法都是通过session来调用的

**瞬时状态(Transient)**：
刚用new语句创建，还没有被持久化，且不处于Session的缓存中
**持久状态(Persistent)**：
已经被持久化，且加入到Session的缓存中
**游离状态(Detached)**：
已经被持久化，但不再处于Session的缓存中


### 持久化

通俗的讲，就是瞬时数据（比如内存中的数据，是不能永久保存的）持久化为持久数据（比如持久化至数据库中，能够长久保存)

![](https://s1.ax1x.com/2020/07/17/UsrMes.jpg)


## 相互转换

**session方法**

1. session.save():该方法可以把一个对象从临时装填转换成持久化状态
2. session.get():从数据库中根据主键提取出一个对象，该对象就是一个持久化状态的对象
3. session.update():把一个对象变为持久化对象
4. session.evict():把一个持久化状态的对象变为脱管状态
5. session.clear():把所有Hibernate中的持久化状态的对象变为脱管状态的对象


![](https://s1.ax1x.com/2020/07/17/Usr3F0.md.png)


## 测试session的各个方法
	
	public class SessionfatoryTest {
		SessionFactory factory;
		Session session ;
		Transaction transaction;
		@Before
		public void init(){
			factory =  HibernateUtils.getSessionFactory();
			session = factory.openSession();
			transaction = session.beginTransaction();
		}
		
		/**
		 * 把一个临时对象变为持久化对象
		 */
		@Test
		public void testSave(){
			User user = new User();
			user.setAge(11);
			user.setName("aaaa");
			session.save(user);
		}
		
		@Test
		public void testGet(){
			User user = (User) session.get(User.class, 1);
			System.out.println(user);
		}
		
		@Test
		public void testUpdate(){
			User user = (User) session.get(User.class, 1);
			user.setAge(102);
			//session.update(user);
			//因为user本身就是持久化对象,所以即使不写session.update(),在
			//transaction.commit()时也会发出SQL语句,因为该对象的数据与
			//hibernate中的该对象的快照(副本)不一致,在flush()时,就会发出sql
		}
		
		@Test
		public void testUpdate2(){
			
			User user = (User) session.get(User.class, 1);
			session.evict(user);//脱管状态
			user.setAge(100);
			session.update(user);//变为持久化状态
		}
	 
		@After
		public void destory(){
			transaction.commit();
			session.close();
			factory.close();
		}
	}

## Hibernate对象的副本(快照)

![](https://s1.ax1x.com/2020/07/17/UsrQwn.md.png)

每次进行transaction.commit()时,都会进行flush操作,进而进行对象与副本的数据对比,决定是否发送sql语句，比如取出的user的age是10,又改为10,user.setAge(10);那么这样是不会发送sql语句的。

### session.flush()

实际上sql的发送时刻,发生在session.flsuh()，在transaction.commit()之前,如果有session.flush()操作,就会发送sql语句,而不再等待transaction.commit().

![](https://s1.ax1x.com/2020/07/17/Usrloq.png)

### session.flush()做了哪些事？

在hibernate内部，回去检查所有的持久化对象

1.如果持久化对象是由临时状态转换过来的就发出 insert 语句

2.如果持久化对象是由 get 等方法得到的

3.再次查看副本，如果与副本对照后，一致就不发送sql，如果不一致，就发生update语句

这也就是为什么，不使用session.update() 同样可以发送 update 语句
