# 今日内容介绍
<extoc></extoc>

# 会话技术概述(理解)

## 什么是会话：一次会话中包含多次请求和响应。(理解)
    一次会话：浏览器第一次给服务器资源发送请求，会话建立，直到有一方断开为止
## 会话的作用：在一次会话的范围内的多次请求间，共享数据(理解)

每个用户与服务器进行交互过程中，产生一些各自的数据，程序想要把这些数据进行保存，就需要使用会话技术。

例如：用户点击超链接购买一个商品，程序应该保存用户所购买的商品到购物车，以便于用户点击结账可以得到用户所购买的商品信息。

![](assets/markdown-img-paste-20180728131055317.png)

![](assets/markdown-img-paste-2018072813112554.png)

## 会话的实现方式(理解)
		1. 客户端会话技术：Cookie
		2. 服务器端会话技术：Session


# 会话技术-Cookie(掌握)
## 快速入门(掌握)
### 使用步骤(掌握)
```
1. 创建Cookie对象，绑定数据
  * new Cookie(String name, String value)
2. 发送Cookie对象
  * response.addCookie(Cookie cookie)
3. 获取Cookie，拿到数据
  * Cookie[]  request.getCookies()  
```
### 代码实现(掌握)
#### 向浏览器写Cookie
```java
@WebServlet("/cookieDemo1")
public class CookieDemo1 extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1.创建Cookie对象
        Cookie c = new Cookie("msg","hello");
        //2.发送Cookie
        response.addCookie(c);
    }
}
```

#### 获取浏览器发送到服务器的Cookie
```java
@WebServlet("/cookieDemo2")
public class CookieDemo2 extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
       //3. 获取Cookie
        Cookie[] cs = request.getCookies();
        //获取数据，遍历Cookies
        if(cs != null){
            for (Cookie c : cs) {
                String name = c.getName();
                String value = c.getValue();
                System.out.println(name+":"+value);
            }
        }
    }
}
```


## Cookie实现原理及细节(理解)
### Cookie的实现原理(理解)
基于响应头set-cookie和请求头cookie实现

![](assets/markdown-img-paste-20180728131204124.png)

### cookie的细节(熟悉)
#### 一次可不可以发送多个cookie?
```
一次响应可以发送多个Cookie, 可以创建多个Cookie对象，使用response调用多次addCookie方法发送cookie即可。
```		 

#### 2. cookie在浏览器中保存多长时间？
```
1. 默认情况下，当浏览器关闭后，Cookie数据被销毁
2. 持久化存储：
  * setMaxAge(int seconds)
    1. 正数：将Cookie数据写到硬盘的文件中。持久化存储。并指定cookie存活时间，时间到后，cookie文件自动失效
    2. 负数：默认值
    3. 零：删除cookie信息
```

#### cookie能不能存中文？
```
* 在tomcat 8 之前 cookie中不能直接存储中文数据。
  * 需要将中文数据转码---一般采用URL编码(%E3)
* 在tomcat 8 之后，cookie支持中文数据。特殊字符还是不支持，建议使用URL编码存储，URL解码解析
```


#### cookie共享问题？
```
1. 假设在一个tomcat服务器中，部署了多个web项目，那么在这些web项目中cookie能不能共享？
  * 默认情况下cookie不能共享
  * setPath(String path):设置cookie的获取范围。默认情况下，设置当前的虚拟目录
    * 如果要共享，则可以将path设置为"/"

2. 不同的tomcat服务器间cookie共享问题？
  * setDomain(String path):如果设置一级域名相同，那么多个服务器之间cookie可以共享
    * setDomain(".baidu.com"),那么tieba.baidu.com和news.baidu.com中cookie可以共享
```

### Cookie的特点和作用(了解)
#### 特点
		1. cookie存储数据在客户端浏览器
		2. 浏览器对于单个cookie 的大小有限制(4kb) 以及 对同一个域名下的总cookie数量也有限制(20个)

#### 作用
    1. cookie一般用于存出少量的不太敏感的数据
    2. 在不登录的情况下，完成服务器对客户端的身份识别

