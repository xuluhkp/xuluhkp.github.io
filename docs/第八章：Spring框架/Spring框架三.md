# 今日内容介绍

<extoc></extoc>

# 代码存在的问题与动态代理介绍(理解)

之前我们写的代码,如果一个业务中包含多次的增删改操作,当我们执行时,如果执行过程中出现了异常,则会引发一系列的问题.

例如:我们现在完成一个转账的功能,一个转账功能包含多个数据库的操作,例如:

    1.根据名称查询转出账户
    2.根据名称查询转入账户
    3.转出账户减钱
    4.转入账户加钱
    5.更新转出账户余额
    6.更新转入账户余额

如果在执行过程中程序出现了异常,那么可能会出现一些我们不希望看到的结果,下面通过代码演示问题

## 准备(理解)
### 创建数据库(理解)
```sql
CREATE TABLE `account` (
  `id` int(11) PRIMARY KEY auto_increment COMMENT '编号',
  `username` varchar(20) default NULL COMMENT '用户编号',
  `money` double default NULL COMMENT '金额'
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert  into `account`(id, `username`, money) values   (1, 'zhangsan', 1000),(2, 'lisi', 1000);
```

### 创建maven项目(理解)
![](assets/markdown-img-paste-20180907171941877.png)

### 引入maven依赖(理解)
```xml
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
        <artifactId>spring-test</artifactId>
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

        <!--加载src目录下的静态文件-->
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
        </resources>
</build>
```

### 配置Spring容器(理解)
#### 引入数据库配置文件`db.properties`
```
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/spring_02
jdbc.username=root
jdbc.password=zl
```

#### 使用spring注解配置容器
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

    @Bean(name="dataSource")
    public DataSource getDataSource() throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setDriverClass(driver);
        dataSource.setJdbcUrl(url);
        dataSource.setUser(username);
        dataSource.setPassword(password);
        return dataSource;
    }

    @Bean("queryRunner")
    public QueryRunner getQueryRunner(DataSource dataSource){
        return new QueryRunner(dataSource);
    }
}
```


## 转账业务的代码实现(理解)
### 编写转账的数据访问层接口和实现类(理解)
#### 接口`AccountService`
```java
public interface AccountDao {

    /**
     * 根据用户名称查询用户信息
     * @param name
     * @return
     */
    Account findByName(String name);

    /**
     * 更新账户信息
     * @param source
     */
    void update(Account source);
}
```
#### 实现类`AccountServiceImpl`
```java
@Repository("accountDao")
public class AccountDaoImpl implements AccountDao {

    @Autowired
    private QueryRunner queryRunner ;

    @Override
    public Account findByName(String name) throws SQLException {
        String sql = "select * from account where username = ? ";
        return queryRunner.query(sql,new BeanHandler<Account>(Account.class),name);
    }

    @Override
    public void update(Account account) throws SQLException {
        String sql = "update account set money = ? where username = ? ";
        queryRunner.update(sql,account.getMoney(),account.getUsername());
    }
}
```

### 编写转账功能的业务层接口和实现类(理解)
#### 接口
```java
public interface AccountService {

    /**
     * 转账业务功能
     * @param sourceName  转出账户
     * @param targetName  转入账户
     * @param money         转账金额
     * @throws SQLException
     */
    public void transfer(String sourceName,String targetName,double money) throws SQLException;
}
```
#### 实现类
```java
@Service("accountService")
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao ;

    @Override
    public void transfer(String sourceName, String targetName, double money) throws SQLException {
        //1.根据名称查询转出账户
        Account source = accountDao.findByName(sourceName);
        //2.根据名称查询转入账户
        Account target = accountDao.findByName(sourceName);
        //3.转出账户减钱
        source.setMoney(source.getMoney()-money);
        //4.转入账户加钱
        source.setMoney(source.getMoney()+money);
        //5.更新转出账户余额
        accountDao.update(source);
        //6.更新转入账户余额
        accountDao.update(target);
    }
}
```

### 编写测试类(理解)
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConfig.class)
public class AccountServiceTest {

    @Autowired
    private AccountService accountService ;

    @Test
    public void transfer() throws SQLException {
        //zhangsan给lisi转账1000元
        accountService.transfer("zhangsan","lisi",1000);
    }
}
```

