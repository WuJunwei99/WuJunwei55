---
layout: post
title: "struts2的值栈和OGNL"
date: 2020-07-16 18:46:55 +0800
categories: notes
tags: struts2
img: https://s1.ax1x.com/2020/07/16/UDF2WR.png
---
值栈;OGNL；常用的OGNL访问操作；ActionContext和ServletActionContext



## 值栈

### 值栈的介绍

简单说：就是对应每一个请求对象的轻量级的内存数据中心。

Struts2引入值栈最大的好处就是：在大多数情况下，用户根本无须关心值栈，不管它在哪里，不用管它里面有什么，只需要去获取自己需要的数据就可以了。

### 值栈的作用

简单说：就是能够线程安全的为每一个请求提供公共的数据式服务。

### 值栈包括

值栈包含Map栈和对象栈，值栈通过ActionContext的getValueStack()方法来获取值栈，但是在通常情况下，向valuestack中压入值都是由Struts2去完成的，而访问valuestack多是由OGNL去获取，实际向valuestack压入值的情况并不多。

### 应用

#### 定义和配置Action

	public class UserAction extends ActionSupport {
		private User User;
		public String execute() throws Exception {
			return Action.SUCCESS;
		}
		省略get和set方法
	}

xml：

	<struts>
		<package name="Struts 2" extends="struts-default">
	 <action name=“login" class="com.lesson.action.UserAction">
		<result name="success">login_Success.jsp</result>
		<result name="input">login.jsp</result>
		</action>		
		</package>
	</struts>


创建页面

* 登录页面login.jsp

	<html>
	 <head><title>电话页面</title></head>
		<body>
			<h2>信息录入</h2>
			<br/>
		<s:form action=“login" method="post" >
		<s:textfield name="user.userName" label="用户名” />
		<s:textfield name=“user.pwd” label=“密码” />  <s:submit></s:submit>
		    </s:form>		
			</body>
		</html>


* 成功页面login_Success.jsp 

		<html>
			<head>	
				<title>成功</title>	
			</head>
			<body>
		 调试：<s:debug></s:debug>
		获取值栈中的username属性：
		<s:property value="user.userName"/> 
		</body>
		</html>

当输入用户名，密码后和没有输入任何信息直接提交后

