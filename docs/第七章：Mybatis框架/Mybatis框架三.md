# 今日内容介绍

<extoc></extoc>

# Mybatis连接池与事务深入(熟悉)
## Mybatis的连接池技术(熟悉)

我们在前面的WEB课程中也学习过类似的连接池技术,而在Mybatis中也有连接池技术,但是它采用的是自己的连接池技术。

在Mybatis的SqlMapConfig.xml配置文件中,通过<dataSource type=”pooled”>来实现Mybatis中连接池的配置。

连接池是一个用于存储连接的容器,这个容器就是一个集合对象。

容器必须是线程安全的,不能两个线程拿到同一个连接。容器还必须实现队列即先进先出原则。

在mybatis中将数据源分为三类:

    UNPOOLED– 这个数据源的实现只是每次被请求时打开和关闭连接。虽然有点慢,但对于在数据库连接可用性方面没有太高要求的简单应用程序来说,是一个很好的选择。
              不同的数据库在性能方面的表现也是不一样的,对于某些数据库来说,使用连接池并不重要,这个配置就很适合这种情形。UNPOOLED 类型的数据源仅仅需要配置以下 5 种属性:

                 driver – 这是 JDBC 驱动的 Java 类的完全限定名（并不是 JDBC 驱动中可能包含的数据源类）。
                 url – 这是数据库的 JDBC URL 地址。
                 username – 登录数据库的用户名。
                 password – 登录数据库的密码。
                 defaultTransactionIsolationLevel – 默认的连接事务隔离级别。

              作为可选项,你也可以传递属性给数据库驱动。要这样做,属性的前缀为“driver.”,例如:

                 driver.encoding=UTF8

    POOLED– 这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来,避免了创建新的连接实例时所必需的初始化和认证时间。
            这是一种使得并发 Web 应用快速响应请求的流行处理方式。

            除了上述提到 UNPOOLED 下的属性外,还有更多属性用来配置 POOLED 的数据源:

               poolMaximumActiveConnections – 在任意时间可以存在的活动（也就是正在使用）连接数量,默认值:10
               poolMaximumIdleConnections – 任意时间可能存在的空闲连接数。
               poolMaximumCheckoutTime – 在被强制返回之前,池中连接被检出（checked out）时间,默认值:20000 毫秒（即 20 秒）
               poolTimeToWait – 这是一个底层设置,如果获取连接花费了相当长的时间,连接池会打印状态日志并重新尝试获取一个连接（避免在误配置的情况下一直安静的失败）,默认值:20000 毫秒（即 20 秒）。
               poolMaximumLocalBadConnectionTolerance – 这是一个关于坏连接容忍度的底层设置, 作用于每一个尝试从缓存池获取连接的线程. 如果这个线程获取到的是一个坏的连接,那么这个数据源允许这个线程尝试重新获取一个新的连接,但是这个重新尝试的次数不应该超过 poolMaximumIdleConnections 与 poolMaximumLocalBadConnectionTolerance 之和。 默认值:3 (新增于 3.4.5)
               poolPingQuery – 发送到数据库的侦测查询,用来检验连接是否正常工作并准备接受请求。默认是“NO PING QUERY SET”,这会导致多数数据库驱动失败时带有一个恰当的错误消息。
               poolPingEnabled – 是否启用侦测查询。若开启,需要设置 poolPingQuery 属性为一个可执行的 SQL 语句（最好是一个速度非常快的 SQL 语句）,默认值:false。
               poolPingConnectionsNotUsedFor – 配置 poolPingQuery 的频率。可以被设置为和数据库连接超时时间一样,来避免不必要的侦测,默认值:0（即所有连接每一时刻都被侦测 — 当然仅当 poolPingEnabled 为 true 时适用）。


      JNDI – 这个数据源的实现是为了能在如 EJB 或应用服务器这类容器中使用,容器可以集中或在外部配置数据源,然后放置一个 JNDI 上下文的引用。这种数据源配置只需要两个属性:

              initial_context – 这个属性用来在 InitialContext 中寻找上下文（即,initialContext.lookup(initial_context)）。这是个可选属性,如果忽略,那么 data_source 属性将会直接从 InitialContext 中寻找。
              data_source – 这是引用数据源实例位置的上下文的路径。提供了 initial_context 配置时会在其返回的上下文中进行查找,没有提供时则直接在 InitialContext 中查找。

              和其他数据源配置类似,可以通过添加前缀“env.”直接把属性传递给初始上下文。比如:

              env.encoding=UTF8

