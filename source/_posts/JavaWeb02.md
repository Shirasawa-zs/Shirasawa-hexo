---
title: JavaWeb(黑马笔记版)
date: 2022-04-20 09:25:00
author: scy
top: false
hide: false
cover: false
toc: false
summary: JavaWeb02
categories: 后端 
tags:
  - Java
  - JavaWeb
---


# 1.前言

最近复习到了JavaWeb的知识，刷视频的效率实在低的吓人，所以打算以笔记的形式进行复习，复习的笔记选择了黑马培训班的笔记，学习方式是通过看笔记 + 做测试来实现，所以这是一篇偏转载的博客。里面大部分文字性知识来自黑马的笔记。

学习目标

> * 掌握Request对象的概念与使用
> * 掌握Response对象的概念与使用
> * 能够完成用户登录注册案例的实现
> * 能够完成SqlSessionFactory工具类的抽取

# 3.Request和Response的概述

```xml
1. service方法的两个参数request和response是由tomcat创建的
	void service(ServletRequest var1, ServletResponse var2)
2. request 表示请求数据, tomcat将浏览器发送过来的请求数据解析并封装到request对象中
		servlet开发者可以通过request对象获得请求数据
3. response 表示响应数据,服务器发送给浏览器的数据
		servlet开发者可以通过response对象设置响应数据
```

Request是请求对象，Response是响应对象。这两个对象在我们使用Servlet的时候有看到：

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230420164543.png)

此时，我们就需要思考一个问题request和response这两个参数的作用是什么?

* request:==获取==请求数据
  * 浏览器会发送HTTP请求到后台服务器[Tomcat]
  * HTTP的请求中会包含很多请求数据[请求行+请求头+请求体]
  * 后台服务器[Tomcat]会对HTTP请求中的数据进行解析并把解析结果存入到一个对象中
  * 所存入的对象即为request对象，所以我们可以从request对象中获取请求的相关参数
  * 获取到数据后就可以继续后续的业务，比如获取用户名和密码就可以实现登录操作的相关业务
* response:==设置==响应数据
  * 业务处理完后，后台就需要给前端返回业务处理的结果即响应数据
  * 把响应数据封装到response对象中
  * 后台服务器[Tomcat]会解析response对象,按照[响应行+响应头+响应体]格式拼接结果
  * 浏览器最终解析结果，把内容展示在浏览器给用户浏览

对于上述所讲的内容，我们通过一个案例来初步体验下request和response对象的使用。

```java
@WebServlet(name = "helloServlet", value = "/hello-servlet")
public class HelloServlet extends HttpServlet {
    private String message;
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //使用request对象 获取请求数据
        String name = request.getParameter("name");//url?name=zhangsan

        //使用response对象 设置响应数据
        response.setHeader("content-type","text/html;charset=utf-8");
        response.getWriter().write("<h1>"+name+",欢迎您！</h1>");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("Post...");
    }
}
```

**小结**

在这节中，我们主要认识了下request对象和reponse对象:

* request对象是用来封装浏览器请求tomcat服务器数据的对象
* response对象是用来封装tomcat服务器响应给浏览器的数据的对象

目前我们只知道这两个对象是用来干什么的，那么它们具体是如何实现的，就需要我们继续深入的学习。接下来，就先从Request对象来学习,主要学习下面这些内容:

* request继承体系

* request获取请求参数
* request请求转发

# 4.Request

### 4.1 Request继承体系

在学习这节内容之前，我们先思考一个问题，前面在介绍Request和Reponse对象的时候，比较细心的同学可能已经发现：

* 当我们的Servlet类实现的是Servlet接口的时候，service方法中的参数是ServletRequest和ServletResponse
* 当我们的Servlet类继承的是HttpServlet类的时候，doGet和doPost方法中的参数就变成HttpServletRequest和HttpServletReponse

那么，

* ServletRequest和HttpServletRequest的关系是什么?
* request对象是有谁来创建的?
* request提供了哪些API,这些API从哪里查?

首先，我们先来看下Request的继承体系:

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230420165850.png)

上图中可以看出，ServletRequest和HttpServletRequest都是Java提供的

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230420170052.png)

所以ServletRequest和HttpServletRequest是继承关系，并且两个都是接口，接口是无法创建对象，这个时候就引发了下面这个问题:

**这些参数中的对象由谁创建**

这个时候，我们就需要用到Request继承体系中的`RequestFacade`:

* 该类实现了HttpServletRequest接口，也间接实现了ServletRequest接口。
* Servlet类中的service方法、doGet方法或者是doPost方法最终都是由Web服务器[Tomcat]来调用的，所以Tomcat提供了方法参数接口的具体实现类，并完成了对象的创建
* 要想了解RequestFacade中都提供了哪些方法，我们可以直接查看JavaEE的API文档中关于ServletRequest和HttpServletRequest的接口文档，因为RequestFacade实现了其接口就需要重写接口中的方法

对于上述结论，要想验证，可以编写一个Servlet，在方法中把request对象打印下，就能看到最终的对象是不是RequestFacade,代码如下:

```java
@WebServlet("/demo2")
public class ServletDemo2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println(request);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    }
}
```

**小结**

