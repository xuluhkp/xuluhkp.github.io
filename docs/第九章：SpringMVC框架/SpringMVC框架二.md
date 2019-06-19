# 今日内容介绍
<extoc></extoc>

# SpringMVC响应(掌握)

## SpringMVC返回值类型(掌握)

### 控制方法返回值为字符串(掌握)
```java
/**
 * 指定逻辑视图名，经过视图解析器解析为jsp物理路径：/WEB-INF/pages/success.jsp
 */
@RequestMapping("/str")
public String respStr(User user,HttpSession session){
    System.out.println("aaaaaaaaaaaa");
    session.setAttribute("logined",user);
    return "success";
}
```

### 控制方法无返回值(掌握)
Servlet原始API可以作为控制器中方法的参数,在controller方法形参上可以定义request和response，使用request或response指定响应结果：

#### 1. 使用request转向页面
```java
/**
 * 返回值如果是void,则方法名称作为视图名称
 * 可以通过重定向和转发实现页面跳转
 *
 */
@RequestMapping("/void")
public void respStr(User user,  HttpServletRequest request,HttpServletResponse response) throws ServletException, IOException {
    System.out.println("aaaaaaaaaaaa");
    request.setAttribute("logined",user);
    request.getRequestDispatcher("/WEB-INF/pages/success.jsp").forward(request,response);
}
```

#### 2.  也可以通过response页面重定向跳转页面： `response.sendRedirect("testRetrunString") `需要注意的是:重定向不能访问到WEB-INF目录下的资源
```java
/**
 * 返回值如果是void,则方法名称作为视图名称
 * 可以通过重定向和转发实现页面跳转
 */
@RequestMapping("/void")
public void respStr(User user,  HttpServletRequest request,HttpServletResponse response) throws ServletException, IOException {
    //业务操作...
    response.sendRedirect(request.getContextPath()+"/index.jsp");
}
```

#### 3. 也可以通过response指定响应结果，例如响应json数据：
```java
/**
 * 返回值如果是void,则方法名称作为视图名称
 * 可以通过重定向和转发实现页面跳转
 */
@RequestMapping("/void")
public void respStr(User user,  HttpServletRequest request,HttpServletResponse response) throws ServletException, IOException {
    //业务操作...
    response.setCharacterEncoding("utf-8");
    response.setContentType("application/json;charset=utf-8");
    response.getWriter().write("json串");
}
```

### 控制方法返回`ModelAndView`(掌握)

ModelAndView是SpringMVC为我们提供的一个对象，该对象也可以用作控制器方法的返回值。

他其中有二个常用方法:

| 方法名称 | 作用     |
| :------------- | :------------- |
| addObject(String name,Object value)      |   向Model中设置值      |
| setViewName(String viewName)  | 设置需要跳转的视图名称  |

```java
/**
 * @return ModelAndView  可以设置数据与跳转视图
 */
@RequestMapping("/mv")
public ModelAndView mv(User user) throws ServletException, IOException {
    ModelAndView mv = new ModelAndView();
    //将数据保存到Model中
    mv.addObject("logined", user);
    //设置视图名称
    mv.setViewName("success");

    return mv;
}
```

## SpringMVC页面跳转方式(了解)

controller方法在提供了String类型的返回值之后，默认就是请求转发。我们也可以写成：

### 转发跳转页面(了解)
```java
/**
 * 返回值如果是void,则方法名称作为视图名称
 * 可以通过重定向和转发实现页面跳转
 */
@RequestMapping("/forward")
public String forward(User user,HttpSession session) throws ServletException, IOException {

    //将数据保存到session中
    session.setAttribute("logined", user);

    return "forward:/WEB-INF/pages/success.jsp";
}
```

    需要注意的是，如果用了formward：则路径必须写成实际视图url，不能写逻辑视图。

    它相当于“request.getRequestDispatcher("url").forward(request,response)”。

    使用请求转发，既可以转发到jsp，也可以转发到其他的控制器方法。

### 重定向跳转页面(了解)

contrller方法提供了一个String类型返回值之后，它需要在返回值里使用:redirect:

```java
/**
 * 返回值如果是void,则方法名称作为视图名称
 * 可以通过重定向和转发实现页面跳转
 */
@RequestMapping("/redirect")
public String redirect(User user,HttpSession session) throws ServletException, IOException {

    //将数据保存到session中
    session.setAttribute("logined", user);

    return "redirect:/index.jsp";
}
```

它相当于“response.sendRedirect(url)”。需要注意的是，如果是重定向到jsp页面，则jsp页面不能写在WEB-INF目录中，否则无法找到。

## SpringMVC响应JSON格式数据(掌握)

### 添加依赖(掌握)
```xml
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>
    <version>2.7.3</version>
</dependency>
```

### 过滤静态资源(掌握)
```xml
<!--配置资源目录,springMVC框架不拦截-->
<!--js目录下的所有资源不拦截-->
<mvc:resources mapping="/js/**" location="/js/**"></mvc:resources>
<!--imgs目录下的所有资源不拦截
<mvc:resources mapping="/imgs/**" location="/imgs/**"></mvc:resources>
-->
<!--css目录下的所有资源不拦截
<mvc:resources mapping="/css/**" location="/css/**"></mvc:resources>
-->
```

### 发送异步请求(掌握)
```html
<h1>用户登录功能---异步ajax请求,响应json格式数据</h1>
<input id="btn" type="button" value="点击异步获取数据">

<script src="js/jquery-3.3.1.min.js"></script>
<script>
$(function(){
    $("#btn").click(function () {

        //发送异步请求
        var url = "${pageContext.request.contextPath}/response/json";
        var params = "{username:'zs',password:'123456'}";
        $.ajax({
            type:"POST",
            url: url,
            data: params,
            contentType:"application/json;charset=utf-8",
            dataType:"json",
            success: function(data){
              alert(data)
            }
        });

    })
})
</script>
```

### 响应JSON格式数据(掌握)
```java
/**
 * @RequestBody注解 可以将发送到服务器的json格式数据封装到参数中
 * @ResponseBody注解 可以将响应的对象转化为json格式数据响应给客户端
 * @param user
 * @return
 */
@RequestMapping("/json")
public @ResponseBody User json(@RequestBody User user){
    System.out.println(user);

    user.setUsername("张三");
    user.setPassword("456789");

    return user;
}
```

    @RequestBody注解 可以将发送到服务器的json格式数据封装到参数中
    @ResponseBody注解 可以将方法返回的对象转化为json格式数据响应给客户端

# SpringMVC文件上传(掌握)
## 文件上传的必要前提(掌握)
    1. form表单的enctype取值必须是：multipart/form-data (默认值是:application/x-www-form-urlencoded) enctype:是表单请求正文的类型
    2. method属性取值必须是Post
    3. 提供一个文件选择域<input type=”file” />

## 文件上传的原理分析(掌握)
### 没有设置enctype属性(掌握)
```
POST /web06/jsp/upload.jsp HTTP/1.1
Accept: text/html, application/xhtml+xml, */*
X-HttpWatch-RID: 22006-10011
Referer: http://localhost:8080/web06/jsp/upload.jsp
Accept-Language: zh-CN
User-Agent: Mozilla/5.0 (Windows NT 6.3; WOW64; Trident/7.0; rv:11.0) like Gecko
Content-Type: application/x-www-form-urlencoded
Accept-Encoding: gzip, deflate
Host: localhost:8080
Content-Length: 53
DNT: 1
Connection: Keep-Alive
Cache-Control: no-cache
Cookie: JSESSIONID=D51DCB996556C94861B2C72C4D978010

info=info&upload=C%3A%5CUsers%5Cjt%5CDesktop%5Caa.txt

```

