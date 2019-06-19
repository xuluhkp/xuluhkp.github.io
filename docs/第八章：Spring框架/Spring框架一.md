# 今日内容介绍

<extoc></extoc>

# Spring概述(了解)

## 什么是Spring(了解)

    Spring是分层的Java SE/EE应用 full-stack轻量级开源框架,以IOC（Inverse Of Control：反转控制）和AOP（Aspect Oriented Programming：面向切面编程）为内核,提供了展现层Spring MVC和持久层Spring JDBC以及业务层事务管理等众多的企业级应用技术,还能整合开源世界众多著名的第三方框架和类库,逐渐成为使用最多的Java EE企业应用开源框架.

![](assets/markdown-img-paste-20180903141425829.png)

## Spring的发展历程(了解)

    1997年IBM提出了EJB的思想
    1998年,SUN制定开发标准规范EJB1.0
    1999年,EJB1.1发布
    2001年,EJB2.0发布
    2003年,EJB2.1发布
    2006年,EJB3.0发布
    Rod Johnson（spring之父）
    Expert One-to-One J2EE Design and Development(2002)
    阐述了J2EE使用EJB开发设计的优点及解决方案
    Expert One-to-One J2EE Development without EJB(2004)
    阐述了J2EE开发不使用EJB的解决方式（Spring雏形）
    2017年9月份发布了spring的最新版本spring 5.0通用版（GA）

## Spring的优势(了解)
    方便解耦,简化开发

        通过Spring提供的IOC容器,可以将对象间的依赖关系交由Spring进行控制,避免硬编码所造成的过度程序耦合.用户也不必再为单例模式类、属性文件解析等这些很底层的需求编写代码,可以更专注于上层的应用.

    AOP编程的支持

        通过Spring的AOP功能,方便进行面向切面的编程,许多不容易用传统OOP实现的功能可以通过AOP轻松应付.

    声明式事务的支持

        可以将我们从单调烦闷的事务管理代码中解脱出来,通过声明式方式灵活的进行事务的管理,提高开发效率和质量.

    方便程序的测试

        可以用非容器依赖的编程方式进行几乎所有的测试工作,测试不再是昂贵的操作,而是随手可做的事情.

    方便集成各种优秀框架

        Spring可以降低各种框架的使用难度,提供了对各种优秀框架（Struts、Hibernate、Hessian、Quartz等）的直接支持.

    降低JavaEE API的使用难度

        Spring对JavaEE API（如JDBC、JavaMail、远程调用等）进行了薄薄的封装层,使这些API的使用难度大为降低.

    Java源码是经典学习范例

        Spring的源代码设计精妙、结构清晰、匠心独用,处处体现着大师对Java设计模式灵活运用以及对Java技术的高深造诣.它的源代码无意是Java技术的最佳实践的范例.

## Spring的体系结构(了解)

![](assets/markdown-img-paste-2018090314163839.png)

# SpringIOC(控制反转)(理解)

## 程序的耦合和解耦(理解)

### 什么是程序的耦合(理解)
    耦合就是程序间的依赖关系.解藕就是降低程序间的依赖关系.在开发过程中应做到编译期不依赖,运行时再依赖

### 之前代码的问题(理解)

例如:

![](assets/markdown-img-paste-20180903142826689.png)

    业务层调用持久层,并且此时业务层在依赖持久层的接口和实现类.

    如果此时没有持久层实现类,编译将不能通过.或者入门我们项更换新的实现类,需要修改源码重新编译

    这种编译期依赖关系,应该在我们开发中杜绝.我们需要优化代码解决.

再比如:

![](assets/markdown-img-paste-20180903142852643.png)

早期我们的JDBC操作,注册驱动时,我们为什么不使用DriverManager的register方法,而是采用Class.forName的方式？

原因就是： 我们的类依赖了数据库的具体驱动类（MySQL）,如果这时候更换了数据库品牌（比如Oracle）,需要修改源码来重新数据库驱动.这显然不是我们想要的.

### 解耦思路(理解)