![](assets/markdown-img-paste-20180831142655665.png)

在这三种数据源中,我们一般采用的是POOLED数据源（很多时候我们所说的数据源就是为了更好的管理数据库连接,也就是我们所说的连接池技术）

注意:如果不是web或maven工程是不能使用jndi方式的

## Mybatis中数据源的配置(熟悉)

![](assets/markdown-img-paste-20180831143313702.png)

## Mybatis中DataSource的存取(熟悉)(扩展补充)

MyBatis是通过工厂模式来创建数据源DataSource对象的,MyBatis定义了抽象的工厂接口:org.apache.ibatis.datasource.DataSourceFactory

通过其getDataSource()方法返回数据源DataSource。

通过这个特性,你可以通过实现接口 org.apache.ibatis.datasource.DataSourceFactory 来使用第三方数据源,例如:c3p0连接池

### 在pom.xml中引入C3P0依赖包(熟悉)
```xml
<dependency>
    <groupId>c3p0</groupId>
    <artifactId>c3p0</artifactId>
    <version>0.9.1.2</version>
</dependency>
```

### 引入c3p0配置文件(熟悉)
![](assets/markdown-img-paste-20180831150707156.png)

### 创建自定义工厂类(熟悉)
```java
/**
 * 自定义连接池工厂
 */
public class MyDataSourceFactory implements DataSourceFactory {

    private Properties properties;

    @Override
    public void setProperties(Properties properties) {
        this.properties = properties ;
    }

    @Override
    public DataSource getDataSource() {

        return new ComboPooledDataSource();
    }
}
```

### 编写SqlMapConfig.xml配置数据源(熟悉)
![](assets/markdown-img-paste-20180831150842964.png)

## Mybatis中连接的获取过程分析(理解)
![](assets/markdown-img-paste-20180831152306790.png)

## Mybatis的事务控制(理解)

在JDBC中我们可以通过手动方式将事务的提交改为手动方式,通过setAutoCommit()方法可以设置事物的提交方式。

Mybatis框架因为是对JDBC的封装,所以Mybatis框架的事务控制方式,本身也是用JDBC的setAutoCommit()方法来设置事务提交方式的。

### Mybatis中事务提交方式(理解)

Mybatis中事务的提交方式,本质上就是调用JDBC的setAutoCommit()来实现事务控制。

Mybatis在执行sql语句的时候调用了setAutoCommit()方法,在执行时将它的值设置为false,所以我们在CUD操作中,必须通过sqlSession.commit()方法来执行提交操作。

![](assets/markdown-img-paste-20180831152852570.png)

### Mybatis自动提交事务的设置(熟悉)
    Mybatis中可以让我们选择事物的提交方式是自动提交还是手动提交,默认是手动提交的,如果想要设置自动提交可以在打开连接时传递参数false即可

![](assets/markdown-img-paste-20180831153618610.png)

# Mybatis的动态SQL语句(掌握)

Mybatis的映射文件中,前面我们的SQL都是比较简单的,有些时候业务逻辑复杂时,我们的SQL是动态变化的,此时在前面的学习中我们的SQL就不能满足要求了。

MyBatis 的强大特性之一便是它的动态 SQL。如果你有使用 JDBC 或其它类似框架的经验,你就能体会到根据不同条件拼接 SQL 语句的痛苦。

例如拼接时要确保不能忘记添加必要的空格,还要注意去掉列表最后一个列名的逗号。利用动态 SQL 这一特性可以彻底摆脱这种痛苦。

虽然在以前使用动态 SQL 并非一件易事,但正是 MyBatis 提供了可以被用在任意 SQL 映射语句中的强大的动态 SQL 语言得以改进这种情形。

动态 SQL 元素和 JSTL 或基于类似 XML 的文本处理器相似。

MyBatis 3 大大精简了元素种类,现在只需学习原来一半的元素便可。

    if
    choose (when, otherwise)
    trim (where, set)
    foreach

