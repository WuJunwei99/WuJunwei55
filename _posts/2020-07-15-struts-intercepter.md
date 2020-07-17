---
layout: post
title: "struts2的拦截器"
date: 2020-07-15 20:52:43 +0800
categories: notes
tags: struts2
img: https://s1.ax1x.com/2020/07/16/UDF2WR.png
---
系统架构；运行流程；拦截器的好处；


## 系统架构

![](https://s1.ax1x.com/2020/07/17/UsAAC4.th.png)

关于图中的Key：

* Servlet Filters：过滤器链，客户端的所有请求都要经过Filter链的处理。
* Struts Core：Struts2的核心部分，但是Struts2已经帮我们做好了，我们不需要去做这个
* Interceptors，Struts2的拦截器。Struts2提供了很多默认的拦截器，可以完成日常开发的绝大部分工作；而我们自定义的拦截器，用来实现实际的客户业务需要的功能。
* User Created，由开发人员创建的，包括struts.xml、Action、Template，这些是每个使用Struts2来进行开发的人员都必须会的。

#### FilterDispatcher

FilterDispatcher是整个Struts2的调度中心，也就是MVC中的C（控制中心），根据ActionMapper的结果来决定是否处理请求，如果ActionMapper指出该URL应该被Struts2处理，那么它将会执行Action处理，并停止过滤器链上还没有执行的过滤器。

#### ActionMapper 

根据请求的URI查找是否存在对应Action调用，会判断这个请求是否应该被Struts2处理，如果需要Struts2处理，ActionMapper会返回一个对象来描述请求对应的ActionInvocation的信息。

#### ActionProxy

在XWork和真正的Action之间充当代理 ，它会创建一个ActionInvocation实例，位于Action和xwork之间，使得我们在将来有机会引入更多的实现方式，比如通过WebService来实现等。

#### ActionMapping 

保存调用Action的映射信息，如namespace、name等

#### ConfigurationManager

是xwork配置的管理中心，可以把它看做struts.xml这个配置文件在内存中的对应。

#### struts.xml

是开发人员必须光顾的地方。是Stuts2的应用配置文件，负责诸如URL与Action之间映射关系的配置、以及执行后页面跳转的Result配置等。

#### ActionInvocation

表示Action的执行状态，保存拦截器、Action实例，真正调用并执行Action，它拥有一个Action实例和这个Action所依赖的拦截器实例。ActionInvocation会按照指定的顺序去执行这些拦截器、Action以及相应的Result。

#### Interceptor(拦截器)

可以在请求处理之前或者之后执行的Struts 2组件，Struts 2绝大多数功能通过拦截器完成。

它是Struts2的基石，类似于JavaWeb的Filter，拦截器是一些无状态的类，拦截器可以自动拦截Action，它们给开发者提供了在Action运行之前或Result运行之后来执行一些功能代码的机会。

#### Action

用来处理请求，封装数据。


## 运行流程

1. 当用户的发出请求，比如http:localhost:8080/Struts2/helloworld/helloworldAction.action,请求会被Tomcat接收到，Tomcat服务器来选择处理这个请求的Web应用，那就是由helloworld这个web工程来处理这个请求。
2. Web容器会去读取helloworld这个工程的web.xml，在web.xml中进行匹配，但发现，由struts2这个过滤器来进行处理（也就是StrutsPrepareAndExecuteFilter），根据Filter的配置，找到FilterDispatcher（Struts2的调度中心）
3. 然后会获取FilterDispatcher实例，然后回调doFilter方法，进行真正的处理

PS：FilterDispatcher是任何一个Struts2应用都需要配置的，通常情况下，web.xml文件中还有其他过滤器时，FilterDispatcher是放在滤器链的最后；如果在FilterDispatcher前出现了如SiteMesh这种特殊的过滤器，还必须在SiteMesh前引用Struts2的ActionContextCleanUp过滤器

![](https://s1.ax1x.com/2020/07/17/UskL4g.png)

4.这时FilterDispatcher会将请求转发给ActionMapper。ActionMapper负责识别当前的请求是否需要Struts2做出处理。

![](https://s1.ax1x.com/2020/07/17/Uskj3j.png)

5.如果需要Struts2处理，ActionMapper会通知FilterDispatcher，需要处理这个请求，FilterDispatcher会停止过滤器链以后的部分，（这也就是为什么，FilterDispatcher应该出现在过滤器链的最后的原因）。然后建立一个ActionProxy实例，这个对象作为Action与xwork之间的中间层，会代理Action的运行过程。

![](https://s1.ax1x.com/2020/07/17/Uskvgs.png)

6.ActionProxy对象在被创建出来的时候，并不知道要运行哪个Action，它手里只有从FilterDispatcher中拿到的请求的URL。

而真正知道要运行哪个Action的是ConfigurationManager。因为只有它才能读取我们的strtus.xml

（在服务器启动的时候，ConfigurationManager就会把struts.xml中的所有信息读到内存里，并缓存，当ActionProxy带着URL向他询问要运行哪个Action的时候，就可以直接匹配、查找并回答了）

![](https://s1.ax1x.com/2020/07/17/UskXCQ.png)

7.ActionProxy知道自己该干什么事之后（运行哪个Action、相关的拦截器以及所有可能使用的result信息），然后马上建立ActionInvocation对象了，ActionInvocation对象描述了Action运行的整个过程。

注意：Action完整的调用过程都是由ActionInvocation对象负责

![](https://s1.ax1x.com/2020/07/17/Uskxvn.png)

8.在execute方法之前，好像URL请求中的参数已经赋值到了Action的属性上，这就是我们的拦截器。

拦截器的运行被分成两部分，一部分在Action之前运行，一部分在Result之后运行，而且顺序是刚好反过来的。也就是在Action执行前的顺序，比如是拦截器1、拦截器2、拦截器3，那么运行Result之后，再次运行拦截器的时候，顺序就变成拦截器3、拦截器2、拦截器1了。

所以ActionInvocation对象执行的时候需要通过很多复杂的过程，按照指定拦截器的顺序依次执行。

![](https://s1.ax1x.com/2020/07/17/UsApD0.png)

9.执行Action的execute方法,然后根据execute方法返回的结果（Result），去struts.xml中匹配选择下一个页面

![](https://s1.ax1x.com/2020/07/17/UsASuq.png)

10.根据结果(Result)找到页面后，在页面上(有很多Struts2提供的模板)，可以通过Struts2自带的标签库来访问需要的数据，并生成最终页面

注意：这时还没有给客户端应答，只是生成了页面

![](https://s1.ax1x.com/2020/07/17/UsA9bV.png)

11.最后，ActionInvocation对象倒序执行拦截器

![](https://s1.ax1x.com/2020/07/17/UsAPET.png)

12.ActionInvocation对象执行完毕后，已经得到响应对象（HttpServletResponse）了，最后按与过滤器（Filter）配置定义相反的顺序依次经过过滤器，向客户端展示出响应的结果

![](https://s1.ax1x.com/2020/07/17/UsAF5F.th.png)

## 拦截器

### 拦截器的好处

1. 简化Action的实现，可以把很多功能从Action中独立出来，大量减少了Action的代码书写。
2. 功能更单一，把功能从Action中分离出来，分散到不同的拦截器里面，这样使每个拦截器和Action的功能更加单一，便于维护。
3. 通用代码模块化和重用性，把Action中的功能分离出来，放到拦截器去实现，这样就能实现多个Action中通用的代码进行模块化，多个Action公用一个拦截器
4. 实现AOP，Struts2通过拦截器实现了AOP（面向切面编程），AOP是一种分散实现关注功能的编程模式。
5. 拦截器将通用需求功能从不相关的Action之中分离出来，能够使得很多Action共享同一个行为，一旦行为发生变化，不必修改很多Action，只要修改这个行为就可以了。

### 什么是拦截器

1. Struts 2大多数核心功能是通过拦截器实现的，每个拦截器完成某项功能 
2. 拦截器方法在Action执行之前或者之后执行
3. 拦截器栈
	* 从结构上看，拦截器栈相当于多个拦截器的组合
	* 在功能上看，拦截器栈也是拦截器
4. 拦截器与过滤器原理很相似

为Action提供附加功能时，无需修改Action代码，使用拦截器来提供

### 拦截器工作原理

![](https://s1.ax1x.com/2020/07/17/UsAiUU.png)

三阶段执行周期：

1. 做一些Action执行前的预处理
2. 将控制交给后续拦截器或返回结果字符串
3. 做一些Action执行后的处理

### 拦截器示例

    public class MyTimerInterceptor extends AbstractInterceptor{	
    	public String intercept(ActionInvocation invocation) 
    						throws Exception {
    		//预处理工作
    		long startTime = System.currentTimeMillis();
      //执行后续拦截器或Action
    		String result = invocation.invoke();
      //后续处理工作
    	   long execTime = System.currentTimeMillis() - startTime;
     System.out.println("The interval time is "+execTime+" ms");
    		//返回结果字符串
      return result;
    	}
    }

## 自带拦截器

* Params拦截器:负责将请求参数设置为Action属性，把请求参数设置到相应的Action的属性去的，并自动进行类型转换。
* servletConfig拦截器:将源于Servlet API的各种对象注入到Action
* fileUpload拦截器:对文件上传提供支持
* exception拦截器:捕获异常，并且将异常映射到用户自定义的错误页面
* validation拦截器:调用验证框架进行数据验证 
* workflow拦截器:调用Action类的validate()，执行编码验证
* prepare拦截器，在Action执行之前调用Action的prepare()方法，这个方法是用来准备Action执行之前要做的工作。它要求我们的Action必需实现com.opensymphony.xwork2.Preparable接口，一般都是与ModelDriven一起使用
* modelDriven拦截器，如果Action实现ModelDriven接口，它将getModel()取得的模型对象存入OgnlValueStack中
* chain拦截器，将前一个执行结束的Action属性设置到当前的Action中。它被用在ResultType为“chain”所指定的结果的Action中，该结果Action对象会从值栈中获得前一个Action对应的属性，它实现Action链之间的数据传递
* validation拦截器，调用验证框架读取 *-validation.xml文件，并且应用在这些文件中声明的校验
* timer拦截器，记录ActionInvocation余下部分执行的时间，并做为日志信息记录下来，便于查找性能优化。
* token拦截器（令牌拦截器），核对当前Action请求（request）的有效标识，防止重复提交。使用标签<s:token>可以生成表单令牌，该标签会在session中设置一个预期的值并且在表单中创建一个隐藏的input字段。Token拦截器会检查这个令牌，如果不合法，将不会执行action，注意这个拦截器需要手工添加，还需要配置一个invalid.token的result。


### 预定义的拦截器栈

在struts-default.xml中给出了Struts2的默认的预定义拦截器栈defaultStack

    <interceptors>
    <interceptor>。。。。
    <interceptor-stack>。。。
    <default-interceptor-ref>。。。。
    </interceptors
    
* <interceptor>用来定义一个拦截器，仅仅是一个定义，还没有被Action来引用它。name属性作为唯一标志，而class属性就是这个拦截器的实现类的全类名。拦截器的实现类都应该是com.opensymphony.xwork2.interceptor.Interceptor这个接口的实现类
* <interceptor-stack>定义了一个拦截器栈，这个栈中可以引用其他已经定义好的拦截器。拦截器栈简化了动作类Action在引用拦截器时的操作。
因为大多数动作类Action在引用拦截器的时候都不会仅仅引用一个拦截器，而是引用一组拦截器。Action在引用的时候，只需要引用这个拦截器栈就可以了，而不是引用每一个拦截器


### 拦截器的调用顺序

（1）.要找它自己有没有声明拦截器的引用，即<action>元素有没有<interceptor-ref>子元素，如果有，则不用继续再找，如果没有，（2）。

（2)：在这个<action>所在的包有没有声明默认的拦截器引用，即<package>元素的<default-interceptor-ref>子元素, 如果有，则不用继续再找，直接使用这些拦截器，如果没有，(3)。

 (3)：递归地寻找当前包的父包有没有声明默认的拦截器引用，直到找到有拦截器引用就为止

注意：三者是覆盖关系


## 自定义拦截器

所有的拦截器都要实现com.opensymphony.xwork2.interceptor.Interceptor接口

    public interface Interceptor {
	    void destroy();
	    void init();
    String intercept(ActionInvocation invocation) throws Exception;
    }

### 配置struts.xml文件

    <package name="packName" extends="struts-default" namespace="/manage">
    	<interceptors>
    		<!-- 定义拦截器 -->
    		<interceptor name="interceptorName" class="interceptorClass" />
    		<!-- 定义拦截器栈 -->
    		<interceptor-stack name="interceptorStackName">
    			<!--指定引用的拦截器-->
    			<interceptor-ref name="interceptorName|interceptorStackName" />
    		</interceptor-stack>
    	</interceptors>
    	<!--定义默认的拦截器引用-->
    	<default-interceptor-ref name="interceptorName|interceptorStackName" />
    	<action name="actionName" class="actionClass">
    	  <!—为Action指定拦截器引用-->
    		<interceptor-ref name="interceptorName|interceptorStackName" />
    		<!--省略其他配置-->
    	</action>
    </package>

#### 实现Interceptor接口

1. void init()：初始化拦截器所需资源
2. void destroy()：释放在init()中分配的资源
3. String intercept(ActionInvocation ai) throws Exception
	1. 实现拦截器功能
	2. 利用ActionInvocation参数获取Action状态
	3. 返回结果码（result）字符串


#### 继承AbstractInterceptor类 

1. 提供了init()和destroy()方法的空实现
2. 只需要实现intercept方法即可
3. 推荐使用


## 登录验证

为开发一个权限验证拦截器来判断用户是否登录

1. 当用户请求受保护资源时，先检查用户是否登录
2. 如果没有登录，则向用户显示登录页面
3. 如果已经登录，则继续操作 

实现步骤

1. 开发权限验证拦截器
2. 在配置文件中定义拦截器并引用它

#### UserAction
    
    public String login(){  
	     Map map= ActionContext.getContext().getSession();
	     if("admin".equals(this.getUsername())&& 
	       "admin".equals(this.getPassword())){  
	       map.put("name",getUsername());   
	    return "success";  
	    }  
	    return "error";  
    }
    
#### ShowAction

    public String show() {  
     	return "success";  
     } 

#### Welcome.jsp

     <a href="show.action">show</a> 


#### Show This Page   

        登录后执行此页面<br> 

#### LoginInterceptor登录拦截器

    public class LoginInterceptor extends AbstractInterceptor{
     @Override  
     public String intercept(ActionInvocation invocation) throws Exception {  
	    // 取得请求相关的ActionContext实例  
	    ActionContext ctx = invocation.getInvocationContext();  
	    Map session = ctx.getSession();  
	     String username = (String) session.get("name");  
	    // 如果没有登陆，或者登陆的用户名不是admin，都返回重新登陆 
	    if (username != null && username.equals("admin")) {  
	    return invocation.invoke();  
	    }  
	    return Action.LOGIN;  
	    }  
    }
    
#### 无拦截器的登录验证

    <package name="noauthority" extends="struts-default" >
    <!-- 定义全局Result -->  
    <global-results>  
	    <!-- 当返回login视图名时，转入/login.jsp页面 -->  
	    <result name="login">/login.jsp</result>  
    </global-results>  
    <action name="login" class="com.lesson.action.UserAction" method="login">  
	    <result name="success">/welcome.jsp</result>  
	    <result name="error">/login.jsp</result>  
	    <result name="input">/login.jsp</result>  
    </action>  
    <action name="show" class="com.lesson.action.ShowAction">  
	    <result name="success">/show.jsp</result>  
    </action>
    </package>


#### 有拦截器的登录验证


    <package name="authority" extends="struts-default" >
     <interceptors> 
	     <interceptor name="authority"  
	    class="com.lesson.interceptor.LoginInterceptor">  
	    </interceptor>  
	    <!-- 拦截器栈 -->  
	    <interceptor-stack name="mydefault">  
	    <interceptor-ref name="defaultStack" />  
	    <interceptor-ref name="authority" />  
	    </interceptor-stack>  
    </interceptors> 
    	…..
     <action name="show" class="com.lesson.action.ShowAction">  
	    <result name="success">/show.jsp</result>  
	    <!-- 使用此拦截器 -->  
	    <interceptor-ref name="mydefault" />  
    </action>
    </package>
