# 今日内容介绍

<extoc></extoc>

# 基于注解的IOC和DI配置(掌握)
## 常用注解(掌握)

    1. 用于创建对象：作用和xml配置中的<bean>标签的作用是一样的
    2. 用于注入数据：作用和xml配置中的<bean>标签中的<property>标签作用一样
    3. 用于改变作用范围：作用和xml配置中的<bean>标签的scope属性的作用一样
    4. 生命周期相关：作用和xml配置中的<bean>标签的init-method和destroy-method属性的作用是一样的

## 环境准备(掌握)

### 创建maven项目(掌握)
![](assets/markdown-img-paste-20180904111400905.png)

### 引入依赖包(掌握)
```xml
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>9</maven.compiler.source>
    <maven.compiler.target>9</maven.compiler.target>
</properties>

<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.0.2.RELEASE</version>
    </dependency>
</dependencies>
```

### 创建Spring核心配置文件`beans.xml`(掌握)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

</beans>
```

<font color="red">注意:如果要使用注解,需要导入`context`名称空间 `xmlns:context="http://www.springframework.org/schema/context"`</font>

### 创建数据访问层接口及实现类(掌握)
#### 接口`UserDao`
```java
public interface UserDao {

    public void add();
}
```

#### 实现类
```java
public class UserDaoImpl implements UserDao {
    @Override
    public void add() {
        System.out.println("数据访问层添加方法执行....");
    }
}
```


## 开启注解扫描(掌握)

要想使用注解,必须在`beans.xml`中添加一段配置开启注解扫描

```xml
<!--配置注解扫描,扫描指定包下的注解-->
<context:component-scan base-package="com.itheima"></context:component-scan>
```

![](assets/markdown-img-paste-2018090411300490.png)


## 用于创建对象的注解(掌握)

相当于：`<bean id="" class="">`

### `@Component`(掌握)

    作用：
        把资源让spring来管理。相当于在xml中配置一个bean。

    属性：
        value：指定bean的id。如果不指定value属性，默认bean的id是当前类的类名。首字母小写。

![](assets/markdown-img-paste-2018090411384128.png)

![](assets/markdown-img-paste-20180904113916921.png)

### `@Controller` `@Service` `@Repository`(掌握)

      他们三个注解都是针对一个的衍生注解，他们的作用及属性都是一模一样的。 他们只不过是提供了更加明确的语义化。

      @Controller：一般用于表现层的注解。

      @Service：一般用于业务层的注解。

      @Repository：一般用于持久层的注解。

      细节：如果注解中有且只有一个属性要赋值时，且名称是value，value在赋值是可以不写。

![](assets/markdown-img-paste-20180904114241357.png)


## 用于注入数据的注解(掌握)

相当于 ：`<property name="" ref="">`

        `<property name="" value="">`

### `@Autowired`(掌握)

    作用：
        自动按照类型注入。当使用注解注入属性时，set方法可以省略。它只能注入其他bean类型。当有多个类型匹配时，使用要注入的对象变量名称作为bean的id，在spring容器查找，找到了也可以注入成功。找不到就报错。

![](assets/markdown-img-paste-2018090411480475.png)

### `@Qualifier`(掌握)

    作用：
        在自动按照类型注入的基础之上，再按照Bean的id注入。它在给字段注入时不能独立使用，必须和@Autowire一起使用；但是给方法参数注入时，可以独立使用。
    属性：
        value：指定bean的id。

![](assets/markdown-img-paste-20180904115458691.png)

### `@Resource`(掌握)

    作用：
        直接按照Bean的id注入。只能注入其他bean类型。

    属性：
        name：指定bean的id。

![](assets/markdown-img-paste-20180904135827600.png)

注意:
    `@Resource`是由JDK提供的注解,在JDK1.9之后移除到扩展模块之中.所以如果想使用`@Resource`注解注入数据,请使用1.8及以下版本JDK


### `@Value`(掌握)

    作用：
        注入基本数据类型和String类型数据的

    属性：
        value：用于指定值

![](assets/markdown-img-paste-20180904141052522.png)

![](assets/markdown-img-paste-20180904141311443.png)

## 用于改变作用范围的注解(掌握)

相当于：<bean id="" class="" scope="">

### `@Scope`(掌握)

    作用：
        指定bean的作用范围。

    属性：
        value：指定范围的值。 取值：`singleton` `prototype` `request` `session` `globalsession`  

![](assets/markdown-img-paste-20180904141240109.png)

![](assets/markdown-img-paste-20180904141256459.png)

## 和生命周期相关的注解(了解)

相当于：`<bean id="" class="" init-method="" destroy-method="" />`

### `@PostConstruct`

    作用：
        用于指定初始化方法。

### `@PreDestroy`

    作用：
        用于指定销毁方法

![](assets/markdown-img-paste-20180904141606525.png)

# 使用Spring的IOC实现CRUD操作(掌握)

## 准备(掌握)
### 创建maven项目(掌握)
![](assets/markdown-img-paste-20180904150224342.png)
### 导入相关依赖包(掌握)
```xml
<packaging>jar</packaging>