## 动态SQL之<if>标签(掌握)

If标签用来判断输入参数是否符合规范,最常见的用法就是做多条件查询

### 在UserDao接口中编写方法(掌握)
```java
/**
 * 根据用户信息,查询用户列表
 * @param user
 * @return
 */
List<User> findByUser(User user);
```

### 在UserDao.xml中编写添加SQL语句(掌握)
```xml
<!-- 动态sql,if标签-->
<select id="findByUser" resultType="user" parameterType="user">
    select * from user where 1=1

    <!-- 如果username不为null ,筛选用户名 -->
    <if test="username!=null and username != '' ">
        and username like '%${username}%'
    </if>

    <!-- 如果address不为null ,筛选地址 -->
    <if test="address != null and address != '' ">
        and address like '%${address}%'
    </if>
</select>
```

### 编写测试方法(掌握)
```java
@Test
public void findByUser() {
    //创建用户对象
    User user = new User();
    user.setUsername("小");
    user.setAddress("长");

    List<User> users = userDao.findByUser(user);
    System.out.println(users);
}
```

## 动态SQL之<where>标签(掌握)

在上面的案例中我们为了能够将两个条件拼接使用了一个小技巧

![](assets/markdown-img-paste-20180831155727989.png)

我们在where后面使用了一个`1=1`的恒等式,保证我们拼接的sql语句不会出现语法问题

mybatis提供了一个`where`标签,也可以帮助我们来解决sql中条件拼接的问题

### 在`UserDao`接口中编写方法(掌握)
```java
/**
 * 根据用户信息,查询用户列表
 * @param user
 * @return
 */
List<User> findLikeUser(User user);
```

### 在`UserDao.xml`中编写添加SQL语句(掌握)
```xml
<!-- 动态sql,if标签  where标签-->
<select id="findLikeUser" resultType="user" parameterType="user">
    select * from user
    <where>
        <!-- 如果username不为null ,筛选用户名 -->
        <if test="username!=null and username != '' ">
            and username like '%${username}%'
        </if>

        <!-- 如果address不为null ,筛选地址 -->
        <if test="address != null and address != '' ">
            and address like '%${address}%'
        </if>
    </where>
</select>
```

### 编写测试方法(掌握)
```java
@Test
public void findLikeUser() {
    //创建用户对象
    User user = new User();
    user.setUsername("小");
    user.setAddress("长");

    List<User> users = userDao.findLikeUser(user);
    System.out.println(users);
}
```

## 动态标签之<foreach>标签(掌握)

动态 SQL 的另外一个常用的操作需求是对一个集合进行遍历,通常是在构建 IN 条件语句的时候。比如:



### 在UserDao接口中编写方法(掌握)
```java
/**
 * 根据传入的id查询用户信息
 * @param ids
 * @return
 */
List<User> findUsersByIds(Integer... ids);
```
### 在UserDao.xml中编写添加SQL语句(掌握)
```xml
<!--根据多个id查询用户信息-->
<select id="findUsersByIds" resultType="user" >
    SELECT *from USER
    <where>
        <foreach collection="array"  open="and id in (" close=")" separator="," item="uid">
            #{uid}
        </foreach>
    </where>
</select>
```

    注意:我们可以将任何`可迭代对象(如 List、Set)`、`Map对象`或者`数组对象`传递给 foreach 作为集合参数。

    当使用可迭代对象或者数组时,`index`是当前迭代的次数,`item` 的值是本次迭代获取的元素。

    <foreach>标签用于遍历集合,它的属性:
        collection:代表要遍历的集合元素,注意编写时不要写#{}
        open:代表语句的开始部分
        close:代表结束部分
        item:代表遍历集合的每个元素,生成的变量名
        sperator:代表分隔符


collection属性是在使用foreach的时候最关键的也是最容易出错的,该属性是必须指定的,但是在不同情况下,该属性的值是不一样的,主要有一下3种情况:

如果传入的参数是pojo类,`collection`属性值可以写pojo类的集合属性名称

![](assets/markdown-img-paste-20180831163400668.png)

如果传入的是单参数且参数类型是一个List的时候,collection属性值为`list`