* Request的继承体系为ServletRequest(最大父接口)-->HttpServletRequest(可以处理http协议的请求接口)-->RequestFacade(tomcat定义的实现类)
* Tomcat需要解析请求数据，封装为request对象,并且创建request对象传递到service方法
* 使用request对象，可以查阅JavaEE API文档的HttpServletRequest接口中方法说明

### 4.2 Request获取请求数据

HTTP请求数据总共分为三部分内容，分别是==请求行、请求头、请求体==，对于这三部分内容的数据，分别该如何获取，首先我们先来学习请求行数据如何获取?

#### 4.2.1 获取请求行数据

请求行包含三块内容，分别是`请求方式`、`请求资源路径`、`HTTP协议及版本`

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230420170455.png)

对于这三部分内容，request对象都提供了对应的API方法来获取，具体如下:

* 获取请求方式: `GET`

```
String getMethod()
```

* 获取虚拟目录(项目访问路径): `/request-demo`

```
String getContextPath()
```

* 获取URL(统一资源定位符): `http://localhost:8080/request-demo/req1`

```
StringBuffer getRequestURL()
```

* 获取URI(统一资源标识符): `/request-demo/req1`

```
String getRequestURI()
```

* 获取请求参数(GET方式): `username=zhangsan&password=123`

```
String getQueryString()
```

介绍完上述方法后，咱们通过代码把上述方法都使用下:

```java
@WebServlet("/req1")
public class RequestDemo1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // String getMethod()：获取请求方式： GET
        String method = req.getMethod();
        System.out.println(method);//GET
        // String getContextPath()：获取虚拟目录(项目访问路径)：/request-demo
        String contextPath = req.getContextPath();
        System.out.println(contextPath);
        // StringBuffer getRequestURL(): 获取URL(统一资源定位符)：http://localhost:8080/request-demo/req1
        StringBuffer url = req.getRequestURL();
        System.out.println(url.toString());
        // String getRequestURI()：获取URI(统一资源标识符)： /request-demo/req1
        String uri = req.getRequestURI();
        System.out.println(uri);
        // String getQueryString()：获取请求参数（GET方式）： username=zhangsan
        String queryString = req.getQueryString();
        System.out.println(queryString);
    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    }
}
```

启动服务器，访问`http://localhost:8080/request-demo/req1?username=zhangsan&passwrod=123`

输出结果

```xml
GET
/demo_war_exploded
http://localhost:8080/demo_war_exploded/req1
/demo_war_exploded/req1
username=zhangsan&passwrod=123
```

#### 4.2.2 获取请求头数据

对于请求头的数据，格式为`key: value`如下:

所以根据请求头名称获取对应值的方法为:

```
String getHeader(String name) 参数name书写的是请求头冒号左边的内容例如：User-Agent
	User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) 	Chrome/96.0.4664.110 Safari/537.36
```

接下来，在代码中如果想要获取客户端浏览器的版本信息，则可以使用

```java
/**
 * request 获取请求数据
 */
@WebServlet("/req1")
public class RequestDemo1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //获取请求头: user-agent: 浏览器的版本信息
        String agent = req.getHeader("user-agent");
		System.out.println(agent);
    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    }
}

```

重新启动服务器后，`http://localhost:8080/request-demo/req1?username=zhangsan&passwrod=123`，获取的结果如下:

```xml
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36 Edg/112.0.1722.48
```

#### 4.2.3 获取请求体数据

**浏览器在发送GET请求的时候是没有请求体的，所以需要把请求方式变更为POST**，请求体中的数据格式如下:

对于请求体中的数据，Request对象提供了如下两种方式来获取其中的数据，分别是:

* 获取字节输入流，如果前端发送的是字节数据，比如传递的是文件数据，则使用该方法

```
ServletInputStream getInputStream()
该方法可以获取字节
```

* 获取字符输入流，如果前端发送的是纯文本数据，则使用该方法

```
BufferedReader getReader()
```

接下来，大家需要思考，要想获取到请求体的内容该如何实现?

具体实现的步骤如下:

> 1.准备一个页面，在页面中添加form表单,用来发送post请求
>
> 2.在Servlet的doPost方法中获取请求体数据
>
> 3.在doPost方法中使用request的getReader()或者getInputStream()来获取
>
> 4.访问测试

1. 在项目的webapp目录下添加一个html页面，名称为：`req.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<!-- 
    action:form表单提交的请求地址
    method:请求方式，指定为post
-->
<form action="/request-demo/req1" method="post">
    <input type="text" name="username">
    <input type="password" name="password">
    <input type="submit">
</form>
</body>
</html>
```

2. 在Servlet的doPost方法中获取数据

```java
/**
 * request 获取请求数据
 */
@WebServlet("/req1")
public class RequestDemo1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //在此处获取请求体中的数据
    }
}
```

3. 调用getReader()或者getInputStream()方法，因为目前前端传递的是纯文本数据，所以我们采用getReader()方法来获取

```java
/**
 * request 获取请求数据
 */
@WebServlet("/req1")
public class RequestDemo1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
         //获取post 请求体：请求参数
        //1. 获取字符输入流
        BufferedReader br = req.getReader();
        //2. 读取数据
        String line = br.readLine();
        System.out.println(line);
    }
}
```

==注意==