![](https://s1.ax1x.com/2020/07/17/UseTk6.png)


当直接在地址栏里面通过...Login.acton运行后

![](https://s1.ax1x.com/2020/07/17/Use501.png)

### Action在ValueStack中的位置

* struts2监听到客户端的login.action请求，按配置文件要求，把此请求交给LoginAction处理。这表示会new LoginAction()， 当struts2  new出此Action对象后会把这个对象放在context map中，只是这个Action非常特殊，所以放在值栈中，而放在值栈中的对象是可以直接引用的
* 如果我们在用户名和密码什么都不输，再来用debug来看值栈中的user是，发现它仍会new此对象，因为尽管我们没用输入值，但是后台的set方法还是要被调用，所以会new出此对象
* 如果我们直接输入.../login.action时我们会发现跳转到loginSuccess.jsp页面时，用debug来看值栈中此User user，发现它的值为null。注意的是我们必须为User类提供默认的构造方法，否则将会出现如下错误： java.lang.InstantiationException: 


#### 总结

* Action会在请求时被创建，且会把创建的对象放到值栈中
* Action中的对象字段只有在需要时才会以new 的形式初始化，而且这些对象字段必须提供默认的构造方法
* ValueStack对象贯穿整个Action的生命周期（每个Action类的对象实例会拥有一个ValueStack对象）。当Struts 2接收到一个.action的请求后，会先建立Action类的对象实例，但并不会调用Action方法，而是先将Action类的相应属性放到ValueStack对象的顶层节点（vs对象相当于一个栈）
* 值栈（根）对象也可以直接使用EL表达式访问，比如这里可以直接通过${user.username}来获取username的值

## OGNL基础

* Object Graph Navigation Language
* 开源项目，取代页面中Java脚本，简化数据访问
* 和EL同属于表达式语言，但功能更为强大 


### OGNL上下文

* OGNL表达式的计算围绕OGNL上下文进行
* 由ognl.OgnlContext类表示，实现了Map接口 
* OGNL上下文中可以以键值对的形式包含多个对象，可以将其中一个指定为根对象
	* 访问根对象，直接书写对象的属性
	* 访问其他对象必须使用“#key”前缀

### 数据转移和类型转换

* 开发Web应用程序中最常见的一个任务是从基于字符串的HTTP请求向Java语言的不同数据类型移动和转换数据 
* 数据转移和类型转换上发生在请求处理周期的两端 
* Struts 2提供了强大的数据转移和类型转换功能，由框架自动完成

### OGNL在Struts 2中的作用

#### 表达式语言

将表单或Struts 2标签与特定的Java数据绑定起来，用来将数据移入、移出框架 

#### 类型转换

将表单或Struts 2标签与特定的Java数据绑定起来，用来将数据移入、移出框架 

### OGNL融入Struts 2 

![](https://s1.ax1x.com/2020/07/17/Use4mR.png)


## 常用的OGNL访问操作

### 访问JavaBean 

	public class Address { // 家庭地址
		private String country; // 国家
		private String city; // 城市
		private String street; // 街道
		...  //省略各个属性的setter和getter方法
	}
	
	public class User { //用户类
		private String name; //姓名
		private int age;     //年龄
		private Address address; //家庭地址
		...  //省略各个属性的setter和getter方法
	}

#### User对象作为OGNL上下文的对象，其键名为user

![](https://s1.ax1x.com/2020/07/17/UseITx.png)

#### User对象作为OGNL上下文的根对象，其键名为user

![](https://s1.ax1x.com/2020/07/17/UseW6J.png)

### 访问列表

#### 定义列表

 {value1,value2,values3,...,valueN }

 {"ACCP","BENET","BETEST"} 

	<%
		  List list = new ArrayList();
		  list.add("ACCP");
		  list.add("BENET");
		  list.add("BETEST");
		  return list;
	%>


#### 访问列表

![](https://s1.ax1x.com/2020/07/17/UsefX9.png)

### 访问数组

#### 定义数组

new int[ ]{1,2,3,4}

new double[4]


#### 访问数组

![](https://s1.ax1x.com/2020/07/17/UseHfO.png)

### 访问Map

定义map

  \#{key1:value1,key2:value2,key3:values3,..., keyN,valueN }

\#{"cn":"China","us":"the United States","fr":"France","jp":"Japan"} 

	<%
		Map map = new HashMap();
		map.put("cn", "China");
		map.put("us", "the United States");
		map.put("fr", "France");
		map.put("jp", "Japan");
		return map;
	%>

![](https://s1.ax1x.com/2020/07/17/Use7tK.png)


### 访问非值栈对象 

![](https://s1.ax1x.com/2020/07/17/UseO6H.png)

![](https://s1.ax1x.com/2020/07/17/UseL1e.png)

<s:set>标签将一个值赋给指定范围的变量

<s:property>标签用于输出指定对象的属性值

	<%--			
		request.setAttribute("age",10);
		session.setAttribute("username","jbit");
		application.setAttribute("count",5);				
	--%>
	<s:set name="age" value="10" scope="request"/>
	<s:set name="username" value="'jbit'" scope="session"/>
	<s:set name="count" value="5" scope="application"/>		
	#request.age:<s:property value="#request.age"/><br/>
	#session.username:<s:property value="#session.username"/><br/>
	#application.count:<s:property value="#application.count"/><br/>
	#attr.count:<s:property value="#attr.count" /><br />		
	======================================<br>
	<s:set name="country1" value=“China"/>	
	<s:set name="country2" value="'China'" />
	#country1:<s:property value="#country1"/><br/>
	#country2:<s:property value="#country2"/><br/>	
	#request.country2:<s:property value="#request.country2"/><br/>		

为避免出错，如果分不清一个属性值的类型是不是字符串类型的，可以直接加上%{...}


	<html>
		...
		<body>
		<s:set name="myurl" value="'http://www.jb-aptech.com.cn'"/>
		1：<s:url value="#myurl"/>	<br>
		2：<s:url value="%{#myurl}"/><br>	
		========================================<br/>
		3：<s:property value="#myurl"/><br/>
		4：<s:property value="%{#myurl}"/><br>	
		========================================<br/>
		5：<s:url value="http://www.jb-aptech.com.cn"/><br>
		6：<s:url value="'http://www.jb-aptech.com.cn'"/><br>	
		</body>
	</html>


### 访问集合元素

使用<s:iterator>标签来迭代一个集合或者数组


#### 访问列表

	<s:set name="list" value="{‘java',‘html',‘sql'} "/>
	#list[0]:<s:property value="#list[0]" /><br/>
	#list[2]:<s:property value="#list[2]" /><br/>
	#list.size:<s:property value="#list.size" /><br/>
	list-iterator:
	<s:iterator value="#list">
		<s:property />			
	</s:iterator>

#### 访问数组

	<s:set name="array" value="new int[]{1,2,3,4}"/>
	#array[0]:<s:property value= "#array[0]" /><br/>
	#array[2]:<s:property value="#array[2]" /><br/>
	#array.length:<s:property value="#array.length" /><br/>
	array-iterator:
	<s:iterator value="#array">			
		<s:property />
	</s:iterator>

#### 访问map

	<s:set name="map"
	  value="#{'cn':'China','us':'the United States','fr':'France'}"/>
	#map['cn']:<s:property value="#map['cn']" /><br />
	#map.cn:<s:property value="#map.cn" /><br />
	map-iterator:
	<s:iterator value="#map">
	     <s:property value="key"/>---<s:property value="value"/>,
	</s:iterator>

### 访问Action属性 


#### Action

	public class OgnlAction extends ActionSupport {
		private User user;
		public User getUser() {	return user;	}
		public void setUser(User user) {
			this.user = user;	
		}
		public String execute(){
			user = new User();
			user.setName("jbit");
			user.setAge(23);
			Address address = new Address();
			address.setCountry("China");
			address.setCity("beijing");
			address.setStreet("chengfu street");
			user.setAddress(address);		
			return SUCCESS;
		}
	}

#### Struts.xml

	<struts>
		<constant name="struts.custom.i18n.resources" value="message"/>
		<package name="Struts 2" extends="struts-default">	
			<action name="ognl" class="cn.jbit.action.OgnlAction">
				<result>testognl4.jsp</result>
			</action>
		</package>
	</struts>

#### JSP页面
	
	用户名：<s:property value="user.name"/><br>
	用户名：<s:property value="user.name.toUpperCase()"/><br>
	年龄：<s:property value="user.age"/><br>
	国家：<s:property value="user.address.country"/><br>	

访问Action属性无需使用“#”前缀。Struts 2总是把Action实例放在值栈的栈顶

## OGNL进阶

#### LoginAction


	
	public class OgnlAction extends ActionSupport {
		private User user;
		private List studentList = new ArrayList();	public String execute(){
	studentList.add(new Student("jack", 20, 86.0f));
	studentList.add(new Student("lily", 22, 96.5f));
	studentList.add(new Student("tom", 23, 56.5f));
		return SUCCESS;
		}
		…..省略get和set
	}

#### Login_success.jsp

	获取List中的Student对象：<s:property value="studentList"/><br>
	利用投影获取List中的name属性：
		<s:property value="studentList.{name}"/><br>
	利用投影获取List中的age属性：
		<s:property value="studentList.{age}"/><br>
	利用投影获取List中的第一个对象的name属性：
		<s:property value="studentList.[0]{name}"/>   
		或者<s:property value="studentList.{name}[0]"/><br>
	利用选择获取List中grade>60的student信息：
		<s:property value="studentList.{?#this.grade>60}"/><br>
	利用选择获取List中grade>60的student名字信息：
	<s:property value="studentList.{?#this.grade>60}.{name}"/><br>
	利用选择获取List中grade>60的第一个student名字信息：
	<s:property value="studentList.{?#this.grade>60}.{name}[0]"/><br>
	利用选择获取List中grade>60的第一个student名字信息(链表)：
	<s:property value="studentList.{^#this.grade>60}.{name}"/><br>
	利用选择获取List中grade>60的最后一个student名字信息(链表)：
	<s:property value="studentList.{$#this.grade>60}.{name}"/><br>

### ?,#,^说明

studentList中有许多Stutdent对象,我们可以用条件来限制取哪些对象，这些条件必须以?#开始，并且条件要用{}括起。而this是指在判断studentList中的对象是否符合条件的当前对象。?#是指取出符合条件的所有Student对象，而^#是指取出符合条件的第一个对象，$#是指取出符合条件的最后一个对象

$用于i18n和struts配置文件，#取得ActionContext的值，%将原来的文本串解析为ognl，对于本来就是ognl的文本不起作用。形式：%{要解析的文本串}


#### Login_success.jsp

	
	N语法[0]：<s:property value="[0]"/><br>
	N语法[1]：<s:property value="[1]"/><br>
	N语法[0].top：<s:property value="[0].top"/><br>
	N语法[1].top：<s:property value="[1].top"/><br>
	N语法top：<s:property value="top"/><br>
	N语法取值：
	<s:property value="[0].user.userName"/><br>
	N语法取值：
	<s:property value="top.user.userName"/><br>

### N语法

规定栈顶的对象为[0],而我们只使用[0]的意思是从值栈中第一个对象取，一直取至栈底。N的意思是从值栈中的第N个对象开始，取到栈底为止。如果要想访问某个对象，需要使用[N].top，它的意思是取出符合N语法的栈顶对象，比如在这里，[0]会取出两个对象，而[0].top是取出这两个对象的栈顶对象。纯top可以简洁地取出值栈中的栈顶对象

$用于i18n和struts配置文件，#取得ActionContext的值，%将原来的文本串解析为ognl，对于本来就是ognl的文本不起作用。形式：%{要解析的文本串}

### 总结

OGNL是Object Graphic Navigation Language(对象图导航语言)的缩写，它是一个开源项目。Struts2使用OGNL作为默认的表达式语言。

相对于EL表达式，它提供了平时我们需要的一些功能，如：支持对象方法调用，支持各类静态方法调用和值访问，支持操作集合对象。OGNL有一个上下文的概念，这个上下文件实质就是一个Map结构，它实现了java.utils.Map接口，在struts2中上下文的实现为ActionContext，当struts2接受一个请求时，会迅速创建ActionContext，ValueStack，action。然后把action存放进ValueStack，所以action的实例变量可以接受OGNL访问，访问上下文中的对象需要使用#号标注命名空间，如#application、#session

在struts2中，根对象的实现类为OgnlValueStack，该对象不是我们想象的只存放单个值，而是存放一组对象，在OgnlValueStack类里有一个List类型的变量，就是使用这个List变量来存放一组对象。在root变量（List类型）中处于第一位的对象叫栈顶对象，通常我们在Ognl表达式里直接写上属性的名称即可访问root变量里对象的属性，搜索顺序是从栈顶对象开始寻找，如果栈顶对象不存在该属性，就会从第二个对象寻找，如果没有找到就从第三个对象寻找，依次往下寻找。 

注意：struts2中 ，OGNL表达式需要配合struts的标签才可以使用。另外OGNL会设定一个根对象，在struts2中根对象就是ValueStack值栈对象，如果要访问根对象中对象的属性，则可以省略#命名空间，直接访问该对象的属性即可

### ActionContext上下行文

![](https://s1.ax1x.com/2020/07/17/UseqpD.png)

## ActionContext和 ServletActionContext

### ActionContext

ActionContext是线程安全的，使用ThreadLocal为每一个使用该变量的线程都提供一个副本，使每一个线程都有自己的副本独立运行，而不会和其他线程的副本冲突。

存放在ActionContext中的数据都放在这个ThreadLocal的属性中，而这个属性只会在对应的当前线程中可见，从而保证线程安全。

### ServletActionContext

在实际应用中，有时候需要使用类似于Cookie，那么这时，AcrionContext就不行了，需要使用ServletActionContext来获取。这种方式，是采用与Servlet耦合的方式。


在获取如request、session时，建议使用XXXAware接口的形式，不建议使用ServletXXXAware的方式，尽可能的降低于Servlet的耦合，所以在实际开发中，优先考虑使用ActionContext。

注意:在使用ActionCotnext时，不要在Action的构造函数里使用ActionContext.getContext()，因为这个时候ActionContext里的一些值也许还没有设置，这时通过ActionContext取得值也许是Null。