当是我们讲解jdbc时,是通过反射来注册驱动的,代码如下：
```java
Class.forName("com.mysql.jdbc.Driver");//此处只是一个字符串
```
此时的好处是,我们的类中不再依赖具体的驱动类,此时就算删除mysql的驱动jar包,依然可以编译（运行就不要想了,没有驱动不可能运行成功的）.

同时,也产生了一个新的问题,mysql驱动的全限定类名字符串是在java类中写死的,一旦要改还是要修改源码.

解决这个问题也很简单,使用配置文件将这些经常可变的参数提取到了配置文件中

解耦的思路与步骤:

    1. 使用反射来创建对象,而不使用new关键字
    2. 提取配置及类的全限定名到配置文件中
    3. 通过读取配置文件来获取要创建的对象全限定名
    4. 使用反射创建对象

### 工厂模式解耦(理解)

    在实际开发中我们可以把三层的对象都使用配置文件配置起来,当启动服务器应用加载的时候,让一个类中的方法通过读取配置文件,把这些对象创建出来并存起来.

    在接下来的使用的时候,直接拿过来用就好了. 那么,这个读取配置文件,创建和获取三层对象的类就是工厂.


#### 创建配置文件`beans.xml`(理解)
![](assets/markdown-img-paste-20180903153435649.png)

#### 使用工厂模式解析配置文件创建对象(理解)
```java
public class BeanFactory {

    public static Object  getBean(String name){
        //读取配置文件到内存中
        String path = BeanFactory.class.getClassLoader().getResource("beans.xml").getPath();

        try {
            //解析xml文件,获取文档对象
            Document document = Jsoup.parse(new File(path), "utf-8");
            //解析xml文件获取bean标签
            Elements beans = document.select("bean[id="+name+"]");
            //判断是否找到对应的标签
            if(beans.size()>0){
                //获取标签对象 : <bean id="UserService" class="com.itheima.servcie.impl.UserServiceImpl"></bean>
                Element bean = beans.get(0);
                //获取class属性值
                String str = bean.attr("class");
                // 加载类到内存
                Class<?> clzz = Class.forName(str);
                // 创建对象并返回
                return clzz.getDeclaredConstructor().newInstance();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }

        return null ;
    }
}
```

#### 代码测试(理解)
![](assets/markdown-img-paste-20180903154136867.png)

#### 工厂模式优化(理解)

    此种方式每次调用`getBean`方法都会创建对象,方法执行完毕之后就会销毁,这样对象会不停的创建和销毁对象,,执行效率比较低,会影响我们程序的执行效率,可以对以上方法做一个优化

    可以将创建的对象保存在容器中,下一次用到这个对象的时候,直接从容器中获取就可以了,提高程序的执行效率


```java
public class BeanFactory {

    private static Map<String,Object> map = new HashMap<String, Object>();

    static {
        //读取配置文件到内存中
        String path = BeanFactory.class.getClassLoader().getResource("beans.xml").getPath();

        try {
            //解析xml文件,获取文档对象
            Document document = Jsoup.parse(new File(path), "utf-8");
            //解析xml文件获取bean标签
            Elements beans = document.getElementsByTag("bean");
            //判断是否找到对应的标签
            for (int i = 0; i < beans.size() ; i++) {
                //获取标签对象 : <bean id="UserService" class="com.itheima.servcie.impl.UserServiceImpl"></bean>
                Element bean = beans.get(i);
                //获取class属性值
                String id = bean.attr("id");
                String str = bean.attr("class");
                // 加载类到内存
                Class<?> clzz = Class.forName(str);
                // 创建对象并返回
                Object obj = clzz.getDeclaredConstructor().newInstance();
                //将创建的对象放入到容器中
                map.put(id,obj);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static Object  getBean(String name){
        //从容器中获取对象并返回
        return map.get(name);
    }
}
```

## SpringIOC概述(理解)

![](assets/markdown-img-paste-2018090316010975.png)

之前我们在获取对象时,都是采用new的方式.是主动的.

![](assets/markdown-img-paste-20180903155953686.png)