<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.0.2.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>5.0.2.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>c3p0</groupId>
        <artifactId>c3p0</artifactId>
        <version>0.9.1.2</version>
    </dependency>
    <dependency>
        <groupId>javax.annotation</groupId>
        <artifactId>jsr250-api</artifactId>
        <version>1.0</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.38</version>
    </dependency>
    <dependency>
        <groupId>commons-dbutils</groupId>
        <artifactId>commons-dbutils</artifactId>
        <version>1.6</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.7.0</version>
            <configuration>
                <target>1.8</target>
                <source>1.8</source>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### 创建spring核心配置文件`beans.xml`并配置(掌握)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">


</beans>
```

### 编写数据访问层接口及实现类(掌握)
#### 接口`UserDao.java`

```java
public interface UserDao {

    /**
     * 添加用户
     * @param user
     */
    void add(User user) throws SQLException;

    /**
     * 更新用户
     * @param user
     */
    void update(User user) throws SQLException;

    /**
     * 删除用户
     * @param id
     */
    void delete(Integer id) throws SQLException;

    /**
     * 根据id查询用户
     * @param id
     * @return
     */
    User findById(Integer id) throws SQLException;

    /**
     * 查询所有用户
     * @return
     */
    List<User> findAll() throws SQLException;

}

```

#### 实现类(掌握)

```java
public class UserDaoImpl implements UserDao {

    private QueryRunner queryRunner;

    public void setQueryRunner(QueryRunner queryRunner) {
        this.queryRunner = queryRunner;
    }

    @Override
    public void add(User user) throws SQLException {
        String sql = "insert  into `user` values (null,?,?,?,?)";
        queryRunner.update(sql, user.getUsername(), user.getBirthday(), user.getSex(), user.getAddress());
    }

    @Override
    public void update(User user) throws SQLException {
        String sql = "update user set username = ? ,birthday = ? ,sex = ? ,address = ? where id = ? ";
        queryRunner.update(sql, user.getUsername(), user.getBirthday(), user.getSex(), user.getAddress(), user.getId());
    }

    @Override
    public void delete(Integer id) throws SQLException {
        String sql = "delete from user where id = ? ";
        queryRunner.update(sql, id);
    }

    @Override
    public User findById(Integer id) throws SQLException {
        String sql = "select * from user where id = ? ";
        return queryRunner.query(sql, new BeanHandler<User>(User.class), id);
    }

    @Override
    public List<User> findAll() throws SQLException {
        String sql = "select * from user ";
        return queryRunner.query(sql, new BeanListHandler<User>(User.class));
    }
}

```

### 编写业务层接口及实现类(掌握)
#### 接口
```java
public interface UserService {

    /**
     * 添加用户
     * @param user
     */
    void add(User user) throws SQLException;

    /**
     * 更新用户
     * @param user
     */
    void update(User user) throws SQLException;

    /**
     * 删除用户
     * @param id
     */
    void delete(Integer id) throws SQLException;

    /**
     * 根据ID查找用户
     * @param id
     * @return
     */
    User findById(Integer id) throws SQLException;

    /**
     * 查询所有用户
     * @return
     */
    List<User> findAll() throws SQLException;
}

```

#### 实现类
```java
public class UserServiceImpl implements UserService {

    private UserDao userDao ;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    @Override
    public void add(User user) throws SQLException {
        userDao.add(user);
    }

    @Override
    public void update(User user) throws SQLException {
        userDao.update(user);
    }

