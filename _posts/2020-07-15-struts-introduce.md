---
layout: post
title: "struts2的介绍及配置详解"
date: 2020-07-15 13:24:12 +0800
categories: notes
tags: struts2
img: https://s1.ax1x.com/2020/07/16/UDF2WR.png
---
struts2的简介；开发基本步骤；配置详解；Action；Result配置

## 简介

Struts 2是一个MVC框架，以WebWork设计思想为核心，吸收了Struts 1的部分优点

Struts 2拥有更加广阔的前景，自身功能强大，还对其他框架下开发的程序提供很好的兼容性

### Struts 2的资源获取 

1. Struts官方地址：http://www.apache.com
2. 选取Struts 2.1.8进行讲解
3. Struts 2 目录结构

![](https://s1.ax1x.com/2020/07/16/UDFgY9.png)

* apps目录：Struts2示例应用程序
* docs目录：Struts2指南、向导、API文档
* lib目录：Struts 2的发行包及其依赖包
* src目录：Struts 2项目源代码

## 基本步骤

1. 加载Struts2 类库
2. 配置web.xml
3. 开发视图层页面
4. 开发控制层Action
5. 配置Struts 2的配置文件（struts.xml）
6. 部署、运行项目

### 加载Struts2 类库

必须加载的5个jar文件：

1. struts2-core-2.1.6.jar：Struts 2框架的核心类库
2. xwork-2.1.2.jar：XWork类库，Struts 2的构建基础
3. ognl-2.6.11.jar：Struts 2使用的一种表达式语言类库
4. freemarker-2.3.13.jar：Struts 2的标签模板使用类库
5. commons-fileupload-1.2.1.jar：Struts 2文件上传依赖包

### 配置web.xml

    <filter>
    	<filter-name>struts2</filter-name>
    	<filter-class>
    		org.apache.struts2.dispatcher.ng.filter.
    					StrutsPrepareAndExecuteFilter
    	</filter-class>
    </filter>
    	
    <filter-mapping>
    	<filter-name>struts2</filter-name>
		//将全部请求定位到指定的Struts 2过滤器中
    	<url-pattern>/*</url-pattern>
    </filter-mapping>
    
### 开发视图层页面-helloWorld.jsp
    
    …
    <h1>Hello World</h1>
    <div>
    	<h1>
    		<!-- 显示Struts Action中message属性内容 -->
    		${message}
    	</h1>
    </div>
    <hr />
    <div>
    	<form action="helloWorld.action">
    		请输入您的姓名：
    		<input name="name" type="text" />
    		<input type="submit" value="提交" />
    	</form>
    </div>
    …

### 开发控制层Action-HelloWorldAction

    public class HelloWorldAction {	
    	private String name = ""; // 用户输入的姓名
    	private String message = ""; // 向用户显示的信息	
    	/**
    	 * 当Struts 2处理用户请求时，在默认配置下调用的方法
    	 */
    	public String execute() {
    		// 根据用户输入的姓名，进行“Hello，XXXX！”的封装
    		this.setMessage("Hello,"+this.getName()+"！");
    		// 处理完毕，返回“helloWorld”
    		return "helloWorld";
    	}
    	…	//省略setter、getter方法
    }
    
### 配置Struts 2配置文件（struts.xml）

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE struts PUBLIC
    "-//Apache Software Foundation//DTD Struts Configuration 2.0//EN"
    "http://struts.apache.org/dtds/struts-2.0.dtd">
    <struts>
    	<package name="default" namespace="/" extends="struts-default">
    		<action name="helloWorld" 
    					class="com.strutsdemo.HelloWorldAction">	
    			<result name="helloWorld">helloWorld.jsp</result>
    		</action>
    	</package>
    </struts>
    

### DTD来源

    Document Type Definition 
    <!DOCTYPE struts PUBLIC"-//Apache Software Foundation//DTD Struts Configuration 2.1.7//EN""http://struts.apache.org/dtds/struts-2.1.7.dtd">
    来自struts2-core-2.1.XXX.jar/struts-default.xml
    
## Action属性的数据绑定