现在执行测试没有任何问题,`zhangsan`的账户扣了`1000`元`lisi`的账户加了`1000`元

![](assets/markdown-img-paste-20180907174724477.png)

![](assets/markdown-img-paste-20180907174804343.png)

但是如果我们的业务执行过程中出现了异常,就会出现一些问题

![](assets/markdown-img-paste-20180907175239686.png)

![](assets/markdown-img-paste-2018090717494938.png)

`zhangsan`账户钱扣了,`lisi`钱却没有加（不符合事务的一致性） ,转账失败.因为我们是每次执行持久层方法都是独立事务,导致无法实现事务控制


## JDBC事物控制(理解)

此转帐操作同时使用了四个dao方法,也就是说连接了四次数据库,产生了四个数据库连接对象,而每个连接对象都是有独立的事务的,

为了保证操作数据的一致性我们需要将事务来进行统一的管理.也就是说不管这个功能由几个操作组成,都只有一个事务

### 创建JDBC工具类(理解)
```java
/**
 * JDBC工具类,提供获取连接的操作,连接使用ThreadLocal与线程绑定
 */
@Component
public class JdbcUtils {

    private  ThreadLocal<Connection> tl = new ThreadLocal<Connection>(); //Map

    //数据源,使用C3P0连接池
    @Autowired
    private   DataSource dataSource ;

    /**
     * 获取连接池的方法
     * @return
     */
    public DataSource getDataSource(){
        return dataSource ;
    }

    /**
     * 获取当前线程上的连接
     * @return
     */
    public  Connection getThreadConnection() {
        try{
            //1.先从ThreadLocal上获取
            Connection conn = tl.get();
            //2.判断当前线程上是否有连接
            if (conn == null) {
                //3.从数据源中获取一个连接,并且存入ThreadLocal中
                conn = dataSource.getConnection();
                tl.set(conn);
            }
            //4.返回当前线程上的连接
            return conn;
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

    /**
     * 把连接和线程解绑
     */
    public  void release(){
        tl.remove();
    }

}
```

### 创建JDBC事物管理器(理解)
```java
/**
 * JDBC事物管理器
 */
@Component
public class TransactionManager {

    @Autowired
    private JdbcUtils jdbcUtils ;

    /**
     * 开启事务
     */
    public  void beginTransaction() {
        try {
            jdbcUtils.getThreadConnection().setAutoCommit(false);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 提交事务
     */
    public  void commit() {
        try {
            jdbcUtils.getThreadConnection().commit();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 回滚事务
     */
    public  void rollback() {
        try {
            jdbcUtils.getThreadConnection().rollback();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 释放连接
     */
    public  void release() {
        try {
            jdbcUtils.getThreadConnection().close();//还回连接池中
            jdbcUtils.release();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```