BufferedReader流是通过request对象来获取的，当请求完成后request对象就会被销毁，request对象被销毁后，BufferedReader流就会自动关闭，所以此处就不需要手动关闭流了。

4. 启动服务器，通过浏览器访问`http://localhost:8080/request-demo/req.html`

点击`提交`按钮后，就可以在控制台看到前端所发送的请求数据

**小结**

HTTP请求数据中包含了`请求行`、`请求头`和`请求体`，针对这三部分内容，Request对象都提供了对应的API方法来获取对应的值:

* 请求行
  * getMethod()获取请求方式 GET POST
  * getContextPath()获取项目访问虚拟路径 /day06
  * getRequestURL()获取请求URL http://localhost:8080/day06/demo01
  * getRequestURI()获取请求URI /day06/demo01
  * getQueryString()获取GET请求方式的请求参数，获取的请求参数放到一个字符串中 了解
* 请求头
  * getHeader(String name)根据请求头名称获取其对应的值
* 请求体
  * 注意: ==浏览器发送的POST请求才有请求体==
  * 如果是纯文本数据:getReader()  了解
  * 如果是字节数据如文件数据:getInputStream()

#### 4.2.4 获取请求参数的通用方式(很重要)

在学习下面内容之前，我们先提出两个问题:

* 什么是请求参数?
* 请求参数和请求数据的关系是什么?

1.什么是请求参数?

为了能更好的回答上述两个问题，我们拿用户登录的例子来说明

1.1 想要登录网址，需要进入登录页面

1.2 在登录页面输入用户名和密码

1.3 将用户名和密码提交到后台

1.4 后台校验用户名和密码是否正确

1.5 如果正确，则正常登录，如果不正确，则提示用户名或密码错误

上述例子中，**用户名和密码其实就是我们所说的请求参数。**

**get请求：请求参数位于url后面。**

**post请求：请求参数位于请求体中。**

2.什么是请求数据?

请求数据则是包含请求行、请求头和请求体的所有数据

3.请求参数和请求数据的关系是什么?

3.1 请求参数是请求数据中的部分内容

3.2 如果是GET请求，请求参数在请求行中

3.3 如果是POST请求，请求参数一般在请求体中

对于请求参数的获取,常用的有以下两种:

* GET方式:

```
String getQueryString()
```

* POST方式:

```
BufferedReader getReader();
```

有了上述的知识储备，我们来实现一个案例需求:

（1）发送一个GET请求并携带用户名，后台接收后打印到控制台

（2）发送一个POST请求并携带用户名，后台接收后打印到控制台

此处大家需要注意的是GET请求和POST请求接收参数的方式不一样，具体实现的代码如下:

```java
@WebServlet("/req1")
public class RequestDemo1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        String result = req.getQueryString();
        System.out.println(result);

    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        BufferedReader br = req.getReader();
        String result = br.readLine();
        System.out.println(result);
    }
}
```

GET请求和POST请求获取请求参数的方式不一样，在获取请求参数这块该如何实现呢?==

要想实现，我们就需要思考:

GET请求方式和POST请求方式区别主要在于获取请求参数的方式不一样，是否可以提供一种统一获取请求参数的方式，从而统一doGet和doPost方法内的代码?

解决方案一:

```java
@WebServlet("/req1")
public class RequestDemo1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //获取请求方式
        String method = req.getMethod();
        //获取请求参数
        String params = "";
        if("GET".equals(method)){
            params = req.getQueryString();
        }else if("POST".equals(method)){
            BufferedReader reader = req.getReader();
            params = reader.readLine();
        }
        //将请求参数进行打印控制台
        System.out.println(params);
      
    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        this.doGet(req,resp);
    }
}
```

使用request的getMethod()来获取请求方式，根据请求方式的不同分别获取请求参数值，这样就可以解决上述问题，但是以后每个Servlet都需要这样写代码，实现起来比较麻烦，这种方案我们不采用

解决方案二:

request对象已经将上述获取请求参数的方法进行了封装，并且request提供的方法实现的功能更强大，以后只需要调用request提供的方法即可，在request的方法中都实现了哪些操作?

(1)根据不同的请求方式获取请求参数，获取的内容如下:

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230420174433.png)

(2)把获取到的内容进行分割，内容如下:

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230420174437.png)

(3)把分割后端数据，存入到一个Map集合中:

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230420174440.png)

**注意**:因为参数的值可能是一个，也可能有多个，所以Map的值的类型为String数组。

基于上述理论，request对象为我们提供了如下方法:

基于上述理论，request对象为我们提供了如下方法:

* 获取所有参数Map集合

```
Map<String,String[]> getParameterMap()
```

* 根据名称获取参数值（数组）

```
String[] getParameterValues(String name)
```

* 根据名称获取参数值(单个值)

```
String getParameter(String name)
```

接下来，我们通过案例来把上述的三个方法进行实例演示:

1.修改req.html页面，添加爱好选项，爱好可以同时选多个

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="/request-demo/req2" method="get">
    <input type="text" name="username"><br>
    <input type="password" name="password"><br>
    <input type="checkbox" name="hobby" value="1"> 游泳
    <input type="checkbox" name="hobby" value="2"> 爬山 <br>
    <input type="submit">