## 案例：记住上一次访问时间(掌握)
### 需求(掌握)
			1. 访问一个Servlet，如果是第一次访问，则提示：您好，欢迎您首次访问。
			2. 如果不是第一次访问，则提示：欢迎回来，您上次访问时间为:显示时间字符串

### 分析(掌握)
			1. 可以采用Cookie来完成
			2. 在服务器中的Servlet判断是否有一个名为lastTime的cookie
				1. 有：不是第一次访问
					1. 响应数据：欢迎回来，您上次访问时间为:2018年6月10日11:50:20
					2. 写回Cookie：lastTime=2018年6月10日11:50:01
				2. 没有：是第一次访问
					1. 响应数据：您好，欢迎您首次访问
					2. 写回Cookie：lastTime=2018年6月10日11:50:01

### 代码实现(掌握)
```java
@WebServlet("/cookieTest")
public class CookieTest extends HttpServlet {
  protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
      //设置响应的消息体的数据格式以及编码
      response.setContentType("text/html;charset=utf-8");

      //1.获取所有Cookie
      Cookie[] cookies = request.getCookies();
      boolean flag = false;//没有cookie为lastTime
      //2.遍历cookie数组
      if(cookies != null && cookies.length > 0){
          for (Cookie cookie : cookies) {
              //3.获取cookie的名称
              String name = cookie.getName();
              //4.判断名称是否是：lastTime
              if("lastTime".equals(name)){
                  //有该Cookie，不是第一次访问

                  flag = true;//有lastTime的cookie

                  //设置Cookie的value
                  //获取当前时间的字符串，重新设置Cookie的值，重新发送cookie
                  Date date  = new Date();
                  SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss");
                  String str_date = sdf.format(date);
                  System.out.println("编码前："+str_date);
                  //URL编码
                  str_date = URLEncoder.encode(str_date,"utf-8");
                  System.out.println("编码后："+str_date);
                  cookie.setValue(str_date);
                  //设置cookie的存活时间
                  cookie.setMaxAge(60 * 60 * 24 * 30);//一个月
                  response.addCookie(cookie);


                  //响应数据
                  //获取Cookie的value，时间
                  String value = cookie.getValue();
                  System.out.println("解码前："+value);
                  //URL解码：
                  value = URLDecoder.decode(value,"utf-8");
                  System.out.println("解码后："+value);
                  response.getWriter().write("<h1>欢迎回来，您上次访问时间为:"+value+"</h1>");

                  break;

              }
          }
      }


      if(cookies == null || cookies.length == 0 || flag == false){
          //没有，第一次访问

          //设置Cookie的value
          //获取当前时间的字符串，重新设置Cookie的值，重新发送cookie
          Date date  = new Date();
          SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss");
          String str_date = sdf.format(date);
          System.out.println("编码前："+str_date);
          //URL编码
          str_date = URLEncoder.encode(str_date,"utf-8");
          System.out.println("编码后："+str_date);

          Cookie cookie = new Cookie("lastTime",str_date);
          //设置cookie的存活时间
          cookie.setMaxAge(60 * 60 * 24 * 30);//一个月
          response.addCookie(cookie);

          response.getWriter().write("<h1>您好，欢迎您首次访问</h1>");
      }


  }

  protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
      this.doPost(request, response);
  }
}
```			


# JSP入门(熟悉)

```
JSP: Java Server Pages： java服务器端页面
  * 可以理解为：一个特殊的页面，其中既可以指定定义html标签，又可以定义java代码
  * 用于简化书写！！！
```

## JSP原理(理解)

![](assets/03/20180506-d97e015c.png)  

JSP本质上就是一个Servlet
JSP = HTML代码+JAVA代码+JSP指令和动作标签

## JSP脚本(熟悉)
```
1. <%  代码 %>：定义的java代码，在service方法中。service方法中可以定义什么，该脚本中就可以定义什么。
2. <%! 代码 %>：定义的java代码，在jsp转换后的java类的成员位置。
3. <%= 代码 %>：定义的java代码，会输出到页面上。输出语句中可以定义什么，该脚本中就可以定义什么。
```