### 在我们的代码中添加事物管理操作(理解)
#### 业务层进行事物控制`AccountServiceImpl`
```java
@Service("accountService")
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountDao accountDao ;

    @Autowired
    private TransactionManager transactionManager ;

    @Override
    public void transfer(String sourceName, String targetName, double money) throws SQLException {
        try {
            //开启事物-----第一个数据库操作之前开启事物
            transactionManager.beginTransaction();

            //1.根据名称查询转出账户
            Account source = accountDao.findByName(sourceName);
            //2.根据名称查询转入账户
            Account target = accountDao.findByName(targetName);
            //3.转出账户减钱
            source.setMoney(source.getMoney()-money);
            //4.转入账户加钱
            target.setMoney(target.getMoney()+money);
            //5.更新转出账户余额
            accountDao.update(source);
            int i = 10 / 0 ; //特意添加的异常代码,让程序执行异常
            //6.更新转入账户余额
            accountDao.update(target);

            //提交事物------所有的数据库操作执行成功之后提交事务
            transactionManager.commit();
        } catch (SQLException e) {
            e.printStackTrace();
            //回滚事物 ----业务代码出现了异常,回滚事物
            transactionManager.rollback();
        }finally {
            //释放资源------数据库操作执行完毕时候释放资源
            transactionManager.release();
        }
    }
}
```
#### 数据访问层,使用传入连接的方法
```java
@Repository("accountDao")
public class AccountDaoImpl implements AccountDao {

    @Autowired
    private QueryRunner queryRunner ;

    @Autowired
    private JdbcUtils jdbcUtils ;

    @Override
    public Account findByName(String name) throws SQLException {
        String sql = "select * from account where username = ? ";
        return queryRunner.query(jdbcUtils.getThreadConnection(),sql,new BeanHandler<Account>(Account.class),name);
    }

    @Override
    public void update(Account account) throws SQLException {
        String sql = "update account set money = ? where username = ? ";
        queryRunner.update(jdbcUtils.getThreadConnection(),sql,account.getMoney(),account.getUsername());
    }
}
```

### 重新完成测试(理解)

![](assets/markdown-img-paste-20180912111748111.png)

通过对业务层改造,我们发现已经可以实现事务控制了,但是由于我们添加了事务控制,也产生了一个新的问题:业务层方法变得臃肿了,里面充斥着很多重复代码.并且业务层方法和事务控制方法耦合了.试想一下,如果我们此时提交,回滚,释放资源中任何一个方法名变更,都需要修改业务层的代码,况且这还只是一个业务层实现类,而实际的项目中这种业务层实现类可能有十几个甚至几十个.如何解决这类问题？

![](assets/markdown-img-paste-20180912112923429.png)

我们可以在不修改源代码的情况下,对业务层的代码进行统一的增强,这个时候我们想到了之前学过的动态代理

![](assets/markdown-img-paste-20180912113445267.png)

## 使用动态代理解决业务层的事物问题(理解)

动态代理特点: 字节码随用随创建,随用随加载,不修改源码的基础上对方法增强.

动态代理分为两类: 基于接口的动态代理(JDK)、基于子类的动态代理(CGLIB)

### JDK动态代理(熟悉)

#### 接口
```java
/**
 * 对生产厂家要求的接口
 */
public interface IProducer {
    /**
     * 销售
     * @param money
     */
    public void saleProduct(float money);

    /**
     * 售后
     * @param money
     */
    public void afterService(float money);
}
```

#### 实现类(真实对象)
```java
public class ProducerImpl implements  IProducer {
    /**
     * 销售
     * @param money
     */
    @Override
    public void saleProduct(float money) {
         System.out.println("销售产品,并拿到钱:"+money);
    }

    /**
     * 售后
     * @param money
     */
    @Override
    public void afterService(float money) {
        System.out.println("提供售后服务,并拿到钱:"+money);
    }
}
```

#### 使用JDK动态代理创建代理对象
```java
@Test
public void producerTest() {
    final ProducerImpl producer = new ProducerImpl();
    /**
     * newProxyInstance方法参数
     *参数1: ClassLoader,类加载器它是用于加载代理对象字节码的.和被代理对象使用相同的类加载器.固定写法.
     *参数2: Class[],字节码数组它是用于让代理对象和被代理对象有相同方法.固定写法
     *参数3: InvocationHandler,用于提供增强的代码,它是让我们写如何代理.
     *       我们一般都是些一个该接口的实现类,通常情况下都是匿名内部类,但不是必须的.
     */
    IProducer proxyProducer = (IProducer) Proxy.newProxyInstance(producer.getClass().getClassLoader(),producer.getClass().getInterfaces(), new InvocationHandler() {
          /**
           * 执行被代理对象的任何接口方法都会经过该方法
           *
           * @param proxy  代理对象的引用
           * @param method 当前执行的方法
           * @param args   当前执行方法所需的参数
           * @return 和被代理对象方法有相同的返回值
           * @throws Throwable
           */
          @Override
          public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
              //提供增强的代码
              Object object = null;
              //获取方法执行的参数
              Float money = (Float) args[0];//表示获取第一个参数
              //判断当前方法是不是销售方法,"saleProduct"为销售方法名称
              if ("saleProduct".equals(method.getName())) {
                  //是销售方法执行提成操作,代理商提取20%的利润
                  object = method.invoke(producer,money * 0.8f);
              }
              return object;
          }
    });
    //调用销售方法
    proxyProducer.saleProduct(1000f);
}
```