用户登录成功后，将用户名保存在session中，供多个页面和Action使用

在Struts 2 中通过Action属性的数据绑定降低了对request的使用需求，但是对于session、application还是有使用需求的

* 与Servlet API解耦的访问方式
* 与Servlet API耦合的访问方式

### 与Servlet API解耦的访问方式

1. Struts 2对Servlet API进行封装，提供了三个Map对象代替request、session、application
2. 通过ActionContext类获取这三个Map对象
	* Object get("request")
	* Map getSession()
	* Map getApplication()


    public class LoginAction {
    	private static final String CURRENT_USER = "CURRENT_USER";	
    	… //省略username、password属性及其setter和getter方法
    	public String execute() {
    		if("jbit".equals(username) && "bdqn".equals(password)) {
    			Map<String,Object> session = null;
				//获取session
    			session = ActionContext.getContext().getSession();
    			if(session.containsKey(CURRENT_USER)) {				
    				session.remove(CURRENT_USER);
    			}
				//将用户名存入session
    			session.put(CURRENT_USER, username);
    			return "success";
    		} else {
    			return "fail";
    		}
    	}
    }

### 与Servlet API耦合的访问方式

1. 通过ServletActionContext类获取Servlet API对象
	* ServletContext getServletContext()
	* HttpServletResponse getResponse()
	* HttpServletRequest  getRequest()
	* 通过request.getSession()获取session对象
2. 通过xxx.setAttribute()和xxx.getAttribute() 功能，在不同的页面或Action中传递数据

    public class LoginAction {
    	private static final String CURRENT_USER = "CURRENT_USER";	
    	… //省略username、password属性及其setter和getter方法
    	public String execute() {			
    		if("jbit".equals(username) && "bdqn".equals(password)) {
				//获取session	
    			HttpSession session = null;
    			session = ServletActionContext.getRequest().getSession(); 			if(session.getAttribute(CURRENT_USER) != null) {
    				session.removeAttribute(CURRENT_USER);
    			}
				//将用户名存入session
    			session.setAttribute(CURRENT_USER, username);	
    			return "success";
    		} else {			
    			return "fail";
    		}
    	}
    }
    

## 配置详解

#### 登录程序运行流程图 