## JSP内置对象(熟悉)
```
什么是内置对象:在jsp页面中不需要获取和创建，可以直接使用的对象

jsp一共有9个内置对象,今天学习3个：
* request
* response
* out：字符输出流对象。可以将数据输出到页面上。和response.getWriter()类似
  * response.getWriter()和out.write()的区别：
    * 在tomcat服务器真正给客户端做出响应之前，会先找response缓冲区数据，再找out缓冲区数据。
    * response.getWriter()数据输出永远在out.write()之前
```

## 案例:改造记录用户上次访问时间的案例(了解)
```java
<%@ page import="java.util.Date" %>
<%@ page import="java.text.SimpleDateFormat" %>
<%@ page import="java.net.URLEncoder" %>
<%@ page import="java.net.URLDecoder" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>itcast</title>
</head>
<body>

<%
    //1.获取所有Cookie
    Cookie[] cookies = request.getCookies();
    boolean flag = false;//没有cookie为lastTime
    //2.遍历cookie数组
    if(cookies != null && cookies.length > 0){
        for (Cookie cookie : cookies) {
            //3.获取cookie的名称
            String name = cookie.getName();
            //4.判断名称是否是：lastTime
            if("lastTime".equals(name)){
                //有该Cookie，不是第一次访问

                flag = true;//有lastTime的cookie

                //设置Cookie的value
                //获取当前时间的字符串，重新设置Cookie的值，重新发送cookie
                Date date  = new Date();
                SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss");
                String str_date = sdf.format(date);
                System.out.println("编码前："+str_date);
                //URL编码
                str_date = URLEncoder.encode(str_date,"utf-8");
                System.out.println("编码后："+str_date);
                cookie.setValue(str_date);
                //设置cookie的存活时间
                cookie.setMaxAge(60 * 60 * 24 * 30);//一个月
                response.addCookie(cookie);


                //响应数据
                //获取Cookie的value，时间
                String value = cookie.getValue();
                System.out.println("解码前："+value);
                //URL解码：
                value = URLDecoder.decode(value,"utf-8");
                System.out.println("解码后："+value);
%>
            <h1>欢迎回来，您上次访问时间为:<%=value%></h1>
<%
                break;

            }
        }
    }

    if(cookies == null || cookies.length == 0 || flag == false){
        //没有，第一次访问

        //设置Cookie的value
        //获取当前时间的字符串，重新设置Cookie的值，重新发送cookie
        Date date  = new Date();
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy年MM月dd日 HH:mm:ss");
        String str_date = sdf.format(date);
        System.out.println("编码前："+str_date);
        //URL编码
        str_date = URLEncoder.encode(str_date,"utf-8");
        System.out.println("编码后："+str_date);

        Cookie cookie = new Cookie("lastTime",str_date);
        //设置cookie的存活时间
        cookie.setMaxAge(60 * 60 * 24 * 30);//一个月
        response.addCookie(cookie);

%>
        <h1>您好，欢迎您首次访问</h1>
<%
    }
%>
</body>
</html>
```

# 会话技术-Session(掌握)
## 快速入门(掌握)
### 获取HttpSession对象(掌握)
```java
@WebServlet("/sessionDemo1")
public class SessionDemo1 extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //使用session共享数据

        //1.获取session
        HttpSession session = request.getSession();
        //2.存储数据
        session.setAttribute("msg","hello session");
    }
}
```
### 使用HttpSession对象(掌握)
| 返回值 | 方法名称    |  方法介绍  |
| :------------- | :------------- |:------------- |
| void           | **setAttribute(String name,Object obj)** |  存储数据   |
| Object         | **getAttribute(String name)** | 通过键获取值 |
| void           | removeAttribute(String name) | 通过键删除数据  |

## Session的实现原理及细节(理解)

### Session的实现原理(理解)
![](assets/markdown-img-paste-20180730231533228.png)

Session的实现是依赖于Cookie的。

