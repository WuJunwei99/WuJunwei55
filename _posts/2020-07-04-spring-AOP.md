---
layout: post
title: "Spring 学习笔记（6）——AOP切面编程"
date: 2020-07-04 13:22:18 +0800
categories: notes
tags: spring
img: https://s1.ax1x.com/2020/07/02/NqzFiV.png
---
AOP切面编程介绍；专业术语；Spring实现AOP简单切面编程；切入点表达式；代理对象；执行顺序；基于xml配置aop程序


## 介绍

### 什么是AOP

AOP是面向切面编程。全称：Aspect Oriented Programming

面向切面编程指的是：程序是运行期间，动态地将某段代码插入到原来方法代码的某些位置中（前面、后面等）。这就叫面向切面编程。

## 日记处理


### 一个简单计算数功能加日记

    public interface Calculate {
    
    	public int add(int num1, int num2);
    
    	public int add(int num1, int num2, int num3);
    
    	public int div(int num1, int num2);
    	
    }
    
    
    public class Calculator implements Calculate {
    
    	@Override
    	public int add(int num1, int num2) {
    		int result = num1 + num2;
    		return result;
    	}
    
    	@Override
    	public int add(int num1, int num2, int num3) {
    		int result = num1 + num2 + num3;
    		return result;
    	}
    
    	@Override
    	public int div(int num1, int num2) {
    		int result = num1 / num2;
    		return result;
    	}
    
    }

