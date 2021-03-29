# Spring
Spring是一个轻量级的控制反转（IOC）和面向切面编程（AOP）的框架。本文主要针对IOC和AOP展开学习。
###### Spring工作方式
> 个人理解：
Spring是一个封装了很多功能（比如IOC、AOP）的框架，通过读取配置文件（默认叫applicationContext.xml）控制Spring完成相关操作。
对于开发者来说只需要按照Spring的规范去编写配置文件。所以对于一个应用来说，Spring帮助把一些逻辑从业务代码中抽离到配置文件，有些场景下只需要修改配置文件就可以完成需求，而不用直接去修改业务代码，使得应用更方便维护。
**特点：**
* 非侵入式，指的是引入Spring不会侵入到业务代码，不用继承框架提供的类，而是通过配置完成依赖注入后，就可以使用
* 轻量级
* 控制反转（IOC）
* 面向切面编程（AOP）
* 支持事务

## 核心模块
spring七大核心模块：

**核心容器（Spring Core）**
核心容器提供Spring框架的基本功能。Spring以bean的方式组织和管理Java应用中的各个组件及其关系。Spring使用BeanFactory来产生和管理Bean，它是工厂模式的实现。BeanFactory使用控制反转(IoC)模式将应用的配置和依赖性规范与实际的应用程序代码分开。

**应用上下文（Spring Context）**
Spring上下文是一个配置文件，向Spring框架提供上下文信息。Spring上下文包括企业服务，如JNDI、EJB、电子邮件、国际化、校验和调度功能。

**Spring面向切面编程（Spring AOP）**
通过配置管理特性，Spring AOP 模块直接将面向方面的编程功能集成到了 Spring框架中。所以，可以很容易地使 Spring框架管理的任何对象支持 AOP。Spring AOP 模块为基于 Spring 的应用程序中的对象提供了事务管理服务。通过使用 Spring AOP，不用依赖 EJB 组件，就可以将声明性事务管理集成到应用程序中。

**JDBC和DAO模块（Spring DAO）**
JDBC、DAO的抽象层提供了有意义的异常层次结构，可用该结构来管理异常处理，和不同数据库供应商所抛出的错误信息。异常层次结构简化了错误处理，并且极大的降低了需要编写的代码数量，比如打开和关闭链接。

**对象实体映射（Spring ORM）**
Spring框架插入了若干个ORM框架，从而提供了ORM对象的关系工具，其中包括了Hibernate、JDO和 IBatis SQL Map等，所有这些都遵从Spring的通用事物和DAO异常层次结构。

**Web模块（Spring Web）**
Web上下文模块建立在应用程序上下文模块之上，为基于web的应用程序提供了上下文。所以Spring框架支持与Struts集成，web模块还简化了处理多部分请求以及将请求参数绑定到域对象的工作。

**MVC模块（Spring Web MVC）**
MVC框架是一个全功能的构建Web应用程序的MVC实现。通过策略接口，MVC框架变成为高度可配置的。MVC容纳了大量视图技术，其中包括JSP、POI等，模型来有JavaBean来构成，存放于m当中，而视图是一个街口，负责实现模型，控制器表示逻辑代码，由c的事情。Spring框架的功能可以用在任何J2EE服务器当中，大多数功能也适用于不受管理的环境。Spring的核心要点就是支持不绑定到特定J2EE服务的可重用业务和数据的访问的对象，毫无疑问这样的对象可以在不同的J2EE环境，独立应用程序和测试环境之间重用。

### IOC控制反转
所谓IOC就是把创建示例对象（new 一个类）这件事交给Spring去做，然后并把创建好的对象放到Spring容器，使用的时候从Spring容器中取。
IOC是一种编程思想，目的是为了降低程序耦合。

##### 认识bean标签
applicationContext.xml配置文件中的\<bean>标签是用来告诉哪些类需要交给Spring管理，相当于是一个注册（装配）过程。
一个正常的\<bean\>标签大致长这个样子：
```
<beans>
  <bean id="mybean" class="某个类">
    <!-- 依赖注入 -->
    ...
  </bean>
</beans>
```
### 依赖注入（DI）
通常情况创建对象（new 一个类）希望初始化实例数据，主要通过构造方法和set方法两种方式初始化。
```
// User.java
public class User() {
  private String name;
  public User(name) {
    this.name = name
  }
  public String setName(name) {
    this.name = name;
  }
}

// Main.java
public static void main(String[] args) {
  User user = new User("张三");
  user.setName("李四");
}
```
已知IOC是Spring也是帮我们创建实例对象的，那么他支持这两种方式初始化实例对象数据（以及更常用的注解方式），初始化实例对象数据的过程也就是所说的依赖注入(DI)：
#### 一、使用构造器注入
* 使用无参构造函数创建对象（默认）
    ```
    <bean id="xxx" class="类的路径"></bean>
    ```