### Session的使用细节(理解)
```
1. 当客户端关闭后，服务器不关闭，两次获取session是否为同一个？
  * 默认情况下。不是。
  * 如果需要相同，则可以创建Cookie,键为JSESSIONID，设置最大存活时间，让cookie持久化保存。
     Cookie c = new Cookie("JSESSIONID",session.getId());
         c.setMaxAge(60*60);
         response.addCookie(c);

2. 客户端不关闭，服务器关闭后，两次获取的session是同一个吗？
  * 不是同一个，但是要确保数据不丢失。tomcat自动完成以下工作
    * session的钝化：
      * 在服务器正常关闭之前，将session对象系列化到硬盘上
    * session的活化：
      * 在服务器启动后，将session文件转化为内存中的session对象即可。

3. session什么时候被销毁？
  1. 服务器关闭
  2. session对象调用invalidate() 。
  3. session默认失效时间 30分钟
    选择性配置修改
    <session-config>
          <session-timeout>30</session-timeout>
      </session-config>
```

### session的特点(理解)
    1. session用于存储一次会话的多次请求的数据，存在服务器端
    2. session可以存储任意类型，任意大小的数据

### session与Cookie的区别(理解)
    1. session存储数据在服务器端，Cookie在客户端
    2. session没有数据大小限制，Cookie有
    3. session数据安全，Cookie相对于不安全


## 案例：验证码(掌握)
### 案例需求(理解)
```
1. 访问带有验证码的登录页面login.jsp
2. 用户输入用户名，密码以及验证码。
  * 如果用户名和密码输入有误，跳转登录页面，提示:用户名或密码错误
  * 如果验证码输入有误，跳转登录页面，提示：验证码错误
  * 如果全部输入正确，则跳转到主页success.jsp，显示：用户名,欢迎您
```

### 分析(理解)
![](assets/markdown-img-paste-20180730231843756.png)

### 代码实现(掌握)
#### 步骤一:编写登录页面引入验证码图片
```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>login</title>
    <script>
        window.onload = function(){
            document.getElementById("img").onclick = function(){
                this.src="/day16/checkCodeServlet?time="+new Date().getTime();
            }
        }
    </script>
    <style>
        div{
            color: red;
        }
    </style>
</head>
<body>

    <form action="/day16/loginServlet" method="post">
        <table>
            <tr>
                <td>用户名</td>
                <td><input type="text" name="username"></td>
            </tr>
            <tr>
                <td>密码</td>
                <td><input type="password" name="password"></td>
            </tr>
            <tr>
                <td>验证码</td>
                <td><input type="text" name="checkCode"></td>
            </tr>
            <tr>
                <td colspan="2"><img id="img" src="/day16/checkCodeServlet"></td>
            </tr>
            <tr>
                <td colspan="2"><input type="submit" value="登录"></td>
            </tr>
        </table>
    </form>

    <div><%=request.getAttribute("cc_error") == null ? "" : request.getAttribute("cc_error")%></div>
    <div><%=request.getAttribute("login_error") == null ? "" : request.getAttribute("login_error") %></div>
</body>
</html>
```

#### 步骤二:编写登录的Servlet代码
```java
@WebServlet("/loginServlet")
public class LoginServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1.设置request编码
        request.setCharacterEncoding("utf-8");
        //2.获取参数
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        String checkCode = request.getParameter("checkCode");
        //3.先获取生成的验证码
        HttpSession session = request.getSession();
        String checkCode_session = (String) session.getAttribute("checkCode_session");
        //删除session中存储的验证码
        session.removeAttribute("checkCode_session");
        //3.先判断验证码是否正确
        if(checkCode_session!= null && checkCode_session.equalsIgnoreCase(checkCode)){
            //忽略大小写比较
            //验证码正确
            //判断用户名和密码是否一致
            if("zhangsan".equals(username) && "123".equals(password)){//需要调用UserDao查询数据库
                //登录成功
                //存储信息，用户信息
                session.setAttribute("user",username);
                //重定向到success.jsp
                response.sendRedirect(request.getContextPath()+"/success.jsp");
            }else{
                //登录失败
                //存储提示信息到request
                request.setAttribute("login_error","用户名或密码错误");
                //转发到登录页面
                request.getRequestDispatcher("/login.jsp").forward(request,response);
            }
        }else{
            //验证码不一致
            //存储提示信息到request
            request.setAttribute("cc_error","验证码错误");
            //转发到登录页面
            request.getRequestDispatcher("/login.jsp").forward(request,response);
        }
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request, response);
    }
}
```
