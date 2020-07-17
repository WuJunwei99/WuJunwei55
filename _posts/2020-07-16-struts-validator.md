---
layout: post
title: "struts2的验证器"
date: 2020-07-16 10:28:45 +0800
categories: notes
tags: struts2
img: https://s1.ax1x.com/2020/07/16/UDF2WR.png
---
Struts2的预定义验证器；自定义验证器



### 一个好的验证框架必须要考虑的事情:

1. 验证功能的复用性。
2. 验证功能的可扩展性。
3. 验证与业务逻辑分离

Struts2提供了强大的验证框架：在 xwork-core-2.3.24.jar 包下，在 \com\opensymphony\xwork2\validator\validators 路径下找一个名字为“ default.xml ”的 xml 文件

## Struts2的预定义验证器

    <validators>
	    <validator name="required" class="com.opensymphony.xwork2.validator.validators.RequiredFieldValidator"/>
	    <validator name="requiredstring" class="com.opensymphony.xwork2.validator.validators.RequiredStringValidator"/>
	    <validator name="int" class="com.opensymphony.xwork2.validator.validators.IntRangeFieldValidator"/>
	    <validator name="long" class="com.opensymphony.xwork2.validator.validators.LongRangeFieldValidator"/>
	    <validator name="short" class="com.opensymphony.xwork2.validator.validators.ShortRangeFieldValidator"/>
	    <validator name="double" class="com.opensymphony.xwork2.validator.validators.DoubleRangeFieldValidator"/>
	    <validator name="date" class="com.opensymphony.xwork2.validator.validators.DateRangeFieldValidator"/>
	    <validator name="expression" class="com.opensymphony.xwork2.validator.validators.ExpressionValidator"/>
	    <validator name="fieldexpression" class="com.opensymphony.xwork2.validator.validators.FieldExpressionValidator"/>
	    <validator name="email" class="com.opensymphony.xwork2.validator.validators.EmailValidator"/>
	    <validator name="url" class="com.opensymphony.xwork2.validator.validators.URLValidator"/>
	    <validator name="visitor" class="com.opensymphony.xwork2.validator.validators.VisitorFieldValidator"/>
	    <validator name="conversion" class="com.opensymphony.xwork2.validator.validators.ConversionErrorFieldValidator"/>
	    <validator name="stringlength" class="com.opensymphony.xwork2.validator.validators.StringLengthFieldValidator"/>
	    <validator name="regex" class="com.opensymphony.xwork2.validator.validators.RegexFieldValidator"/>
	    <validator name="conditionalvisitor" class="com.opensymphony.xwork2.validator.validators.ConditionalVisitorFieldValidator"/>
    </validators>

当需要验证某些字段的时候,用户提交的个人信息等，可以在 需要验证的 xxxAction 同包下创建 ActionClassName-validation.xml，那么就会对这个Action中的所有方法都进行验证，如果只希望对Action中的某个方法起作用，那么验证文件取名为 ActionClassName-actionName-validation.xml，需要注意的是actionName是Struts.xml 中的 <action name="">的action的名称。 

UserAction中有个login方法,Struts.xml中 <action name="user_login">,那么验证文件名为:
UserAction-user_login-validation.xml