![](assets/markdown-img-paste-20180831163655965.png)

如果传入的是单参数且参数类型是一个array数组的时候,collection的属性值为`array`

![](assets/markdown-img-paste-20180831163457628.png)

如果传入的参数是多个的时候,我们就需要把它们封装成一个Map了,当然单参数也可以封装成map,所以这个时候collection属性值map的集合属性的key.

![](assets/markdown-img-paste-20180831165004678.png)

### 编写测试方法(掌握)
```java
@Test
public void findUsersByIds() {
    List<User> users = userDao.findUsersByIds(41,42,54,56);
    System.out.println(users);
}
```

## Mybatis中简化编写的SQL片段(掌握)

Sql中可将重复的sql提取出来，使用时用include引用即可，最终达到sql重用的目的。

我们之前写的功能中,每一个功能都会有 `SELECT *from USER` 这么一条SQL,对于这些重复性比较高的SQL语句我们可以提取出来,定义一个SQL片段,最终达到sql重用的目的.

### 定义SQL片段(掌握)
```xml
<!--定义sql片段-->
<sql id="select_user">
    select * from user
</sql>
```

### 引用SQL片段(掌握)
![](assets/markdown-img-paste-20180831165804272.png)

<include>标签的refid属性的值就是<sql> 标签定义id的取值。

如果引用其它mapper.xml的sql片段，则在引用时需要加上namespace`<include refid="namespace.sql片段”/>`

# Mybatis多表查询(掌握)

## 多表之间的关系(掌握)

    一对多关系

        例如:

            一个部门下可以有多个员工，一个员工只能属于某一个部门。

            一个分类下有多个商品,一个商品只属于一个分类

            一个用户名下有多张银行卡,一个银行卡只属于一个用户

    多对多关系

        例如:

            一个学生可以选择多门课程，一门课程可以被多个学生选择。

            一个订单可以有多个商品,一个商品也可以属于多个订单

            一个用户可以使用多辆自行车,一个自行车也可以被多个用户使用

     一对一关系

        例如:

          一个公司可以有一个注册地址，一个注册地址只能对一个公司。

          一个人只能有一张身份证,一个身份证也只属于一个人

## 环境准备(掌握)

### 数据库准备(掌握)
使用用户和帐户表来说明表之间的级联操作

![](assets/markdown-img-paste-20180902124503645.png)

```sql
DROP TABLE IF EXISTS `user`;

CREATE TABLE `user` (
  `id` int(11) NOT NULL auto_increment,
  `username` varchar(32) NOT NULL COMMENT '用户名称',
  `birthday` datetime default NULL COMMENT '生日',
  `sex` char(1) default NULL COMMENT '性别',
  `address` varchar(256) default NULL COMMENT '地址',
  PRIMARY KEY  (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert  into `user`(`id`,`username`,`birthday`,`sex`,`address`) values (41,'老王','2018-02-27 17:47:08','男','北京'),(42,'小二王','2018-03-02 15:09:37','女','北京金燕龙'),(43,'小二王','2018-03-04 11:34:34','女','北京金燕龙'),(45,'传智播客','2018-03-04 12:04:06','男','北京金燕龙'),(46,'老王','2018-03-07 17:37:26','男','北京'),(48,'小马宝莉','2018-03-08 11:44:00','女','北京修正');

DROP TABLE IF EXISTS `account`;

CREATE TABLE `account` (
  `ID` int(11) NOT NULL COMMENT '编号',
  `UID` int(11) default NULL COMMENT '用户编号',
  `MONEY` double default NULL COMMENT '金额',
  PRIMARY KEY  (`ID`),
  KEY `FK_Reference_8` (`UID`),
  CONSTRAINT `FK_Reference_8` FOREIGN KEY (`UID`) REFERENCES `user` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert  into `account`(`ID`,`UID`,`MONEY`) values (1,41,1000),(2,45,1000),(3,41,2000);


DROP TABLE IF EXISTS `role`;

CREATE TABLE `role` (
  `ID` int(11) NOT NULL COMMENT '编号',
  `ROLE_NAME` varchar(30) default NULL COMMENT '角色名称',
  `ROLE_DESC` varchar(60) default NULL COMMENT '角色描述',
  PRIMARY KEY  (`ID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert  into `role`(`ID`,`ROLE_NAME`,`ROLE_DESC`) values (1,'院长','管理整个学院'),(2,'总裁','管理整个公司'),(3,'校长','管理整个学校');