**注意:Jdk实现的动态代理,只能给有接口的类做代理**

### CGLIB动态代理(熟悉)

创建一个类,不用实现任何接口,也可以实现接口,cglib动态代理,有没有接口都可以做代理.
基于子类的动态代理使用Enhancer类中的create方法创建代理对象

#### 导入CGLIB的相关jar包
```xml
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.2.2</version>
</dependency>
```
#### 创建被代理类(真实对象)
```java
public class Producer {

    /**
     * 销售
     * @param money
     */
    public void saleProduct(float money) {
        System.out.println("销售产品,并拿到钱:"+money);
    }

    /**
     * 售后
     * @param money
     */
    public void afterService(float money) {
        System.out.println("提供售后服务,并拿到钱:"+money);
    }
}
```

#### 使用CGLIB动态代理创建代理对象
```java
public class CglibProxyTest {
    @Test
    public void client(){
        final Producer producer=new Producer();
        /**
         * create方法的参数
         * 参数1:Class,字节码它是用于指定被代理对象的字节码
         *参数2:Callback,用于提供增强的代码它是让我们写如何代理.
         *      我们一般都是些一个该接口的实现类,通常情况下都是匿名内部类,但不是必须的.
         *我们一般写的都是该接口的子接口实现类:MethodInterceptor
         */
        Producer cglibProducer= (Producer) Enhancer.create(producer.getClass(), new MethodInterceptor(){
            /**
             *执行被代理对象的任何接口方法都会经过该方法
             * @param proxy  代理对象的引用
             * @param method 当前执行的方法
             * @param args   当前执行方法所需的参数
             * @param methodProxy 当前执行方法的代理对象
             * @return
             * @throws Throwable
             */
            @Override
            public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
                //提供增强的代码
                Object returnValue = null;

                //1.获取方法执行的参数
                Float money = (Float)args[0];
                //2.判断当前方法是不是销售
                if("saleProduct".equals(method.getName())) {
                    returnValue = method.invoke(producer, money*0.8f);
                }
                return returnValue;
            }
        });

        cglibProducer.saleProduct(12000f);
    }
}
```

### 使用动态代理解决事物的问题(熟悉)
#### 编写创建代理对象的工厂

```java
@Component
public class BeanFactory {

    @Autowired
    private TransactionManager transactionManager ;

    @Bean("transaction_accountService")
    public AccountService getAccountService(@Qualifier("accountService") AccountService accountService){

        ClassLoader loader = accountService.getClass().getClassLoader();

        Class[] interfaces = accountService.getClass().getInterfaces();

        AccountService proxy = (AccountService) Proxy.newProxyInstance(loader,  interfaces, new InvocationHandler() {
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    try {
                        //开启事务
                        transactionManager.beginTransaction();
                        //执行操作
                        Object object = method.invoke(accountService, args);
                        //提交事务
                        transactionManager.commit();
                        //返回结果
                        return object;
                    } catch (Exception e) {
                        //回滚事务
                        transactionManager.rollback();
                        throw new RuntimeException(e);
                    } finally {
                        //释放连接
                        transactionManager.release();
                    }
                }
            });

        return proxy;
    }
}
```

