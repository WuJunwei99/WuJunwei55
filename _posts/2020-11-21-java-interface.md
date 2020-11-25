---
layout: post
title: "抽象、接口"
date: 2020-11-21 09:12:25 +0800
categories: notes
tags: java
img: https://s1.ax1x.com/2020/07/12/U1oQN4.png
---
代码块、抽象类与抽象方法、接口(interface)、内部类

## 代码块

* 代码块(或初始化块)的作用：
	* 对Java类或对象进行初始化
* 代码块(或初始化块)的分类：
	* 一个类中代码块若有修饰符，则只能被static修饰，称为静态代码块(static block)，没有使用static修饰的，为非静态代码块。
* static代码块通常用于初始化static的属性

	class Person {
	public static int total;
	static {
	total = 100;//为total赋初值
	}
	…… //其它属性或方法声明
	}

静态代码块：用static修饰的代码块

1. 可以有输出语句。
2. 可以对类的属性、类的声明进行初始化操作。
3. 不可以对非静态的属性初始化。即：不可以调用非静态的属性和方法。
4. 若有多个静态的代码块，那么按照从上到下的顺序依次执行。
5. 静态代码块的执行要先于非静态代码块。
6. 静态代码块随着类的加载而加载，且只执行一次。

非静态代码块：没有static修饰的代码块

1. 可以有输出语句。 
2. 可以对类的属性、类的声明进行初始化操作。
3. 除了调用非静态的结构外，还可以调用静态的变量或方法。
4. 若有多个非静态的代码块，那么按照从上到下的顺序依次执行。
5. 每次创建对象的时候，都会执行一次。且先于构造器执行。

### 程序中成员变量赋值的执行顺序