</form>
</body>
</html>
```

2.在Servlet代码中获取页面传递GET请求的参数值

 2.1获取GET方式的所有请求参数

```java
/**
 * request 通用方式获取请求参数
 */
@WebServlet("/req2")
public class RequestDemo2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //GET请求逻辑
        System.out.println("get....");
        //1. 获取所有参数的Map集合
        Map<String, String[]> map = req.getParameterMap();
        for (String key : map.keySet()) {
            // username:zhangsan lisi
            System.out.print(key+":");

            //获取值
            String[] values = map.get(key);
            for (String value : values) {
                System.out.print(value + " ");
            }

            System.out.println();
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    }
}
```

获取的结果为:

```xml
get....
username:fe 
password:fe 
hobby:1 2 
```



 2.2获取GET请求参数中的爱好，结果是数组值

```java
/**
 * request 通用方式获取请求参数
 */
@WebServlet("/req2")
public class RequestDemo2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //GET请求逻辑
        //...
        System.out.println("------------");
        String[] hobbies = req.getParameterValues("hobby");
        for (String hobby : hobbies) {
            System.out.println(hobby);
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    }
}
```

获取的结果为: 1, 2

 2.3获取GET请求参数中的用户名和密码，结果是单个值

```java
/**
 * request 通用方式获取请求参数
 */
@WebServlet("/req2")
public class RequestDemo2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //GET请求逻辑
        //...
        String username = req.getParameter("username");
        String password = req.getParameter("password");
        System.out.println(username);
        System.out.println(password);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    }
}
```

获取的结果为:账户和密码

3.在Servlet代码中获取页面传递POST请求的参数值

 3.1将req.html页面form表单的提交方式改成post

 3.2将doGet方法中的内容复制到doPost方法中即可

**小结**

* req.getParameter()和getParameterMap()方法使用的频率会比较高

### 4.3 解决post请求乱码问题

#### 内容讲解

html页面：

~~~html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="/httpServletRequestDemo04Servlet" method="post">
    <input type="text" name="username"><br>   
    <input type="submit">
</form>
</body>
</html>
~~~



【1】从tomcat8开始以后，对于get请求乱码，tomcat已经解决。对于post请求中文乱码没有解决，需要我们自己处理。

【2】post请求乱码产生的原因和解决思路

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230420175853.png)

说明：

> 1)页面使用的编码表是UTF-8编码，tomcat使用的是默认编码表ISO-8859-1进行解码，编码和解码使用的编码表不一致导致乱码。
>
> 2)解决思路：先按照ISO-8859-1编码，在按照UTF-8进行重新解码



【3】解决方案

解决方案有三种：

> 1.方案一
>
> ~~~java
> 【1】方式一
>           使用URLEncoder类进行编码:static String encode(String s, String enc)
>                           参数：
>                               s:编码的字符串
>                               enc:使用编码表
>           使用URLDecoder进行解码：static String decode(String s, String enc)
>                           参数：
>                               s:解码的字符串
>                               enc:使用编码表
> ~~~
>
> 2.方案二
>
> ~~~java
> 【2】方式二：
>           使用String类中的方法进行编码：    byte[] getBytes(String charsetName)
>                                             参数表示指定的编码表，返回值表示编码后的字节数组
>           使用String类中的构造方法进行解码：String(byte[] bytes, String charsetName)
>                                           参数：
>                                               bytes：字节数组
>                                               charsetName：表示指定的编码表
>                                           返回值：解码后的字符串
> ~~~
>
> 3.方案三
>
> ~~~java
> 【3】方式三：
>           如果是get请求，tomcat8底层已经帮助我们解决完了，我们只需要解决post乱码即可，但是上述
>           两种方式对于post请求可以解决乱码，对于get请求本身获取到的已经是正确的数据，处理
>           后又乱码了。
>           我们的想法是：get请求不用我们自己书写代码处理乱码，只需要我们书写代码处理post乱码。
>           我们接下来学习第三种解决方案：
>           只解决来自于请求体数据的乱码。而get请求体没有数据，post请求体含有数据，所以我们可以理解为第三种处理方案只				是用来解决post乱码的。使用的api是ServletRequest接口中的：
>               void setCharacterEncoding(String env)
>                   参数：指定的编码表
>           注意：该方式的代码必须书写在获取请求数据之前
> ~~~

【4】代码实现

```java
package com.itheima.sh.web;

import com.itheima.sh.pojo.User;
import com.itheima.sh.service.UserServcie;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.net.URLDecoder;
import java.net.URLEncoder;

@WebServlet("/httpServletRequestDemo04Servlet")
public class LoginServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1.获取浏览器的请求数据
//        String username = request.getParameter("username");

        /*
            解决post乱码问题有三种方式：
            【1】方式一
                使用URLEncoder类进行编码:static String encode(String s, String enc)
                                参数：
                                    s:编码的字符串
                                    enc:使用编码表
                使用URLDecoder进行解码：static String decode(String s, String enc)
                                参数：
                                    s:解码的字符串
                                    enc:使用编码表
         */
        //1)编码 ： 使用URLEncoder类进行编码:static String encode(String s, String enc)
