---
layout: post
title: "SpringMvc 学习笔记（1）——SpringMVC介绍和注解配置"
date: 2020-07-05 20:34:27 +0800
categories: notes
tags: springMvc
img: https://s1.ax1x.com/2020/07/06/UC7LAH.png
---
SpringMVC介绍；架构；推荐使用的注解配置；springMvc的hello world


# SpringMVC介绍

### MVC回顾

1、 模型（Model）：负责封装应用的状态，并实现应用的功能。通常分为数据模型和业务逻辑模型，数据模型用来存放业务数据，比如订单信息、用户信息等；而业务逻辑模型包含应用的业务操作，比如订单的添加或者修改等。通常由java开发人员编写程序完成，代码量最多

2、 视图（View）：视图通过控制器从模型获得要展示的数据，然后用自己的方式展现给用户，相当于提供界面来与用户进行人机交互。通常有前端和java开发人员完成，代码量较多。

3、 控制器（Controller）：用来控制应用程序的流程和处理用户所发出的请求。当控制器接收到用户的请求后，会将用户的数据和模型的更新相映射，也就是调用模型来实现用户请求的功能；然后控制器会选择用于响应的视图，把模型更新后的数据展示给用户。起到总调度的作用，Controller通常由框架实现，使用时基本不需要编写代码