![](https://s3.ax1x.com/2020/11/25/DaISC6.md.png)

## 抽象类与抽象方法

随着继承层次中一个个新子类的定义，类变得越来越具体，而父类则更一般，更通用。类的设计应该保证父类和子类能够共享特征。有时将一个父类设计得非常抽象，以至于它没有具体的实例，这样的类叫做抽象类。

* 用abstract关键字来修饰一个类，这个类叫做抽象类。
	* 此类不能实例化
	* 抽象类中一定有构造器，便于子类实例化时调用（涉及：子类对象实例化的全过程）
	* 开发中，都会提供抽象类的子类，让子类对象实例化，完成相关的操作
* 用abstract来修饰一个方法，该方法叫做抽象方法。
	* 抽象方法：只有方法的声明，没有方法的实现。以分号结束：
	* 比如：public abstract void talk(); 
	* 包含抽象方法的类，一定是一个抽象类。反之，抽象类中可以没有抽象方法的。
* 含有抽象方法的类必须被声明为抽象类。
* 抽象类不能被实例化。抽象类是用来被继承的，抽象类的子类必须重写父类的抽象方法，并提供方法体。若没有重写全部的抽象方法，仍为抽象类。
* 不能用abstract修饰变量、代码块、构造器；
* 不能用abstract修饰私有方法、静态方法、final的方法、final的类

### 抽象类应用

抽象类是用来模型化那些父类无法确定全部实现，而是由其子类提供具体实现的对象的类。

![](https://s3.ax1x.com/2020/11/25/DaICvD.md.png)

#### 解决方案

Java允许类设计者指定：超类声明一个方法但不提供实现，该方法的实现由子类提供。这样的方法称为抽象方法。有一个或更多抽象方法的类称为抽象类。

Vehicle是一个抽象类，有两个抽象方法。

	public abstract class Vehicle{
	public abstract double calcFuelEfficiency(); //计算燃料效率的抽象方法
	public abstract double calcTripDistance(); //计算行驶距离的抽象方法
	}
	public class Truck extends Vehicle{
	public double calcFuelEfficiency( ) { //写出计算卡车的燃料效率的具体方法 }
	public double calcTripDistance( ) { //写出计算卡车行驶距离的具体方法 } }
	public class RiverBarge extends Vehicle{
	public double calcFuelEfficiency( ) { //写出计算驳船的燃料效率的具体方法 }
	public double calcTripDistance( ) { //写出计算驳船行驶距离的具体方法} }

注：抽象类不能实例化，new Vihicle()是非法的


### 多态的应用：模板方法设计模式(TemplateMethod)

抽象类体现的就是一种模板模式的设计，抽象类作为多个子类的通用模板，子类在抽象类的基础上进行扩展、改造，但子类总体上会保留抽象类的行为方式。

解决的问题：

* 当功能内部一部分实现是确定的，一部分实现是不确定的。这时可以把不确定的部分暴露出去，让子类去实现。
* 换句话说，在软件开发中实现一个算法时，整体步骤很固定、通用，这些步骤已经在父类中写好了。但是某些部分易变，易变部分可以抽象出来，供不同子类实现。这就是一种模板模式。

		abstract class Template {
		public final void getTime() {
		long start = System.currentTimeMillis();
		code();
		long end = System.currentTimeMillis();
		System.out.println("执行时间是：" + (end - start));
		}
		public abstract void code();
		}
		class SubTemplate extends Template {
		public void code() {
		for (int i = 0; i < 10000; i++) {
		System.out.println(i);
		} } }

#### 应用实例

模板方法设计模式是编程中经常用得到的模式。各个框架、类库中都有他的影子，比如常见的有：

* 数据库访问的封装
* Junit单元测试
* JavaWeb的Servlet中关于doGet/doPost方法调用
* Hibernate中模板程序
* Spring中JDBCTemlate、HibernateTemplate等

## 接口(interface)

### 提出问题

* 一方面，有时必须从几个类中派生出一个子类，继承它们所有的属性和方法。但是，Java不支持多重继承。有了接口，就可以得到多重继承的效果。
* 另一方面，有时必须从几个类中抽取出一些共同的行为特征，而它们之间又没有is-a的关系，仅仅是具有相同的行为特征而已。例如：鼠标、键盘、打印机、扫描仪、摄像头、充电器、MP3机、手机、数码相机、移动硬盘等都支持USB连接。
* 接口就是规范，定义的是一组规则，体现了现实世界中“如果你是/要...则必须能...”的思想。继承是一个"是不是"的关系，而接口实现则是 "能不能"的关系。
* 接口的本质是契约，标准，规范，就像我们的法律一样。制定好后大家都要遵守。

### 概念与特点

* 接口(interface)是抽象方法和常量值定义的集合。
* 接口的特点：
	* 用interface来定义。
	* 接口中的所有成员变量都默认是由public static final修饰的。
	* 接口中的所有抽象方法都默认是由public abstract修饰的。
	* 接口中没有构造器。
	* 接口采用多继承机制。
	* 接口定义举例

![](https://s3.ax1x.com/2020/11/25/DaIp8K.md.png)


* 定义Java类的语法格式：先写extends，后写implements
	* class SubClass extends SuperClass implements InterfaceA{ }
* 一个类可以实现多个接口，接口也可以继承其它接口。
* 实现接口的类中必须提供接口中所有方法的具体实现内容，方可实例化。否则，仍为抽象类。
* 接口的主要用途就是被实现类实现。（面向接口编程）
* 与继承关系类似，接口与实现类之间存在多态性
* 接口和类是并列关系，或者可以理解为一种特殊的类。从本质上讲，接口是一种特殊的抽象类，这种抽象类中只包含常量和方法的定义(JDK7.0及之前)，而没有变量和方法的实现

![](https://s3.ax1x.com/2020/11/25/DaI9gO.md.png)

### 代理模式

代理模式是Java开发中使用较多的一种设计模式。代理设计就是为其他对象提供一种代理以控制对这个对象的访问

![](https://s3.ax1x.com/2020/11/25/Da5x4x.png)

		interface Network {
		public void browse();
		}

		// 被代理类
		class RealServer implements Network { @Override
		public void browse() {
		System.out.println("真实服务器上
		网浏览信息");
		} }

		// 代理类
		class ProxyServer implements Network {
		private Network network;
		public ProxyServer(Network network) {
		this.network = network; }
		public void check() {
		System.out.println("检查网络连接等操作");
		}

		public void browse() {
		check();
		network.browse();
		} }

		public class ProxyDemo {
		public static void main(String[] args) {
		Network net = new ProxyServer(newRealServer());
		net.browse();
		} }

#### 应用场景

* 安全代理：屏蔽对真实角色的直接访问。  远程代理：通过代理类处理远程方法调用（RMI）
* 延迟加载：先加载轻量级的代理对象，真正需要再加载真实对象

比如你要开发一个大文档查看软件，大文档中有大的图片，有可能一个图片有100MB，在打开文件时，不可能将所有的图片都显示出来，这样就可以使用代理模式，当需要查看图片时，用proxy来进行大图片的打开。

#### 分类

* 静态代理（静态定义代理类）
* 动态代理（动态生成代理类）
	* JDK自带的动态代理，需要反射等知识


### 工厂模式

工厂模式：实现了创建者与调用者的分离，即将创建对象的具体过程屏蔽隔离起来，达到提高灵活性的目的。

#### 分类

* 简单工厂模式：用来生产同一等级结构中的任意产品。（对于增加新的产品，需要修改已有代码）
* 工厂方法模式：用来生产同一等级结构中的固定产品。（支持增加任意产品）
* 抽象工厂模式：用来生产不同产品族的全部产品。（对于增加新的产品，无能为力；支持增加产品族）

#### 核心本质：

实例化对象，用工厂方法代替 new 操作。

将选择实现类、创建对象统一管理和控制。从而将调用者跟我们的实现类解耦。

#### 无工厂模式

	interface Car{
	void run();
	}
	class Audi implements Car{
	public void run() {
	System.out.println("奥迪在跑");
	} }
	class BYD implements Car{
	public void run() {
	System.out.println("比亚迪在跑");
	} }
	public class Client01 {
	public static void main(String[] args) {
	Car a = new Audi();
	Car b = new BYD();
	a.run();
	b.run();
	} }

#### 简单工厂模式

简单工厂模式，从命名上就可以看出这个模式一定很简单。它存在的目的很简单：定义一个用于创建对象的工厂类

	interface Car {
	void run();
	}
	class Audi implements Car {
	public void run() {
	System.out.println("奥迪在跑");
	} }
	class BYD implements Car {
	public void run() {
	System.out.println("比亚迪在跑");
	} }

	//工厂类
	class CarFactory {

	//方式一
	public static Car getCar(String type) {
	if ("奥迪".equals(type)) {
	return new Audi();
	} else if ("比亚迪".equals(type)) {
	return new BYD();
	} else {
	return null; } }

	//方式二
	// public static Car getAudi() {
	// return new Audi();
	// }
	//
	// public static Car getByd() {
	// return new BYD();
	// } }

	public class Client02 {
	public static void main(String[] args) {
	Car a = CarFactory.getCar("奥迪");
	a.run();
	Car b = CarFactory.getCar("比亚迪");
	b.run();
	} }

调用者只要知道他要什么，从哪里拿，如何创建，不需要知道。分工，多出了一个专门生产 Car 的实现类对象的工厂类。把调用者与创建者分离。

**小结**

简单工厂模式也叫静态工厂模式，就是工厂类一般是使用静态方法，通过接收的参数的不同来返回不同的实例对象。

缺点：对于增加新产品，不修改代码的话，是无法扩展的。违反了开闭原则（对扩展开放；对修改封闭）。


#### 工厂方法模式

为了避免简单工厂模式的缺点，不完全满足 OCP（对扩展开放，对修改关闭）。工厂方法模式和简单工厂模式最大的不同在于，简单工厂模式只有一个（对于一个项目或者一个独立的模块而言）工厂类，而工厂方法模式有一组实现了相同接口的工厂类。这样在简单工厂模式里集中在工厂方法上的压力可以由工厂方法模式里不同的工厂子类来分担。

	interface Car{
	void run();
	}

	//两个实现类
	class Audi implements Car{
	public void run() {
	System.out.println("奥迪在跑");
	} }
	class BYD implements Car{
	public void run() {
	System.out.println("比亚迪在跑");
	} }

	//工厂接口
	interface Factory{
	Car getCar();
	}

	//两个工厂类
	class AudiFactory implements Factory{
	public Audi getCar(){
	return new Audi();
	} }

	class BydFactory implements Factory{
	public BYD getCar(){
	return new BYD();
	} }

	public class Client {
	public static void main(String[] args) {
	Car a = new AudiFactory().getCar();
	Car b = new BydFactory().getCar();
	a.run();
	b.run();
	} }

**总结：**

简单工厂模式与工厂方法模式真正的避免了代码的改动了？

没有。在简单工厂模式中，新产品的加入要修改工厂角色中的判断语句；而在工厂方法模式中，要么将判断逻辑留在抽象工厂角色中，要么在客户程序中将具体工厂角色写死（就像上面的例子一样）。而且产品对象创建条件的改变必然会引起工厂角色的修改。面对这种情况，**Java 的反射机制与配置文件的巧妙结合突破了限制——这在Spring 中完美的体现了出来。**

#### 抽象工厂模式

抽象工厂模式和工厂方法模式的区别就在于需要创建对象的复杂程度上。而且抽象工厂模式是三个里面最为抽象、最具一般性的。抽象工厂模式的用意为：给客户端提供一个接口，可以创建多个产品族中的产品对象。

而且使用抽象工厂模式还要满足一下条件：

1) 系统中有多个产品族，而系统一次只可能消费其中一族产品。
2) 同属于同一个产品族的产品以其使用。


### Java 8中关于接口的改进

Java 8中，你可以为接口添加静态方法和默认方法。从技术角度来说，这是完全合法的，只是它看起来违反了接口作为一个抽象定义的理念。

静态方法：使用 static 关键字修饰。可以通过接口直接调用静态方法，并执行其方法体。我们经常在相互一起使用的类中使用静态方法。你可以在标准库中找到像Collection/Collections或者Path/Paths这样成对的接口和类。

默认方法：默认方法使用 default 关键字修饰。可以通过实现类对象来调用。我们在已有的接口中提供新方法的同时，还保持了与旧版本代码的兼容性。

比如：java 8 API中对Collection、List、Comparator等接口提供了丰富的默认
方法

		public interface AA {
		double PI = 3.14;
		public default void method() {
		System.out.println("北京");
		}
		default String method1() {
		return "上海";
		}
		public static void method2() {
		System.out.println(“hello lambda!");
		} }

		
		public static void main(String[] args) {
				SubClass s = new SubClass();
				
		//		s.method1();
		//		SubClass.method1();
				//知识点1：接口中定义的静态方法，只能通过接口来调用。
				CompareA.method1();
				//知识点2：通过实现类的对象，可以调用接口中的默认方法。
				//如果实现类重写了接口中的默认方法，调用时，仍然调用的是重写以后的方法
				s.method2();
				//知识点3：如果子类(或实现类)继承的父类和实现的接口中声明了同名同参数的默认方法，
				//那么子类在没有重写此方法的情况下，默认调用的是父类中的同名同参数的方法。-->类优先原则
				//知识点4：如果实现类实现了多个接口，而这多个接口中定义了同名同参数的默认方法，
				//那么在实现类没有重写此方法的情况下，报错。-->接口冲突。
				//这就需要我们必须在实现类中重写此方法
				s.method3();
				
			}
			
		}

		//知识点5：如何在子类(或实现类)的方法中调用父类、接口中被重写的方法
			public void myMethod(){
				method3();//调用自己定义的重写的方法
				super.method3();//调用的是父类中声明的
				//调用接口中的默认方法
				CompareA.super.method3();
				CompareB.super.method3();
			}

### 内部类

当一个事物的内部，还有一个部分需要一个完整的结构进行描述，而这个内部的完整的结构又只为外部事物提供服务，那么整个内部的完整结构最好使用内部类。

在Java中，允许一个类的定义位于另一个类的内部，前者称为内部类，后者称为外部类。

Inner class一般用在定义它的类或语句块之内，在外部引用它时必须给出完整的名称。

Inner class的名字不能与包含它的外部类类名相同；

**分类：** 

* 成员内部类（static成员内部类和非static成员内部类）
* 局部内部类（不谈修饰符）、匿名内部类


**成员内部类作为类的成员的角色：**

* 和外部类不同，Inner class还可以声明为private或protected；
* 可以调用外部类的结构
* Inner class 可以声明为static的，但此时就不能再使用外层类的非static的成员变量；

**成员内部类作为类的角色：**

* 可以在内部定义属性、方法、构造器等结构
* 可以声明为abstract类 ，因此可以被其它的内部类继承
* 可以声明为final的  编译以后生成OuterClass$InnerClass.class字节码文件（也适用于局部内部类）

 【注意】

1. 非static的成员内部类中的成员不能声明为static的，只有在外部类或static的成员内部类中才可声明static成员。
2. 外部类访问成员内部类的成员，需要“内部类.成员”或“内部类对象.成员”的方式
3. 成员内部类可以直接使用外部类的所有成员，包括私有的数据
4. 当想要在外部类的静态成员部分使用内部类时，可以考虑内部类声明为静态的


#### 举例
		
		class Outer {
		private int s;
		public class Inner {
		public void mb() {
		s = 100;
		System.out.println("在内部类Inner中s=" + s);
		} }
		public void ma() {
		Inner i = new Inner();
		i.mb();
		} }
		public class InnerTest {
		public static void main(String args[]) {
		Outer o = new Outer();
		o.ma();
		} }


		public class Outer {
		private int s = 111;
		public class Inner {
		private int s = 222;
		public void mb(int s) {
		System.out.println(s); // 局部变量s
		System.out.println(this.s); // 内部类对象的属性s
		System.out.println(Outer.this.s); // 外部类对象属性s } }
		public static void main(String args[]) {
		Outer a = new Outer();
		Outer.Inner b = a.new Inner();
		b.mb(333);
		} }

#### 如何声明局部内部类

		class 外部类{
		方法(){
		class 局部内部类{ } }{
		class 局部内部类{ } } }

#### 如何使用局部内部类

* 只能在声明它的方法或代码块中使用，而且是先声明后使用。除此之外的任何地方都不能使用该类
* 但是它的对象可以通过外部方法的返回值返回使用，返回值类型只能是局部内部类的父类或父接口类型

#### 局部内部类的特点

* 内部类仍然是一个独立的类，在编译之后内部类会被编译成独立的.class文件，但是前面冠以外部类的类名和$符号，以及数字编号。
* 只能在声明它的方法或代码块中使用，而且是先声明后使用。除此之外的任何地方都不能使用该类。
* 局部内部类可以使用外部类的成员，包括私有的。
* 局部内部类可以使用外部方法的局部变量，但是必须是final的。由局部内部类和局部变量的声明周期不同所致。
* 局部内部类和局部变量地位类似，不能使用public,protected,缺省,private
* 局部内部类不能使用static修饰，因此也不能包含静态成员

### 匿名内部类

* 匿名内部类不能定义任何静态成员、方法和类，只能创建匿名内部类的一个实例。一个匿名内部类一定是在new的后面，用其隐含实现一个接口或实现一个类。
* 格式：

	new 父类构造器（实参列表）|实现接口(){
	//匿名内部类的类体部分
	}

* 匿名内部类的特点
	* 匿名内部类必须继承父类或实现接口
	* 匿名内部类只能有一个对象
	* 匿名内部类对象只能使用多态形式引用

			interface A{
			public abstract void fun1();
			}
			public class Outer{
			public static void main(String[] args) {
			new Outer().callInner(new A(){
			//接口是不能new但此处比较特殊是子类对象实现接口，只不过没有为对象取名
			public void fun1() {
			System.out.println(“implement for fun1");
			}
			});// 两步写成一步了
			}
			public void callInner(A a) {
			a.fun1();
			}
			}