#### 修改业务层方法,删除事物控制代码
```java
@Service("accountService")
public class AccountServiceImpl2 implements AccountService {

    @Autowired
    private AccountDao accountDao ;

    @Override
    public void transfer(String sourceName, String targetName, double money) throws SQLException {

        //1.根据名称查询转出账户
        Account source = accountDao.findByName(sourceName);
        //2.根据名称查询转入账户
        Account target = accountDao.findByName(targetName);
        //3.转出账户减钱
        source.setMoney(source.getMoney()-money);
        //4.转入账户加钱
        target.setMoney(target.getMoney()+money);
        //5.更新转出账户余额
        accountDao.update(source);
        int i = 10 / 0 ; //特意添加的异常代码,让程序执行异常
        //6.更新转入账户余额
        accountDao.update(target);
    }
}
```
#### 测试方法,使用代理对象完成转账操作
![](assets/markdown-img-paste-20180912142533990.png)

# APO的相关概念(熟悉)

## APO概述(熟悉)
![](assets/markdown-img-paste-20180912144046970.png)

    AOP:全称是 Aspect Oriented Programming 即:面向切面编程.

    简单的说它就是把我们程序重复的代码抽取出来,在需要执行的时候,使用动态代理的技术,在不修改源码的基础上,对我们的已有方法进行增强.

    Aop的作用:在程序运行期间,不修改源码对已有方法进行增强.

    Aop优势:减少重复代码、提高开发效率、维护方便

    AOP的实现方式:使用动态代理技术

## AOP的相关术语(熟悉)
    1. Joinpoint( 连接点):所谓连接点是指那些被拦截到的点,在spring中这些点指的是方法,因为spring只支持方法类型的连接点

    2. Pointcut( 切入点):所谓切入点 表示一组 joint point,这些 joint point 或是通过逻辑关系组合起来,或是通过通配、正则表达式等方式集中起来,它定义了相应的 Advice 将要发生的地方.简单说切入点是指我们要对哪些连接点进行拦截的定义

    3. Advice( 通知/ 增强):所谓通知是指拦截到 Joinpoint 之后所要做的事情就是通知.通知的类型:前置通知,后置通知,异常通知,最终通知,环绕通知.

    4. Aspect( 切面):是切入点和通知（引介）的结合.

    5. Introduction( 引介):引介是一种特殊的通知在不修改类代码的前提下, Introduction 可以在运行期为类动态地添加一些方法或 Field.

    6. Target( 目标对象):代理的目标对象.

    7. Proxy（代理）:一个类被 AOP 织入增强后,就产生一个结果代理类.

    8. Weaving( 织入):是指把增强应用到目标对象来创建新的代理对象的过程.spring 采用动态代理织入,而 AspectJ 采用编译期织入和类装载期织入.

## 学习AOP需要明确的事情(熟悉)

1. 开发阶段（我们做的）

    编写核心业务代码（开发主线）:大部分程序员来做,要求熟悉业务需求.
    把公用代码抽取出来,制作成通知.（开发阶段最后再做）
    在配置文件中,声明切入点与通知间的关系,即切面.

2. 运行阶段（Spring  框架完成的）
    Spring 框架监控切入点方法的执行.一旦监控到切入点方法被运行,使用代理机制,动态创建目标对象的代理对象,根据通知类别,在代理对象的对应位置,将通知对应的功能织入,完成完整的代码逻辑运行.



# Spring中的AOP(熟悉)

    在spring中,框架会根据目标类是否实现了接口来决定采用哪种动态代理的方式.

## SpringAOP环境准备(熟悉)
### 创建Maven项目(熟悉)
![](assets/markdown-img-paste-20180912145948824.png)

### 导入依赖jar包(熟悉)
```xml
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
            <artifactId>spring-test</artifactId>
            <version>5.0.2.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>cglib</groupId>
            <artifactId>cglib</artifactId>
            <version>3.2.2</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.8.7</version>
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

### 创建业务层接口及实现类(熟悉)
#### 接口
```java
public interface AccountService {

