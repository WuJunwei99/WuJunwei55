---
layout: post
title: "java 学习笔记（4）——数组"
date: 2020-07-12 19:38:17 +0800
categories: notes
tags: java
img: https://s1.ax1x.com/2020/07/12/U1oQN4.png
---


## 概述

数组(Array)，是多个**相同类型数据**按**一定顺序排列**的集合，并使用**一个名字命名**，并通过**编号**的方式对这些数据进行统一管理。

* 数组本身是**引用数据类型**，而数组中的元素可以是**任何数据类型**，包括基本数据类型和引用数据类型
* 创建数组对象会在内存中开辟一整块**连续的空间**，而数组名中引用的是这块连续空间的首地址
* 数组的长度一旦确定，就**不能**修改。
* 我们可以直接通过下标(或索引)的方式调用指定位置的元素，速度很快。
* 数组的分类：
	* 按照维度：一维数组、二维数组、三维数组...
	* 按照元素的数据类型分：基本数据类型元素的数组、引用数据类型元素的数组(即对象数组)

## 一维数组的使用

#### 声明方式

type var[] 或 type[] var；


例如：
    
    int a[];
    int[] a1;
    double b[];
    String[] c; //引用类型变量数组

Java语言中声明数组时不能指定其长度(数组中元素的数)， 例如： int a[5]; //非法

#### 动态初始化

数组声明且为数组元素分配空间与赋值的操作分开进行

    int[] arr = new int[3];
    arr[0] = 3;
    arr[1] = 9;
    arr[2] = 8;
    
    String names[];
    names = new String[3];
    names[0] = “钱学森”;
    names[1] = “邓稼先”;
    names[2] = “袁隆平”;

#### 静态初始化

在定义数组的同时就为数组元素分配空间并赋值。

    int arr[] = new int[]{ 3, 9, 8};或int[] arr = {3,9,8}
    
    String names[] = {“李四光”,“茅以升”,“华罗庚” }

### 数组元素的引用

* 定义并用运算符new为之分配空间后，才可以引用数组中的每个元素；
* 数组元素的引用方式：数组名[数组元素下标]
	* 数组元素下标可以是整型常量或整型表达式。如a[3] , b[i] , c[6*i];
	* 数组元素下标从0开始；长度为n的数组合法下标取值范围: 0 —>n-1；如int a[]=new int[3]; 可引用的数组元素为a[0]、a[1]、a[2]
* 每个数组都有一个属性length指明它的长度，例如：a.length 指明数组a的长度(元素个数)
	* 数组一旦初始化，其长度是不可变的

### 数组元素的默认初始化值


数组是引用类型，它的元素相当于类的成员变量，因此数组一经分配空间，其中的每个元素也被按照成员变量同样的方式被隐式初始化。例如

    public class Test {
    	public static void main(String argv[]){
    		int a[]= new int[5];
    		System.out.println(a[3]); //a[3]的默认值为0
    	 } 
    }

* 对于基本数据类型而言，默认初始化值各有不同
* 对于引用数据类型而言，默认初始化值为null(注意与0不同！)

![](https://s1.ax1x.com/2020/07/12/U3xuPx.png)

### 创建基本数据类型数组 

Java中使用关键字new来创建数组

如下是创建基本数据类型元素的一维数组

    public class Test{
    	public static void main(String args[]){
    		int[] s;
    		s = new int[10];
			//int[] s=new int[10];
			//基本数据类型数组在显式赋值之前，
    		for ( int i=0; i<10; i++ ) {
    			s[i] =2*i+1;
    			System.out.println(s[i]);
    		} 
    	} 
    }

![](https://s1.ax1x.com/2020/07/12/U3xmI1.png)

![](https://s1.ax1x.com/2020/07/12/U3xKG6.md.png)

![](https://s1.ax1x.com/2020/07/12/U3xMRK.md.png)

## 多维数组的使用

Java 语言里提供了支持多维数组的语法

如果说可以把一维数组当成几何中的线性图形，那么二维数组就相当于是一个表格

对于二维数组的理解，我们可以看成是一维数组array1又作为另一个一维数组array2的元素而存在。其实，从数组底层的运行机制来看，其实没有多维数组。

### 动态初始化

#### 格式1

int[][] arr = new int[3][2];

定义了名称为arr的二维数组

二维数组中有3个一维数组

每一个一维数组中有2个元素

一维数组的名称分别为arr[0], arr[1], arr[2]

给第一个一维数组1脚标位赋值为78写法是：arr[0][1] = 78

#### 格式2

int[][] arr = new int[3][];

二维数组中有3个一维数组。

每个一维数组都是默认初始化值null (注意：区别于格式1）

可以对这个三个一维数组分别进行初始化

arr[0] = new int[3]; 
arr[1] = new int[1]; 
arr[2] = new int[2];

注：int[][]arr = new int[][3]; //非法

### 静态初始化

int[][] arr = new int[][]{{3,8,2},{2,7},{9,0,1,6}};

定义一个名称为arr的二维数组，二维数组中有三个一维数组

每一个一维数组中具体元素也都已初始化

第一个一维数组 arr[0] = {3,8,2};

第二个一维数组 arr[1] = {2,7};

第三个一维数组 arr[2] = {9,0,1,6};

第三个一维数组的长度表示方式：arr[2].length

注意特殊写法情况：int[] x,y[]; x是一维数组，y是二维数组。

Java中多维数组不必都是规则矩阵形式

![](https://s1.ax1x.com/2020/07/12/U3xeaR.md.png)

![](https://s1.ax1x.com/2020/07/12/U3xQxO.md.png)

![](https://s1.ax1x.com/2020/07/12/U3x1MD.md.png)

## Arrays工具类的使用

java.util.Arrays类即为操作数组的工具类，包含了用来操作数组（比如排序和搜索）的各种方法

![](https://s1.ax1x.com/2020/07/12/U3zz90.png)

java.util.Arrays类的sort()方法提供了数组元素排序功能：

    import java.util.Arrays;
    
    public class SortTest {
    	public static void main(String[] args) {
    		int [] numbers = {5,900,1,5,77,30,64,700};
    		Arrays.sort(numbers);
    		for(int i = 0; i < numbers.length; i++){
    			System.out.println(numbers[i]);
    		 }
    	 } 
    }

## 数组使用中的常见异常

编译时，不报错！！

### 数组脚标越界异常(ArrayIndexOutOfBoundsException)

int[] arr = new int[2];

System.out.println(arr[2]);

System.out.println(arr[-1]);

访问到了数组中的不存在的脚标时发生

### 空指针异常(NullPointerException)

int[] arr = null;

System.out.println(arr[0]);

arr引用没有指向实体，却在操作实体中的元素时。