### 设置了enctype属性(掌握)
```
POST /web06/jsp/upload.jsp HTTP/1.1
Accept: text/html, application/xhtml+xml, */*
X-HttpWatch-RID: 22006-10026
Referer: http://localhost:8080/web06/jsp/upload.jsp
Accept-Language: zh-CN
User-Agent: Mozilla/5.0 (Windows NT 6.3; WOW64; Trident/7.0; rv:11.0) like Gecko
Content-Type: multipart/form-data; boundary=---------------------------7e139d10110a64
Accept-Encoding: gzip, deflate
Host: localhost:8080
Content-Length: 322
DNT: 1
Connection: Keep-Alive
Cache-Control: no-cache
Cookie: JSESSIONID=D51DCB996556C94861B2C72C4D978010

-----------------------------7e139d10110a64
Content-Disposition: form-data; name="info"

aaa
-----------------------------7e139d10110a64
Content-Disposition: form-data; name="upload"; filename="C:\Users\jt\Desktop\aa.txt"
Content-Type: text/plain

hello world！！！
-----------------------------7e139d10110a64--
```

### 原理分析(掌握)

![](assets/markdown-img-paste-20180919190025650.png)

## 文件上传环境搭建(掌握)
### 创建maven项目(掌握)
![](assets/markdown-img-paste-20180919190610804.png)

### 在`pom.xml`中导入相关依赖(掌握)
```xml
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <maven.compiler.source>1.8</maven.compiler.source>
  <maven.compiler.target>1.8</maven.compiler.target>
  <!--spring版本锁定-->
  <spring.version>5.0.2.RELEASE</spring.version>
</properties>

<dependencies>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>${spring.version}</version>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>${spring.version}</version>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>${spring.version}</version>
  </dependency>
  <dependency>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>tomcat-api</artifactId>
    <version>7.0.47</version>
  </dependency>
  <dependency>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>tomcat-jasper-el</artifactId>
    <version>7.0.47</version>
  </dependency>
  <dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.1</version>
  </dependency>
</dependencies>
```

### 在`web.xml`中配置SpringMVC核心控制器(掌握)
```xml

<web-app>

  <!--配置SpringMVC核心控制器-->
  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

    <!-- 配置Servlet的初始化参数，读取springmvc的配置文件，创建spring容器 -->
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc.xml</param-value>
    </init-param>

    <!-- 配置servlet启动时加载对象 -->
    <load-on-startup>1</load-on-startup>
  </servlet>

  <!--配置springmvc过请求过滤器，/表示所有请求都经过springmvc处理-->
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>

</web-app>
```

### 配置SpringMVC核心配置文件`springmvc.xml`(掌握)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 配置创建spring容器要扫描的包 -->
    <context:component-scan base-package="com.itheima"></context:component-scan>

    <!--SpringMVC注解驱动-->
    <mvc:annotation-driven></mvc:annotation-driven>

    <!-- 配置视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/pages/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>

</beans>
```

## 传统方式文件上传(熟悉)
### 编写文件上传页面`upload.jsp`(熟悉)
```html
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>文件上传表单</title>
</head>
<body>
    <form action="${pageContext.request.contextPath }/UploadServlet" method="post" enctype="multipart/form-data" >
        文件描述：<input type="text" name="fileDescribe"><br>
        文件上传：<input type="file" name="file"><br>
        <input type="submit" value="开始上传">
    </form>
</body>
</html>
```

### 编写文件上传的控制层方法`FileController`(熟悉)
```java
@Controller
@RequestMapping("/file")
public class FileController {