//        String encodeUsername = URLEncoder.encode(username, "ISO-8859-1");
//        //2)解码：使用URLDecoder进行解码：static String decode(String s, String enc)
//        username = URLDecoder.decode(encodeUsername, "UTF-8");

        /*
             解决post乱码问题有三种方式：
            【2】方式二：
                使用String类中的方法进行编码：    byte[] getBytes(String charsetName)
                                                  参数表示指定的编码表，返回值表示编码后的字节数组
                使用String类中的构造方法进行解码：String(byte[] bytes, String charsetName)
                                                参数：
                                                    bytes：字节数组
                                                    charsetName：表示指定的编码表
                                                返回值：解码后的字符串

         */
        //1)编码 ： 使用String类中的方法进行编码：    byte[] getBytes(String charsetName)
//        byte[] bytes = username.getBytes("ISO-8859-1");
//        //2)解码：使用String类中的构造方法进行解码：String(byte[] bytes, String charsetName)
//        username = new String(bytes, "UTF-8");

        //username = new String(username.getBytes("ISO-8859-1"), "UTF-8");

        /*
            解决post乱码问题有三种方式：
            【3】方式三：
                如果是get请求，tomcat8底层已经帮助我们解决完了，我们只需要解决post乱码即可，但是上述
                两种方式对于post请求可以解决乱码，对于get请求本身获取到的已经是正确的数据，处理
                后又乱码了。
                我们的想法是：get请求不用我们自己书写代码处理乱码，只需要我们书写代码处理post乱码。
                我们接下来学习第三种解决方案：
                只解决来自于请求体数据的乱码。而get请求体没有数据，post请求体含有数据，所以我们可以理解为第三种处理方案只是用来解决
                post乱码的。使用的api是ServletRequest接口中的：
                    void setCharacterEncoding(String env)
                        参数：指定的编码表
                注意：该方式的代码必须书写在获取请求数据之前
         */
        request.setCharacterEncoding("utf-8");//告知tomcat使用UTF-8解码页面请求数据

        //  1.获取浏览器的请求数据
        String username = request.getParameter("username");
        System.out.println("username = " + username);
    }
}
```

### 4.4 Request请求转发(前后端分离后使用较少，面试考)

1. 请求转发(forward):一种在服**务器内部**的资源**跳转**方式。

(1)浏览器发送请求给服务器，服务器中对应的资源A接收到请求

(2)资源A处理完请求后将请求发给资源B

(3)资源B处理完后将结果响应给浏览器

(4)请求从资源A到资源B的过程就叫请求转发

2. 请求转发的实现方式:

```java
req.getRequestDispatcher("资源B路径").forward(req,resp);
说明：
	1）RequestDispatcher dispatcher = req.getRequestDispatcher("资源B路径");
	    RequestDispatcher表示转发器，该接口中有一个方法：forward(request,response)
```

具体如何来使用，我们先来看下需求:

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230420182635.png)

针对上述需求，具体的实现步骤为:

>1.创建一个RequestDemo5类，接收/req5的请求，在doGet方法中打印`demo5`
>
>2.创建一个RequestDemo6类，接收/req6的请求，在doGet方法中打印`demo6`
>
>3.在RequestDemo5的方法中使用
>
>​	req.getRequestDispatcher("/req6").forward(req,resp)进行请求转发
>
>4.启动测试

(1)创建RequestDemo5类

```java
/**
 * 请求转发
 */
@WebServlet("/req5")
public class RequestDemo5 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("demo5...");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

(2)创建RequestDemo6类

```java
/**
 * 请求转发
 */
@WebServlet("/req6")
public class RequestDemo6 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("demo6...");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

(3)在RequestDemo5的doGet方法中进行请求转发

```java
/**
 * 请求转发
 */
@WebServlet("/req5")
public class RequestDemo5 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("demo5...");
        //请求转发
        request.getRequestDispatcher("/req6").forward(request,response);
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

(4)启动测试

访问`http://localhost:8080/request-demo/req5`,就可以在控制台看到如下内容:

```XML
demo5...
demo6...
```

说明请求已经转发到了`/req6`

3. 请求转发资源间共享数据:使用Request对象

此处主要解决的问题是把请求从`/req5`转发到`/req6`的时候，如何传递数据给`/req6`。

需要使用request对象提供的三个方法:

* 存储数据到request域[范围,数据是存储在request对象]中

```
void setAttribute(String name,Object o);
```

* 根据key获取值

```
Object getAttribute(String name);
```

* 根据key删除该键值对

```
void removeAttribute(String name);
```

接着上个需求来:

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230420182803.png)

> 1.在RequestDemo5的doGet方法中转发请求之前，将数据存入request域对象中
>
> 2.在RequestDemo6的doGet方法从request域对象中获取数据，并将数据打印到控制台
>
> 3.启动访问测试

(1)修改RequestDemo5中的方法

```java
@WebServlet("/req5")
public class RequestDemo5 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("demo5...");
        //存储数据
        request.setAttribute("msg","hello");
        //请求转发
        request.getRequestDispatcher("/req6").forward(request,response);

    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

(2)修改RequestDemo6中的方法

```java
/**
 * 请求转发
 */
@WebServlet("/req6")
public class RequestDemo6 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("demo6...");
        //获取数据
        Object msg = request.getAttribute("msg");
        System.out.println(msg);

    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

(3)启动测试