    @Override
    public void delete(Integer id) throws SQLException {
        userDao.delete(id);
    }

    @Override
    public User findById(Integer id) throws SQLException {
        return userDao.findById(id);
    }

    @Override
    public List<User> findAll() throws SQLException {
        return userDao.findAll();
    }
}

```

### 编写测试类(掌握)
```java
public class UserServiceTest {

    private UserService userService ;

    @Before
    public void init (){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");
        userService = (UserService) applicationContext.getBean("userService");
    }

    @Test
    public void add() throws SQLException {
        //创建用户对象
        User user = new User();
        user.setUsername("赵敏");
        user.setAddress("金融港");
        user.setSex("男");
        user.setBirthday(new Date());

        userService.add(user);
    }

    @Test
    public void update() throws SQLException {
        //创建用户对象
        User user = new User();
        user.setId(45);
        user.setUsername("张无忌");
        user.setAddress("金融港");
        user.setSex("男");
        user.setBirthday(new Date());

        userService.update(user);
    }

    @Test
    public void delete() throws SQLException {

        userService.delete(48);
    }

    @Test
    public void findById() throws SQLException {
        User user = userService.findById(45);
        System.out.println(user);
    }

    @Test
    public void findAll() throws SQLException {
        List<User> users = userService.findAll();
        System.out.println(users);
    }
}
```

## 基于XML的实现方式(掌握)
### 为数据访问层实现类中的`template`属性提供`setter`方法(掌握)

![](assets/markdown-img-paste-20180904152329588.png)

### 为业务层层实现类中的`userDao`属性提供`setter`方法(掌握)

![](assets/markdown-img-paste-20180904152312899.png)

### 配置Spring的IOC和DI(掌握)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!--配置数据源-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/spring_02"></property>
        <property name="user" value="root"></property>
        <property name="password" value="zl"></property>
    </bean>

    <!--配置DbUtils核心运行类queryRunner,注入数据源-->
    <bean id="queryRunner" class="org.apache.commons.dbutils.QueryRunner">
        <constructor-arg name="ds" ref="dataSource"></constructor-arg>
    </bean>

    <!--配置数据访问层-->
    <bean id="userDao" class="com.itheima.dao.impl.UserDaoImpl">
        <property name="queryRunner" ref="queryRunner"></property>
    </bean>

    <!--配置业务逻辑层-->
    <bean id="userService" class="com.itheima.service.impl.UserServiceImpl">
        <property name="userDao" ref="userDao"> </property>
    </bean>

</beans>
```

## 基于注解的实现方式(掌握)
### 配置组件扫描与template模板类(掌握)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
    <!--配置组件扫描-->
    <context:component-scan base-package="com.itheima"></context:component-scan>

    <!--配置数据源-->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/spring_02"></property>
        <property name="user" value="root"></property>
        <property name="password" value="zl"></property>
    </bean>

    <!--配置DbUtils核心运行类queryRunner,注入数据源-->
    <bean id="queryRunner" class="org.apache.commons.dbutils.QueryRunner">
        <constructor-arg name="ds" ref="dataSource"></constructor-arg>
    </bean>