  /**
   * 传统方式文件上传
   * @param request  请求对象 包含文件上传信息
   * @param response
   * @throws Exception
   */
  @RequestMapping(path="/upload",method = RequestMethod.POST)
  public void upload(HttpServletRequest request, HttpServletResponse response) throws Exception {
      //1. 创建磁盘文件工厂
      DiskFileItemFactory factory = new DiskFileItemFactory();
      //2. 创建核心的解析类
      ServletFileUpload upload = new ServletFileUpload(factory);

      //3. 解析请求对象,获取到各个部分
      List<FileItem> fileItems = upload.parseRequest(request);

      //4. 获取一个文件输出的磁盘路径
      String path = "C:\\Users\\Administrator\\Desktop\\upload";
      File dir = new File(path);
      if(!dir.exists()){
          dir.mkdir();
      }

      //5. 遍历表单的各项
      for (FileItem fileItem : fileItems) {

          //6. 判断是普通项还是文件上传项
          if (fileItem.isFormField()) { // 普通项
              String name = fileItem.getFieldName(); //获取普通项name属性的值
              String value = fileItem.getString();   //获取普通项value属性的值
              System.out.println(name + " " + value);
          } else { //文件上传项
              String name = fileItem.getName();//获取文件名称  a.txt
              InputStream is = fileItem.getInputStream();//获取文件内容

              //创建输出流
              OutputStream os = new FileOutputStream(path + "/" + name);

              // 将文件内容通过输出流写到磁盘文件中
              int i = 0;
              byte[] bys = new byte[1024];

              while ((i = is.read()) != -1) {
                  os.write(bys, 0, i);
              }
              os.close();
              is.close();
          }
      }
  }
}
```

### 总结(熟悉)
    1.	创建磁盘文件工厂DiskFileItemFactory factory = new DiskFileItemFactory();

    2.	创建核心的解析类 ServletFileUpload upload = new ServletFileUpload(factory);

    3.	解析文件上传的请求,获取文件上传的各个部分 List<FileItem> fileItems = upload.parseRequest(request);

    4.	遍历文件上传的各个部分,判断是一个文件上传项还是一个普通的表单项
        如果是一个文件上传项,获取文件名称和文件内容,创建一个输出流将文件写到磁盘上
        如果是一个普通的表单项,获取name和value就可以了

## SpringMVC实现文件上传(掌握)

![](assets/markdown-img-paste-20180919192759402.png)

### 在`springmvc.xml`中配置文件解析器(掌握)
```xml
<!--
    配置文件上传解析器
    id的值是固定的
    class的值也是固定的
 -->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <!-- 设置上传文件的最大尺寸为5MB -->
    <property name="maxUploadSize">
        <value>5242880</value>
    </property>
</bean>
```
### 编写文件上传页面`upload.jsp`(掌握)
```html
<h1>SpringMVC方式文件上传</h1>
<form action="${pageContext.request.contextPath }/file/upload2" method="post" enctype="multipart/form-data" >
    文件描述：<input type="text" name="desc"><br>
    文件上传：<input type="file" name="file"><br>
    <input type="submit" value="开始上传">
</form>
```

### 编写控制层代码`FileController`(掌握)
```java
@RequestMapping(path="/upload2",method = RequestMethod.POST)
public String upload2(MultipartFile file) throws Exception {
    //1. 获取一个文件输出的磁盘路径
    String path = "C:\\Users\\Administrator\\Desktop\\upload";
    File dir = new File(path);
    if(!dir.exists()){
        dir.mkdir();
    }

    //2. 获取文件名称
    String filename = file.getOriginalFilename();

    //3. 获取文件内容
    InputStream is = file.getInputStream();//获取文件内容

    //4. 创建输出流
    OutputStream os = new FileOutputStream(path + "/" + filename);

    //5. 将文件内容通过输出流写到磁盘文件中
    int i = 0;
    byte[] bys = new byte[1024];

    while ((i = is.read()) != -1) {
        os.write(bys, 0, i);
    }
    os.close();

    return "success";
}
```

## SpringMVC实现多文件上传(掌握)
### 编写文件上传页面`upload.jsp`(掌握)
```html
<h1>SpringMVC方式多选文件上传</h1>
<form action="${pageContext.request.contextPath }/file/multi" method="post" enctype="multipart/form-data" >
    文件描述：<input type="text" name="desc"><br>
    文件上传：<input type="file" name="files"><br>
    文件上传：<input type="file" name="files"><br>
    <input type="submit" value="开始上传">