![](https://s1.ax1x.com/2020/07/16/UDFfQx.png)

### package元素 

* 包的作用：简化维护工作，提高重用性
* 包可以“继承”已定义的包，并可以添加自己包的配置
* name属性为必需的且唯一，用于指定包的名称
* extends属性指定要扩展的包
* namespace属性定义该包中action的命名空间 ，为可选属性
    …
    <struts>
      	<constant name="" value=""/>
    	<package name="default" namespace="/" extends="struts-default">
    		<action name="" class="">
    			<result name=""></result>			
    		</action>
    	</package>
    </struts>

注：除非有令人信服原因，自定义的包应该总是扩展**struts-default**包

### 核心控制器

* 需要在web.xml中进行配置
* 对框架进行初始化，以及处理所有的请求

    <filter>
    	<filter-name>struts2</filter-name>
    	<filter-class>
    		org.apache.struts2.dispatcher.ng.filter.
    					StrutsPrepareAndExecuteFilter
    	</filter-class>
    </filter>	
    <filter-mapping>
    	<filter-name>struts2</filter-name>
    	<url-pattern>/*</url-pattern>
    </filter-mapping>

Struts 2.0版本的核心控制器为:org.apache.struts2.dispatcher.FilterDispatcher

### Action

在\struts2-core-2. XXX\template\simple目录下找到fielderror.ftl实现Action接口（错误对应标签是s:fielderror如果是actionerror就是actionerror.ftl）

    …
    <struts>
    	<package name="default" namespace="/" extends="struts-default">
    		<action name="login" class="com.hrent.action.LoginAction">
    			<result name="success">/page/manage.jsp</result>
    			<result name="input">/page/login.jsp</result>						<result name="error">/page/error.jsp</result>
    		</action>
    	</package>
    </struts>

### Result

作用:调度视图以哪种形式体现给客户端(Action处理结束后，系统下一步将要做什么)

name属性表示result逻辑名，result元素的值指定对应的实际资源位置

    …
    <struts>
    	<package name="default" namespace="/" extends="struts-default">
    		<action name="login" class="com.hrent.action.LoginAction">
    			<result name="success">/page/manage.jsp</result>
    			<result name="input">/page/login.jsp</result>						<result name="error">/page/error.jsp</result>
    		</action>
    	</package>
    </struts>



### constant元素

* 配置常量，可以改变Struts 2框架的一些行为
* name属性表示常量名称，value属性表示常量值 	
    
    …
    <struts>
      	<constant name="struts.i18n.encoding" value="UTF-8"/>
    	<constant name="struts.ui.theme" value="simple"/>
     	<package name="" namespace="" extends="">
    		<action name="" class="">
    			<result name=""></result>			
    		</action>
    	</package>
    </struts>


常见constant元素

指定Web应用的默认编码集，相当于调用 HttpServletRequest的setCharacterEncoding方法  

`<constant name="struts.i18n.encoding" value="UTF-8" />  ` 

该 属性指定需要Struts 2处理的请求后缀，该属性的默认值是action，即 所有匹配*.action的请求都由Struts 2处理。如 果用户需要指定多个请求后缀，则多个后缀之间以英文逗号（，）隔开
    
    <constant name="struts.action.extension" value="do,action" />   

设 置浏览器是否缓存静态内容，默认值为true（生产环境下使用），开发阶段最好 关闭

    <constant name="struts.serve.static.browserCache " value="false" />  

当 struts的配置文件修改后，系统是否自动重新加载该文件，默认值为false（生 产环境下使用），开发阶段最好打开

    <constant name="struts.configuration.xml.reload" value="true" />

开发模式下使用，这样可以打印出更详细的错误信息

    <constant name="struts.devMode" value="true" />   

默认的视图主题, 该属性的默认值是xhtml

    <constant name="struts.ui.theme" value="simple" />   

Constant里面的配置的位置
struts2-core-2.1.XXX.jar\org\apache\struts2 
default.properties

### struts.xml

核心配置文件，主要负责管理Action

通常放在WEB-INF/classes目录下，在该目录下的struts.xml文件可以被自动加载
    …
    <struts>
      	<constant name="" value=""/>
    	<package name="" namespace="" extends="">
    		<action name="" class="">
    			<result name=""></result>			
    		</action>
    	</package>
    </struts>

### struts-default.xml 

Struts 2默认配置文件，会自动加载

struts-default包在struts-default.xml文件中定义

### struts-plugin.xml 

Struts 2插件使用的配置文件
 
如果不开发插件，不需要编写该配置文件

### 加载顺序

struts-default.xml—>struts-plugin.xml—>struts.xml —>web.xml

## Action

### Action的作用

1. 封装工作单元
2. 数据转移的场所 
3. 返回结果字符串 

    public class HelloWorldAction implements Action {	
    	  private String name = "";
    	  private String message = "";	
    	  public String execute() {
    		  this.setMessage("Hello,"+this.getName()+"！");
    		  return SUCCESS;
    	  }
    	  //...省略setter/getter方法 
    }
    

### 动态方法调用

作用：减少Action数量

使用：actionName!methodName.action 

禁用：将属性strutsenableDynamicMethodInvocation设置为false 

    public class UserAction implements Action {	
    	  …	
    	  public String login() {
    		  …		
    	  }
    	public String register() {
    		  …		
    	  }
    }
    
#### struts.xml

    <action name="user" class="com.houserent.action.UserAction">
    	  <result name="login">/page/manage.jsp</result>
    	  <result name="register">/page/success.jsp</result>
    	  <result name="login_input">/page/login.jsp</result>
    	  <result name="register_input">/page/register.jsp</result>
    	  <result name="error">/page/error.jsp</result>
    </action>

![](https://s1.ax1x.com/2020/07/16/UDFcFJ.png)
    
### 使用method属性

优点：避免动态方法调用的安全隐患

缺陷：导致大量的Action配置 

#### struts.xml

    <action name="login" 
    	  class="com.houserent.action.UserAction" method="login">
    	  <result>/page/manage.jsp</result>
    	  <result name="input">/page/login.jsp</result>
    	  <result name="error">/page/error.jsp</result>
    </action>
    <action name="register" 
    	class="com.houserent.action.UserAction" method="register">
    	  <result>/page/success.jsp</result>
    	  <result name="input">/page/register.jsp</result>
    	  <result name="error">/page/error.jsp</result>
    </action>
    

![](https://s1.ax1x.com/2020/07/16/UDFyo4.png)

### 通配符(*)的使用

另一种形式的动态方法调用

    <action name= "*User" 
    		class="com.houserent.action.UserAction" method="{1}">
    	  <result>/page/{1}_success.jsp</result>
    	  <result name="input">/page/{1}.jsp</result>
    	  <result name="error">/page/error.jsp</result>
    </action>
    

![](https://s1.ax1x.com/2020/07/16/UDFWS1.png)

通配符(*)的使用最常用的例子

    <action name= “*_*" 
    		class="com.houserent.action.{1}Action" method="{2}">
    	  <result>/page/{1}_{2}_success.jsp</result>
    </action>
    
### 配置默认Action

* 如果没有一个Action匹配请求，默认Action将被执行
* 通过<default-action-ref … />元素配置默认Action


    <struts>
    	<package name="default" extends="struts-default">
    	 <default-action-ref name="defaultAction"/ >
			//省略class属性，将使用ActionSupport类
    		<action name="defaultAction">
				//如果请求的Action不存在，将转发到error.jsp
    			<result>error.jsp</result>			
    		</action>
    	</package>
    </struts>



## Result配置

### 常用结果类型 

1. dispatcher类型:默认结果类型，后台使用RequestDispatcher() 转发请求 
2. redirect类型:后台使用的sendRedirect()将请求重定向至指定的URL 
3. redirectAction类型:主要用于重定向到Action 

    <action name= "login" class="com.hrent.action.LoginAction" >
    	<result name="success" type="redirectAction">manage</result>
    	<result name="error" type="redirect">error.jsp</result>
    	<result name="input" type="dispatcher" >login.jsp</result>
    </action>
    <action name="manage"  class= ="com.hrent.action.UserAction" >
    	…
    </action>

### 动态结果

配置时不知道执行后的结果是哪一个，运行时才知道哪个结果作为视图显示给用户

    public class UserAction extends ActionSupport {
    	private String nextDispose;
    	public String login() {
    		...
    		if(user.isManager()){
    			nextDispose = "manager";
    		}else{
    			nextDispose = "common";
    		}
    		return  nextDispose;
    	}
    	public String getNextDispose(){
    		return nextDispose;
    	}
    	...
    }
    
注：nextDispose要在Action中存在，并且提供其getter方法 

#### struts.xml

    <struts>
    	<package name="default" extends="struts-default">
    		<action name="login"
    			class="com.houserent.action.UserAction" method="login">
    			<result type="redirectAction">${nextDispose}</result>
    			<result name="error">/page/error.jsp</result>
    		</action>
    		<action name="common"
    			class="com.houserent.action.CommonUserAction">
    			…
    		</action>
    		<action name="manage"
    			class="com.houserent.action.ManagerAction">
    			...
    		</action>
    	</package>
    </struts>

### 全局结果

全局结果可满足一个包中多个Action共享一个结果

    <struts>	
    	<default-action-ref name="defaultAction"/ >
    	<package name="default" extends="struts-default">
    		<global-results>
    			<result name="error">/page/error.jsp</result>
    			<result name="login" type="redirect">/page/login.jsp</result>
    		</global-results>
    		<action name="login" 
    	class="com.houserent.action.UserAction" method="login">
    	<result>/page/manage.jsp</result>
    	<result name="input">/page/login.jsp</result>
    	<result name="error">/page/error.jsp</result>
    	</action>		
    	…
    	</package>
    </struts>

不需要在包内的Action中指定error Result了