</beans>
```

### 为数据访问层实现类添加注解(掌握)
![](assets/markdown-img-paste-20180907163507125.png)

### 为业务层实现类添加注解(掌握)
![](assets/markdown-img-paste-2018090716353116.png)

# Spring新注解(熟悉)

## `@Configuration`注解(熟悉)

    作用：
        用于指定当前类是一个spring配置类，当创建容器时会从该类上加载注解。获取容器时需要使用AnnotationApplicationContext(有@Configuration注解的类.class)。它的作用和bean.xml是一样的。当配置类作为AnnotationConfigApplicationContext对象创建的参数时，该注解可以不写。

    属性：
        value:用于指定配置类的字节码

## `@ComponentScan`注解(熟悉)

    作用：
        用于指定spring在初始化容器时要扫描的包。作用和在spring的xml配置文件中的: `<context:component-scan base-package="com.itheima"/>`是一样的。

    属性：
        basePackages：用于指定要扫描的包。和该注解中的value属性作用一样。


![](assets/markdown-img-paste-2018090416191993.png)

![](assets/markdown-img-paste-20180904163813687.png)

## `@Bean`注解(熟悉)

    `@Bean`:用于把当前方法的返回值作为bean对象存入spring的ioc容器中,他的name属性用于指定bean的id。当不写时，默认值是当前方法的名称

![](assets/markdown-img-paste-20180904163625920.png)

![](assets/markdown-img-paste-2018090416382685.png)

## `@Import`注解(熟悉)

    @Import用于导入其他的配置类

### 创建一个配置类(熟悉)

![](assets/markdown-img-paste-20180904164056540.png)    

### 引入配置类(熟悉)
![](assets/markdown-img-paste-20180904164124280.png)

### 测试代码(熟悉)
![](assets/markdown-img-paste-2018090416382685.png)

## `@PropertySource`注解(熟悉)

    作用:
        `@PropertySource`用于加载.properties文件中的配置。例如我们配置数据源时，可以把连接数据库的信息写到properties配置文件中，就可以使用此注解指定properties配置文件的位置。
    属性：
        value[]：用于指定properties文件位置。如果是在类路径下，需要写上classpath

### 创建`jdbc.properties`文件(熟悉)
```
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/spring_02
jdbc.username=root
jdbc.password=zl
```

### 使用`@PropertySource`指定配置文件(熟悉)
![](assets/markdown-img-paste-20180904170243943.png)


### 从配置类中读取配置文件(熟悉)
```java
@Configuration
@ComponentScan("com.itheima")
@PropertySource("classpath:db.properties")
public class BeanFactory {

    @Value("${jdbc.driver}")
    private String driver ;
    @Value("${jdbc.url}")
    private String url ;
    @Value("${jdbc.username}")
    private String username ;
    @Value("${jdbc.password}")
    private String password ;

    @Bean(name="queryRunner")
    public QueryRunner getQueryRunner(DataSource dataSource){
        return  new QueryRunner(dataSource);
    }

    @Bean(name="dataSource")
    public DataSource getDataSource() throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setDriverClass(driver);
        dataSource.setJdbcUrl(url);
        dataSource.setUser(username);
        dataSource.setPassword(password);
        return dataSource;
    }
}
```

## `@Qualifier`注解(熟悉)

    作用:
        用来在形参中为参数取别名
    属性:
        value:参数的别名

![](assets/markdown-img-paste-20180907164229470.png)

## 使用Spring新注解,替换掉原来的`beans.xml`配置文件(熟悉)
### 创建配置文件`db.properties`(熟悉)
```
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/spring_02
jdbc.username=root
jdbc.password=zl
```

### 创建配置类`SpringConfig`(熟悉)
```java
@Configuration
@ComponentScan("com.itheima")
@PropertySource("classpath:db.properties")
public class SpringConfig {

    @Value("${jdbc.driver}")
    private String driver ;
    @Value("${jdbc.url}")
    private String url ;
    @Value("${jdbc.username}")
    private String username ;
    @Value("${jdbc.password}")
    private String password ;

    @Bean(name="queryRunner")
    public QueryRunner getQueryRunner(@Qualifier("ds") DataSource dataSource){
        return  new QueryRunner(dataSource);
    }

    @Bean(name="ds")
    public DataSource getDataSource() throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setDriverClass(driver);
        dataSource.setJdbcUrl(url);
        dataSource.setUser(username);
        dataSource.setPassword(password);
        return dataSource;
    }
}
```
### 修改测试类获取Spring容器操作(熟悉)
![](assets/markdown-img-paste-20180904173702103.png)

# Spring整合Junit(熟悉)

## 导入Spring与Junit整合的依赖包(熟悉)
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.0.2.RELEASE</version>
</dependency>
```

## 在单元测试类上添加注解(熟悉)
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConfig.class)
public class UserServiceTest {

    private UserService userService ;

    @Test
    public void findAll() {
        List<User> users = userService.findAll();
        System.out.println(users);
    }
}
```

    注意:

      1. 如果使用注解配置,  `@ContextConfiguration` 应该指定配置类的字节码
      @ContextConfiguration(classes = SpringConfig.class)

      2. 如果使用配置文件,`@ContextConfiguration` 应该指定配置文件的位置
      @ContextConfiguration(locations = "classpath:beans.xml")

## 给测试类中的变量注入数据(熟悉)
![](assets/markdown-img-paste-20180904172520210.png)