访问`http://localhost:8080/request-demo/req5`,就可以在控制台看到如下内容:

此时就可以实现在转发多个资源之间共享数据。

4. 请求转发的特点

* 浏览器地址栏路径不发生变化

  虽然后台从`/req5`转发到`/req6`,但是浏览器的地址一直是`/req5`,未发生变化

* 只能转发到当前服务器的内部资源

  不能从一个服务器通过转发访问另一台服务器

* 一次请求，可以在转发资源间使用request共享数据

  虽然后台从`/req5`转发到`/req6`，但是这个==只有一次请求==

* 问题：request.getParameter()

~~~java
request.getParameter(String name)和request.getAttribute(String name);区别
    1.request.getParameter(String name):获取来自于浏览器的数据 <input type="text" name="username" value="锁哥"/>
    request.getParameter("username"); 获取的是锁哥
     
    2.request.getAttribute(String name)获取的是服务器中的代码：request.setAttibute(String name,Object obj);的数据
    
    request.setAttribute("msg","黑马程序员"); 
    String msg = (String) request.getAttribute("msg");
    
~~~



### 4.5request的生命周期

1.何时创建?

浏览器第一次访问tomcat服务器的时候

2.谁创建?

tomcat创建

3.创建对象做什么？

浏览器第一次访问tomcat服务器的时候，tomcat创建request对象和response对象，传递给servlet中的service方法，然后我们可以在servlet中使用request对象调用方法获取请求数据(请求行 头 体)，然后处理业务逻辑，处理完毕，然后tomcat将响应数据给浏览器，浏览器接收到响应之后，tomcat立刻销毁request和response对象。

## 5.HTTP响应详解(理解)

### 1.使用抓包查看响应报文协议内容

注意：

http响应报文协议包括：

~~~java
1.响应行
2.响应头
3.响应体
~~~

响应数据：是服务器响应给浏览器

【1】步骤

~~~markdown
1.创建html页面
2.创建servlet
~~~

【2】实现

1.创建html页面

~~~html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Insert title here</title>
</head>
<body>
<h2>get请求</h2>
<form action="/getServlet" method="get">
    用户名：<input type="text" name="username" value="suoge" /> <br/>
    密码：<input type="text" name="password" value="1234" /> <br/>
    <input type="submit" value="get提交" />
</form>
<h2>post请求</h2>
<form action="/postServlet" method="post">
    用户名：<input type="text" name="username" value="suoge" /> <br/>
    密码：<input type="text" name="password" value="1234" /> <br/>
    <input type="submit" value="post提交" />
</form>
</body>
</html>
~~~

2.创建servlet

~~~java
package com.itheima.sh.a_http_01;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet("/getServlet")
public class GetServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //响应给浏览器数据
        response.getWriter().print("get....");
    }
}

~~~

~~~java
package com.itheima.sh.a_http_01;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet("/postServlet")
public class PostServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //响应给浏览器数据
        response.getWriter().print("post....");
    }
}

~~~

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230420184309.png)

#### 内容小结

​	1.由于浏览器的原因，浏览器会把请求行和响应行信息放在了一起；

​	2.get和post请求的响应没有区别；



### 2.HTTP响应报文协议介绍

#### 学习目标

- 理解响应报文协议的组成部分

#### 内容讲解

【1】响应行

响应行格式：协议/版本  状态码   

>  如：HTTP/1.1 200 ;

| 状态码  | 状态码描述            | 说明                                                         |
| ------- | --------------------- | ------------------------------------------------------------ |
| **200** | OK                    | 请求已成功，请求所希望的响应头或数据体将随此响应返回。出现此状态码是表示正常状态。 |
| **302** | Move temporarily      | 重定向，请求的资源临时从不同的 地址响应请求。                |
| **304** | Not Modified          | 从**浏览器**缓存中读取数据，不从服务器重新获取数据。例如，用户第一次从浏览器访问服务器端图片资源，以后在访问该图片资源的时候就不会再从服务器上加载而直接到浏览器缓存中加载，这样效率更高。 |
| **404** | Not Found             | 请求资源不存在。通常是用户路径编写错误，也可能是服务器资源已删除。 |
| 403     | Forbidden             | 服务器已经理解请求，但是拒绝执行它                           |
| 405     | Method Not Allowed    | 请求行中指定的请求方法不能被用于请求相应的资源               |
| **500** | Internal Server Error | 服务器内部错误。通常程序抛异常                               |

【2】响应头

响应头也是用的键值对key:value，服务器基于**响应头**通知浏览器的行为。

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230420184819.png)

**常见的响应头** ：

|      响应头Key      |                         响应头value                          |
| :-----------------: | :----------------------------------------------------------: |
|      location       |     指定响应的路径，需要与状态码302配合使用，完成重定向      |
|    content-Type     | 响应正文的类型（MIME类型，属于服务器里面的一种类型，例如文件在window系统有自己的类型，.txt  .doc  .jpg。文件在服务器中也有自己的类型），同时还可以解决乱码问题。例如：text/html;charset=UTF-8 |
| content-disposition | 通过浏览器以附件形式解析正文，例如：attachment;filename=xx.zip |
|       refresh       | 页面刷新，例如：3;url=www.itcast.cn    //三秒刷新页面到www.itcast.cn |

