---
layout: post
title: "Java反射机制"
date: 2020-11-26 15:21:37 +0800
categories: notes
tags: java 反射
img: https://s1.ax1x.com/2020/07/12/U1oQN4.png
---
Java反射机制概述;理解Class类并获取Class实例;类的加载与ClassLoader的理解;创建运行时类的对象;获取运行时类的完整结构;调用运行时类的指定结构;反射的应用：动态代理


## Java反射机制概述

* Reflection（反射）是被视为动态语言的关键，反射机制允许程序在执行期借助于Reflection API取得任何类的内部信息，并能直接操作任意对象的内部属性及方法。 
* 加载完类之后，在堆内存的方法区中就产生了一个Class类型的对象（一个类只有一个Class对象），这个对象就包含了完整的类的结构信息。我们可以通过这个对象看到类的结构。这个对象就像一面镜子，透过这个镜子看到类的结构，所以，我们形象的称之为：反射

![](https://s3.ax1x.com/2020/12/07/DxomAe.png)

### 补充：动态语言 vs 静态语言


#### 动态语言

是一类在运行时可以改变其结构的语言：例如新的函数、对象、甚至代码可以被引进，已有的函数可以被删除或是其他结构上的变化。通俗点说就是在运行时代码可以根据某些条件改变自身结构。

主要动态语言：Object-C、C#、JavaScript、PHP、Python、Erlang。 


#### 静态语言

与动态语言相对应的，运行时结构不可变的语言就是静态语言。如Java、C、C++。

Java不是动态语言，但Java可以称之为“准动态语言”。即Java有一定的动态性，我们可以利用反射机制、字节码操作获得类似动态语言的特性。Java的动态性让编程的时候更加灵活！


### Java反射机制研究及应用

Java反射机制提供的功能

* 在运行时判断任意一个对象所属的类
* 在运行时构造任意一个类的对象
* 在运行时判断任意一个类所具有的成员变量和方法
* 在运行时获取泛型信息
* 在运行时调用任意一个对象的成员变量和方法
* 在运行时处理注解
* 生成动态代理


### 反射相关的主要API

* java.lang.Class:代表一个类 
* java.lang.reflect.Method:代表类的方法
* java.lang.reflect.Field:代表类的成员变量
* java.lang.reflect.Constructor:代表类的构造器

### 问题

* 疑问1：通过直接new的方式或反射的方式都可以调用公共的结构，开发中到底用那个？
	* 建议：直接new的方式。
	* 什么时候会使用：反射的方式。 反射的特征：动态性
* 疑问2：反射机制与面向对象中的封装性是不是矛盾的？如何看待两个技术？
	* 不矛盾。

## 理解Class类并获取Class实例


### Class类

在Object类中定义了以下的方法，此方法将被所有子类继承：

● public final Class getClass()

以上的方法返回值的类型是一个Class类，此类是Java反射的源头，实际上所谓反射从程序的运行结果来看也很好理解，即：可以通过对象反射求出类的名称。


对象照镜子后可以得到的信息：某个类的属性、方法和构造器、某个类到底实现了哪些接口。对于每个类而言，JRE 都为其保留一个不变的 Class 类型的对象。一个 Class 对象包含了特定某个结构(class/interface/enum/annotation/primitive type/void/[])的有关信息。 

* Class本身也是一个类
* Class 对象只能由系统建立对象
* 一个加载的类在 JVM 中只会有一个Class实例
* 一个Class对象对应的是一个加载到JVM中的一个.class文件
* 每个类的实例都会记得自己是由哪个 Class 实例所生成
* 通过Class可以完整地得到一个类中的所有被加载的结构
* Class类是Reflection的根源，针对任何你想动态加载、运行的类，唯有先获得相应的Class对象

### Class类的常用方法

![](https://s3.ax1x.com/2020/12/07/DxoZ7D.md.png)

### 反射的应用举例

		Class clazz = Person.class;
        //1.通过反射，创建Person类的对象
        Constructor cons = clazz.getConstructor(String.class,int.class);
        Object obj = cons.newInstance("Tom", 12);
        Person p = (Person) obj;
        System.out.println(p.toString());
        //2.通过反射，调用对象指定的属性、方法
        //调用属性
        Field age = clazz.getDeclaredField("age");
        age.set(p,10);
        System.out.println(p.toString());

        //调用方法
        Method show = clazz.getDeclaredMethod("show");
        show.invoke(p);

        System.out.println("*******************************");

        //通过反射，可以调用Person类的私有结构的。比如：私有的构造器、方法、属性
        //调用私有的构造器
        Constructor cons1 = clazz.getDeclaredConstructor(String.class);
        cons1.setAccessible(true);
        Person p1 = (Person) cons1.newInstance("Jerry");
        System.out.println(p1);

        //调用私有的属性
        Field name = clazz.getDeclaredField("name");
        name.setAccessible(true);
        name.set(p1,"HanMeimei");
        System.out.println(p1);

        //调用私有的方法
        Method showNation = clazz.getDeclaredMethod("showNation", String.class);
        showNation.setAccessible(true);
        String nation = (String) showNation.invoke(p1,"中国");//相当于String nation = p1.showNation("中国")
        System.out.println(nation);


### 关于java.lang.Class类的理解

1. 类的加载过程：
	程序经过javac.exe命令以后，会生成一个或多个字节码文件(.class结尾)。接着我们使用java.exe命令对某个字节码文件进行解释运行。相当于将某个字节码文件加载到内存中。此过程就称为类的加载。加载到内存中的类，我们就称为运行时类，此运行时类，就作为Class的一个实例。

2. 换句话说，Class的实例就对应着一个运行时类。
3. 加载到内存中的运行时类，会缓存一定的时间。在此时间之内，我们可以通过不同的方式
来获取此运行时类。

### 获取Class类的实例(四种方法)

1）前提：若已知具体的类，通过类的class属性获取，该方法最为安全可靠，程序性能最高

实例：Class clazz = String.class;

2）前提：已知某个类的实例，调用该实例的getClass()方法获取Class对象

实例： Person p1 = new Person();
       
 Class clazz2 = p1.getClass();

3）前提：已知一个类的全类名，且该类在类路径下，可通过Class类的静态方法forName()获取，可能抛出ClassNotFoundException

实例：Class clazz = Class.forName(“java.lang.String”);

4）其他方式(不做要求)

ClassLoader cl = this.getClass().getClassLoader();

Class clazz4 = cl.loadClass(“类的全类名”)

### 哪些类型可以有Class对象？

1. class： 外部类，成员(成员内部类，静态内部类)，局部内部类，匿名内部类 
2. interface：接口
3. []：数组
4. enum：枚举
5. annotation：注解@interface
6. primitive type：基本数据类型
7. void

		Class c1 = Object.class;
		Class c2 = Comparable.class;
		Class c3 = String[].class;
		Class c4 = int[][].class;
		Class c5 = ElementType.class;
		Class c6 = Override.class;
		Class c7 = int.class;
		Class c8 = void.class;
		Class c9 = Class.class;
		int[] a = new int[10];
		int[] b = new int[100];
		Class c10 = a.getClass();
		Class c11 = b.getClass();
		// 只要元素类型与维度一样，就是同一个Class
		System.out.println(c10 == c11);

## 类的加载与ClassLoader的理解

当程序主动使用某个类时，如果该类还未被加载到内存中，则系统会通过如下三个步骤来对该类进行初始化

![](https://s3.ax1x.com/2020/12/07/DxoV0O.png)

* 加载：将class文件字节码内容加载到内存中，并将这些静态数据转换成方法区的运行时数据结构，然后生成一个代表这个类的java.lang.Class对象，作为方法区中类数据的访问入口（即引用地址）。所有需要访问和使用类数据只能通过这个Class对象。这个加载的过程需要类加载器参与。
* 链接：将Java类的二进制代码合并到JVM的运行状态之中的过程。
	* 验证：确保加载的类信息符合JVM规范，例如：以cafe开头，没有安全方面的问题
	* 准备：正式为类变量（static）分配内存并设置类变量默认初始值的阶段，这些内存都将在方法区中进行分配。 
	* 解析：虚拟机常量池内的符号引用（常量名）替换为直接引用（地址）的过程。 
* 初始化:
	* 执行类构造器<clinit>()方法的过程。类构造器<clinit>()方法是由编译期自动收集类中所有类变量的赋值动作和静态代码块中的语句合并产生的。（类构造器是构造类信息的，不是构造该类对象的构造器）。 
	* 当初始化一个类的时候，如果发现其父类还没有进行初始化，则需要先触发其父类的初始化。 
	* 虚拟机会保证一个类的<clinit>()方法在多线程环境中被正确加锁和同步。

![](https://s3.ax1x.com/2020/12/07/DxoEnK.png)

#### 什么时候会发生类初始化？

* 类的主动引用（一定会发生类的初始化）
	* 当虚拟机启动，先初始化main方法所在的类 
	* new一个类的对象
	* 调用类的静态成员（除了final常量）和静态方法
	* 使用java.lang.reflect包的方法对类进行反射调用
	* 当初始化一个类，如果其父类没有被初始化，则先会初始化它的父类
* 类的被动引用（不会发生类的初始化） 
	* 当访问一个静态域时，只有真正声明这个域的类才会被初始化
	* 当通过子类引用父类的静态变量，不会导致子类初始化
	* 通过数组定义类引用，不会触发此类的初始化
	* 引用常量不会触发此类的初始化（常量在链接阶段就存入调用类的常量池中了）

### 类加载器的作用

**类加载的作用**：

* 将class文件字节码内容加载到内存中，并将这些静态数据转换成方法区的运行时数据结构，然后在堆中生成一个代表这个类的java.lang.Class对象，作为方法区中类数据的访问入口。 
* **类缓存**：标准的JavaSE类加载器可以按要求查找类，但一旦某个类被加载到类加载器
中，它将维持加载（缓存）一段时间。不过JVM垃圾回收机制可以回收这些Class对象。

### 了解：ClassLoader

类加载器作用是用来把类(class)装载进内存的。JVM 规范定义了如下类型的类的加载器。

![](https://s3.ax1x.com/2020/12/07/DxokX6.md.png)

		//对于自定义类，使用系统类加载器进行加载
        ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
        System.out.println(classLoader);
        //调用系统类加载器的getParent()：获取扩展类加载器
        ClassLoader classLoader1 = classLoader.getParent();
        System.out.println(classLoader1);
        //调用扩展类加载器的getParent()：无法获取引导类加载器
        //引导类加载器主要负责加载java的核心类库，无法加载自定义类的。
        ClassLoader classLoader2 = classLoader1.getParent();
        System.out.println(classLoader2);

        ClassLoader classLoader3 = String.class.getClassLoader();
        System.out.println(classLoader3);

		//4.测试当前类由哪个类加载器进行加载
		• classloader = Class.forName("exer2.ClassloaderDemo").getClassLoader();
		• System.out.println(classloader);
		• //5.测试JDK提供的Object类由哪个类加载器加载
		• classloader = 
		• Class.forName("java.lang.Object").getClassLoader();
		• System.out.println(classloader);
		• //*6.关于类加载器的一个主要方法：getResourceAsStream(String str):获取类路
		径下的指定文件的输入流
		• InputStream in = null;
		• in = this.getClass().getClassLoader().getResourceAsStream("exer2\\test.properties");
		• System.out.println(in);

## 创建运行时类的对象

创建类的对象：调用Class对象的newInstance()方法

要 求： 

1. 类必须有一个无参数的构造器。
2. 类的构造器的访问权限需要足够。


难道没有无参的构造器就不能创建对象了吗？

不是！只要在操作的时候明确的调用类中的构造器，并将参数传递进去之后，才可以实例化操作。
步骤如下：

1. 通过Class类的getDeclaredConstructor(Class … parameterTypes)取得本类的指定形参类型的构造器
2. 向构造器的形参中传递一个对象数组进去，里面包含了构造器中所需的各个参数。
3. 通过Constructor实例化对象。

![](https://s3.ax1x.com/2020/12/07/Dxoi11.png)


 		Class<Person> clazz = Person.class;
        /*
        newInstance():调用此方法，创建对应的运行时类的对象。内部调用了运行时类的空参的构造器。

        要想此方法正常的创建运行时类的对象，要求：
        1.运行时类必须提供空参的构造器
        2.空参的构造器的访问权限得够。通常，设置为public。


        在javabean中要求提供一个public的空参构造器。原因：
        1.便于通过反射，创建运行时类的对象
        2.便于子类继承此运行时类时，默认调用super()时，保证父类有此构造器

         */
        Person obj = clazz.newInstance();
        System.out.println(obj);

    /*
    创建一个指定类的对象。
    classPath:指定类的全类名
     */
    public Object getInstance(String classPath) throws Exception {
       Class clazz =  Class.forName(classPath);
       return clazz.newInstance();
    }

*** 

1.根据全类名获取对应的Class对象

	String name = “atguigu.java.Person";
	Class clazz = null;
	clazz = Class.forName(name);

2.调用指定参数结构的构造器，生成Constructor的实例

	Constructor con = clazz.getConstructor(String.class,Integer.class);

3.通过Constructor的实例创建对应类的对象，并初始化类属性

	Person p2 = (Person) con.newInstance("Peter",20);
	System.out.println(p2)

## 获取运行时类的完整结构

通过反射获取运行时类的完整结构

Field、Method、Constructor、Superclass、Interface、Annotation


* 实现的全部接口
* 所继承的父类
* 全部的构造器
* 全部的方法
* 全部的Field

***

使用反射可以取得：

1.实现的全部接口
	public Class<?>[] getInterfaces() 

确定此对象所表示的类或接口实现的接口。

2.所继承的父类

	public Class<? Super T> getSuperclass()

返回表示此 Class 所表示的实体（类、接口、基本类型）的父类的Class。

3.全部的构造器

* public Constructor<T>[] getConstructors()
	* 返回此 Class 对象所表示的类的所有public构造方法。获取当前运行时类中声明为public的构造器
* public Constructor<T>[] getDeclaredConstructors(
	* 返回此 Class 对象表示的类声明的所有构造方法。获取当前运行时类中声明的所有的构造器

* Constructor类中：
	* 取得修饰符: public int getModifiers();
	* 取得方法名称: public String getName();
	* 取得参数的类型：public Class<?>[] getParameterTypes()

4.全部的方法

* public Method[] getDeclaredMethods()
	* 返回此Class对象所表示的类或接口的全部方法。获取当前运行时类及其所有父类中声明为public权限的方法
* public Method[] getMethods() 
	* 返回此Class对象所表示的类或接口的public的方法。获取当前运行时类中声明的所有方法。（不包含父类中声明的方法）

* Method类中：
	* public Class<?> getReturnType()取得全部的返回值
	* public Class<?>[] getParameterTypes()取得全部的参数
	* public int getModifiers()取得修饰符
	* public Class<?>[] getExceptionTypes()取得异常信息

5.全部的Field

* public Field[] getFields() 
	* 返回此Class对象所表示的类或接口的public的Field。获取当前运行时类及其父类中声明为public访问权限的属性
* public Field[] getDeclaredFields() 
	* 返回此Class对象所表示的类或接口的全部Field。 获取当前运行时类中声明的所有属性。（不包含父类中声明的属性）

* Field方法中：
	* public int getModifiers() 以整数形式返回此Field的修饰符
	* public Class<?> getType() 得到Field的属性类型
	* public String getName() 返回Field的名称。
	* 
6.Annotation相关

* get Annotation(Class<T> annotationClass) 
* getDeclaredAnnotations() 

7.泛型相关

* 获取父类泛型类型：Type getGenericSuperclass()
* 泛型类型：ParameterizedType
* 获取实际的泛型类型参数数组：getActualTypeArguments()

8.类所在的包 Package getPackage()

## 调用运行时类的指定结构

### 调用指定方法

通过反射，调用类中的方法，通过Method类完成。步骤：

1. 通过Class类的getMethod(String name,Class…parameterTypes)方法取得一个Method对象，并设置此方法操作时所需要的参数类型。
2. 之后使用Object invoke(Object obj, Object[] args)进行调用，并向方法中传递要设置的obj对象的参数信息

![](https://s3.ax1x.com/2020/12/07/DxoPpR.png)

	 /*
    如何操作运行时类中的指定的方法 -- 需要掌握
     */
    @Test
    public void testMethod() throws Exception {

        Class clazz = Person.class;

        //创建运行时类的对象
        Person p = (Person) clazz.newInstance();

        /*
        1.获取指定的某个方法
        getDeclaredMethod():参数1 ：指明获取的方法的名称  参数2：指明获取的方法的形参列表
         */
        Method show = clazz.getDeclaredMethod("show", String.class);
        //2.保证当前方法是可访问的
        show.setAccessible(true);

        /*
        3. 调用方法的invoke():参数1：方法的调用者  参数2：给方法形参赋值的实参
        invoke()的返回值即为对应类中调用的方法的返回值。
         */
        Object returnValue = show.invoke(p,"CHN"); //String nation = p.show("CHN");
        System.out.println(returnValue);

        System.out.println("*************如何调用静态方法*****************");

        // private static void showDesc()

        Method showDesc = clazz.getDeclaredMethod("showDesc");
        showDesc.setAccessible(true);
        //如果调用的运行时类中的方法没有返回值，则此invoke()返回null
		//        Object returnVal = showDesc.invoke(null);
        Object returnVal = showDesc.invoke(Person.class);
        System.out.println(returnVal);//null

    }


#### invoke

Object invoke(Object obj, Object … args)

说明：

1. Object 对应原方法的返回值，若原方法无返回值，此时返回null
2. 若原方法若为静态方法，此时形参Object obj可为null
3. 若原方法形参列表为空，则Object[] args为null
4. 若原方法声明为private,则需要在调用此invoke()方法前，显式调用


方法对象的setAccessible(true)方法，将可访问private的方法

### 调用指定属性

在反射机制中，可以直接通过Field类操作类中的属性，通过Field类提供的set()和get()方法就可以完成设置和取得属性内容的操作。

* public Field getField(String name) 返回此Class对象表示的类或接口的指定的public的Field。
* public Field getDeclaredField(String name)返回此Class对象表示的类或接口的指定的Field。

* 在Field中：
	* public Object get(Object obj) 取得指定对象obj上此Field的属性内容
	* public void set(Object obj,Object value) 设置指定对象obj上此Field的属性内容

       Class clazz = Person.class;

        //创建运行时类的对象
        Person p = (Person) clazz.newInstance();

        //获取指定的属性：要求运行时类中属性声明为public
        //通常不采用此方法
        Field id = clazz.getField("id");

        /*
        设置当前属性的值

        set():参数1：指明设置哪个对象的属性   参数2：将此属性值设置为多少
         */

        id.set(p,1001);

        /*
        获取当前属性的值
        get():参数1：获取哪个对象的当前属性值
         */
        int pId = (int) id.get(p);
        System.out.println(pId);


***

    /*
    如何操作运行时类中的指定的属性 -- 需要掌握
     */
    @Test
    public void testField1() throws Exception {
        Class clazz = Person.class;

        //创建运行时类的对象
        Person p = (Person) clazz.newInstance();

        //1. getDeclaredField(String fieldName):获取运行时类中指定变量名的属性
        Field name = clazz.getDeclaredField("name");

        //2.保证当前属性是可访问的
        name.setAccessible(true);
        //3.获取、设置指定对象的此属性值
        name.set(p,"Tom");

        System.out.println(name.get(p));
    }

### 关于setAccessible方法的使用

* Method和Field、Constructor对象都有setAccessible()方法。
* setAccessible启动和禁用访问安全检查的开关。
* 参数值为true则指示反射的对象在使用时应该取消Java语言访问检查。 
	* 提高反射的效率。如果代码中必须用反射，而该句代码需要频繁的被调用，那么请设置为true。 
	* 使得原本无法访问的私有成员也可以访问
* 参数值为false则指示反射的对象应该实施Java语言访问检查

#### 调用指定构造器

    /*
    如何调用运行时类中的指定的构造器
     */
    @Test
    public void testConstructor() throws Exception {
        Class clazz = Person.class;

        //private Person(String name)
        /*
        1.获取指定的构造器
        getDeclaredConstructor():参数：指明构造器的参数列表
         */

        Constructor constructor = clazz.getDeclaredConstructor(String.class);

        //2.保证此构造器是可访问的
        constructor.setAccessible(true);

        //3.调用此构造器创建运行时类的对象
        Person per = (Person) constructor.newInstance("Tom");
        System.out.println(per);

    }

## 反射的应用：动态代理

### 代理设计模式的原理

使用一个代理将对象包装起来, 然后用该代理对象取代原始对象。任何对原始对象的调用都要通过代理。代理对象决定是否以及何时将方法调用转到原始对象上。

* 之前为大家讲解过代理机制的操作，属于静态代理，特征是代理类和目标对象的类都是在编译期间确定下来，不利于程序的扩展。同时，每一个代理类只能为一个接口服务，这样一来程序开发中必然产生过多的代理。**最好可以通过一个代理类完成全部的代理功能**。
* 动态代理是指客户通过代理类来调用其它对象的方法，并且是在程序运行时根据需要动态创建目标类的代理对象。
* 动态代理使用场合:
	* 调试
	* 远程方法调用

#### 动态代理相比于静态代理的优点

抽象角色中（接口）声明的所有方法都被转移到调用处理器一个集中的方法中处理，这样，我们可以更加灵活和统一的处理众多的方法。

### Java动态代理相关API

* Proxy ：专门完成代理的操作类，是所有动态代理类的父类。通过此类为一个或多个接口动态地生成实现类。
* 提供用于创建动态代理类和动态代理对象的静态方法
	* static Class<?> getProxyClass(ClassLoader loader, Class<?>... interfaces) 创建一个动态代理类所对应的Class对象
	* static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) 直接创建一个动态代理对象

![](https://s3.ax1x.com/2020/12/07/DxoF6x.md.png)

### 动态代理步骤

1.创建一个实现接口InvocationHandler的类，它必须实现invoke方法，以完成代理的具体操作。

	public Object invoke(Object theProxy, Method method, Object[] params) 
	throws Throwable{
	try{
	Object retval = method.invoke(targetObj, params);
	// Print out the result
	System.out.println(retval);
	return retval;
	}catch (Exception exc){}
	}

![](https://s3.ax1x.com/2020/12/07/Dxo9h9.md.png)

2.创建被代理的类以及接口

![](https://s3.ax1x.com/2020/12/07/DxIv0U.png)

3.通过Proxy的静态方法

newProxyInstance(ClassLoader loader, Class[] interfaces, InvocationHandler h) 创建一个Subject接口代理

	RealSubject target = new RealSubject();
	// Create a proxy to wrap the original implementation
	DebugProxy proxy = new DebugProxy(target);
	// Get a reference to the proxy through the Subject interface
	Subject sub = (Subject) Proxy.newProxyInstance(
	Subject.class.getClassLoader(),new Class[] { Subject.class }, proxy);

4.通过 Subject代理调用RealSubject实现类的方法

	String info = sub.say(“Peter", 24);
	System.out.println(info);

***


	/*
	要想实现动态代理，需要解决的问题？
	问题一：如何根据加载到内存中的被代理类，动态的创建一个代理类及其对象。
	问题二：当通过代理类的对象调用方法a时，如何动态的去调用被代理类中的同名方法a。
	
	
	 */
	
	class ProxyFactory{
	    //调用此方法，返回一个代理类的对象。解决问题一
	    public static Object getProxyInstance(Object obj){//obj:被代理类的对象
	        MyInvocationHandler handler = new MyInvocationHandler();
	
	        handler.bind(obj);
	
	        return Proxy.newProxyInstance(obj.getClass().getClassLoader(),obj.getClass().getInterfaces(),handler);
	    }
	
	}
	
	class MyInvocationHandler implements InvocationHandler{
	
	    private Object obj;//需要使用被代理类的对象进行赋值
	
	    public void bind(Object obj){
	        this.obj = obj;
	    }
	
	    //当我们通过代理类的对象，调用方法a时，就会自动的调用如下的方法：invoke()
	    //将被代理类要执行的方法a的功能就声明在invoke()中
	    @Override
	    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
	
	        HumanUtil util = new HumanUtil();
	        util.method1();
	
	        //method:即为代理类对象调用的方法，此方法也就作为了被代理类对象要调用的方法
	        //obj:被代理类的对象
	        Object returnValue = method.invoke(obj,args);
	
	        util.method2();
	
	        //上述方法的返回值就作为当前类中的invoke()的返回值。
	        return returnValue;
	
	    }
	}
	
	public class ProxyTest {
	
	    public static void main(String[] args) {
	        SuperMan superMan = new SuperMan();
	        //proxyInstance:代理类的对象
	        Human proxyInstance = (Human) ProxyFactory.getProxyInstance(superMan);
	        //当通过代理类对象调用方法时，会自动的调用被代理类中同名的方法
	        String belief = proxyInstance.getBelief();
	        System.out.println(belief);
	        proxyInstance.eat("四川麻辣烫");
	
	        System.out.println("*****************************");
	
	        NikeClothFactory nikeClothFactory = new NikeClothFactory();
	
	        ClothFactory proxyClothFactory = (ClothFactory) ProxyFactory.getProxyInstance(nikeClothFactory);
	
	        proxyClothFactory.produceCloth();
	
	    }

### 动态代理与AOP（Aspect Orient Programming)

前面介绍的Proxy和InvocationHandler，很难看出这种动态代理的优势，下面介绍一种更实用的动态代理机制

![](https://s3.ax1x.com/2020/12/07/DxoptJ.png)

![](https://s3.ax1x.com/2020/12/07/DxoSk4.png)

改进后的说明：代码段1、代码段2、代码段3和深色代码段分离开了，但代码段1、2、3又和一个特定的方法A耦合了！最理想的效果是：代码块1、2、3既可以执行方法A，又无须在程序中以硬编码的方式直接调用深色代码的方法

		public interface Dog{
		void info();
		void run();
		}
		
		public class HuntingDog implements Dog{
		public void info(){
		System.out.println("我是一只猎狗");
		}
		public void run(){
		System.out.println("我奔跑迅速");
		} }
		
		public class DogUtil{
		public void method1(){
		System.out.println("=====模拟通用方法一=====");
		}
		public void method2(){
		System.out.println("=====模拟通用方法二=====");
		} }

***

	public class MyInvocationHandler implements InvocationHandler{
	// 需要被代理的对象
	private Object target;
	public void setTarget(Object target){
	this.target = target;}
	// 执行动态代理对象的所有方法时，都会被替换成执行如下的invoke方法
	public Object invoke(Object proxy, Method method, Object[] args)
	throws Exception{
	DogUtil du = new DogUtil();
	// 执行DogUtil对象中的method1。
	du.method1();
	// 以target作为主调来执行method方法
	Object result = method.invoke(target , args);
	// 执行DogUtil对象中的method2。
	du.method2();
	return result;}}

***

	public class MyInvocationHandler implements InvocationHandler{
	// 需要被代理的对象
	private Object target;
	public void setTarget(Object target){
	this.target = target;}
	// 执行动态代理对象的所有方法时，都会被替换成执行如下的invoke方法
	public Object invoke(Object proxy, Method method, Object[] args)
	throws Exception{
	DogUtil du = new DogUtil();
	// 执行DogUtil对象中的method1。
	du.method1();
	// 以target作为主调来执行method方法
	Object result = method.invoke(target , args);
	// 执行DogUtil对象中的method2。
	du.method2();
	return result;}} 
	
	public class Test{
	public static void main(String[] args) 
	throws Exception{
	// 创建一个原始的HuntingDog对象，作为target
	Dog target = new HuntingDog();
	// 以指定的target来创建动态代理
	Dog dog = (Dog)MyProxyFactory.getProxy(target);
	dog.info();
	dog.run();
	} }

* 使用Proxy生成一个动态代理时，往往并不会凭空产生一个动态代理，这样没有太大的意义。通常都是为指定的目标对象生成动态代理
* 这种动态代理在AOP中被称为AOP代理，AOP代理可代替目标对象，AOP代理包含了目标对象的全部方法。但AOP代理中的方法与目标对象的方法存在差异：AOP代理里的方法可以在执行目标方法之前、之后插入一些通用处理


![](https://s3.ax1x.com/2020/12/07/DxIx7F.png)