使用工厂模式解耦合之后,我们获取对象时,同时跟工厂要,有工厂为我们查找或者创建对象.是被动的.

![](assets/markdown-img-paste-20180903160037350.png)

这种被动接收的方式获取对象的思想就是控制反转,它是spring框架的核心之一.

## SpringIOC作用(理解)

IOC的作用: 削减计算机程序的耦合(解除我们代码中的依赖关系).

# SpringIOC使用(掌握)
## 准备(掌握)

官网: http://spring.io/

下载地址: http://repo.springsource.org/libs-release-local/org/springframework/spring

解压:

![](assets/markdown-img-paste-20180903160750581.png)                                    


我们上课使用的版本是 spring5.0.2.

特别说明:

spring5 版本是用 jdk8 编写的,所以要求我们的 jdk 版本是 8 及以上. 同时 tomcat 的版本要求 8.5 及以上.

## 搭建Spring开发环境(掌握)

### 创建maven项目(掌握)
![](assets/markdown-img-paste-2018090316243862.png)

### 导入Spring的相关依赖(掌握)
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.0.2.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
</dependencies>
```

### 编写配置文件,引入文档声明`beans.xml`(掌握)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">


</beans>
```

## 使用Spring解决耦合问题(掌握)
### 编写业务层接口及实现类(掌握)
#### 业务层接口
```java
public interface UserService {

    public void add();
}
```
#### 编写业务层实现类
```java
public class UserServiceImpl implements UserService {

    private UserDao userDao = new UserDaoImpl() ; //此处需要使用SpringIOC完成解耦和

    public void add() {
        userDao.add();
    }
}
```

### 编写数据访问层接口及实现类(掌握)
#### 数据访问层接口
```java
public interface UserDao {

    public void add();
}

```

#### 数据访问层实现类
```java
public class UserDaoImpl implements UserDao {
    public void add() {
        System.out.println("保存账户信息....");
    }
}
```

### 在`beans.xml`配置文件中配置IOC(掌握)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--配置dao和service层的对象-->
    <!--
        配置service对象
           id:唯一标识
           class:要创建对象的全限定名
    -->
    <bean id="userService" class="com.itheima.service.impl.UserServiceImpl"></bean>

    <!--配置userdao对象-->
    <bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl"></bean>

</beans>
```

### 编写测试类,使用SpringIOC完成解耦(掌握)
```java
public static void main(String[] args) {

    //加载Spring配置文件
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");

    //获取业务层bean对象
    UserService userService = (UserService) applicationContext.getBean("userService");

    //调用业务层方法
    userService.add();
}
```

业务层中调用数据访问层,可以同上方式进行bean对象获取

## ApplicationContext的实现关系解析(熟悉)

### spring中工厂的类结构图(熟悉)
![](assets/markdown-img-paste-20180903165430114.png)

### `BeanFactory`和`ApplicationContext`的区别(熟悉)

    1. BeanFactory是ApplicationContext的顶层接口

    2. BeanFactory是采用延迟加载策略.只有真正使用getBean时才会实例化Bean.适用于多例模式

    3. ApplicationContext配置文件加载时就会立即初始化Bean,适用于单例模式,实际开发时一般采用这种.


### 常用的三个实现类(熟悉)
    1. ClassPathXmlApplicationContext:加载类路径下的配置文件,要求配置文件必须在类路径下（常用）

    2. FileSystemXmlApplicationContext:加载磁盘任意路径下的配置文件(必须有访问权限)

    3. AnnotationConfigApplicationContext:读取注解配置容器


## IOC中bean标签和管理对象细节(掌握)
### 创建bean的三种方式(掌握)
#### 使用默认无参构造函数创建
![](assets/markdown-img-paste-20180903171942273.png)

#### 使用普通工厂方法创建
```java
public class UserServiceFactory {

    public UserService getBean(){
        return new UserServiceImpl();
    }
}
```

![](assets/markdown-img-paste-2018090317223876.png)

#### 使用静态工厂方法创建
```java
public class UserServiceStaticFactory {