常见的MIME类型：就是文件在tomcat服务器中的文件类型：

~~~java
				windows    tomcat(MIME类型)
超文本标记语言文本 .html      text/html ***
xml文档 .xml 				text/xml
XHTML文档 .xhtml 			application/xhtml+xml
普通文本         .txt 	   text/plain ***
PDF文档 .pdf 				application/pdf
Microsoft Word文件 .word 	application/msword
PNG图像 .png  			image/png **
GIF图形 .gif 				image/gif
JPEG图形 .jpeg,.jpg 		image/jpeg **
......
~~~





【3】响应体

​	响应体，就是服务器发送给浏览器的数据。当前浏览器向服务器请求的资源是hello.html，所以服务器给浏览器响应的数据是一个html页面。

请求资源路径：url

响应结果：页面或者资源

如果请求是servlet,那么浏览器的响应体接收到的是servlet响应的数据：

#### 内容小结

1.响应行：

​	协议版本号 状态码 200(一切正常) 404(找不到资源路径) 500(服务器报异常) 302(和location一起使用，实现重定向) 304(从浏览器缓存中读取数据) 405(服务器的servlet没有重写doGet和doPost方法)

2.响应头：

​	location  指定响应的路径

​	**content-type:告诉浏览器文件格式，告诉浏览器不要解析html文件(text/plain)，解决中文乱码问题**

​	refresh 定时刷新

​	content-disposition 以附件形式展示图片等资源

3.响应体：

​	服务器处理的结果响应到浏览器中

## 5.Response对象

### 1 Response对象介绍

前面讲解完Request对象，接下来我们回到刚开始的那张图:

* Request:使用request对象来==获取==请求数据
* Response:使用response对象来==设置==响应数据

Reponse的继承体系和Request的继承体系也非常相似:

HttpServletResponse  response = new ResponseFacade();多态

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230420185543.png)

 介绍完Response的相关体系结构后，接下来对于Response我们需要学习如下内容:

* Response设置响应数据的功能介绍
* Response完成重定向
* Response响应字符数据
* Response响应字节数据

### 2 Response设置响应数据功能介绍

HTTP响应数据总共分为三部分内容，分别是==响应行、响应头、响应体==，对于这三部分内容的数据，respone对象都提供了哪些方法来进行设置?

1. 响应行

```xml
HTTP/1.1 200 OK
```

对于响应行，比较常用的就是设置响应状态码:

```
void setStatus(int sc);
```

2. 响应头

```xml
Content-Type: text/html
```

设置响应头键值对：

```
void setHeader(String name,String value);
响应头：name的值
	location  指定响应的路径
	content-type:告诉浏览器文件格式，告诉浏览器不要解析html文件(text/plain)，解决中文乱码问题
	refresh 定时刷新
	content-disposition 以附件形式展示图片等资源

```

3. 响应体

```xml
返回的数据静态页面也可以
```

对于响应体，是通过字符、字节输出流的方式往浏览器写，

获取字符输出流:

```
PrintWriter getWriter();
```

获取字节输出流

```
ServletOutputStream getOutputStream();
```

介绍完这些方法后，后面我们会通过案例把这些方法都用一用，首先先来完成下重定向的功能开发。

### 3 Respones请求重定向

1. ==Response重定向(redirect):一种资源跳转方式(服务器外部的)。==

(1)浏览器发送请求给服务器，服务器中对应的资源A接收到请求

(2)资源A现在无法处理该请求，就会给浏览器响应一个302的状态码+location的一个访问资源B的路径

(3)浏览器接收到响应状态码为302就会重新发送请求到location对应的访问地址去访问资源B

(4)资源B接收到请求后进行处理并最终给浏览器响应结果，这整个过程就叫==重定向==

2. 重定向的实现方式:

```
resp.setStatus(302);设置响应状态码是302
resp.setHeader("location","资源B的访问路径");
```

具体如何来使用，我们先来看下需求:

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230420190235.png)

针对上述需求，具体的实现步骤为:

> 1.创建一个ResponseDemo1类，接收/resp1的请求，在doGet方法中打印`resp1....`
>
> 2.创建一个ResponseDemo2类，接收/resp2的请求，在doGet方法中打印`resp2....`
>
> 3.在ResponseDemo1的方法中使用
>
> ​	response.setStatus(302);
>
> ​	response.setHeader("Location","/request-demo/resp2") 来给前端响应结果数据
>
> 4.启动测试

(1)创建ResponseDemo1类

```java
@WebServlet("/resp1")
public class ResponseDemo1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("resp1....");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

(2)创建ResponseDemo2类

```java
@WebServlet("/resp2")
public class ResponseDemo2 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("resp2....");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

(3)在ResponseDemo1的doGet方法中给前端响应数据

```java
@WebServlet("/resp1")
public class ResponseDemo1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("resp1....");
        //重定向
        //1.设置响应状态码 302
        response.setStatus(302);
        //2. 设置响应头 Location
        response.setHeader("Location","/request-demo/resp2");
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

(4)启动测试

访问`http://localhost:8080/request-demo/resp1`,就可以在控制台看到如下内容:

```xml
resp1...
resp2...
```

说明`/resp1`和`/resp2`都被访问到了。到这重定向就已经完成了。