</form>
```

### 编写服务器web层代码`FileController`(掌握)
```java
@RequestMapping(path="/multi",method = RequestMethod.POST)
public String upload3(MultipartFile[] files) throws Exception {

    //1. 获取一个文件输出的磁盘路径
    String path = "C:\\Users\\Administrator\\Desktop\\upload";
    File dir = new File(path);
    if(!dir.exists()){
        dir.mkdir();
    }

    //遍历文件集合
    for (MultipartFile file : files) {
        //2. 获取文件名称
        String filename = file.getOriginalFilename();
        //3. 获取文件内容
        InputStream is = file.getInputStream();//获取文件内容

        //4. 创建输出流
        OutputStream os = new FileOutputStream(path + "/" + filename);

        //5. 将文件内容通过输出流写到磁盘文件中
        int i = 0;
        byte[] bys = new byte[1024];

        while ((i = is.read()) != -1) {
            os.write(bys, 0, i);
        }
        os.close();
    }

    return "success";
}
```

## 跨服务器的文件上传(了解)
![](assets/markdown-img-paste-20180919195728856.png)

### 创建一个文件服务器(了解)
![](assets/markdown-img-paste-20180919200318580.png)
![](assets/markdown-img-paste-20180919200401694.png)

#### 修改默认的Servlet的相关配置`web.xml`
```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <servlet>
    <servlet-name>default</servlet-name>
    <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
    <init-param>
      <param-name>debug</param-name>
      <param-value>0</param-value>
    </init-param>
    <init-param>
      <param-name>readonly</param-name>
      <param-value>false</param-value>
    </init-param>
    <init-param>
      <param-name>listings</param-name>
      <param-value>false</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>default</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```

加入此行的含义是：接收文件的目标服务器可以支持写入操作。

### 编写文件上传代码(了解)
#### 导入依赖jar包
```xml
<dependency>
  <groupId>com.sun.jersey</groupId>
  <artifactId>jersey-client</artifactId>
  <version>1.19</version>
</dependency>
```

```java
@RequestMapping(path="/cross",method = RequestMethod.POST)
public String cross(MultipartFile file) throws Exception {
    //1. 定义需要上传的文件服务器路径
    String url = "http://localhost:8090/upload/";
    //1. 获取文件名称
    String filename = file.getOriginalFilename();
    //2. 获取文件内容
    InputStream is = file.getInputStream();//获取文件内容

    //3. 创建客户端
    Client client = Client.create();
    //4. 与文件服务器建立连接
    WebResource resource = client.resource(url + filename);
    //5. 将文件保存到文件服务器上
    resource.put(file.getBytes());

    return "success";
}
```

# SpringMVC异常处理(熟悉)

    系统中异常包括两类：预期异常和运行时异常RuntimeException，前者通过捕获异常从而获取异常信息，后者主要通过规范代码开发、测试通过手段减少运行时异常的发生。

    系统的dao、service、controller出现都通过throws Exception向上抛出，最后由springmvc前端控制器交由异常处理器进行异常处理，如下图：

![](assets/markdown-img-paste-20180919201743826.png)

## 环境搭建(熟悉)
### 创建maven项目(熟悉)
![](assets/markdown-img-paste-2018091920525492.png)

### 在`pom.xml`中导入依赖(熟悉)
```xml
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <maven.compiler.source>1.8</maven.compiler.source>
  <maven.compiler.target>1.8</maven.compiler.target>
  <!--spring版本锁定-->
  <spring.version>5.0.2.RELEASE</spring.version>
</properties>

<dependencies>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>${spring.version}</version>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>${spring.version}</version>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>${spring.version}</version>
  </dependency>
  <dependency>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>tomcat-api</artifactId>
    <version>7.0.47</version>
  </dependency>
  <dependency>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>tomcat-jasper-el</artifactId>
    <version>7.0.47</version>
  </dependency>
</dependencies>
</project>
```

### 在`web.xml`中编写核心控制器(熟悉)
```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <!--配置SpringMVC核心控制器-->
  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

    <!-- 配置Servlet的初始化参数，读取springmvc的配置文件，创建spring容器 -->
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc.xml</param-value>
    </init-param>

    <!-- 配置servlet启动时加载对象 -->
    <load-on-startup>1</load-on-startup>
  </servlet>

  <!--配置springmvc过请求过滤器，/表示所有请求都经过springmvc处理-->
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>

```

### 编写SpringMVC核心配置文件`springmvc.xml`(熟悉)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 配置创建spring容器要扫描的包 -->
    <context:component-scan base-package="com.itheima"></context:component-scan>

    <!--SpringMVC注解驱动-->
    <mvc:annotation-driven></mvc:annotation-driven>

    <!-- 配置视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/pages/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>


</beans>
```