    /**
     * 静态工厂方法
     * @return
     */
    public static UserService getBean(){
        return new UserServiceImpl();
    }
}
```
![](assets/markdown-img-paste-20180903172451221.png)

### bean对象的作用范围(掌握)

在bean声明时它有一个scope属性，它是用于描述bean的作用域。

![](assets/markdown-img-paste-20180903172721906.png)

可取值：

    1. singleton: 单例 代表在SpringIOC容器中只有一个Bean实例 (默认的scope)

    2. prototype多例 每一次从Spring容器中获取时，都会返回一个新的实例

    3. request 用在web开发中，将bean对象request.setAttribute()存储到request域中

    4. session 用在web开发中，将bean对象session.setAttribute()存储到session域中


### bean对象的生命周期(熟悉)

    单例对象:scope="singleton" 一个应用只有一个对象的实例。它的作用范围就是整个引用。

        生命周期:
            创建:当应用加载，创建容器时，对象就被创建了。
            活跃:只要容器在，对象一直活着。
            销毁:当应用卸载，销毁容器时，对象就被销毁了。

    多例对象:scope="prototype" 每次访问对象时，都会重新创建对象实例。

        生命周期:
          创建:当使用对象时，创建新的对象实例。
          活跃:只要对象在使用中，就一直活着。
          销毁:当对象长时间不用时，被 java 的垃圾回收器回收了。

# SpringDI使用(掌握)

## SpringDI概述(掌握)

    依赖注入：Dependency Injection。

    是指在spring框架负责创建Bean对象时，动态将依赖对象注入到Bean中。

    之前我们都需要自己通过`getBean`方法获取到对象,使用DI之后就可以让Spring框架帮我们把对象注入到bean中

        依赖注入能注入的数据类型：
          1. 基本类型和string
          2. 其他Bean类型（在配置文件中或注解配置过的bean）
          3. 复杂类型/集合类型

        依赖注入的方式：
          1. 通过构造函数
          2. 使用set方法
          3. 使用注解

## 构造函数注入(掌握)
### 创建创建pojo类`User.java`(掌握)
```java
package com.itheima.bean;

import java.util.Date;

public class User {

    private String username ;
    private String adress ;
    private String sex ;
    private Date birthday ;

    public User(){

    }

    public User(String username, String adress, String sex, Date birthday) {
        this.username = username;
        this.adress = adress;
        this.sex = sex;
        this.birthday = birthday;
    }



    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getAdress() {
        return adress;
    }

    public void setAdress(String adress) {
        this.adress = adress;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }
}
```

### 在bean.xml中为`user`对象注入值(掌握)
```xml
<!--事件日期格式对象,相对与new Date()-->
<bean id="now" class="java.util.Date"></bean>

<!--配置bean对象,使用构造函数注入值-->
<bean id="user" class="com.itheima.bean.User">
    <constructor-arg name="username" value="张三丰"></constructor-arg>
    <constructor-arg name="adress" value="金融港"></constructor-arg>
    <constructor-arg name="sex" value="男"></constructor-arg>
    <constructor-arg name="birthday" ref="now"></constructor-arg>
</bean>
```

### 构造函数注入测试(掌握)
```java
@Test
public  void test1() {
    //加载spring配置文件
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");

    //从Spring容器中获取bean对象
    User user = (User) applicationContext.getBean("user");

    System.out.println(user);//User{username='张三丰', adress='金融港', sex='男', birthday=Mon Sep 03 18:20:58 CST 2018}
}
```

### 构造函数注入总结(掌握)

    类中需要提供一个对应参数列表的构造函数。

    涉及的标签： constructor-arg

    涉及的属性：
          index:指定参数在构造函数参数列表的索引位置
          type:指定参数在构造函数中的数据类型
          name:指定参数在构造函数中的名称 用这个找给谁赋值

          =======上面三个都是找给谁赋值，下面两个指的是赋什么值的==============

          value:它能赋的值是基本数据类型和String类型
          ref:它能赋的值是其他bean类型，也就是说，必须得是在配置文件中配置过的bean

    构造注入优缺点:

       优势:在获取bean对象时，数据注入是必须的，否则无法创建成功

       缺点:改变了bean对象的实例化方式，使我们在创建对象时如果用不到某此数据同样也要提供。

## Set方法注值(掌握)

会调用bean的set方法完成值的注入

### 在bean.xml中为`user`对象注入值(掌握)
```xml
<bean id="now" class="java.util.Date"></bean>

