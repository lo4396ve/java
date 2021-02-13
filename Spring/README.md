## Spring
Spring是一个轻量级的控制反转（IOC）和面向切面编程（AOP）的框架。
**特点：**
* 非侵入式，指的是引入Spring不会侵入到业务代码，不用继承框架提供的类，而是通过配置完成依赖注入后，就可以使用
* 轻量级
* 控制反转（IOC）
* 面向切面编程（AOP）
* 支持事务

## 核心模块
spring包括七大核心模块：

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

## 核心概念
**IOC控制反转**
IOC是一种编程思想，目的是为了降低程序耦合。就是把创建对象交给Spring去做，业务程序不创建对象只接搜对象。

**思想来源：**
（1）从一个demo来理解IOC思想，假如有个User实体类，UserDao接口，UserDaoImpl实现类，UserServer接口，UserServiceImpl实现类以及一个UserTest测试类。

UserDao:
```
public interface UserDao {
  String getUserName();
}
```
UserDaoMysqlImpl:
```
public class UserDaoImpl implements UserDao{
  @Override
  punlic String getUserName() {
    System.out.println("Mysql方式获取用户名")
  }
}
```
UserServer:
```
public interface UserService {
  String getUserName();
}
```
UserServiceImpl:
```
public class UserServiceImpl implements UserService{
  private UserDao userDao = new UserDaoMysqlImpl();
  @Override
  public String getUserName() {
      return userDao.getUserName();
  }
}
```
UserTest:
```
public class UserTest {
  public static void main(String[] args) {
    UserService userService = new UserServiceImpl();
    userService,getUserName();
  }
}
```
（2）上面代码是一个简单的查询用户名的demo。正常运行是没有问题的。如果现在要加一个Oracle查询的方式，需要对UserDao添加一个UserDaoOracleImpl实现类。
UserDaoOracleImpl：
```
@Override
  punlic String getUserName() {
    System.out.println("Oracle方式获取用户名")
  }
```

然后还需要手动到业务层UserServiceImpl修改代码：
```
public class UserServiceImpl implements UserService{
  private UserDao userDao = new UserDaoOracleImpl();
  @Override
  public String getUserName() {
      return userDao.getUserName();
  }
}
```
（3）如果还有更多的方式去获取用户名字，每次都要手动修改业务层代码。现在把UserServiceImpl改成下面这样：
```
public class UserServiceImpl implements UserService{
  private UserDao userDao

  public void setUserDao(UserDao userDao) {
    this.userDao = userDao;
  }

  @Override
  public String getUserName() {
      return userDao.getUserName();
  }
}
```
UserTest也改一下：
```
public class UserTest {
  public static void main(String[] args) {
    UserService userService = new UserServiceImpl();

    userService .setUserDao(new UserDaoOracleImpl())

    userService,getUserName();
  }
}
```

其实只是把new UserDaoOracleImpl()这一步骤从业务层UserService中脱离出来放到了UserTest中去执行，但是从设计模式的角度考虑，现在已经降低了业务层的耦合度。这就是IOC的一个思想。

Spring做的就是把new一个对象交给spring去做，业务不需要去做这个事情，只需要关心业务逻辑。

（4）Spring配置文件里面的\<bean\>标签就是用来做这个事的，一个正常的\<bean\>标签大致长这个样子：
```
<beans>
  <bean id="mybean" class="某个类">
    <property name="对应该类中的某个set方法" value="想要设置的值">
  </bean>
</beans>
```
所有被\<bean\>标签配置的类都会放到Spring容器，什么时候想用这个类的实例对象，只需要从Spring容器中取，而不用自己去new实例对象。这就是Spring的IOC思想。

**Spring创建对象的方式：**
* 使用无参构造函数创建对象（默认）
```
<bean id="xxx" class="类的路径"></bean>
```
* 使用有参构造函数创建对象
  1) 下标赋值
  ```
  <bean id="xxx" class="类的路径">
    <constructor-arg index="0" value="xxx">
  </bean>
  ```
  2) 参数类型赋值
   ```
  <bean id="xxx" class="类的路径">
    <constructor-arg type="java.lang.String" value="xxx">
  </bean>
  ```
  3) 参数名赋值（用的比较多）
  ```
  <bean id="xxx" class="类的路径">
    <constructor-arg name="参数名" value="xxx">
  </bean>
  ```
其中在配置文件加载的时候，容器中管理的对象已经初始化了。