### 编写前端页面`index.jsp`(熟悉)
```html
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<html>
<head>
    <title>SpringMVC异常处理</title>
</head>
<body>
<h2>SpringMVC异常处理</h2>
<a href="${pageContext.request.contextPath}/exception/test">异常处理</a>
</body>
</html>
```

### 编写web层代码`ExceptionController`(熟悉)
```java
package com.itheima.web;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/exception")
public class ExceptionController {

    @RequestMapping("/test")
    public String testException() throws Exception{
        System.out.println("控制器执行成功...");
        int i = 10/0;
        return "success";
    }
}
```

## 处理异常(熟悉)

### 编写自定义异常`CustomException`(熟悉)
```java
package com.itheima.exception;

public class CustomException extends Exception {

    private String message;

    public CustomException(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }
}
```

### 自定义异常处理器`CustomExceptionResolver`(熟悉)
```java

public class CustomExceptionResolver implements HandlerExceptionResolver {

    @Override
    public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception ex) {
        ex.printStackTrace();
        CustomException customException = null;
        //如果抛出的是系统自定义异常则直接转换
        if (ex instanceof CustomException) {
            customException = (CustomException) ex;
        } else {
            //如果抛出的不是系统自定义异常则重新构造一个系统错误异常。
            customException = new CustomException("系统错误，请与系统管理 员联系！");
        }

        //创建ModelAndView对象
        ModelAndView modelAndView = new ModelAndView();
        //保存错误信息
        modelAndView.addObject("message", customException.getMessage());
        //设置跳转的视图名称
        modelAndView.setViewName("error");

        return modelAndView;
    }
}
```

### 编写错误页面`error.jsp`(熟悉)
```html
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<html>
<head>
    <title>执行失败</title>
</head>
<body>
 执行失败！ ${message }
</body>
</html>
```

### 在`springmvc.xml`中配置异常处理器(熟悉)
```xml
<!-- 配置自定义异常处理器 -->
<bean id="handlerExceptionResolver" class="com.itheima.exception.CustomExceptionResolver"/>
```


# SpringMVC拦截器(熟悉)
```
1. SpringMVC框架中的拦截器用于对处理器进行预处理和后处理的技术。
2. 可以定义拦截器链，连接器链就是将拦截器按着一定的顺序结成一条链，在访问被拦截的方法时，拦截器链中的拦截器会按着定义的顺序执行。
3. 拦截器和过滤器的功能比较类似，有区别
    1. 过滤器是Servlet规范的一部分，任何框架都可以使用过滤器技术。
    2. 拦截器是SpringMVC框架独有的。
    3. 过滤器配置了/*，可以拦截任何资源。
    4. 拦截器只会对控制器中的方法进行拦截。
4. 拦截器也是AOP思想的一种实现方式
5. 想要自定义拦截器，需要实现HandlerInterceptor接口。
```

## 环境搭建(熟悉)
### 创建maven项目(熟悉)
![](assets/markdown-img-paste-2018091920525492.png)

### 在`pom.xml`中导入依赖(熟悉)
```xml
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <maven.compiler.source>1.8</maven.compiler.source>
  <maven.compiler.target>1.8</maven.compiler.target>
  <!--spring版本锁定-->
  <spring.version>5.0.2.RELEASE</spring.version>
</properties>

<dependencies>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>${spring.version}</version>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>${spring.version}</version>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>${spring.version}</version>
  </dependency>
  <dependency>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>tomcat-api</artifactId>
    <version>7.0.47</version>
  </dependency>
  <dependency>
    <groupId>org.apache.tomcat</groupId>
    <artifactId>tomcat-jasper-el</artifactId>
    <version>7.0.47</version>
  </dependency>
</dependencies>
</project>
```

### 在`web.xml`中编写核心控制器(熟悉)
```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <!--配置SpringMVC核心控制器-->
  <servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

    <!-- 配置Servlet的初始化参数，读取springmvc的配置文件，创建spring容器 -->
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc.xml</param-value>
    </init-param>

    <!-- 配置servlet启动时加载对象 -->
    <load-on-startup>1</load-on-startup>
  </servlet>

  <!--配置springmvc过请求过滤器，/表示所有请求都经过springmvc处理-->
  <servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>

```