DROP TABLE IF EXISTS `user_role`;

CREATE TABLE `user_role` (
  `UID` int(11) NOT NULL COMMENT '用户编号',
  `RID` int(11) NOT NULL COMMENT '角色编号',
  PRIMARY KEY  (`UID`,`RID`),
  KEY `FK_Reference_10` (`RID`),
  CONSTRAINT `FK_Reference_10` FOREIGN KEY (`RID`) REFERENCES `role` (`ID`),
  CONSTRAINT `FK_Reference_9` FOREIGN KEY (`UID`) REFERENCES `user` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert  into `user_role`(`UID`,`RID`) values (41,1),(45,1),(41,2);

```

### 创建maven项目,搭建mybatis开发环境(掌握)

![](assets/markdown-img-paste-20180831172000730.png)

### 创建pojo实体类(掌握)
#### `Account.java`类
![](assets/markdown-img-paste-20180831172039411.png)

一个账户信息只能供某个用户使用

#### `User.java`类
![](assets/markdown-img-paste-20180831172155636.png)

一个用户可以有多个账户

## Mybatis多表查询之一对一(掌握)

    需求:查询所有账户信息，关联查询下单用户信息。

    注意：
        因为一个账户信息只能供某个用户使用，所以从查询账户信息出发关联查询用户信息为一对一查询。

        如果从用户信息出发查询用户下的账户信息则为一对多查询，因为一个用户可以有多个账户。


### 方式一:通过继承扩展Account类(掌握)

#### 分析SQL语句
实现查询账户信息时，也要查询账户所对应的用户信息。

```sql
SELECT account.*, user.username, user.address FROM account, user WHERE account.uid = user.id
```
在MySQL中测试的查询结果如下：

![](assets/markdown-img-paste-20180831172429366.png)

为了能够封装上面SQL语句的查询结果，我们可以通过继承扩展Account类

定义 AccountUser类中要包含账户信息同时还要包含用户信息，所以我们要在定义AccountUser类时可以继承User类。

#### 定义AccountUser类
![](assets/markdown-img-paste-20180831172955245.png)

#### 定义账户的持久层Dao接口
```java
public interface AccountDao {

    /**
     * 查询所有账户信息,包含用户信息
     * @return
     */
    public AccountUser findAll();
}
```

#### 编写SQL映射文件`AccountDao.xml`
```xml
<select id="findAll" resultType="AccountUser">
    select  a.*,u.username,u.address from  account a inner join user u on a.UID = u.id
</select>
```


#### 编写测试类
```java
@Test
public void findAll() {
    List<AccountUser> aus = accountDao.findAll();
    System.out.println(aus);
}
```
### 方式二: 使用resultMap，定义专门的resultMap用于映射一对一查询结果。(掌握)

通过面向对象的(has a)关系可以得知，我们可以在Account类中加入一个User类的对象来代表这个账户是哪个用户的。

#### 定义Account类,添加代表账户所属用户的User属性
![](assets/markdown-img-paste-20180902120248146.png)


#### 在`AccountDao`接口中编写方法
```JAVA
/**
 * 查询所有账户信息,包含用户对象
 * @return
 */
public List<Account> findAllAccountWithUser();
```

#### 编写SQL映射文件`AccountDao.xml`
```xml
<!--定义结果集映射-->
<resultMap id="AccountUserMap" type="Account" >
    <id column="id" property="id"></id>
    <result column="uid" property="uid"></result>
    <result column="money" property="money"></result>

    <!--
        配置一对一映射,一个账户只属于一个用户
    -->
    <association property="user" javaType="User">
        <result column="username" property="username"></result>
        <result column="address" property="address"></result>
    </association>
</resultMap>

<!--定义SQL语句-->
<select id="findAllAccountWithUser" resultMap="AccountUserMap">
    select  a.*,u.username,u.address from  account a inner join user u on a.UID = u.id
</select>
```

#### 编写测试类
```JAVA
@Test
public void findAllAccountWithUser() {
    List<Account> accounts = accountDao.findAllAccountWithUser();
    System.out.println(accounts);
}
```

## Mybatis多表查询之一对多(掌握)

在帐户表中我们可以看到编号为41的用户有两个帐号，那么此时用户和帐号之间的关系为一对多，即一个用户拥有多个帐号。

现在我们来看一下如何通过用户来获得与他相关的所有帐号信息

![](assets/markdown-img-paste-20180902121501191.png)

### 修改用户实体类(掌握)
![](assets/markdown-img-paste-20180902121610661.png)

**注意getter和setter方法**


#### 在`UserDao`接口中编写方法
```java
/**
 * 查询所有用户信息,以及对应的账户信息
 * @return
 */
public User findAllWithAccount();
```

#### 编写SQL映射文件`UserDao.xml`
```xml
<!--定义结果集映射-->
<resultMap id="UserAccountMap" type="User">
    <id column="id" property="id"></id>
    <result column="username" property="username"></result>
    <result column="birthday" property="birthday"></result>
    <result column="sex" property="sex"></result>
    <result column="address" property="address"></result>

    <!--
        配置一对多映射,一个用户可以有多个账户
    -->
    <collection property="accounts" ofType="Account" >
        <id column="aid" property="id"></id>
        <result column="id" property="uid"></result>
        <result column="money" property="money"></result>
    </collection>
</resultMap>

<!--定义查询的sql语句-->
<select id="findAllWithAccount" resultMap="UserAccountMap" >
    select u.*,a.id aid,a.MONEY  from  user u inner join account a on u.id = a.uid
</select>
```

#### 编写测试类
```java
@Test
public void findAllWithAccount() {
    List<User> users = userDao.findAllWithAccount();
    System.out.println(users);
}
```

## Mybatis多表查询之多对多(掌握)

多对多关系其实是双向的一对多关系。此处我们使用用户表和角色表.一个用户可以有多个角色,一个角色也可以属于多个用户

![](assets/markdown-img-paste-20180902124758256.png)

### 需求分析(掌握)

实现在需要查询所有角色并且加载它所分配的用户信息。

分析： 查询角色我们需要用到Role表，但角色分配的用户的信息我们并不能直接找到用户信息，而是要通过中间表(USER_ROLE表)才能关联到用户信息。

下面是实现的SQL语句：

```sql
SELECT
  r.*,u.id uid, u.username username, u.birthday birthday, u.sex sex, u.address address
FROM
  ROLE r
INNER JOIN
  USER_ROLE ur
ON  
    r.id = ur.rid
INNER JOIN
  USER u
ON
    ur.uid = u.id
```

### 准备: 创建Role实体类
```java
/**
 * 角色实体
 */
public class Role {

    private int id ;

    private String role_name ;

    private String role_desc ;

    //多对多的关系映射：一个角色可以赋予多个用户
    private List<User> users;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getRole_name() {
        return role_name;
    }

    public void setRole_name(String role_name) {
        this.role_name = role_name;
    }

    public String getRole_desc() {
        return role_desc;
    }

    public void setRole_desc(String role_desc) {
        this.role_desc = role_desc;
    }

    public List<User> getUsers() {
        return users;
    }

    public void setUsers(List<User> users) {
        this.users = users;
    }
}
```

### 定义角色的持久层Dao接口(掌握)
```java
public interface RoleDao {

    /**
     * 查询所有的角色信息,包含拥有此角色的用户信息
     * @return
     */
    public List<Role> findAll();
}
```

### 编写SQL映射文件`RoleDao.xml`(掌握)
```xml
<!--配置结果集映射-->
<resultMap id="RoleUsersMapper" type="Role">
    <id column="id" property="id"></id>
    <result column="role_name" property="role_name"></result>
    <result column="role_desc" property="role_desc"></result>

    <!--
        配置一对多映射,一个角色可以赋予多个用户
    -->
    <collection property="users" ofType="User">
        <id column="uid" property="id"></id>
        <result column="username" property="username"></result>
        <result column="birthday" property="birthday"></result>
        <result column="sex" property="sex"></result>
        <result column="address" property="address"></result>
    </collection>
</resultMap>

<select id="findAll" resultMap="RoleUsersMapper">
    SELECT
      r.*,u.id uid, u.username username, u.birthday birthday, u.sex sex, u.address address
    FROM
      ROLE r
    INNER JOIN
      USER_ROLE ur
    ON
        r.id = ur.rid
    INNER JOIN
      USER u
    ON
        ur.uid = u.id
</select>
```

#### 编写测试类
```java
@Test
public void findAll() {
    List<Role> roles = roleDao.findAll();
    System.out.println(roles);
}
```

## 总结(掌握)
    resultMap中特殊节点说明:
      1. constructor: 指定使用指定参数列表的构造函数来实例化模型。注意：其子元素顺序必须与参数列表顺序对应, 用来对主实体类中对象属性做映射
      2. association：主要是用来解决一对一关系的。用来对主实体类中对象属性做映射
      3. collection: 用来解决一对多级联。用来对主实体类中的集合对象属性做映射


# JNDI数据源介绍(了解)
## JNDI介绍(了解)

JNDI就是将数据库连接信息写到tomcat的配置文件中，我们在程序中直接引用tomcat的配置文件即可

## 创建Maven的web项目(了解)
![](assets/markdown-img-paste-20180902162706321.png)

## 在`webapp`文件下创建`META-INF`目录(了解)
![](assets/markdown-img-paste-20180902163017542.png)

## 在`META-INF`目录下创建Context.xml配置文件(了解)
![](assets/markdown-img-paste-20180902163233121.png)

## 在`context.xml`中配置数据源相关信息(了解)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Context>
<!--
<Resource
name="jdbc/eesy_mybatis"                  数据源的名称
type="javax.sql.DataSource"                   数据源类型
auth="Container"                        数据源提供者
maxActive="20"                         最大活动数
maxWait="10000"                            最大等待时间
maxIdle="5"                               最大空闲数
username="root"                            用户名
password="1234"                            密码
driverClassName="com.mysql.jdbc.Driver"          驱动类
url="jdbc:mysql://localhost:3306/eesy_mybatis" 连接url字符串
/>
 -->
<Resource
    name="jdbc/mybatis_03_JNDI"
    type="javax.sql.DataSource"
    auth="Container"
    maxActive="20"
    maxWait="10000"
    maxIdle="5"
    username="root"
    password="zl"
    driverClassName="com.mysql.jdbc.Driver"
    url="jdbc:mysql://localhost:3306/mybatis_01"
    />
</Context>
```

## 在`SqlMapConfig.xml`中使用JNDI数据源(了解)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

    <typeAliases>
        <!-- 单个别名定义 -->
        <typeAlias alias="user" type="com.itheima.domain.User"/>
    </typeAliases>

    <environments default="mysql">
        <environment id="mysql">
            <transactionManager type="JDBC"></transactionManager>
            <dataSource type="JNDI">
                <property name="data_source" value="java:comp/env/jdbc/mybatis_03_JNDI"></property>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <package name="com.itheima.dao"></package>
    </mappers>

</configuration>
```

## 编写接口及映射文件(了解)
```java
public interface UserDao {
    /**
     * 查询所有用户
     * @return
     */
    public List<User> findAll();
}
```
```xml
<select id="findAll" resultType="user">
    select * from user
</select>
```

## 在Servlet中或者JSP中执行UserDao中的方法(了解)
```java
@WebServlet("/ServletDemo1")
public class ServletDemo1 extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //1. 创建sqlSessionBuilder构造者对象
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();

        //2. 创建SqlSessionFactory对象
        //2.1 加载配置文件
        InputStream is = Resources.getResourceAsStream("SqlMapConfig.xml");
        //2.2 构建SqlSession工厂
        SqlSessionFactory sf = builder.build(is);

        //3. 获取SqlSession对象
        SqlSession session = sf.openSession(true);

        //4. 获取mapper类的代理对象
        UserDao userDao = session.getMapper(UserDao.class);

        //5. 执行查询所有操作
        List<User> users = userDao.findAll();
        System.out.println(users);
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
```

## 发布项目,启动服务器(了解)

打开浏览器访问`/ServletDemo1`完成测试