* 使用有参构造函数创建对象
  1. 下标赋值
      ```
      <bean id="xxx" class="类的路径">
        <constructor-arg index="0" value="xxx">
      </bean>
      ```
  2. 参数类型赋值
      ```
      <bean id="xxx" class="类的路径">
        <constructor-arg type="java.lang.String" value="xxx">
      </bean>
      ```
  3. 参数名赋值
      ```
      <bean id="xxx" class="类的路径">
        <constructor-arg name="参数名" value="xxx">
      </bean>
      ```
#### 二、使用Set方式注入

* 普通值注入
   ```
     <bean id="xxx" class="类的路径">
       <property name="xxx" value="xxx">
     </bean>
   ```
* 注入其他bean
   ```
   <bean id="bean1" class="类的路径">
     ...
   </bean>
   <bean id="bean2" class="类的路径">
     <property name="xxx" ref="bean1"></property>
   </bean>
   ```
* 注入数组
   ```
   <bean id="xxx" class="类的路径">
     <property name="xxx">
       <array>
         <value>aaa</value>
         <value>bbb</value>
         ...
       </array>
     </property>
   </bean>
   ```
* list注入
   ```
   <bean id="xxx" class="类的路径">
     <property name="xxx">
       <list>
         <value>list1</value>
         <value>list2</value>
         ...
       </list>
     </property>
   </bean>
   ```
* map注入
   ```
   <bean id="xxx" class="类的路径">
     <property name="xxx">
       <map>
         <entry key="xxx" value="xxx"></entry>
         ...
       </map>
     </property>
   </bean>
   ```
* prop注入
   ```
   <bean id="xxx" class="类的路径">
     <property name="xxx">
       <props>
         <prop key="xxx">xxx</prop>
         ...
       </props>
     </property>
   </bean>
   ```

第三种方式注解注入，参考下一章：Spring注解。

### Spring注解
之前所说的都是在xml配置文件中实现bean的装配和依赖注入。除了在xml配置文件配置bean标签，还有一种更主流的方式就是使用Spring提供的注解装配和注入bean。
**用于装配的注解**
Spring提供了很多用来装配的注解，其中比较常用的有@Component、@Repository、@Service和@Controlle，他们四个的功能是一样的，都是用来完成类的装配工作，也就是代替了\<bean>标签的作用。后面三个都是@Component衍生而来的，使用起来更具有语义化。
**用于依赖注入的注解**
Spring也提供了很多用于依赖注入的注解，其中比较常用的有@Autowired和@Resource。

##### 如何使用注解：

还是要创建一个Spring的配置文件applicationContext.xml，要想注解生效，需要在xml配置添加注解的支持\<context:annotation-config/>，或者添加Spring对业务包的扫描配置\<context:component-scan base-package=”XX.XX”/> 。不再需要编写繁琐的\<bean>标签。
```
applicationContext.xml

<!-- 注解支持配置 -->
<context:annotation-config/>
<!-- 或者Spring扫描指定包配置-->
<context:component-scan base-package=”com.demo.xxx”/> 
```

开发业务代码：

```
// UserEntity.java
public class UserEntity implements Serializable {
    private Long id;
    private String username;
    private String password;
    ...
    get/set方法
    ...
}

// UserDao.java
@Repository("UserDao") // 该注解完成UserDao类的装配
public interface UserDao {
    List<UserEntity> findAllUser();
    String findPasswordByName(@Param("username")String username);
}

// UserService.java
public interface UserService {
    List<UserEntity> findAllUser();
    String findPasswordByName(String username);
}

// UserServiceImpl.java
@Service("UserService")
public class UserServiceImpl implements UserService {
    // 只声明userDao的类型，@Resource默认根据变量名字从Spring容器匹配对应的类，并完成注入
    @Resource // 该注解会根据声明的变量名
    private UserDao userDao;
    
    @Override
    public List<UserEntity> findAllUser() {
        return userDao.findAllUser();
    }

    @Override
    public String findPasswordByName(String username) {
        return userDao.findPasswordByName(username);
    }
}

// UserController.java
@Controller
public class UserController {
    // 只声明userService类型，@Autowired默认根据类型从Spring容器匹配对应的类完成注入
    @Autowired
    private UserService userService;

    @RequestMapping("/userlist")
    @ResponseBody
    public List<UserEntity> getAllUser() {
        return userService.findAllUser();
    }
}
```


<!-- ### @Configuration配置类注解代替xml文件 -->


## AOP
AOP实现原理就是利用代理，没有了解过代理模式的，最好先看一下[代理模式](./代理模式/README.MD)。
如果被代理目标是接口的实现类，Spring使用JDKProxy动态代理的方式实现AOP，如果被代理目标是一个普通类，Spring则借助cglib工具提供的动态代理实现AOP。

**概念：**
面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是OOP的延续，是软件开发中的一个热点，也是Spring框架的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的重用性，同时提高了开发的效率。

