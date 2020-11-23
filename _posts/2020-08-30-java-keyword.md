---
layout: post
title: "关键字和包装类的使用"
date: 2020-08-30 12:28:52 +0800
categories: notes
tags: java
img: https://s1.ax1x.com/2020/07/12/U1oQN4.png
---
this的使用、package、super、import的使用、Object类的使用、包装类的使用、static、final、main方法


## this的使用

在Java中，this关键字比较难理解，它的作用和其词义很接近

* 它在方法内部使用，即这个方法所属对象的引用；
* 它在构造器内部使用，表示该构造器正在初始化的对象。

this可以调用类的属性、方法和构造器

当在方法内需要用到调用该方法的对象时，就用this。

具体的：我们可以用this来区分属性和局部变量。

比如：this.name = name;

![](https://s1.ax1x.com/2020/08/29/dHJ9I0.md.png)

![](https://s1.ax1x.com/2020/08/29/dHJSZn.md.png)

注意：

* 可以在类的构造器中使用"this(形参列表)"的方式，调用本类中重载的其他构造器
* 明确：构造器中不能通过"this(形参列表)"的方式调用自身构造器
* 如果一个类中声明了n个构造器，则最多有 n - 1个构造器中使用了"this(形参列表)"
* "this(形参列表)"必须声明在类的构造器的首行！
* 在类的一个构造器中，最多只能声明一个"this(形参列表)"

## package、import的使用

### package

package语句作为Java源文件的第一条语句，指明该文件中定义的类所在的包。(若缺省该语句，则指定为无名包)。它的格式为：

package 顶层包名.子包名 ;


包对应于文件系统的目录，package语句中，用 “.” 来指明包(目录)的层次；

包通常用小写单词标识。通常使用所在公司域名的倒置：com.atguigu.xxx

#### 作用

* 包帮助管理大型软件系统：将功能相近的类划分到同一个包中。比如：MVC的设计模式
* 包可以包含类和子包，划分项目层次，便于管理
* 解决类命名冲突的问题
* 控制访问权限

![](https://s1.ax1x.com/2020/08/29/dHJpaq.md.png)



### JDK中主要的包介绍

1. java.lang----包含一些Java语言的核心类，如String、Math、Integer、 System和Thread，提供常用功能
2. java.net----包含执行与网络相关的操作的类和接口
3. java.io ----包含能提供多种输入/输出功能的类
4. java.util----包含一些实用工具类，如定义系统特性、接口的集合框架类、使用与日期日历相关的函数。
5. java.text----包含了一些java格式化相关的类‘
6. java.sql----包含了java进行JDBC数据库编程的相关类/接口
7. java.awt----包含了构成抽象窗口工具集（abstract window toolkits）的多个类，这些类被用来构建和管理应用程序的图形用户界面(GUI)。 B/S C/S

### import

为使用定义在不同包中的Java类，需用import语句来引入指定包层次下所需要的类或全部类(.*)。import语句告诉编译器到哪里去寻找类

注意：

1. 在源文件中使用import显式的导入指定包下的类或接口
2. 声明在包的声明和类的声明之间。
3. 如果需要导入多个类或接口，那么就并列显式多个import语句即可
4. 举例：可以使用java.util.*的方式，一次性导入util包下所有的类或接口。
5. 如果导入的类或接口是java.lang包下的，或者是当前包下的，则可以省略此import语句。
6. 如果在代码中使用不同包下的同名的类。那么就需要使用类的全类名的方式指明调用的
是哪个类。
7. 如果已经导入java.a包下的类。那么如果需要使用a包的子包下的类的话，仍然需要导入。
8. import static组合的使用：调用指定类或接口下的静态的属性或方法

## super

在Java类中使用super来调用父类中的指定操作：

* super可用于访问父类中定义的属性
* super可用于调用父类中定义的成员方法
* super可用于在子类构造器中调用父类的构造器

注意：

* 尤其当子父类出现同名成员时，可以用super表明调用的是父类中的成员
* super的追溯不仅限于直接父类
* super和this的用法相像，this代表本类对象的引用，super代表父类的内存空间的标识

![](https://s1.ax1x.com/2020/09/11/wUFV8U.png)

### 调用父类的构造器

* 子类中所有的构造器默认都会访问父类中空参数的构造器
* 当父类中没有空参数的构造器时，子类的构造器必须通过this(参数列表)或者super(参数列表)语句指定调用本类或者父类中相应的构造器。同时，只能”二选一”，且必须放在构造器的首行
* 如果子类构造器中既未显式调用父类或本类的构造器，且父类中又没有无参的构造器，则编译出错
* 在类的多个构造器中，至少有一个类的构造器中使用了"super(形参列表)"，调用父类中的构造器

### this和super的区别

![](https://s1.ax1x.com/2020/09/11/wUFECT.png)

## Object类的使用

![](https://s1.ax1x.com/2020/09/11/wUizvQ.md.png)

### Object类中的主要结构

![](https://s1.ax1x.com/2020/09/11/wUiv8S.png)

### ==操作符与equals方法

基本类型比较值:只要两个变量的值相等，即为true

    int a=5; if(a==6){…} 

引用类型比较引用(是否指向同一个对象)：只有指向同一个对象时，==才返回true。

    Person p1=new Person();
    Person p2=new Person();
    if (p1==p2){…}

用“==”进行比较时，符号两边的数据类型必须兼容(可自动转换的基本数据类型除外)，否则编译出错

* equals()：所有类都继承了Object，也就获得了equals()方法。还可以重写。 
	* 只能比较引用类型，其作用与“==”相同,比较是否指向同一个对象。 
	* 格式:obj1.equals(obj2)
* 特例：当用equals()方法进行比较时，对类File、String、Date及包装类（Wrapper Class）来说，是比较类型及内容而不考虑引用的是否是同一个对象；
	* 原因：在这些类中重写了Object类的equals()方法。
* 当自定义使用equals()时，可以重写。用于比较两个对象的“内容”是否都相等

### 重写equals()方法的原则

* 对称性：如果x.equals(y)返回是“true”，那么y.equals(x)也应该返回是“true”。
* 自反性：x.equals(x)必须返回是“true”。
* 传递性：如果x.equals(y)返回是“true”，而且y.equals(z)返回是“true”，那么z.equals(x)也应该返回是“true”。 
* 一致性：如果x.equals(y)返回是“true”，只要x和y内容一直不变，不管你重复x.equals(y)多少次，返回都是“true”。 
* 任何情况下，x.equals(null)，永远返回是“false”； x.equals(和x不同类型的对象)永远返回是“false”。

### ==和equals的区别

1. == 既可以比较基本类型也可以比较引用类型。对于基本类型就是比较值，对于引用类型就是比较内存地址
2. equals的话，它是属于java.lang.Object类里面的方法，如果该方法没有被重写过默认也是==;我们可以看到String等类的equals方法是被重写过的，而且String类在日常开发中用的比较多，久而久之，形成了equals是比较值的错误观点。
3. 具体要看自定义类里有没有重写Object的equals方法来判断。
4. 通常情况下，重写equals方法，会比较类中的相应属性是否都相等。

![](https://s1.ax1x.com/2020/09/11/wUiO4f.png)

### toString() 方法

* toString()方法在Object类中定义，其返回值是String类型，返回类名和它的引用地址。
* 在进行String与其它类型数据的连接操作时，自动调用toString()方法
    
    Date now=new Date();
    System.out.println(“now=”+now); 相当于
    System.out.println(“now=”+now.toString()); 

* 可以根据需要在用户自定义类型中重写toString()方法

如String 类重写了toString()方法，返回字符串的值。 

    s1=“hello”;
    System.out.println(s1);//相当于System.out.println(s1.toString()); 

* 基本类型数据转换为String类型时，调用了对应包装类的toString()方法
	* int a=10; System.out.println(“a=”+a);

## 包装类的使用

* 针对八种基本数据类型定义相应的引用类型—包装类（封装类）
* 有了类的特点，就可以调用类中的方法，Java才是真正的面向对象

![](https://s1.ax1x.com/2020/09/11/wUiqEt.png)

基本数据类型包装成包装类的实例 ---装箱

* 通过包装类的构造器实现：
	* int i = 500; Integer t = new Integer(i);
* 还可以通过字符串参数构造包装类对象：
	* Float f = new Float(“4.56”);
	* Long l = new Long(“asdf”); //NumberFormatException

* 获得包装类对象中包装的基本类型变量 ---拆箱
	* 调用包装类的.xxxValue()方法：boolean b = bObj.booleanValue()

* JDK1.5之后，支持自动装箱，自动拆箱。但类型必须匹配

### 字符串转换成基本数据类型

* 通过包装类的构造器实现：int i = new Integer(“12”);
* 通过包装类的parseXxx(String s)静态方法：Float f = Float.parseFloat(“12.1”);

### 基本数据类型转换成字符串

* 调用字符串重载的valueOf()方法：String fstr = String.valueOf(2.34f);
* 更直接的方式：String intStr = 5 + “”

### 基本类型、包装类与String类间的转换

![](https://s1.ax1x.com/2020/09/11/wUixgg.md.png)

![](https://s1.ax1x.com/2020/09/11/wUijC8.md.png)

![](https://s1.ax1x.com/2020/09/11/wUiLUP.png)

## static

### 设计思想

类属性作为该类各个对象之间共享的变量。在设计类时,分析哪些属性不因对象的不同而改变，将这些属性设置为类属性。相应的方法设置为类方法。

如果方法与调用者无关，则这样的方法通常被声明为类方法，由于不需要创建对象就可以调用类方法，从而简化了方法的调用。

#### 适用范围

在java类中，可以用static修饰属性、方法、代码块、内部类

#### 被修饰后的特点

* 随着类的加载而加载
* 优先于对象存在
* 修饰的成员，被所有对象所共享
* 访问权限允许时，可不创建对象，直接被类调用

![](https://s3.ax1x.com/2020/11/23/DJf0r6.md.png)

![](https://s3.ax1x.com/2020/11/23/DJfax1.md.png)

没有对象的实例时，可以用类名.方法名()的形式访问由static修饰的类方法

在static方法内部只能访问类的static修饰的属性或方法，不能访问类的非static的结构

![](https://s3.ax1x.com/2020/11/23/DJfwKx.md.png)

this是一个指针，用来指向堆中实例的对象。而static是一个静态修饰符，用来修饰方法。被static修饰的方法即被称作静态方法。JVM 在内存划分时，会把静态方法划分到一个独立的区（方法区），只有类能够调用，所以又名类方法。因此，静态方法中不能调用非静态方法，除非实例化出对象。因此，如果在static方法中使用this指针，指针会因为无法指向对象而报错。super指针则同理。

![](https://s3.ax1x.com/2020/11/23/DJfU2R.md.png)

## 设计模式——单例模式

设计模式是在大量的实践中总结和理论化之后优选的代码结构、编程风格、
以及解决问题的思考方式。设计模式免去我们自己再思考和摸索。就像是经典的棋谱，不同的棋局，我们用不同的棋谱。”套路”

所谓类的单例设计模式，就是采取一定的方法保证在整个的软件系统中，对某个类**只能存在一个对象实例**，并且该类只提供一个取得其对象实例的方法。如果我们要让类在一个虚拟机中只能产生一个对象，我们首先必须将类的构造器的访问权限设置为private，这样，就不能用new操作符在类的外部产生类的对象了，但在类内部仍可以产生该类的对象。因为在类的外部开始还无法得到类的对象，只能调用该类的某个静态方法以返回类内部创建的对象，静态方法只能访问类中的静态成员变量，所以，指向类内部产生的该类对象的变量也必须定义成静态的。


### 饿汉式

    class Singleton {
    	// 1.私有化构造器
    	private Singleton() {
    	}
    	// 2.内部提供一个当前类的实例
    	// 4.此实例也必须静态化
    	private static Singleton single = new Singleton();
    	// 3.提供公共的静态的方法，返回当前类的对象
    	public static Singleton getInstance() {
    		return single;
    	 } 
    }

### 懒汉式

懒汉式暂时还存在线程安全问题，使用多线程可修复

    class Singleton {
    	// 1.私有化构造器
    	private Singleton() {
    	}
    	// 2.内部提供一个当前类的实例
    	// 4.此实例也必须静态化
    	private static Singleton single;
    	// 3.提供公共的静态的方法，返回当前类的对象
    	public static Singleton getInstance() {
	    	if(single == null) {
	    		single = new Singleton();
	    	}
    	return single;
    	 } 
    }

### 区别

**饿汉式：**

* 好处：对象加载时间过长
* 坏处：饿汉式是线程安全的

**懒汉式：**

* 好处：延迟对象的创建
* 坏处：存在线程安全问题

### 优点

由于单例模式只生成一个实例，**减少了系统性能开销**，当一个对象的产生需要比较多的资源时，如读取配置、产生其他依赖对象时，则可以通过在应用启动时直接产生一个单例对象，然后永久驻留内存的方式来解决。


### 适用场景

![](https://s3.ax1x.com/2020/11/23/DJhRkF.md.png)

* 网站的计数器，一般也是单例模式实现，否则难以同步。
* 应用程序的日志应用，一般都使用单例模式实现，这一般是由于共享的日志文件一直处于打开状态，因为只能有一个实例去操作，否则内容不好追加。
* 数据库连接池的设计一般也是采用单例模式，因为数据库连接是一种数据库资源。
* 项目中，读取配置文件的类，一般也只有一个对象。没有必要每次使用配置文件数据，都生成一个对象去读取。
* Application 也是单例的典型应用
* Windows的Task Manager (任务管理器)就是很典型的单例模式
* Windows的Recycle Bin (回收站)也是典型的单例应用。在整个系统运行过程
中，回收站一直维护着仅有的一个实例

## 理解main方法的语法

* 由于Java虚拟机需要调用类的main()方法，所以该方法的访问权限必须是public，又因为Java虚拟机在执行main()方法时不必创建对象，所以该方法必须是static的，该方法接收一个String类型的数组参数，该数组中保存执行Java命令时传递给所运行的类的参数。
* 又因为main() 方法是静态的，我们不能直接访问该类中的非静态成员，必须创建该类的一个实例对象后，才能通过这个对象去访问类中的非静态成员，这种情况，我们在之前的例子中多次碰到。

## 关键字：final

在Java中声明类、变量和方法时，可使用关键字final来修饰,表示“最终的”。 

* final标记的类不能被继承。提高安全性，提高程序的可读性。
	* String类、System类、StringBuffer类 
* final标记的方法不能被子类重写。
	* 比如：Object类中的getClass()。 
* final标记的变量(成员变量或局部变量)即称为常量。名称大写，且只能被赋值一次。
	* final标记的成员变量必须在声明时或在每个构造器中或代码块中显式赋值，然后才能使用。
	* final double MY_PI = 3.14;

### final修饰类

	final class A{
	}
	class B extends A{ //错误，不能被继承。
	}

### final修饰方法

	class A {
	public final void print() {
	System.out.println("A");
	} }
	class B extends A {
	public void print() { // 错误，不能被重写。
	System.out.println("尚硅谷");
	} }

### final修饰变量——常量

	class A {
	private final String INFO = "atguigu"; //声明常量
	public void print() {
	//The final field A.INFO cannot be assigned
	//INFO = "尚硅谷";
	} }

常量名要大写，内容不可修改。

**static final：全局常量**

### 关键字final应用举例

	public final class Test {
	public static int totalNumber = 5;
	public final int ID;
	public Test() {
	ID = ++totalNumber; // 可在构造器中给final修饰的“变量”赋值
	}
	public static void main(String[] args) {
	Test t = new Test();
	System.out.println(t.ID);
	final int I = 10;
	final int J; J = 20;
	J = 30; // 非法
	} }

![](https://s3.ax1x.com/2020/11/23/DJhgTU.md.png)