    /**
     * 模拟保存帐户
     */
    void saveAccount();

    /**
     * 模似拟修改帐户
     */
    void updateAccount(int i);

    /**
     * 模拟删除帐户
     */
    int delAccount();

}
```

#### 实现类
```java
@Service("accountService")
public class AccountServiceImpl implements AccountService {
    /**
     * 模拟保存
     */
    @Override
    public void saveAccount() {
        System.out.println("执行了保存");
    }

    /**
     * 模拟修改
     */
    @Override
    public void updateAccount(int i) {
        System.out.println("执行了修改"+i);
    }

    /**
     * 模拟删除
     */
    @Override
    public int delAccount() {
        System.out.println("执行了删除");
        return 0;
    }
}
```

### 编写Spring核心配置文件(熟悉)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.itheima"></context:component-scan>

</beans>
```

## 需求(熟悉)

我们现在要在所有的业务层功能执行时插入日志记录功能

## 基于XML的AOP配置(熟悉)

### 抽取公用功能(熟悉)
```java
/**
 * 日志工具类
 */
@Component
public class Logger {

    /**
     * 记录日志,在执行切入点方法之前执行
     */
    public void printLog() {
        System.out.println("Logger类中的printLog方法开始记录日志....");
    }
}
```

### 引入AOP名称空间(熟悉)
![](assets/markdown-img-paste-20180912151337617.png)

### 配置AOP完成功能(熟悉)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd">


    <!--配置目标 target-->
    <bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl"></bean>

    <!--配置通知-->
    <bean id="logger" class="com.itheima.utils.Logger"></bean>

    <!--配置Aop-->
    <aop:config>
        <!--配置Aop切面-->
        <aop:aspect id="logAdvice" ref="logger">
            <!--配置通知类型,并建立通知方法和切入点方法的关联-->
            <aop:before method="printLog" pointcut="execution(* com.itheima.service..*.*(..))"></aop:before>
        </aop:aspect>
    </aop:config>

</beans>
```

### 编写测试代码(熟悉)
```java
public class AccountServiceTest {

    @Test
    public void saveAccount() {
        ApplicationContext ApplicationContext = new ClassPathXmlApplicationContext("beans.xml");
        AccountService accountService = (AccountService) ApplicationContext.getBean("accountService");

        accountService.saveAccount();
    }

    @Test
    public void updateAccount() {

        ApplicationContext ApplicationContext = new ClassPathXmlApplicationContext("beans.xml");
        AccountService accountService = (AccountService) ApplicationContext.getBean("accountService");

        accountService.updateAccount(1);
    }

    @Test
    public void delAccount() {
        ApplicationContext ApplicationContext = new ClassPathXmlApplicationContext("beans.xml");
        AccountService accountService = (AccountService) ApplicationContext.getBean("accountService");

        int i = accountService.delAccount();
        System.out.println(i);
    }
}
```

## 总结(熟悉)

### Spring中基于XML的AOP配置步骤(熟悉)
    1. 把通知Bean也交给spring来管理

    2. 使用aop:config标签表明开始AOP的配置

    3. 使用aop:aspect标签表明配置切面
        id属性:是给切面提供一个唯一标识
        ref属性:是指定通知类bean的Id.

    4. 在aop:aspect标签的内部使用对应标签来配置通知的类型
        我们现在示例是让printLog方法在切入点方法执行之前之前:所以是前置通知
            aop:before:表示配置前置通知
            method属性:用于指定Logger类中哪个方法是前置通知
            pointcut属性:用于指定切入点表达式,该表达式的含义指的是对业务层中哪些方法增强

### 切入点表达式的写法(熟悉)
```
execution:匹配方法的执行(常用)

execution(表达式)
    表达式语法:execution([修饰符] 返回值类型 包名.类名.方法名(参数)) 写法说明:

全匹配方式:
    public void com.itheima.service.impl.AccountServiceImpl.saveAccount(com.itheima.domain.Account) 访问修饰符可以省略
           void com.itheima.service.impl.AccountServiceImpl.saveAccount(com.itheima.domain.Account)

返回值可以使用*号,表示任意返回值
    * com.itheima.service.impl.AccountServiceImpl.saveAccount(com.itheima.domain.Account)

包名可以使用*号,表示任意包,但是有几级包,需要写几个*
    * *.*.*.*.AccountServiceImpl.saveAccount(com.itheima.domain.Account)

使用..来表示当前包,及其子包
    * com..AccountServiceImpl.saveAccount(com.itheima.domain.Account)

类名可以使用*号,表示任意类
    * com..*.saveAccount(com.itheima.domain.Account)

方法名可以使用*号,表示任意方法
    * com..*.*( com.itheima.domain.Account)

参数列表可以使用*,表示参数可以是任意数据类型,但是必须有参数
    * com..*.*(*)

参数列表可以使用..表示有无参数均可,有参数可以是任意类型
    * com..*.*(..)

全通配方式:
    * *..*.*(..)


注: 通常情况下,我们都是对业务层的方法进行增强,所以切入点表达式都是切到业务层实现类.
    execution(* com.itheima.service.impl.*.*(..))


例如:
    Public * add(com.itheima.User):*表示匹配所有类型的返回值
    Public void *(com.itheima.User):*表示匹配所有方法名
    Public void add(..):表示匹配所有参数个数和类型
    com.itheima.service.*.*(..):表示匹配com.service包下所有类的所有方法
    com.itheima.service..*.*(..):表示匹配com.service包及其子包下所有类的所有方法
```

### 通知的类型(熟悉)

Spring中通知分为: 前置通知,后置通知,异通知常,最终通知

```xml
<!--配置Aop-->
<!--配置Aop-->
<aop:config>
    <!--配置Aop切面-->
    <aop:aspect id="logAdvice" ref="logger">
        <!--配置切入点-->
        <!--配置通知类型,并建立通知方法和切入点方法的关联-->
        <!--前置通知:目标行为执行之前执行-->
        <aop:before method="printLog" pointcut="execution(* com.itheima.service..*.*(..))"></aop:before>

        <!--后置通知:目标行为执行之后执行-->
        <aop:after-returning method="printLog" pointcut="execution(* com.itheima.service..*.*(..))"></aop:after-returning>

        <!--异常通知:目标行为只有抛出了异常后才会执行这个增强方法-->
        <aop:after-throwing method="printLog" pointcut="execution(* com.itheima.service..*.*(..))"></aop:after-throwing>

        <!--最终通知:无论是否有异常,最终通知都会执行.-->
        <aop:after method="printLog"  pointcut="execution(* com.itheima.service..*.*(..))"></aop:after>

        <!--环绕通知:目标行为执行前后执行-->
        <aop:around method="printLog" pointcut="execution(* com.itheima.service..*.*(..))"></aop:around>

    </aop:aspect>