![](https://s1.ax1x.com/2020/07/06/UCwe6f.md.png)

## SpringMVC介绍

大部分java应用都是web应用，展现层是web应用最为重要的部分。Spring为展现层提供了一个优秀的web层框架——SpringMVC。和众多其他web框架一样，它基于MVC的设计理念，此外，它采用了松散耦合可插拔组件结构，比其他MVC框架更具扩展性和灵活性。

SpringMVC通过一套MVC注解，让POJO成为处理请求的控制器，无需实现任何接口，同时，SpringMVC还支持REST风格的URL请求。
此外，SpringMVC在数据绑定、视图解析、本地化处理以及静态资源处理上都有许多不俗的表现。

它在框架设计、扩展性、灵活性等方面全面超越了Struts、WebWork等MVC框架，从原来的追赶者一跃成为MVC的领跑者。

SpringMVC框架围绕DispatcherServlet这个核心展开，DispatcherServlet是SpringMVC框架的总导演、总策划，它负责截获请求并将其分派给相应的处理器处理。


## SpringMVC架构

![](https://s1.ax1x.com/2020/07/06/UCwmX8.md.png)

## hello world

### 创建web项目（war）

若创建结束提示错误，则右击项目——>Java EE Tools——>Generate Deployment Descriptor Stub.然后系统会在src/main/webapp/WEB_INF文件加下创建web.xml文件。错误解决！

### 引入jar包及源码包

![](https://s1.ax1x.com/2020/07/06/UCwVpt.png)


Spring-web：web开发相关的基础功能包

Spring-webmvc：SpringMVC框架核心包

pom.xml：

    <dependencies>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-beans</artifactId>
      <version>5.1.16.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.1.6.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-core</artifactId>
      <version>5.1.16.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-expression</artifactId>
      <version>5.1.16.RELEASE</version>
    </dependency>
    <dependency>
    	<groupId>org.springframework</groupId>
    	<artifactId>spring-aop</artifactId>
    	<version>5.1.16.RELEASE</version>
    </dependency>
    <dependency>
    	<groupId>org.springframework</groupId>
    	<artifactId>spring-web</artifactId>
    	<version>5.1.16.RELEASE</version>
    </dependency>
    <dependency>
    	<groupId>org.springframework</groupId>
    	<artifactId>spring-webmvc</artifactId>
    	<version>5.1.16.RELEASE</version>
    </dependency>
     	<dependency>
     		<groupId>commons-logging</groupId>
     		<artifactId>commons-logging</artifactId>
     		<version>1.2</version>
     	</dependency>
     	<dependency>
     		<groupId>log4j</groupId>
     		<artifactId>log4j</artifactId>
     		<version>1.2.12</version>
     	</dependency>
    </dependencies>
    
### 配置web.xml

      <servlet>
      	<servlet-name>springmvc</servlet-name>
      	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
      </servlet>
      <servlet-mapping>
      	<servlet-name>springmvc</servlet-name>
      	<!-- 
      		/*：拦截所有请求，包括jsp
      		/ ：拦截所有请求，不包含jsp
      		*.do,*.action
      	 -->
      	<url-pattern>*.do</url-pattern>
      </servlet-mapping>
     
    
## springmvc的配置文件

#### {servlet-name}-servlet.xml

用户发送请求到web容器，并被DispatchServlet拦截之后进入springmvc容器，springmvc该怎么处理那，这就需要springmvc的配置文件。

那么springmvc的配置文件该放在什么位置，又该怎么命名呢？

找到DispatchServlet这个类：

![](https://s1.ax1x.com/2020/07/06/UCwZ1P.png)

![](https://s1.ax1x.com/2020/07/06/UCwK0g.md.png)

由此知道，springmvc默认读取/WEB-INF/{servlet-name}-servlet.xml这个配置文件，因为我们在web.xml中的servlet-name配置的是springmvc，所以在WEB-INF目录下创建springmvc-servlet.xml文件：

springmvc配置文件的头信息和spring一样。

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
    	xmlns:context="http://www.springframework.org/schema/context"
    	xmlns:mvc="http://www.springframework.org/schema/mvc"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    
    </beans>

#### HandlerMapping映射器

![](https://s1.ax1x.com/2020/07/06/UCwunS.png)

![](https://s1.ax1x.com/2020/07/06/UCwlkj.md.png)

    <!-- 配置HandlerMapping -->
    	<!-- 把bean的name属性作为Url -->
    	<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping" />
#### HandlerAdapter适配器

![](https://s1.ax1x.com/2020/07/06/UCwM7Q.png)

![](https://s1.ax1x.com/2020/07/06/UCw1ts.md.png)

    <!-- 配置HandlerAdapter -->
    	<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter" />


#### ViewResolver试图解析器

![](https://s1.ax1x.com/2020/07/06/UCw3hn.png)

![](https://s1.ax1x.com/2020/07/06/UCwGpq.md.png)

![](https://s1.ax1x.com/2020/07/06/UCwJ10.md.png)

由此可见，视图解析器的规则是：prefix+viewName+suffix

#### 完整的配置

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
    	xmlns:context="http://www.springframework.org/schema/context"
    	xmlns:mvc="http://www.springframework.org/schema/mvc"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    
    	<!-- 配置映射器,把bean的name属性作为一个url -->
    	<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping" />
    	
    	<!-- 配置适配器 -->
    	<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter" />
    	
    	<bean name="/hello.do" class="com.atguigu.springmvc.controller.HelloController" />
    	
    	<!-- 配置视图解析器 -->
    	<!-- Example: prefix="/WEB-INF/jsp/", suffix=".jsp", viewname="test" -> "/WEB-INF/jsp/test.jsp"  -->
    	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    		<property name="prefix" value="/WEB-INF/views/"></property>
    		<property name="suffix" value=".jsp"></property>
    	</bean>
    
    </beans>

### HelloController


HelloController内容：
    
![](https://s1.ax1x.com/2020/07/06/UCwYcV.png)


    /**
     * 在整体架构中，通常称为Handler
     * 在具体实现中，通常称为Controller
     * @author joedy
     *
     */
    
    public class HelloController implements Controller {
    
    @Override
	    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response)
	    throws Exception {
	    ModelAndView mv = new ModelAndView();
	    mv.setViewName("hello"); // 试图名称
	    mv.addObject("msg", "这是我的第一个SpringMVC程序！！"); // 数据模型
	    return mv;
	    }
   	 
    }

### 添加jsp页面（hello.jsp）

![](https://s1.ax1x.com/2020/07/06/UCwtXT.png)

    <%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>Insert title here</title>
    </head>
    <body>
    	<div style="color:red; font-size:30px">${msg }</div>
    </body>
    </html>

### 运行测试

通过浏览器访问：http://localhost:8080/springmvc/hello.do

![](https://s1.ax1x.com/2020/07/06/UCw2nO.md.png)

### 添加log日志

![](https://s1.ax1x.com/2020/07/06/UCwUnU.png)

Log4j.properties内容：

    log4j.rootLogger=DEBUG,A1
    log4j.logger.org.mybatis=DEBUG
    log4j.appender.A1=org.apache.log4j.ConsoleAppender
    log4j.appender.A1.layout=org.apache.log4j.PatternLayout
    log4j.appender.A1.layout.ConversionPattern=%-d{yyyy-MM-dd HH:mm:ss,SSS} [%t] [%c]-[%p] %m%n

### 日志打印信息

![](https://s1.ax1x.com/2020/07/06/UCws91.md.png)

### 流程分析
 
![](https://s1.ax1x.com/2020/07/06/UCwd74.md.png)

![](https://s1.ax1x.com/2020/07/06/UCwBN9.md.png)

![](https://s1.ax1x.com/2020/07/06/UCwaBF.md.png)

## 优化helloworld程序

入门程序虽然完成了，但是还有可以改进的空间。结合Spring入门流程，我们一步步的优化入门程序。

### web.xml

#### load-on-startup

结合启动日志，发现入门程序在tomcat的运行完成后并没有加载servlet，而是在用户第一次访问之后才加载。生产环境会影响网站的相应速度

解决方案：让tomcat启动时就去加载DispatcherServlet并初始化Spring容器。

![](https://s1.ax1x.com/2020/07/06/UCw0AJ.md.png)


#### 指定SpringMVC的配置文件

通常情况下SpringMVC的命名规则是{servlet-name}-servlet.xml，一个配置文件名而已，问题不大。

但是，SpringMVC的配置文件的存放路径，如果采用默认配置路径（WEB-INF下），这在项目开发时不利于统一维护管理。能不能把SpringMVC的配置文件和其他Spring配置文件放在一起，方便统一管理呢？

通过dispatcherServlet的初始化参数告诉SpringMVC它的配置文件在哪里。

这时就需要在web.xml中去指定springMVC的存放路径，配置方式：

      <servlet>
      	<servlet-name>springmvc</servlet-name>
      	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
      	<!-- 指定springMVC的配置文件，多个可以用逗号和空格隔开 -->
      	<init-param>
      		<param-name>contextConfigLocation</param-name>
      		<param-value>classpath:springmvc-servlet.xml</param-value>
      	</init-param>
      	<!-- 值为非负的情况下，tomcat启动时，就加载该servlet。值越小加载的优先级就越高 -->
      	<load-on-startup>1</load-on-startup>
      </servlet>
      <servlet-mapping>
      	<servlet-name>springmvc</servlet-name>
      	<!-- 
      		/*：拦截所有请求，包括jsp
      		/ ：拦截所有请求，不包含jsp
      		*.do,*.action
      	 -->
      	<url-pattern>*.do</url-pattern>
      </servlet-mapping>

原理：参见DispatcherServlet父类的注释

![](https://s1.ax1x.com/2020/07/06/UCwDhR.md.png)

DispatchServlet.class源码中：

![](https://s1.ax1x.com/2020/07/06/UCwy1x.md.png)

找到DispatchServlet.properties文件：

![](https://s1.ax1x.com/2020/07/06/UCwcjK.png)

![](https://s1.ax1x.com/2020/07/06/UCwRBD.md.png)


在这个默认的配置文件中，已经配置了映射器和适配器。

所以在springmvc-servlet.xml文件中可以省略之前配置的映射器和适配器

![](https://s1.ax1x.com/2020/07/06/UCwWHe.md.png)

再次测试：

![](https://s1.ax1x.com/2020/07/06/UCw2nO.md.png)

### helloword的缺点

1）每个类需要都实现Controller接口

2）每个类（Controller）只能完成一个用户请求（或者只能处理一个业务逻辑）

3）每个类（Controller）都要在配置文件里，进行配置
解决方案：

**注解程序**

# 注解

## 默认注解配置

在DispatchServlet.properties文件中，已经提供了默认的注解映射器和适配器，所以咱们可以直接书写注解的代码

![](https://s1.ax1x.com/2020/07/06/UC7ONd.md.png)

### 创建hello2Controller

![](https://s1.ax1x.com/2020/07/06/UC7bHe.png)

内容：

    @Controller
    public class Hello2Controller {
    
    @RequestMapping("show1")
    public ModelAndView test1(){
    ModelAndView mv = new ModelAndView();
    mv.setViewName("hello");
    mv.addObject("msg", "这是SpringMVC的第一个注解程序！");
    return mv;
    }
    
    }

### 配置扫描器
    
    在springmvc-servlet.xml中，开启注解扫描
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
    	xmlns:context="http://www.springframework.org/schema/context"
    	xmlns:mvc="http://www.springframework.org/schema/mvc"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    
    	<!-- 配置映射器,把bean的name属性作为一个url -->
    	<!-- <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping" /> -->
    	
    	<!-- 配置适配器 -->
    	<!-- <bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter" /> -->
    	
    	<!-- <bean name="/hello.do" class="com.atguigu.springmvc.controller.HelloController" /> -->
    	
    	<!-- 配置注解扫描，和Spring的配置方式一样 -->
    	<context:component-scan base-package="com.atguigu.springmvc" />
    	
    	<!-- 配置视图解析器 -->
    	<!-- Example: prefix="/WEB-INF/jsp/", suffix=".jsp", viewname="test" -> "/WEB-INF/jsp/test.jsp"  -->
    	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    		<property name="prefix" value="/WEB-INF/views/"></property>
    		<property name="suffix" value=".jsp"></property>
    	</bean>
    
    </beans>
    
### 测试

![](https://s1.ax1x.com/2020/07/06/UC7HBD.md.png)

### 缺点

找到默认的注解映射器和适配器，发现他们都已过时。

![](https://s1.ax1x.com/2020/07/06/UC77nO.md.png)

![](https://s1.ax1x.com/2020/07/06/UC7X4A.md.png)

既然默认配置的映射器和适配器都已经过期，并且springmvc也推荐了相应的支持注解的映射器和适配器

## 推荐使用的注解配置

### springmvc-servlet.xml

    <!-- 推荐使用的注解映射器 -->
    	<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"/>
    	
	<!-- 推荐使用的注解适配器 -->
	<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"/>

## 最佳方案（注解驱动）

### 注解驱动的配置

在springmvc-servlet.xml中配置注解驱动

<mvc:annotation-driven />


	<!-- 注解驱动，替代推荐使用的注解映射器和适配器，并提供了对json的支持 -->
	<mvc:annotation-driven />

### 注解驱动的原理

AnnotationDrivenBeanDefinitionParser的注释

![](https://s1.ax1x.com/2020/07/06/UC7x3t.md.png)

## 注解配置最终方案


使用注解驱动后springmvc-servlet.xml这个配置文件：

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
    	xmlns:context="http://www.springframework.org/schema/context"
    	xmlns:mvc="http://www.springframework.org/schema/mvc"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    
    	<!-- 注解驱动，替代推荐使用的注解映射器和适配器，并提供了对json的支持 -->
    	<mvc:annotation-driven />
    	
    	<!-- 配置注解扫描，和Spring的配置方式一样 -->
    	<context:component-scan base-package="com.atguigu.springmvc" />
    	
    	<!-- 配置视图解析器 -->
    	<!-- Example: prefix="/WEB-INF/jsp/", suffix=".jsp", viewname="test" -> "/WEB-INF/jsp/test.jsp"  -->
    	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    		<property name="prefix" value="/WEB-INF/views/"></property>
    		<property name="suffix" value=".jsp"></property>
    	</bean>
    
    </beans>

目前这些配置已经能够完成springmvc的基本使用，后续还会添加一些高级用法的配置，例如：拦截器、自定义试图、文件上传等