<!--配置bean对象,使用setter方法注入值-->
<bean id="user" class="com.itheima.bean.User">
    <property name="username" value="张无忌"></property>
    <property name="adress" value="长城园"></property>
    <property name="sex" value="男"></property>
    <property name="birthday" ref="now"></property>
</bean>
```

### Set方法注值测试(掌握)
```java
@Test
public  void test1() {
    //加载spring配置文件
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");

    //从Spring容器中获取bean对象
    User user = (User) applicationContext.getBean("user");

    System.out.println(user);//User{username='张无忌', adress='长城园', sex='男', birthday=Mon Sep 03 18:30:23 CST 2018}
}
```

### Set方法注值总结(掌握)

    此种方式赋值一定要有无参构造函数,并且为属性提供公有的getter和setter方法

    通过配置文件给bean中的属性传值：使用set方法的方式

    涉及的标签： property  

    涉及的属性：

        name：找的是类中set方法后面的部分

        ref：给属性赋值是其他bean类型的

        value：给属性赋值是基本数据类型和string类型的 实际开发中，此种方式用的较多


    Set注入优缺点：

         优势:创建对象时没有明确的限制，可以直接使用默认的无参构造函数

         缺点:如果某个成员必须有值，获取对象时set方法可能没有执行


## 注入集合类型(熟悉)

### 在bean.xml中为`user`对象注入值(熟悉)

```xml
<bean id="now" class="java.util.Date"></bean>

<!--配置bean对象,使用setter方法注入值-->
<bean id="user" class="com.itheima.bean.User">
    <property name="username" value="张无忌"></property>
    <property name="adress" value="长城园"></property>
    <property name="sex" value="男"></property>
    <property name="birthday" ref="now"></property>

    <!--为数组注入值-->
    <property name="myArray">
        <array>
            <value>数组1</value>
            <value>数组2</value>
            <value>数组3</value>
        </array>
    </property>

    <!--为list注入值-->
    <property name="myList">
        <list>
            <value>List1</value>
            <value>List2</value>
            <value>List3</value>
        </list>
    </property>

    <!--为Set注入值-->
    <property name="mySet">
        <set>

            <value>SET1</value>
            <value>SET2</value>
            <value>SET3</value>
        </set>
    </property>

    <!--为map注入值
       注意map为键/值对的形式
    -->
    <property name="myMap">
        <map>
            <entry key="map1" value="aaa"></entry>
            <entry key="map2" value="bbb"></entry>
            <entry key="map3" value="ccc"></entry>
        </map>
    </property>

    <!--为Properties注入值-->
    <property name="properties">
        <props>
            <prop key="prop1">AAA</prop>
            <prop key="prop2">BBB</prop>
            <prop key="prop3">CCC</prop>
        </props>
    </property>
</bean>
```

### 注入集合类型测试(熟悉)
```java
@Test
public  void test1() {
    //加载spring配置文件
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");

    //从Spring容器中获取bean对象
    User user = (User) applicationContext.getBean("user");

    //User{username='张无忌', adress='长城园', sex='男', birthday=Mon Sep 03 18:49:59 CST 2018,
    // myArray=[数组1, 数组2, 数组3], myList=[List1, List2, List3],
    // mySet=[SET1, SET2, SET3], myMap={map1=aaa, map2=bbb, map3=ccc},
    // properties={prop2=BBB, prop1=AAA, prop3=CCC}}
    System.out.println(user);
}
```
### 注入集合类型总结(熟悉)

    用于给list注入的标签  `array`、`list`、`set`

    用于给map注入的标签 `	map`、`props`

    结构相同的标签可以互换