</aop:config>
```

### 通用的切入点表达式配置(熟悉)

每次配置通知类型时Pointcut中的表达式都是一样的,造成重复编写,能不能配置一个共用的直接调用呢？

1. 写在<aop:aspect>内:只能当前切面使用
![](assets/markdown-img-paste-20180912160421910.png)

2. 写在<aop:aspect>外:所有切面可用
![](assets/markdown-img-paste-20180912160325178.png)

### Spring中的环绕通知(熟悉)

Spring中的环绕通知是spring框架为我们提供的一种可以在代码中手动控制增强方法何时执行的方式.

#### 问题
    当我们配置了环绕通知之后,切入点方法没有执行,而通知方法执行了.

#### 解决
    Spring框架为我们提供了一个接口:ProceedingJoinPoint.该接口有一个方法proceed(),此方法就相当于明确调用切入点方法.

    该接口可以作为环绕通知的方法参数,在程序执行时,spring框架会为我们提供该接口的实现类供我们使用.

```java
public Object aroundPrintLog(ProceedingJoinPoint pjp) {

    Object rtValue=null;
    try{
        //获得方法执行所需参数
        Object[] args=pjp.getArgs();

        System.out.println("logger类中的aroundPrintLog方法开始记录日志===>前置");

        //明确调用业务层方法
        rtValue=pjp.proceed(args);

        System.out.println("logger类中的aroundPrintLog方法开始记录日志===>后置");
    }catch (Throwable throwable){
        System.out.println("logger类中的aroundPrintLog方法开始记录日志===>异常");
    } finally {
        System.out.println("logger类中的aroundPrintLog方法开始记录日志===>最终");
    }
    return rtValue;
}
```

## 基于注解的AOP配置(熟悉)
### 创建maven项目(熟悉)
![](assets/markdown-img-paste-20180912163221686.png)

### 引入依赖包(熟悉)
```xml
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
      <artifactId>spring-test</artifactId>
      <version>5.0.2.RELEASE</version>
  </dependency>
  <dependency>
      <groupId>cglib</groupId>
      <artifactId>cglib</artifactId>
      <version>3.2.2</version>
  </dependency>
  <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.8.7</version>
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

### 引入业务层接口及实现类(熟悉)
![](assets/markdown-img-paste-20180912163549505.png)

### 使用注解配置切面(熟悉)
```java
package com.itheima.utils;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

/**
 * 日志工具类
 */
@Component
@Aspect //表名此类是一个切面类
public class Logger {

    //定义切入点,方法名称就是切入点id
    @Pointcut("execution(* com.itheima.service..*.*(..))")
    public void logPointCut(){};

    /**
     * 记录日志,在执行切入点方法之前执行
     */
    @Before("logPointCut()")
    public void printLog() {
        System.out.println("Logger类中的printLog方法开始记录日志....");
    }


    /**
     * 后置通知
     */
    @AfterReturning("logPointCut()")
    public void afterReturningPrintLog(){
        System.out.println("后置通知Logger类中的afterReturningPrintLog方法开始记录日志=========");
    }

    /**
     * 异常通知
     */
    @AfterThrowing("logPointCut()")
    public void afterThrowingPrintLog(){
        System.out.println("异常通知Logger类中的afterThrowingPrintLog方法开始记录日志++++++++++++");
    }

    /**
     * 最终通知
     */
    @After("logPointCut()")
    public void afterPrintLog(){
        System.out.println("最终通知Logger类中的afterPrintLog方法开始记录日志&&&&&&&&&&&&&&&&&&");
    }

    @Around("logPointCut()")
    public Object aroundPrintLog(ProceedingJoinPoint pjp) {
        Object rtValue=null;
        try{
            //获得方法执行所需参数
            Object[] args=pjp.getArgs();
            System.out.println("logger类中的aroundPrintLog方法开始记录日志===>前置");
            //明确调用业务层方法
            rtValue=pjp.proceed(args);
            System.out.println("logger类中的aroundPrintLog方法开始记录日志===>后置");
        }catch (Throwable t){
            System.out.println("logger类中的aroundPrintLog方法开始记录日志===>异常");
        } finally {
            System.out.println("logger类中的aroundPrintLog方法开始记录日志===>最终");
        }
        return rtValue;
    }
}
```

### 编写Spring配置文件(熟悉)
![](assets/markdown-img-paste-20180912163837468.png)

### 编写测试类完成测试(熟悉)
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = "classpath:beans.xml")
public class AccountServiceTest {

    @Autowired
    private AccountService accountService ;

    @Test
    public void saveAccount() {
        accountService.saveAccount();
    }

    @Test
    public void updateAccount() {
        accountService.updateAccount(1);
    }

    @Test
    public void delAccount() {
        int i = accountService.delAccount();
    }
}
```