虽然功能已经实现，但是从设置重定向的两行代码来看，会发现除了重定向的地址不一样，其他的内容都是一模一样，所以resposne对象给我们提供了简化的编写方式为:

```
resposne.sendRedirect("/request-demo/resp2")
```

所以第3步中的代码就可以简化为：

```java
@WebServlet("/resp1")
public class ResponseDemo1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("resp1....");
        //重定向
        resposne.sendRedirect("/request-demo/resp2")；
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

3. 重定向的特点

* 浏览器地址栏路径发送变化

  当进行重定向访问的时候，由于是由浏览器发送的两次请求，所以地址会发生变化

  ```xml
  /demo/resp1  --->  /demo/resp2
  ```

* 可以重定向到任何位置的资源(服务内容、外部均可)

  因为第一次响应结果中包含了浏览器下次要跳转的路径，所以这个路径是可以任意位置资源。

* 两次请求，不能在多个资源使用request共享数据

  因为浏览器发送了两次请求，是两个不同的request对象，就无法通过request对象进行共享数据

介绍完==请求重定向==和==请求转发==以后，接下来需要把这两个放在一块对比下:

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230420190612.png)

以后到底用哪个，还是需要根据具体的业务来决定。

```markdown
# 如果需要在资源之间传递共享request数据,使用请求转发, 否则就用重定向
```

### 4 路径问题

1. 问题1：转发的时候路径上没有加`/request-demo`而重定向加了，那么到底什么时候需要加，什么时候不需要加呢?

其实判断的依据很简单，只需要记住下面的规则即可:

* 浏览器使用:需要加虚拟目录(项目访问路径)
* 服务端使用:不需要加虚拟目录

对于转发来说，因为是在服务端进行的，所以不需要加虚拟目录

对于重定向来说，路径最终是由浏览器来发送请求，就需要添加虚拟目录。

掌握了这个规则，接下来就通过一些练习来强化下知识的学习:

* `<a href='路劲'>`
* `<form action='路径'>`
* req.getRequestDispatcher("路径")
* resp.sendRedirect("路径")

答案:

```
1.超链接，从浏览器发送，需要加
2.表单，从浏览器发送，需要加
3.转发，是从服务器内部跳转，不需要加
4.重定向，是由浏览器进行跳转，需要加。
```

### 5 Response响应字符数据

要想将字符数据写回到浏览器，我们需要两个步骤:

* 通过Response对象获取字符输出流： PrintWriter writer = resp.getWriter();

* 通过字符输出流写数据: writer.write("aaa");

接下来，我们实现通过些案例把响应字符数据给实际应用下:

1. 返回一个简单的字符串`aaa`

```java
/**
 * 响应字符数据：设置字符数据的响应体
 */
@WebServlet("/resp3")
public class ResponseDemo3 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        //1. 获取字符输出流
        PrintWriter writer = response.getWriter();
		 writer.write("aaa");
    }
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

2. 返回一串html字符串，并且能被浏览器解析

```
PrintWriter writer = response.getWriter();
//content-type，告诉浏览器返回的数据类型是HTML类型数据，这样浏览器才会解析HTML标签
response.setHeader("content-type","text/html");
writer.write("<h1>aaa</h1>");
```

==注意:==一次请求响应结束后，response对象就会被销毁掉，所以不要手动关闭流。

3. 返回一个中文的字符串`你好`，需要注意设置响应数据的编码为`utf-8`

```
//设置响应的数据格式及数据的编码
response.setContentType("text/html;charset=utf-8");
writer.write("你好");
```

### 6 Response响应字节数据

要想将字节数据写回到浏览器，我们需要两个步骤:

- 通过Response对象获取字节输出流：ServletOutputStream outputStream = resp.getOutputStream();

- 通过字节输出流写数据: outputStream.write(字节数据);

接下来，我们实现通过些案例把响应字节数据给实际应用下:

1. 返回一个图片文件到浏览器

```java
/**
 * 响应字节数据：设置字节数据的响应体
 */
@WebServlet("/resp4")
public class ResponseDemo4 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1. 读取文件
        FileInputStream fis = new FileInputStream("d://a.jpg");
        //2. 获取response字节输出流
        ServletOutputStream os = response.getOutputStream();
        //3. 完成流的copy
        byte[] buff = new byte[1024];
        int len = 0;
        while ((len = fis.read(buff))!= -1){
            os.write(buff,0,len);
        }
        fis.close();
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

上述代码中，对于流的copy的代码还是比较复杂的，所以我们可以使用别人提供好的方法来简化代码的开发，具体的步骤是:

(1)pom.xml添加依赖

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.6</version>
</dependency>
```

(2)调用工具类方法

```
//fis:输入流
//os:输出流
IOUtils.copy(fis,os);
```

优化后的代码:

```java
/**
 * 响应字节数据：设置字节数据的响应体
 */
@WebServlet("/resp4")
public class ResponseDemo4 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1. 读取文件
        FileInputStream fis = new FileInputStream("d://a.jpg");
        //2. 获取response字节输出流
        ServletOutputStream os = response.getOutputStream();
        //3. 完成流的copy
      	IOUtils.copy(fis,os);
        fis.close();
    }

    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```
