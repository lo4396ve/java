# Mybatis
## 1、概念
MyBatis是一个Java持久化框架，它通过XML描述符或注解把对象与存储过程或SQL语句关联起来，映射成数据库内对应的纪录。
不管什么持久化框架，最后都绕不开jdbc，说白了Mybatis就是对jdbc的一个封装，支持定制化 SQL、存储过程以及高级映射，MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息。
> jdbc: Java数据库连接，（Java Database Connectivity，简称JDBC）是Java语言中用来规范客户端程序如何来访问数据库的应用程序接口，提供了诸如查询和更新数据库中数据的方法。

## 2、使用Mybatis
每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为核心的。SqlSessionFactory 的实例可以通过 SqlSessionFactoryBuilder 获得。而 SqlSessionFactoryBuilder 则可以从 XML 配置文件或一个预先配置的 Configuration 实例来构建出 SqlSessionFactory 实例。
以 XML 配置文件构建 SqlSessionFactory 为例：
### 2.1 环境搭建
#### 2.1.1 创建数据库
```
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `username` varchar(255) DEFAULT NULL,
  `address` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8;

BEGIN;
INSERT INTO `user` VALUES (0, '张三', '北京市');
INSERT INTO `user` VALUES (1, '李四', '上海市');
COMMIT;

SET FOREIGN_KEY_CHECKS = 1;

```
#### 2.1.2 创建maven项目导入jar包
```
// pom.xml
<dependencies>
    <!-- 测试 -->
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
    <!-- mybatis -->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.2.8</version>
    </dependency>
    <!-- mysql -->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.46</version>
    </dependency>
  </dependencies>
```
#### 2.1.3 实体类
```
// User.java

public class User implements Serializable {
    private int id;
    private String username;
    private String address;


    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}

```
#### 2.1.4 mapper接口
起名mapper、dao都可以
```
// UserMapper.java
public interface UserMapper {
    // 获取所有用户号列表
    List<User> getAllUser();
}
```
#### 2.1.5 添加Mybatis核心配置文件
Mybatis核心配置包括获取数据库连接实例的数据源（DataSource）、决定事务作用域和控制方式的事务管理器（TransactionManager）以及mapper接口映射配置
```
// mybatis-config.xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!--数据源-->
    <properties>
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/zzz_mybatis"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
    </properties>

    <environments default="development">
        <environment id="development">
            <!--事务管理器-->
            <transactionManager type="JDBC"/>
            <!--数据源配置-->
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>

    <!-- 接口映射配置 -->
    <mappers>
        <mapper resource="mapper/UserMapper.xml"/>
    </mappers>
</configuration>
```

#### 2.1.6 接口映射配置
```
// UserMapper.xml
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.mybatis.mapper.UserMapper">
    <!-- 配置查询所有用户 -->
    <select id="getAllUser" resultType="com.mybatis.domain.User">
        SELECT * FROM user
    </select>
</mapper>
```

### 2.2 测试
Mybatis项目的所有准备工作已经完成了，然写个测试类跑一下。
```
// MyTest.java
public class MyTest {
    @Test
    public void mytest() {
        try{
            // 1、读取配置文件
            InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");

            // 2、构建模式 创建SqlSessionFactory 工厂
            SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);

            // 3、工厂模式 生产并获取SqlSession 对象
            SqlSession sqlSession = factory.openSession();

            // 4、代理模式 创建 Mapper 的代理对象
            UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

            // 5、使用代理对象查询
            List<User> list = userMapper.getAllUser();

            for(User user: list) {
                System.out.println(user);
            }

            // 6、释放资源
            sqlSession.close();
            inputStream.close();

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
* ①：根据Mybatis的概念（Mybatis通过XML描述符或注解把对象与存储过程或SQL语句关联起来，映射成数据库内对应的纪录），要想Mybatis工作，首先要读取Mybatis的核心配置xml文件。
* ②：每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为核心的。SqlSessionFactory 的实例可以通过 SqlSessionFactoryBuilder 获得。所以第二步就是创建SqlSessionFactory。
* ③：SqlSessionFactory是一个工厂（工厂模式），由SqlSessionFactory工厂获取SqlSession示例对象
* ④：UserMapper.java只是一个接口，并没有实现类。所以第四步就是匹配对应的映射UserMapper.xml配置文件，利用动态代理的方式完成对数据库操作的实现。
* ⑤：代理对象已经实现，然后就可以使用代理对象的方法操作数据库
* ⑥：最后别忘了释放资源

这就是一个完整的Mybatis案例。从测试类里面的六个步骤也大致可以猜出Mybatis的工作流程，以及Mybatis的其三个设计模式（构建模式、工厂模式和代理模式）。
## 3、手写迷你版Mybatis
> miniMybatis
> 还不了解构建模式、工厂模式和代理模式的需要先补习一下

看完上述案例，可能还是有很多疑惑，openSession是怎么获取sqlSession对象的？getMapper是怎么匹配映射配置文件的等等问题。不妨反客为主，如果让你去实现Mybatis这么个工具，你应该怎么做？
还是从前一节案例中的测试类里面使用Mybatis的六个步骤切入。