![](https://s1.ax1x.com/2020/07/04/NzbfJ0.md.png)

### 原始方法统一日记处理

工具类

    public class LogUtils {
    
    	public static void logBefore(String method, Object... args) {
    		System.out.println("前置日记是：当前是【" + method + "】操作，参数是：" + Arrays.asList(args));
    	}
    
    	public static void logAfter(String method, Object... args) {
    		System.out.println("后置日记是：当前是【" + method + "】操作，参数是：" + Arrays.asList(args));
    	}
    
    	public static void logAfterThrowing(String method, Exception e) {
    		System.out.println("异常日记是：当前是【" + method + "】操作，异常是：" + e);
    	}
    
    	public static void logAfterReturning(String method, Object result) {
    		System.out.println("返回日记是：当前是【" + method + "】操作，结果是：" + result);
    	}
    
    }

加日记后的计算器：

    public class Calculator implements Calculate {
    
    	@Override
    	public int add(int num1, int num2) {
    		int result = 0;
    		try {
    			try {
    				LogUtils.logBefore("add", num1, num2);
    				result = num1 + num2;
    			} finally {
    				LogUtils.logAfter("add", num1, num2);
    			}
    			LogUtils.logAfterReturning("add", result);
    		} catch (Exception e) {
    			LogUtils.logAfterThrowing("add", e);
    			throw e;
    		}
    		return result;
    	}
    
    	@Override
    	public int add(int num1, int num2, int num3) {
    		int result = 0;
    		try {
    			try {
    				LogUtils.logBefore("add", num1, num2, num3);
    				result = num1 + num2 + num3;
    			} finally {
    				LogUtils.logAfter("add", num1, num2, num3);
    			}
    			LogUtils.logAfterReturning("add", result);
    		} catch (Exception e) {
    			LogUtils.logAfterThrowing("add", e);
    			throw e;
    		}
    		return result;
    	}
    
    	@Override
    	public int div(int num1, int num2) {
    		int result = 0;
    		try {
    			try {
    				LogUtils.logBefore("div", num1, num2);
    				result = num1 / num2;
    			} finally {
    				LogUtils.logAfter("div", num1, num2);
    			}
    			LogUtils.logAfterReturning("add", result);
    		} catch (Exception e) {
    			LogUtils.logAfterThrowing("div", e);
    			throw e;
    		}
    		return result;
    	}
    
    }

![](https://s1.ax1x.com/2020/07/04/NzbhWV.md.png)

![](https://s1.ax1x.com/2020/07/04/Nzb4zT.md.png)

## 实现日记

### 使用jdk动态代理统一日记（尽量掌握）

    public class JdkProxyFactory {
    
    	public static Object createJdkProxy(Object target) {
    		// 创建一个jdk动态代理对象
    		/**
    		 * 代理还可以在原有功能的基本上，添加更多功能。而不改变原来的代码。
    		 */
    		return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(),
    				new InvocationHandler() {
    
    					@Override
    					public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    						// result用来接收目标方法的返回值
    						Object result = null;
    						try {
    							try {
    								// 前置通知
    								LogUtils.logBefore(method.getName(), args);
    								// 调用目标对象的方法
    								result = method.invoke(target, args);
    							} finally {
    								// 后置通知
    								LogUtils.logAfter(method.getName(), args);
    							}
    							// 返回通知
    							LogUtils.logAfterReturning(method.getName(), result);
    						} catch (Exception e) {
    							// 异常通知
    							LogUtils.logAfterThrowing(method.getName(), e);
    							throw e;
    						}
    						// 返回目标对象的返回值
    						return result;
    					}
    				});
    
    	}
    
    	public static void main(String[] args) {
    
    		Calculate target = new Calculator();
    		// jdk动态代理要求被代理的对象必须有接口
    		// Jdk动态代理创建出来的代理对象，会实现所有目标对象的接口
    		Calculator proxy = (Calculator) createJdkProxy(target);
    
    //		System.out.println( proxy instanceof Calculate );
    //		System.out.println( proxy instanceof Calculator );
    		
    		System.out.println(proxy.add(100, 100));
    		System.out.println(proxy.div(100, 0));
    	}
    
    }

优点：这种方式已经解决我们前面所有日记需要的问题。非常的灵活。而且可以方便的在后期进行维护和升级。

缺点：当然使用jdk动态代理，需要有接口。如果没有接口。就无法使用jdk动态代理。

![](https://s1.ax1x.com/2020/07/04/NzbIQU.md.png)

![](https://s1.ax1x.com/2020/07/04/NzbWiq.md.png)

### 使用cglib代理（了解知道）

cglib代理可以在目标对象没有接口的情况下。实现代理。从而达到增强的效果。

Cglib代理其实是在一个指定类的基本上，创建一个类去继承指定类的技术。从而实现增强的效果

    public class CglibProxyFactory {
    
    	public static void main(String[] args) {
    		Calculator target = new Calculator();
    		// jdk动态代理要求被代理的对象必须有接口
    		// Jdk动态代理创建出来的代理对象，会实现所有目标对象的接口
    		Calculator proxy = (Calculator) createCglibProxy(target);
    
    		System.out.println( proxy instanceof Calculate );
    		System.out.println( proxy instanceof Calculator );
    		
    //		System.out.println(proxy.add(100, 100));
    //		System.out.println(proxy.div(100, 0));
    	}
    	
    	
    	public static Object createCglibProxy(Object target) {
    
    		// cglib是通过创建一个类去继承指定的类，达到增强效果
    
    		// 这是Cglibr工具类，可以用来产生代理对象
    		Enhancer enhancer = new Enhancer();
    		// 设置需要继承的类
    		enhancer.setSuperclass(target.getClass());
    		// 设置一个方法拦截器
    		enhancer.setCallback(new MethodInterceptor() {
    			/**
    			 * 在这里给原有方法做增强操作 第一个参数是代理对象<br/>
    			 * 第二个参数是目标方法的反射类<br/>
    			 * 第三个参数是调用方法时传入的参数<br/>
    			 * 第四个参数是调用代理的方法的反射类<br/>
    			 */
    			@Override
    			public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy)
    					throws Throwable {
    
    				// result用来接收目标方法的返回值
    				Object result = null;
    				try {
    					try {
    						// 前置通知
    						LogUtils.logBefore(method.getName(), args);
    						// 调用目标对象的方法
    						result = method.invoke(target, args);
    					} finally {
    						// 后置通知
    						LogUtils.logAfter(method.getName(), args);
    					}
    					// 返回通知
    					LogUtils.logAfterReturning(method.getName(), result);
    				} catch (Exception e) {
    					// 异常通知
    					LogUtils.logAfterThrowing(method.getName(), e);
    					throw e;
    				}
    				// 返回目标对象的返回值
    				return result;
    
    			}
    		});
    		
    		// 创建cglib代理对象
    		return enhancer.create();
    
    	}
    
    }

![](https://s1.ax1x.com/2020/07/04/NzbTL4.md.png)

优点：在没有接口的情况下，同样可以实现代理的效果。

缺点：同样需要自己编码实现代理全部过程。

但是为了更好的整合Spring框架使用。所以我们需要学习一下Spring 的AOP 功能。也就是学习Spring提供的AOP功能。

Spring AOP 底层就是使用的代理。而且不需要我们自己写。配置就可以了


## AOP编程的专业术语

**通知(Advice)**

通知就是增强的代码。比如前置增强的代码。后置增强的代码。异常增强代码。这些就叫通知

**切面(Aspect)**

切面就是包含有通知代码的类叫切面。

**横切关注点**

横切关注点，就是我们可以添加增强代码的位置。比如前置位置，后置位置，异常位置。和返回值位置。这些都叫横切关注点。

**目标(Target)**

目标对象就是被关注的对象。或者被代理的对象。

**代理(Proxy)**

为了拦截目标对象方法，而被创建出来的那个对象，就叫做代理对象。


**连接点(Joinpoint)**

连接点指的是横切关注点和程序代码的连接，叫连接点。

**切入点(pointcut)**

切入点指的是用户真正处理的连接点，叫切入点。

在Spring中切入点通过org.springframework.aop.Pointcut 接口进行描述，它使用类和方法作为连接点的查询条件。

图解AOP专业术语：

![](https://s1.ax1x.com/2020/07/04/NzboyF.md.png)

## 使用Spring实现AOP简单切面编程

java代码

    @Component
    public class Calculator implements Calculate {
    
    
    @Component
    // @Aspect 表示我是一个切面类
    @Aspect
    public class LogUtils {
    	/**
    	 * @Before 表示前置通知<br/>
    	 * 	value 需要写上切入点表达式
    	 */
    	@Before(value="execution(public int com.atguigu.pojo.Calculator.add(int, int))")
    	public static void logBefore() {
    		System.out.println("前置日记是：当前是【】操作，参数是：");
    	}
    }

applicationContext.xml配置


	<context:component-scan base-package="com.atguigu"></context:component-scan>
	<!-- 
		aop:aspectj-autoproxy 支持注解配置AOP切面编程（从而自动实现代理功能）
		@Aspect 就可以生效
	 -->
	<aop:aspectj-autoproxy></aop:aspectj-autoproxy>

测试的代码：

    public class SpringTest {
    
    	@Test
    	public void test1() throws Exception {
    		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
    		
    		Calculate calculate = (Calculate) applicationContext.getBean("calculator");
    		
    		calculate.add(100, 100);
    	}
    	
    }

![](https://s1.ax1x.com/2020/07/04/NzbHeJ.md.png)

## Spring的切入点表达式

切入点表达式，表示了当前通知对哪些方法生效。

@PointCut切入点表达式语法格式是：

    execution(访问权限 返回值类型 方法全限定名(参数类型列表))

    execution(public int com.atguigu.pojo.Calculator.add(int, int))
    public			方法的访问权限 
    int 			返回值类型
    com.atguigu.pojo.Calculator.add			包名+类名+方法名
    (int, int)				参数类型列表

### 限定符

*表示任意的意思：

* 匹配某全类名下，任意或多个方法。
	* public int com.atguigu.pojo.Calculator.*(int, int)
	* 以上星的意思，是方法名任意
* 在Spring中只有public权限能拦截到，访问权限可以省略（访问权限不能写*）。
	* execution(public int com.atguigu.pojo.Calculator.add(int, int))
	* 省略之后是：execution(int com.atguigu.pojo.Calculator.add(int, int))
* 匹配任意类型的返回值，可以使用 * 表示
	* execution(public * com.atguigu.pojo.Calculator.add(int, int))表示任意的返回值类型都可以匹配
* 匹配任意子包
	* execution(public int com.atguigu.*.Calculator.add(int, int))
	* 以上星表示com.atguigu.任意子包（一层）
* 任意类型参数
	* execution(public int com.atguigu.pojo.Calculator.add(int, *))
	* 以上星表示第二个参数是任意参数

..：可以匹配多层路径，或任意多个任意类型参数

* 任意层级的包
	* execution(public int com..pojo.Calculator.add(int, *))
	* 以上点点表示:com和pojo中间是任意层级的包
* 任意个任意类型的参数
	* execution(public int com.atguigu.pojo.Calculator.add(..))
	* 以上点点表示任意个任意类型的参数

### 模糊匹配

全部的方法都会被选上：

// 表示任意返回值，任意方法全限定符，任意参数

execution(* *(..))

// 表示任意返回值，任意包名+任意方法名，任意参数

execution(* *.*(..))

### 精确匹配

execution(public int com.atguigu.pojo.Calculator.add(int,int))

* 明确方法访问权限
* 返回值类型
* 包名
* 类名
* 方法名
* 以及参数个数和参数类型

### 切入点表达式连接：&& 、|| 

// 表示需要同时满足两个表达式

	@Before("execution(public int com.atguigu.aop.Calculator.add(int, int))"
			+ " && "+ "execution(public * com.atguigu.aop.Calculator.add(..))")

// 表示两个条件只需要满足一个，就会被匹配到

	@Before("execution(public int com.atguigu.aop.Calculator.add(int, int))"
			+ " || "
			+ "execution(public * com.atguigu.aop.Calculator.a*(int))")



execution( public * com.atguigu..*Service.*(..) ) 做事务管理用

## Spring切面中的代理对象

在Spring中，可以对有接口的对象和无接口的对象分别进行代理。在使用上有些细微的差别。

1)	如果被代理的对象实现了接口。在获取对象的时候，必须要以接口来接收返回的对象。

![](https://s1.ax1x.com/2020/07/04/NzqX7j.md.png)

2)	如果被代理对象，如果没有实现接口。获取对象的时候使用对象类型本身 

![](https://s1.ax1x.com/2020/07/04/NzqOBQ.md.png)

## Spring通知的执行顺序

    @Component
    // @Aspect 表示我是一个切面类
    @Aspect
    public class LogUtils {
    	/**
    	 * @Before 表示前置通知<br/>
    	 * value 需要写上切入点表达式
    	 */
    	@Before(value = "execution(public int com.atguigu.pojo.Calculator.*(int,int))")
    	public static void logBefore() {
    		System.out.println("前置日记是：当前是【】操作，参数是：");
    	}
    
    	/**
    	 * @After 后置通知<br/>
    	 */
    	@After(value = "execution(public int com.atguigu.pojo.Calculator.*(int,int))")
    	public static void logAfter() {
    		System.out.println("后置日记是：当前是【】操作，参数是：");
    	}
    
    	/**
    	 * @AfterThrowing异常通知<br/>
    	 */
    	@AfterThrowing(value = "execution(public int com.atguigu.pojo.Calculator.*(int,int))")
    	public static void logAfterThrowing() {
    		System.out.println("异常日记是：当前是【】操作，异常是：");
    	}
    
    	/**
    	 * @AfterReturning 是返回通知
    	 */
    	@AfterReturning(value = "execution(public int com.atguigu.pojo.Calculator.*(int,int))")
    	public static void logAfterReturning() {
    		System.out.println("返回日记是：当前是【】操作，结果是：");
    	}
    
    }

Spring通知的执行顺序是:

正常情况：

前置通知====>>>>后置通知=====>>>>返回值之后

![](https://s1.ax1x.com/2020/07/04/Nzbbw9.md.png)

异常情况：

前置通知====>>>>后置通知=====>>>>抛异常通知

![](https://s1.ax1x.com/2020/07/04/NzbqoR.md.png)

## 获取连接点信息

JoinPoint 是连接点的信息。

只需要在通知方法的参数中，加入一个JoinPoint参数。就可以获取到拦截方法的信息。

注意：是org.aspectj.lang.JoinPoint这个类。


	/**
	 * @Before 表示前置通知<br/>
	 *         value 需要写上切入点表达式
	 */
	@Before(value = "execution(public int com.atguigu.pojo.Calculator.*(int,int))")
	public static void logBefore(JoinPoint jp) {
		// jp.getSignature().getName() 获取方法名
		// jp.getArgs() 调用方法时的参数
		System.out.println("前置日记是：当前是【" + jp.getSignature().getName() + "】操作，参数是：" + Arrays.asList(jp.getArgs()));
	}

![](https://s1.ax1x.com/2020/07/04/NzbOF1.md.png)

## 获取拦截方法的返回值和抛的异常信息

#### 获取方法返回的值分为两个步骤：

1、在返回值通知的方法中，追加一个参数 Object result

2、然后在@AfterReturning注解中添加参数returning="参数名"


	/**
	 * @AfterReturning 是返回通知
	 * 	1、在返回通知上添加一个Object result的参数<br/>
	 *  2、在返回通知注解上使用属性returning = "result"标明哪个参数接收返回值
	 */
	@AfterReturning(value = "execution(public int com.atguigu.pojo.Calculator.*(int,int))", returning = "result")
	public static void logAfterReturning(JoinPoint jp, Object result) {
		System.out.println("返回日记是：当前是【" + jp.getSignature().getName() + "】操作，结果是：" + result);
	}

#### 获取方法抛出的异常分为两个步骤：

1、在异常通知的方法中，追加一个参数Exception exception

2、然后在@AfterThrowing 注解中添加参数 throwing="参数名"

	/**
	 * @AfterThrowing异常通知<br/>
	 * 	1、在异常通知参数中添加一个Exception e的参数接收抛出的异常<br/>
	 *  2、在异常通知注解中使用属性throwing="参数名"
	 */
	@AfterThrowing(value = "execution(public int com.atguigu.pojo.Calculator.*(int,int))",throwing="e")
	public static void logAfterThrowing(JoinPoint jp,Exception e) {
		System.out.println("异常日记是：当前是【" + jp.getSignature().getName() + "】操作，异常是：" + e);
	}

![](https://s1.ax1x.com/2020/07/04/NzbXJx.md.png)

## Spring的环绕通知

1. 环绕通知使用@Around注解。
2. 环绕通知如果和其他通知同时执行。环绕通知会优先于其他通知之前执行。
3. 环绕通知一定要有返回值（环绕如果没有返回值。后面的其他通知就无法接收到目标方法执行的结果）。
4. 在环绕通知中。如果拦截异常。一定要往外抛。否则其他的异常通知是无法捕获到异常的。

	/**
	 * @throws Throwable 
	 * @Around 环绕通知 （环绕通知需要自己执行目标方法）
	 * 1、环绕通知优先于普通通知先执行。
	 * 2、环绕通知一定要把目标方法的返回值返回
	 * 3、环绕通知收到异常后，一定要往外抛，否则普通异常通知收不到。
	 */
	@Around(value="execution(public int com.atguigu.pojo.Calculator.*(int,int))")
	public static Object around(ProceedingJoinPoint pjp) throws Throwable {
		
		Object result = null;
		try {
			try {
				System.out.println("环绕 的 前置通知");
				// 执行目标方法
				result = pjp.proceed();
				
			} finally {
				System.out.println("环绕 的 后置通知");
			}
			System.out.println("环绕 的 返回值通知");
		} catch (Exception e) {
			System.out.println("环绕 的 异常通知");
			throw e;
		}
		return result;
	}

![](https://s1.ax1x.com/2020/07/04/NzbxSK.md.png)

## 切入点表达式的复用

//	1、定义一个空的静态方法

//	2、使用@Pointcut注解定义切入点表达式

//	3、在需要复用切入点表达式的地方，使用方法调用代替

	@Pointcut("execution(public int com.atguigu.pojo.Calculator.*(int,int))")
	public static void pointcut1() {}
	
	/**
	 * @Before 表示前置通知<br/>
	 *         value 需要写上切入点表达式
	 */
	@Before(value = "pointcut1()")
	public static void logBefore(JoinPoint jp) {
		// jp.getSignature().getName() 获取方法名
		// jp.getArgs() 调用方法时的参数
		System.out.println("前置日记是：当前是【" + jp.getSignature().getName() + "】操作，参数是：" + Arrays.asList(jp.getArgs()));
	}

![](https://s1.ax1x.com/2020/07/04/NzbzQO.md.png)

## 多个通知的执行顺序

当我们有多个切面，多个通知的时候：

1. 通知的执行顺序默认是由切面类的字母先后顺序决定。
2. 在切面类上使用@Order注解决定通知执行的顺序（值越小，越先执行）

![](https://s1.ax1x.com/2020/07/04/NzbjW6.md.png)

## 如何基于xml配置aop程序

    
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
    	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
    	xmlns:aop="http://www.springframework.org/schema/aop"
    	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd
    		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.3.xsd">
    
    	<!-- 配置目标对象 -->
    	<bean id="calculator" class="com.atguigu.pojo.Calculator"></bean>
    	<!-- 配置切面类实例 -->
    	<bean id="logUtils" class="com.atguigu.pojo.utils.LogUtils"></bean>
    	
    	<!-- 配置aop切面编程 -->
    	<aop:config>
    		<!-- 配置谁是切面类
    				ref="logUtils" 表示logUtils的id为切面类
    		 -->
    		<aop:aspect id="aspect1" ref="logUtils">
    			<!-- 定义可复用的切入点表达式 -->
    			<aop:pointcut expression="execution(public int com.atguigu.pojo.Calculator.*(..))" id="pointcut1"/>
    			<!-- 配置前置通知
    					method 配置前置通知方法
    					pointcut="execution(public int com.atguigu.pojo.Calculator.*(..))" 配置切入点表达式
    			 -->
    			<aop:before method="logBefore" 
    				pointcut="execution(public int com.atguigu.pojo.Calculator.*(..))"/>
    			<!-- 
    				aop:after 配置后置通知
    					method 配置后置通知方法
    			 -->
    			<aop:after method="logAfter" pointcut-ref="pointcut1"/>
    			<!-- 配置异常通知 -->
    			<aop:after-throwing method="logAfterThrowing" pointcut-ref="pointcut1" throwing="e"/>
    			<!-- 配置返回通知 -->
    			<aop:after-returning method="logAfterReturning" pointcut-ref="pointcut1" returning="result"/>
    		</aop:aspect>
    	</aop:config>
    	
    </beans>

![](https://s1.ax1x.com/2020/07/04/NzqSyD.md.png)