### 编写SpringMVC核心配置文件`springmvc.xml`(熟悉)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 配置创建spring容器要扫描的包 -->
    <context:component-scan base-package="com.itheima"></context:component-scan>

    <!--SpringMVC注解驱动-->
    <mvc:annotation-driven></mvc:annotation-driven>

    <!-- 配置视图解析器 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/pages/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
</beans>
```

### 编写页面代码`index.jsp`(熟悉)
```html
<%@ page contentType="text/html;charset=UTF-8" language="java" isELIgnored="false" %>
<html>
<title>SpringMVC拦截器</title>
</head>
<body>
<h1>SpringMVC拦截器</h1>
<a href="${pageContext.request.contextPath}/user/add">添加用户</a>
</body>
</html>
```

### 编写web层代码`UserController`(熟悉)
```java
package com.itheima.web;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/add")
    public String add(){
        System.out.println("控制层add方法执行了.....");
        return "success";
    }
}
```

## 自定义拦截器(熟悉)
### 编写一个普通类实现HandlerInterceptor接口`MyInterceptor1`(熟悉)
```java
package com.itheima.interceptor;

import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class MyInterceptor1 implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle拦截器执行了.....");
        return true;
    }
}
```

### 在`springmvc.xml`中配置拦截器(掌握)
```xml
<!-- 配置拦截器 -->
<mvc:interceptors>
    <mvc:interceptor>
        <!--
            配置拦截器拦截的方法
            /** 代表拦截所有方法
            /user/* 代表拦截/user路径下的所有方法
        -->
        <mvc:mapping path="/user/*"/>
        <!--配置不拦截的方法
        <mvc:exclude-mapping path="" />
        -->
        <bean id="myInterceptor1" class="com.itheima.interceptor.MyInterceptor1"></bean>
    </mvc:interceptor>
</mvc:interceptors>
```

## 拦截器的细节(熟悉)
### 拦截器的放行(熟悉)
放行的含义是指，如果有下一个拦截器就执行下一个，如果该拦截器处于拦截器链的最后一个，则执行控制器中的方法。
![](assets/markdown-img-paste-20180919235121120.png)

![](assets/markdown-img-paste-20180920000014509.png)

### 拦截器中方法的说明(熟悉)
    1. `preHandle()`是controller方法执行前拦截的方法
        1. 可以使用request或者response跳转到指定的页面
        2. return true放行，执行下一个拦截器，如果没有拦截器，执行controller中的方法。
        3. return false不放行，不会执行controller中的方法。

    2. `postHandle()`是controller方法执行后执行的方法，在JSP视图执行前。
        1. 可以使用request或者response跳转到指定的页面
        2. 如果指定了跳转的页面，那么controller方法跳转的页面将不会显示。

    3. `afterCompletion()`方法是在JSP执行后执行
        1. request或者response不能再跳转页面了

### 拦截器的作用路径(熟悉)
```xml
<mvc:interceptor>
    <!--
        配置拦截器拦截的方法
        /** 代表拦截所有方法
        /user/* 代表拦截/user路径下的所有方法
    -->
    <mvc:mapping path="/user/*"/>
    <!--配置不拦截的方法
    <mvc:exclude-mapping path="" />
    -->
    <bean id="myInterceptor1" class="com.itheima.interceptor.MyInterceptor1"></bean>
</mvc:interceptor>
```

### 多个拦截器配置(熟悉)
```xml
<!-- 配置拦截器 -->
<mvc:interceptors>
   <mvc:interceptor>

       <mvc:mapping path="/user/*"/>

       <bean id="myInterceptor1" class="com.itheima.interceptor.MyInterceptor1"></bean>
   </mvc:interceptor>

   <mvc:interceptor>
       <mvc:mapping path="/user/*"/>
       <bean id="myInterceptor2" class="com.itheima.interceptor.MyInterceptor2"></bean>
   </mvc:interceptor>
</mvc:interceptors>
```