所谓AOP面向切面，就好比下水道的过滤网用来过滤水中体积比较大的杂质以防堵下水道，过滤网可以可以放在下水道的任何一个地方，所有的污水都会通过过滤网的过滤，用户只管往下水道倒水，不用担心下水道被堵的问题。
开发者编写的业务代码就好比一个下水道管道，AOP切面就好比一张过滤网，可以插入业务代码某个方法，每次执行这个方法都会经过切面处理，业务只管执行这些方法。
**关键词**
* 连接点（JointPoint）：在应用执行过程中可以插入切面的任何一个地方。就好比下水道任何一个可以插入过滤网的地方，是一个抽象概念
* 切入点（PointCut）：具体的某一个连接点（比如洗菜池的出水口）
* 通知（Advice）：切面在切入点要做的工作
  * 前置通知（Before）：在目标方法被执行之前要做的工作
  * 后置通知（After）：在目标方法被执行之后要做的工作
  * 返回通知（After-returning）：在目标方法成功执行之后要做的工作
  * 异常通知（After-throwing）：在目标方法抛出异常后要做的工作
  * 环绕通知（Around）：在被通知的方法调用之前和之后执行的工作
* 目标（Target）：被通知的对象
* 切面（Aspect）：切面是通知和切点的结合，是一个类
* 织入（Weaving）：织入是把切面应用到目标对象并创建新的代理对象的过程

### Spring AOP使用方式
##### 方式一：xml配置

如果想在service层插入切面。
```
// UserService.java
public interface UserService {
    // 增删改查
    public void add();
    public void delete();
    public void update();
    public void query();
}

// UserServiceImpl.java
public class UserServiceImpl implements UserService {

    @Override
    public void add() {
        System.out.println("新增一个用户");
    }

    @Override
    public void delete() {
        System.out.println("删除一个用户");
    }

    @Override
    public void update() {
        System.out.println("更新一个用户");
    }

    @Override
    public void query() {
        System.out.println("查询一个用户");
    }
}

// 切面类Aspect.java
public class MyAspect {
    public void before() {
        System.out.println("前置通知");
    }
    public void after(){
        System.out.println("后置通知");
    }
    public void afterReturning(){
        System.out.println("成功后置通知");
    }
    public void afterThrowing(){
        System.out.println("抛错通知");
    }
    public Object around(ProceedingJoinPoint point) throws Throwable {
        /**
         * 如果配置了环绕通知，需要手动执行业务目标方法，否则业务方法不执行
         */
        System.out.println("环绕通知1");
        Object result = point.proceed(); // 执行业务方法
        System.out.println("环绕通知2");
        return result;
    }
}
```
```
// applicationContext.xml
<!--AOP配置-->
<bean id="aspect" class="com.aop.log.MyAspect"></bean>

<aop:config>
    <!--切面-切面由切点和通知组成-->
    <aop:aspect ref="aspect">
        <!--切入点-->
        <aop:pointcut id="myPointcut" expression="execution(* com.aop.service.UserServiceImpl.*(..))"></aop:pointcut>
        <!--通知-->
        <aop:before method="before" pointcut-ref="myPointcut"></aop:before>
        <aop:after method="after" pointcut-ref="myPointcut"></aop:after>
        <aop:after-returning method="afterReturning" pointcut-ref="myPointcut"></aop:after-returning>
        <aop:after-throwing method="afterThrowing" pointcut-ref="myPointcut"></aop:after-throwing>
        <aop:around method="around" pointcut-ref="myPointcut"></aop:around>
    </aop:aspect>
</aop:config>
```

```
// 测试

public class MyTest {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserService userService = context.getBean("userService", UserService.class);

        userService.add();
    }
}

// 输出结果
前置通知
环绕通知1
新增一个用户
环绕通知2
成功后置通知
后置通知
```
##### 方式二：注解
使用注解的方式配置AOP，只需要修改切面类Aspect.java和配置文件applicationContext.xml。

```
// Aspect.java
@Aspect
public class MyAspect {
    @Before("execution(* com.aop.service.UserServiceImpl.*(..))")
    public void before(JoinPoint point) {
        System.out.println("前置通知");
    }
    @After("execution(* com.aop.service.UserServiceImpl.*(..))")
    public void after(JoinPoint point){
        System.out.println("后置通知");
    }
    @AfterReturning("execution(* com.aop.service.UserServiceImpl.*(..))")
    public void afterReturning(JoinPoint point){
        System.out.println("成功后置通知");
    }
    @AfterThrowing("execution(* com.aop.service.UserServiceImpl.*(..))")
    public void afterThrowing(JoinPoint point){
        System.out.println("抛错通知");
    }
    @Around("execution(* com.aop.service.UserServiceImpl.*(..))")
    public Object around(ProceedingJoinPoint point) throws Throwable {
        /**
         * 如果配置了环绕通知，需要手动执行业务目标方法，否则业务方法不执行
         */
        System.out.println("环绕通知1");
        Object result = point.proceed(); // 执行业务方法
        System.out.println("环绕通知2");
        return result;
    }
}
```
```
// aplicationContxt.xml
// 删掉<aop:config>配置  添加aop注解方式支持配置
<!--开启spring对AOP注解的支持-->
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
```
```
// 测试

public class MyTest {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        UserService userService = context.getBean("userService", UserService.class);

        userService.add();
    }
}

// 输出结果
前置通知
环绕通知1
新增一个用户
环绕通知2
成功后置通知
后置通知
```


   






  
