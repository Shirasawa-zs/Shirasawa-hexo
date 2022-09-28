---
title: SpringBoot简介1
date: 2022-09-27 20:25:00
author: scy
hide: false
cover: true
toc: false
summary: Spring和SpringBoot1
categories: 后端
tags: 
  - SpringBoot
  - java
---

### SpringBoot和Spring简介

**注解学习：[(32条消息) 历时三个月，史上最详细的Spring注解驱动开发系列教程终于出炉了，给你全新震撼_李阿昀的博客-CSDN博客](https://liayun.blog.csdn.net/article/details/115053350)**

**csdn虽然很垃圾，但里面还是有大佬的...,垃圾堆里刨食**



##### 1.概述

java网页端的开发过程如下javaweb -> ssm -> springboot

其中ssm常规理解是spring + springMVC + Mybatis

本质上来说，springMVC是spring家族的一个模块，也就是人们常说的spring framework

而spring家族的集大成者springboot可以看作是spring framework的一种抽象实现



##### 2.为什么要用springboot

springboot的重点之一就是**自动配置**

在传统的SSM框架集成中，需要编写大量的XML配置文件，比如集成Mybatis时，需要编写mybatis_config.xml文件，在集成springmvc时，需要编写springmvc.xml文件，这些配置文件十分繁琐，还很容易出现错误，导致开发效率低。而Spring Boot采用约定大于配置的思想，将大量的spring配置文件集成到Spring Boot的内部，帮助开发人员自动配置各类XML文件，极大的简化了开发过程。
以上是大部分人对springboot配置的定义，实际上，我感觉，是程序员感觉配置类比xml文件好用，所以决定用配置类和注解



##### 3.IOC

IOC是满足OCP原则的

IOC是一种抽象的概念，java里面具体理解可以分为三个

1.创建容器

2.对象加入容器

3.注入

抽象理解的话就是老生常谈的问题

控制反转

作为开发者，我们只需要考虑把对象加入容器，然后注入我们所需要的对象就行了，作为设计IOC容器的人，他需要考虑的是灵活注入满足更灵活的OCP场景,这也是我们需要学习的。



##### 4.应用

**简单应用：就两个注解就行**

@Component @Autowired

注入容器并取出

```java
@Component
public class Aki {
    public void R(){
        System.out.println("Aki R");
    }
}

```



```JAVA

@RestController
public class TestController {
    @Autowired
    private Aki aki;

    @RequestMapping("/")
    public void get(){
        aki.R();
    }
}
```

更深入一点

@Component

@Controller

@Service

@Repository

@Configuration

前四个用来解决 组件/类和忘了叫什么的new的问题，只不过所属层级不同



##### 5.注入相关

IOC对象的实例化，注入时机都是在容器创建就完成的，可以用@lazy延迟实例化加载，不过如果一个类里面引用了其他对象，那么作为属性的对象的延迟加载无效。

@lazy一般不用，了解一下就行



三种注入方式

属性注入

```JAVA
@Autowired
    private Aki aki;
```

构造器注入

```java
private Aki aki;
    
    @Autowired
    public TestController(Aki aki) {
        this.aki = aki;
    }

    @RequestMapping("/")
    public void get(){
        aki.R();
    }
```

setter注入

```java
 private Aki aki;

    @Autowired
    public void setAki(Aki aki){
        this.aki = aki;
    }
    @RequestMapping("/")
    public void get(){
        aki.R();
    }
```



为了代码的稳定性，我们可以引入抽象的概念

```JAVA
public interface IHero {
    void R();
}

-----------------------------------------------------
@Component
public class Aki implements IHero{

    public Aki(){
        System.out.println("Aki ... constructor");
    }
    public void R(){
        System.out.println("Aki R");
    }
}
-----------------------------------------------------
@Component
public class Niko implements IHero{
    public void R(){
        System.out.println("niko R");
    }
}
------------------------------------------------------
    @RestController
public class TestController {

    @Autowired
    private IHero aki;

    @RequestMapping("/")
    public void get(){
        aki.R();
    }
}
```

满足DIP，高层的实现不依赖于底层,此时我们应该有个疑问，

**@Autowired**

**private IHero aki;**

如何实现自动装配

经过实验，可以得出以下结论

```xml
//默认bytype
IHero < 1 error
      = 1 注入默认存在的一个
      > 1 不一定报错，先根据字段名进行匹配，没有找到就报错
    
//我们可以通过byname指定
    @Autowired
    @Qualifier(value = "字段名")
```

这时我们就该考虑更深层次的问题，如果我们新增多个类，如何减少变化带来的影响，来保持我们代码的稳定性，也为了满足ocp原则

- 制造一个interface,多个类实现这个interface

设计模式的策略模式

- 一个类的属性去解决变化，用配置文件去解决，不过还是建议实现接口，哪怕只有一个类

栗子：MySQL的连接配置文件可以在application.yaml或者application.propertis，把所有的不稳定隔离到配置文件，我们又知道配置文件满足这个

**配置文件可以理解是外部的输入，就如同我们的用户输入意义，是一种变化，它的改变不违法OCP原则**借此来解决。



##### 6.@Configuration

简单的理解，@Configuration是为了解决原来IOC容器无法对属性进行赋值的问题，它的角色更像是原来xml文件的作用，在springboot2还是3引入了这一概念

栗子

```java
@Configuration
public class Hero {

    @Bean
    public IHero createWolf(){
        return new Wolf();
    }
}
```

```java
public class Wolf implements IHero{

    public Wolf(){
        System.out.println("wolf ... constructor");
    }
    @Override
    public void R() {
        System.out.println("wolf R");
    }
}

```

```JAVA
@RestController
public class TestController {

    @Autowired
    private IHero Wolf;

    @RequestMapping("/")
    public void get(){
        Wolf.R();
    }
}
```

通过上述操作，我们实现了在@Configuration实现了bean的注入

接下来需要了解这个注解和xml，我们需要解决以下几个问题

**我们要明白为什么需要spring的配置？**

因为变化的出现，我们需要去隔离这些变化带给我们程序的不稳定，所以我们需要把所有的不稳定放到配置文件中，配置文件属于外部输入，也是一种变化，不影响OCP法则

>还有为什么隔离到配置文件

1.配置文件的集中性

2.业务逻辑的无关性，只需要关心配置就行

**@Configuration是否违反了OCP**

这个问题聚焦到@Configruation是否属于配置，这要从思想上来解决这个问题

**是**：对比其他配置文件，它虽然是以.java文件的后缀，但不否认的是他仍然是一个配置文件，配置文件的定义不应该以后缀为鉴别条件

**否**：相较于其他配置文件的简洁性，类似于SpringBoot的application,@Confiruation好像太过繁琐

> 实现自动配置(有缺陷）

```java
public interface IConnect {
        void connect();
}
```

```JAVA
public class Mysql implements IConnect{


    private String ip = "192.168.0.1";


    private String root = "999";

    public Mysql() {

    }

    public void connect(){
        System.out.println(this.ip + "   " + this.root);
    }

    public Mysql(String ip, String root) {
        this.ip = ip;
        this.root = root;
    }

    @Override
    public String toString() {
        return "Mysql{" +
                "ip='" + ip + '\'' +
                ", root='" + root + '\'' +
                '}';
    }

    public String getIp() {
        return ip;
    }

    public void setIp(String ip) {
        this.ip = ip;
    }

    public String getRoot() {
        return root;
    }

    public void setRoot(String root) {
        this.root = root;
    }
}

```

```java
@Configuration
public class Connect {

    @Value("${mysql.ip}")
    private String ip;

    @Value("${mysql.port}")
    private String root;

    @Bean
    public IConnect mysql(){
        return new Mysql(this.ip, this.root);
    }
}

```

```JAVA
mysql.ip=192.102.2.3
mysql.port=9999
```

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20220927232106.png)

