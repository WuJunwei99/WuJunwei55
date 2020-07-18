---
layout: post
title: "hibernate的关联映射"
date: 2020-07-18 18:55:46 +0800
categories: notes
tags: hibernate
img: https://s1.ax1x.com/2020/07/17/UsU08H.png
---
单向多对一关联;单向一对多关联；双向一对多关联；双向一对一关联；多对多关联；一些属性的设置


### 实体关联关系

* 关联关系：通过一个对象持有另一个对象的实例
* 泛化关系：通过对象之间的继承方法来实现

类与类之间最普遍的关系就是关联关系，在UML语言中，关联是有方向的


## 单向多对一关联

在类与类之间各种各样的关系中，多对一的单向关联关系和关系数据库中的外键参照关系最匹配

单向多对一关联是最常见的单向关联关系

在租房系统中从街道到区的关联就是典型的多对一关联

![](https://s1.ax1x.com/2020/07/17/Uyhwyq.png)

#### district

	public class District implements java.io.Serializable {
	    private Long id;
	    private String name;
	    /** 默认的构造方法 */
	    public District(){
	    }
	    //省略setter/getter方法
	}

#### street

	public class Street implements java.io.Serializable {
	    private Long id;
	    private District district;
	    private String name;
	    /** 默认的构造方法 */
	    public Street(){
	    }
	    public District getDistrict(){
	        return this.district;
	    }
	    public void setDistrict(District district) {
	        this.district = district;
	    }
	    // 省略部分setter/getter方法
	}

### 映射文件

#### District.hbm.xml

	<hibernate-mapping>
	    <class name="cn.jbit.houserent.bean.District" table="district">
	        <id name="id" type="java.lang.Long">
	            <column name="id" />
	            <generator class="native"  />
	        </id>
	        <property name="name" type="java.lang.String">
	            <column name="name" length="50" not-null="true" />
	        </property>
	    </class>
	</hibernate-mapping>


#### Street.hbm.xml

	<hibernate-mapping>
	    <class name="cn.jbit.houserent.bean.Street" table="street”>
	        <id name="id" type="java.lang.Long">
	            <column name="id" />
	            <generator class="native" />
	        </id>
	        <many-to-one name="district" class="cn.jbit.houserent.bean.District">
	            <column name="district_id"  />
	        </many-to-one>
	        <property name="name" type="java.lang.String">
	            <column name="street_name" length="50" not-null="true" />
	        </property>
	    </class>
	</hibernate-mapping>

####Street.hbm.xml

与Street 对应的street 表是通过district_id 的值关联至district 表的

	<hibernate-mapping>
	    <class name="cn.jbit.houserent.bean.Street" table="street”>
	        <id name="id" type="java.lang.Long">
	            <column name="id" />
	            <generator class="native" />
	        </id>
	        <many-to-one name="district" class="cn.jbit.houserent.bean.District">
	            <column name="district_id"  />
	        </many-to-one>
	        <property name="name" type="java.lang.String">
	            <column name="street_name" length="50" not-null="true" />
	        </property>
	    </class>
	</hibernate-mapping>

### many-to-one 元素的常用属性

![](https://s1.ax1x.com/2020/07/17/UyhaSs.png)

### 配置映射文件并测试

#### hibernate.cfg.xml中指定映射文件

	<hibernate-configuration>
	    <!--省略其他配置-->
	    <mapping resource="cn/jbit/houserent/bean/District.hbm.xml" />
	    <mapping resource="cn/jbit/houserent/bean/Street.hbm.xml" />
	</hibernate-configuration>

#### 设置街道和区

	District district = new District();
	Street street1 = new Street();
	Street street2 = new Street();
	Street street3 = new Street();
	district.setName("丰台");          //设置区的名称
	street1.setName("广安路");      //设置街道名称
	street1.setDistrict(district);      //设置街道所在区
	street2.setName("大红门路");
	street2.setDistrict(district);
	street3.setName("南苑路");
	street3.setDistrict(district);

#### 添加街道和区

	SessionFactory sessionFactory = null;
	Session session = null;
	Transaction tx= null;
	try{
	    sessionFactory = 
		new Configuration().configure().buildSessionFactory();
	    session = sessionFactory.openSession();
	    tx= session.beginTransaction();
	    session.save(district);
	    session.save(street1);
	    session.save(street2);
	    session.save(street3);
	    tx.commit();
	}catch (HibernateException e) {
	    tx.rollback();
	    e.printStackTrace();
	} finally{
	    session.close();
	    sessionFactory.close();
	}

![](https://s1.ax1x.com/2020/07/17/UyhrwT.png)

## 单向一对多关联

* 由“一” 的一端加载“多” 的一端，关系由“一”的一端来维护
* 在JavaBean中是在“一”的一端中持有“多”的一端的集合
* Hibernate把这种关系反映到数据库的策略是在“多”的一端的表上加一个外键指向“一”的一端的表
* 在“一”的一端维护关系是不提倡的
	* 将“多”的一端的外键添加非空约束，导致数据不能插入
	* 插入数据效率降低

#### 街道实体类

	public class Street implements java.io.Serializable {
	    private Long id;
	    private Long district_id;
	    private String name;
	    /** 默认的构造方法 */
	    public Street(){
	    }
	    // 省略部分setter/getter方法
	    public Long getDistrict_id(){
	        return district_id;
	    }
	    public void setDistrict_id(Long district_id) {
	        this.district_id = district_id;
	    }
	}



#### 区实体类

	public class District implements java.io.Serializable {
	    private Long id;
	    private String name;
	    private Set streets = new HashSet();
	    /** 默认的构造方法 */
	    public District(){
	    }
	    // 省略部分setter/getter方法
	    public Set getStreets(){
	        return this.streets;
	    }
	    public void setStreets(Set streets) {
	        this.streets = streets;
	    }
	}


#### Street.hbm.xml

	<hibernate-mapping>
	    <class name="cn.jbit.houserent.bean.Street" table="street" >
	        <id name="id" type="java.lang.Long">
	            <column name="id" />
	            <generator class="native" />
	        </id>
	        <property name="name" type="java.lang.String">
	            <column name="name" length="50" not-null="true" />
	        </property>
	        <property name="district_id" type="java.lang.Long">
	            <column name="district_id" />
	        </property>
	    </class>
	</hibernate-mapping>

#### District.hbm.xml

使用set元素和one-to-many元素配置一对多关联

	<hibernate-mapping>
	    <class name="cn.jbit.houserent.bean.District" table="district">
	        <id name="id" type="java.lang.Long">
	            <column name="id" />
	            <generator class="native" />
	        </id>
	        <property name="name" type="java.lang.String">
	            <column name="name" length="50" not-null="true" />
	        </property>
	        <set name="streets" table="street“>
	            <key>
	                <column name="district_id"/>
	            </key>
	            <one-to-many class="cn.jbit.houserent.bean.Street" />
	        </set>
	    </class>
	</hibernate-mapping>

### set元素的常用属性

![](https://s1.ax1x.com/2020/07/17/UyhNWj.png)

### 修改测试类检查结果

#### hibernate.cfg.xml中指定映射文件
	
	<hibernate-configuration>
	    <!--省略其他配置-->
	    <mapping resource="cn/jbit/houserent/bean/District.hbm.xml" />
	    <mapping resource="cn/jbit/houserent/bean/Street.hbm.xml" />
	</hibernate-configuration>


#### 设置街道和区
	
	District district = new District();
	Street street1 = new Street();
	Street street2 = new Street();
	Street street3 = new Street();
	district.setName("海淀");
	street1.setName("中关村大街");
	street2.setName("知春路");
	street3.setName("学院路");
	district.getStreets().add(street1);
	district.getStreets().add(street2);
	district.getStreets().add(street3);

#### 测试
	SessionFactory sessionFactory = null;
	Session session = null;
	Transaction tx= null;
	try{
	    sessionFactory = 
		new Configuration().configure().buildSessionFactory();
	    session = sessionFactory.openSession();
	    tx= session.beginTransaction();
	    session.save(street1);
	    session.save(street2);
	    session.save(street3);
	    session.save(district);
	    tx.commit();
	}catch (HibernateException e) {
	    tx.rollback();
	    e.printStackTrace();
	} finally{
	    session.close();
	    sessionFactory.close();
	}


![](https://s1.ax1x.com/2020/07/17/UyhDmV.th.png)

## 双向一对多关联

* 单向一对多
* 单向多对一
* 同时配置两者就成了双向一对多关联


#### 双向关联one-many中one要注意的
	
	<set name="setStreet“  inverse="true">
	 <!-- 如果在一对多的映射关系中采用一的一端来维护关系的话会存在以下两个缺点：
	 ①如果多的一端那个外键设置为非空时，则多的一端就存不进数据；
	 ②会发出多于的Update语句，这样会影响效率。
	 所以常用对于一对多的映射关系我们在多的一端维护关系，
	 并让一的一端维护关系失效(见下面属性) 
	 inverse="false":一的一端维护关系失效(反转) ：
	 false：可以从一的一端维护关系(默认)；
	true：从一的一端维护关系失效，
	这样如果在一的一端维护关系则不会发出Update语句。
	inverse属性，只影响数据的存储，也就是持久化-->
	 	<key column="district_id“ ></key>
	 	<one-to-many class="com.bean.three.Street"/>
	 </set>

## 双向一对一关联

在类与类之间各种各样的关系中，一对一的双向关联关系

两个对象之间是一对一的关系，如Person-IdCard(人—身份证号)

![](https://s1.ax1x.com/2020/07/17/Uyh0O0.png)

### 创建实体类

#### 人

	public class IdCard implements java.io.Serializable {
	  private int id;
	  private String cardNo;
	  private Person person; //持有Person对象的引用
	/** 默认的构造方法 */
	    public IdCard(){
	    }
	    //省略setter/getter方法
	}

#### 身份证

	public class  Person implements java.io.Serializable {
	    private int id;
	    private String name;
	    private IdCard idCard;//持有IdCard对象的引用
	/** 默认的构造方法 */
	    public Person(){
	    }
	// 省略部分setter/getter方法
	}

### 人和身份证号的映射文件

#### Person.hbm.xml

	<hibernate-mapping>
	   <class name="com.bean.Person" table="t_person">
	     <id name="id" column="id">
	          <generator class="native"/>
	    </id>
	  <property name="name"/>
	  <!-- <many-to-one>:在多的一端(当前Person一端)，加入一个外键(当前为idCard)指向一的一端(当前IdCard),但多对一 关联映射字段是可以重复的，所以需要加入一个唯一条件unique="true",这样就可以此字段唯
	一了。-->
	      <many-to-one name="idCard" unique="true"/>
	   </class>
	</hibernate-mapping>


#### IdCard .hbm.xml

	<hibernate-mapping>
	       <class name="com.bean.IdCard" table="t_idcard">
	            <id name="id" column="id">
	                  <generator class="native"/>
	           </id>
	       <property name="cardNo"/>
	<!--<one-to-one>标签：告诉hibernate如何加载其关联对象
	property-ref属性：是根据哪个字段进行比较加载数据
	-->
	<one-to-one name="person" property-ref="idCard"></one-to-one>
	      </class>
	</hibernate-mapping>

### 修改测试类检查结果

	Person person1=new Person();
	person1.setId(2);
	person1.setName("张三");

	IdCard idcard1=new IdCard();
	idcard1.setId(2);
	idcard1.setCardNo("3602");
	person1.setIdCard(idcard1);

	session.save(idcard1);
	session.save(person1);

## 多对多关联

对雇员和项目需要创建两个表：employee 和project

雇员和项目间是典型的多对多关系

![](https://s1.ax1x.com/2020/07/17/Uyh6kF.png)

#### Project 一方的配置
	
	<class name="Project" table="project" >
	    <set name="members" table="r_emp_proj">
	        <key column="r_proj_id" />
	        <many-to-many class="cn.jbit.aptech.jb.entity.Employee"
	            column="r_emp_id" />
	    </set>
	</class>

#### Employee 一方的配置

	<class name="Employee" table="employee" >
	    <set name="projects" table="r_emp_proj" inverse="true">
	        <key column="r_emp_id" />
	        <many-to-many class="cn.jbit.aptech.jb.entity.Project"
	            column="r_proj_id" />
	    </set>
	</class>

#### 修改测试类

		EmployeeBean em1=new EmployeeBean(“张三”);
		EmployeeBean em2=new EmployeeBean(“李四”);
		EmployeeBean em3=new EmployeeBean(“王五”);
		ProjectBean pb1=new ProjectBean("淘宝网上商城");
		ProjectBean pb2=new ProjectBean("物流管理系统");
		ProjectBean pb3=new ProjectBean("图书管理系统");
	Set<EmployeeBean> empSet1=new HashSet<EmployeeBean>();
		empSet1.add(em1);
		empSet1.add(em3);
		pb1.setSetEmp(empSet1);
	Set<EmployeeBean> empSet2=new HashSet<EmployeeBean>();
		empSet2.add(em2);
		empSet2.add(em3);
		pb2.setSetEmp(empSet2);
		session.save(pb1);
		session.save(pb2);
		session.save(pb3);
		session.save(em1);
		session.save(em2);
		session.save(em3);

	  EmployeeBean   em1=(EmployeeBean)session.load(EmployeeBean.class, 174);
	
	   System.out.println("**"+em1.getEmployeeName());
	   Set<ProjectBean>  set=em1.getSetPro();
	   Iterator<ProjectBean> it=set.iterator();
	   while(it.hasNext()){
	   ProjectBean pb=it.next();
	   System.out.println(pb.getProjectName());
	}

## 属性

### cascade属性

当设置了cascade属性不为none时，Hibernate 会自动持久化所关联的对象

cascade 属性的设置会带来性能上的变动，需谨慎设置

	<set name="streets" table="street" cascade="all" >
	…
	</set>

Cascade属性值


![](https://s1.ax1x.com/2020/07/17/UyhsTU.png)

### inverse属性

* 术语“inverse”直译为“反转”
* 在Hibernate 中，inverse属性指定了关联关系中的方向
* 关联关系中，inverse="false" 的为主动方，由主动方负责维护关联关系
* 在一对多关联中，将one 方的inverse 设置为true，这将有助性能的改善

	<set name="streets" table="street" cascade="all" inverse="true" >
	…
	</set>


inverse指的是关联关系的控制方向，而cascade指的是层级之间的连锁操作
