---
layout: post
title: "SpringMvc 学习笔记（2）——RequestMapping"
date: 2020-07-06 12:22:35 +0800
categories: notes
tags: springMvc
img: https://s1.ax1x.com/2020/07/06/UC7LAH.png
---
RequestMapping（映射请求）:Ant风格;Rest风格;限定请求方法;限定参数;限定请求头信息

## 标准URL映射

@RequestMapping(value=”xxx”)

在springmvc众多Controller以及每个Controller的众多方法中，请求时如何映射到具体的处理方法上

它可以定义在方法上，也可以定义在类上

请求映射的规则：

类上的@RequestMapping的value+方法上的@RequestMapping的value，如果value不以“/”开头，springmvc会自动加上
类上的@RequestMapping可省略，这时请求路径就是方法上的@RequestMapping的value

    
    @RequestMapping("hello")
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

![](https://s1.ax1x.com/2020/07/06/UiLzJe.png)


## Ant风格的映射

* ?：通配一个字符
* *：通配0个或者多个字符
* **：通配0个或者多个路径

    @RequestMapping("aa?/show2")
    public ModelAndView test2(){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello");
        mv.addObject("msg", "ant风格的映射：？");
        return mv;
    }
    
    @RequestMapping("bb*/show3")
    public ModelAndView test3(){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello");
        mv.addObject("msg", "ant风格的映射：*");
        return mv;
    }
    
    @RequestMapping("**/show4")
    public ModelAndView test4(){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello");
        mv.addObject("msg", "ant风格的映射：**");
        return mv;
    }

![](https://s1.ax1x.com/2020/07/06/UiOSRH.png)

## Rest风格的映射（占位符的映射）

![](https://s1.ax1x.com/2020/07/06/UiLxiD.md.png)

@RequestMapping(value=“/user/{userId}/{name} ")

请求URL：http://localhost:8080/user/1001/zhangsan.do

这种方式虽然和通配符“*”类似，却比通配符更加强大，占位符除了可以起到通配的作用，最精要的地方是在于它还可以传递参数。

比如：通过@PathVariable(“userId”) Long id, @PathVariable(“name”)String name获取对应的参数。

注意：@PathVariable(“key”)中的key必须和对应的占位符中的参数名一致，而方法形参的参数名可任意取

    @RequestMapping("show5/{name}/{id}")
    public ModelAndView test5(@PathVariable("name")String name, @PathVariable("id")Long id){
        ModelAndView mv = new ModelAndView();
        mv.setViewName("hello");
        mv.addObject("msg", "rest风格的映射：name=" + name + ",id=" + id);
        return mv;
    }

![](https://s1.ax1x.com/2020/07/06/UiLXdK.png)

如果传递的参数类型和接受参数的形参类型不一致，则会自动转换，如果转换出错（例如：id传了abc字符串，方法形参使用Long来接受参数），则会报400错误（参数列表错误）。

![](https://s1.ax1x.com/2020/07/06/UiLOZ6.png)


## 限定请求方法的映射

@RequestMapping(value=””, method=RequestMethod.POST)

    @RequestMapping(value = "show6", method = RequestMethod.POST)
    public ModelAndView test6(){
    ModelAndView mv = new ModelAndView();
    mv.setViewName("hello");
    mv.addObject("msg", "限定请求方法的映射：POST");
    return mv;
    }

用到了框架提供的RequestMethod枚举类，源代码截图：

![](https://s1.ax1x.com/2020/07/06/UiLzJe.png)

此时show6限定请求方法为POST请求，如果通过浏览器地址栏输入请求路径（也就是GET请求），结果：

![](https://s1.ax1x.com/2020/07/06/UiOpzd.md.png)


输入：http://localhost:8080/springmvc01/show6.do

限定多种请求方法

@RequestMapping(value=””, method={RequestMethod.POST, RequestMethod.GET})

    @RequestMapping(value = "show7", method = {RequestMethod.POST, RequestMethod.GET})
    public ModelAndView test7(){
    ModelAndView mv = new ModelAndView();
    mv.setViewName("hello");
    mv.addObject("msg", "限定请求方法的映射：POST");
    return mv;
    }
    
![](https://s1.ax1x.com/2020/07/06/UiOPsI.md.png)

![](https://s1.ax1x.com/2020/07/06/UiOEo8.md.png)

## 限定参数的映射

@RequestMapping(value=””,params=””)

params=”userId”：请求参数中必须带有userId

params=”!userId”：请求参数中不能包含userId

params=”userId=1”：请求参数中userId必须为1

params=”userId!=1”：请求参数中userId必须不为1，参数中可以不包含userId

params={“userId”, ”name”}：请求参数中必须有userId，name参数


    @RequestMapping(value = "show8", params = "id")
    public ModelAndView test8() {
        ModelAndView mv = new ModelAndView("hello");
        mv.addObject("msg", "springmvc的映射之限定请求参数，id");
        return mv;
    }

    @RequestMapping(value = "show9", params = "!id")
    public ModelAndView test9() {
        ModelAndView mv = new ModelAndView("hello");
        mv.addObject("msg", "springmvc的映射之限定请求参数，!id");
        return mv;
    }

    @RequestMapping(value = "show10", params = "id=1")
    public ModelAndView test10() {
        ModelAndView mv = new ModelAndView("hello");
        mv.addObject("msg", "springmvc的映射之限定请求参数，id=1");
        return mv;
    }

    @RequestMapping(value = "show11", params = "id!=1")
    public ModelAndView test11() {
        ModelAndView mv = new ModelAndView("hello");
        mv.addObject("msg", "springmvc的映射之限定请求参数，id!=1");
        return mv;
    }

    @RequestMapping(value = "show12", params = { "id", "name" })
    public ModelAndView test12() {
        ModelAndView mv = new ModelAndView("hello");
        mv.addObject("msg", "springmvc的映射之限定请求参数，id,name");
        return mv;
    }

http://localhost:8080/springmvc01/show9.do?id=wjw

![](https://s1.ax1x.com/2020/07/06/UiOmWQ.png)


http://localhost:8080/springmvc01/show8.do?id=wjw

![](https://s1.ax1x.com/2020/07/06/UiOiLt.md.png)

![](https://s1.ax1x.com/2020/07/06/UiOeJg.md.png)

## 限定请求头信息的映射

@RequestMapping(value=””, heads=””)

    /**
     * 1.请求头信息必须包含User-Agent
     * 2.User-Agent头参数的值必须为Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36
     *   即：限定浏览器必须是谷歌浏览器，而且版本还是Chrome/69.0.3497.100
     * @return
     */
    @RequestMapping(value = "show13", headers = "User-Agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36")
    public ModelAndView test13() {
    ModelAndView mv = new ModelAndView("hello");
    mv.addObject("msg", "限定请求头信息");
    return mv;
    }

![](https://s1.ax1x.com/2020/07/06/UiOmWQ.png)

火狐浏览器测试：

![](https://s1.ax1x.com/2020/07/06/UiOnzj.png)


## 组合注解

GetMapping：相当于RequestMapping（method = RequestMethod.GET）

PostMapping：相当于RequestMapping（method = RequestMethod.POST）

PutMapping：相当于RequestMapping（method = RequestMethod.PUT）

DeleteMapping：相当于RequestMapping（method = RequestMethod.DELETE）

	 @GetMapping(value = "show14")
    public ModelAndView test14() {
        ModelAndView mv = new ModelAndView("hello");
        mv.addObject("msg", "GetMapping");
        return mv;
    }

    @PostMapping(value = "show15")
    public ModelAndView test15() {
        ModelAndView mv = new ModelAndView("hello");
        mv.addObject("msg", "PostMapping");
        return mv;
    }

    @PutMapping(value = "show16")
    public ModelAndView test16() {
        ModelAndView mv = new ModelAndView("hello");
        mv.addObject("msg", "PutMapping");
        return mv;
    }

    @DeleteMapping(value = "show17")
    public ModelAndView test17() {
        ModelAndView mv = new ModelAndView("hello");
        mv.addObject("msg", "DeleteMapping");
        return mv;
    }

![](https://s1.ax1x.com/2020/07/06/UiOKQs.md.png)

![](https://s1.ax1x.com/2020/07/06/UiOMyn.md.png)