### 几种常用的验证器,解析

	<!-- 必填校验 -->   
	  <!-- 非字段校验 -->   
	  <validator type="required">   
	      <param name="fidleName">field</param>                    
	      <message>请输入数据</message>   
	  </validator>   
	  <!-- 字段校验 -->   
	  <field name="field">   
	      <field-validator type="required">              
	          <message>请输入数据</message>   
	      </field-validator>   
	  </field>   
	  
	  <!-- 必填字符串校验 -->   
	  <!-- 非字段校验 -->   
	  <validator type="requiredstring">   
	      <param name="fidleName">field</param>   
	      <param name="trim">true</param>                   
	      <message>请输入数据</message>   
	  </validator>   
	  <!-- 字段校验 -->   
	  <field name="field">   
	      <field-validator type="requiredstring">   
	          <param name="trim">true</param>             
	          <message>请输入数据</message>   
	      </field-validator>   
	  </field>   
	  
	  <!-- 整数校验 -->   
	  <!-- 非字段校验 -->   
	  <validator type="int">   
	      <param name="fidleName">field</param>                    
	      <param name="min">1</param>   
	      <param name="max">80</param>   
	      <message>数字必须在${min}-${max}岁之间</message>   
	  </validator>   
	  <!-- 字段校验 -->   
	  <field name="field">   
	      <field-validator type="int">   
	          <param name="min">1</param>   
	          <param name="max">80</param>   
	          <message>数字必须在${min}-${max}岁之间</message>   
	      </field-validator>   
	  </field>   
	  
	  <!-- 浮点校验 -->   
	  <!-- 非字段校验 -->   
	  <validator type="double">   
	      <param name="fidleName">field</param>   
	      <param name="minExclusive">0.1</param>   
	      <param name="maxExclusive">10.1</param>              
	      <message>输入浮点无效</message>   
	  </validator>   
	  <!-- 字段校验 -->   
	  <field name="field">   
	      <field-validator type="double">   
	          <param name="minExclusive">0.1</param>   
	          <param name="maxExclusive">10.1</param>              
	          <message>输入浮点无效</message>   
	      </field-validator>   
	  </field>   
	    
	  <!-- 日期校验 -->   
	  <!-- 非字段校验 -->   
	  <validator type="date">   
	      <param name="fidleName">field</param>   
	      <param name="min">2009-01-01</param>   
	      <param name="max">2019-01-01</param>   
	      <message>日期无效</message>   
	  </validator>   
	  <!-- 字段校验 -->   
	  <field name="field">   
	      <field-validator type="date">   
	         <param name="min">2009-01-01</param>   
	         <param name="max">2019-01-01</param>   
	         <message>日期无效</message>   
	      </field-validator>   
	  </field>   
	  
	  <!-- 表达式校验 -->   
	  <!-- 非字段校验 -->   
	  <validator type="expression">   
	      <param name="expression">password==repassword</param>   
	          <message>两者输入不一致</message>   
	  </validator>   
	    
	  <!-- 字段表达式校验 -->   
	  <!-- 非字段校验 -->   
	  <validator type="fieldexpression">   
	      <param name="expression">password==repassword</param>   
	      <message>两者输入不一致</message>   
	  </validator>   
	  <!-- 字段校验 -->   
	  <field name="field">   
	      <field-validator type="fieldexpression">   
	          <param name="expression"><![CDATA[#password==#repassword]]></param>   
	          <message>两者输入不一致</message>   
	      </field-validator>   
	  </field>   
	  
	  <!-- 邮件校验 -->   
	  <!-- 非字段校验 -->   
	  <validator type="email">   
	      <param name="fidleName">field</param>                    
	      <message>非法邮件地址</message>   
	  </validator>   
	  <!-- 字段校验 -->   
	  <field name="field">   
	      <field-validator type="email">   
	          <message>非法邮件地址</message>   
	      </field-validator>   
	  </field>   
	    
	  <!-- 网址校验 -->   
	  <!-- 非字段校验 -->   
	  <validator type="url">   
	      <param name="fidleName">field</param>                
	      <message>无效网址</message>   
	  </validator>   
	  <!-- 字段校验 -->   
	  <field name="field">   
	      <field-validator type="url">                
	          <message>无效网址</message>   
	      </field-validator>   
	  </field>   
	    
	  <!-- visitor校验 -->   
	  <!-- 非字段校验 -->   
	  <validator type="visitor">   
	      <param name="fidleName">field</param>   
	      <param name="context">fieldContext</param>   
	      <param name="appendPrefix">true</param>                
	      <message>输入校验</message>   
	  </validator>   
	  <!-- 字段校验 -->   
	  <field name="field">   
	      <field-validator type="visitor">               
	          <param name="context">fieldContext</param>   
	          <param name="appendPrefix">true</param>                
	          <message>输入校验</message>   
	      </field-validator>   
	  </field>   
	    
	  <!-- 类型转换校验 -->   
	  <!-- 非字段校验 -->   
	  <validator type="conversion">   
	      <param name="fidleName">field</param>   
	      <message>类型转换错误</message>   
	  </validator>   
	  <!-- 字段校验 -->   
	  <field name="field">   
	      <field-validator type="conversion">   
	          <message>类型转换错误</message>   
	      </field-validator>   
	  </field>   
	    
	  <!-- 字符串长度校验 -->   
	  <!-- 非字段校验 -->   
	  <validator type="stringlength">   
	      <param name="fidleName">field</param>   
	      <param name="minLength">1</param>   
	      <param name="maxLength">10</param>   
	      <param name="trim">true</param>                   
	      <message>字符串长度必须为10位</message>   
	  </validator>   
	  <!-- 字段校验 -->   
	  <field name="field">   
	      <field-validator type="stringlength">   
	          <param name="minLength">1</param>   
	          <param name="maxLength">10</param>   
	          <param name="trim">true</param>                
	          <message>字符串长度必须为10位</message>   
	      </field-validator>   
	  </field>   
	    
	  <!-- 正则表达式校验 -->   
	  <!-- 非字段校验 -->   
	  <validator type="regex">   
	      <param name="fidleName">field</param>                    
	      <param name="regexExpression"><![CDATA[(/^13[13567890](\d{8})$/)]]></param>   
	      <message>手机号码必须为数字并且是11位</message>   
	  </validator>   
	  <!-- 字段校验 -->   
	  <field name="field">   
	      <field-validator type="regex">   
	          <param name="regexExpression"><![CDATA[(/^13[13567890](\d{8})$/)]]></param>   
	          <message>手机号码必须为数字并且是11位</message>   
	      </field-validator>   
	  </field> 


### 验证时机

验证发生在execute方法之前，在struts2 的params拦截器已经把请求的参数通过反射设置到 Action 的属性之后,所以,验证框架实际上**验证的是值栈中的值**

### 验证的结果

如果用户输入的参数完全满足验证结果，那么会继续执行execute方法。如果不满足，会跳转到Action配置中的result name="input" 的页面中中

![](https://s1.ax1x.com/2020/07/17/UsAOZ6.png)

## 自定义验证器

检验输入的字符串中是否包含中文：

思路:

1. 获取 中文 和 英文 的长度
2. 比如 "哈哈" 与 "aa" 长度都是 2 ，所以无法判断
3. 一个中文占两个字符,而英文还是占一个字节
4. 所以要判断是否包含中文只需判断字节长度与字符长度是否一致即可。


		import com.opensymphony.xwork2.validator.ValidationException;
		import com.opensymphony.xwork2.validator.validators.FieldValidatorSupport;
		 
		public class ChineseValidator  extends FieldValidatorSupport{

		public void validate(Object object) throws ValidationException {			
			
			//获取字段名
			final String fieldName = this.getFieldName();
			//获取字段值
			final String fieldValue = (String) this.getFieldValue(fieldName, object);		
			//字节数
			final int bytes = fieldValue.getBytes().length;
			//字符数
			final int chars = fieldValue.length();
			
			if(mode.equals("some")){
				
				if (chars==bytes||chars*2==bytes){
					this.addFieldError(fieldName, object);
				}
			
		}
	}

### 声明自定义的验证器

在src下创建validators.xml

	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE validators PUBLIC
	        "-//OpenSymphony Group//XWork Validator Config 1.0//EN"        "http://www.opensymphony.com/xwork/xwork-validator-config-1.0.dtd">
	 
	<validators>
	    <validator name="chinese" class="xx.xxx.ChineseValidator"/>
	</validators>

Struts2的2.0.8版本之前,一旦使用自定义的validators.xml文件,那么系统就不会再调用Struts2预定义的验证器,所以需要将struts2默认的内容拷贝到自定义的validators.xml中。


### 引用自定义的验证器

在UserAction-user_login.validation.xml中


	<validators>
		<field name="username">
			<field-validator type="chinese">
				
				<message>用户账号，只能输入非中文的字符</message>
			</field-validator>
		</field>
	</validators>

在页面上显示验证错误信息

字段验证器:<s:filederror>,不指定filedName，则默认显示所有的字段验证的错

动作验证器:<s:actionerror>,同上


### 验证器短路

当一个字段有多个验证条件的时候，在第一次出现验证错误后，就立即终止验证，后续对这个字段的验证就不再进行了，这就叫验证器短路.

在指明验证器时,添加 short-circuit="true"

	
	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE validators PUBLIC
	        "-//OpenSymphony Group//XWork Validator 1.0.2//EN"        "http://www.opensymphony.com/xwork/xwork-validator-1.0.2.dtd">
	  		
	<validators>
		<field name="user.age">
			<field-validator type="int" short-circuit="true">
				<param name="min">15</param>
				<message>年龄要大于等于15岁</message>
			</field-validator>
		</field>
	</validators>

注意：只有 Validator1.0.2 的版本才能正确执行验证器短路。

