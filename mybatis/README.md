# Mybatis
## 1、概念
MyBatis是一个Java持久化框架，它通过XML描述符或注解把对象与存储过程或SQL语句关联起来，映射成数据库内对应的纪录。

不管什么持久化框架，最后都绕不开JDBC，说白了Mybatis就是对jdbc的一个封装，支持定制化 SQL、存储过程以及高级映射，MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息。

JDBC: Java数据库连接，（Java Database Connectivity，简称JDBC）是Java语言中用来规范客户端程序如何来访问数据库的应用程序接口，提供了诸如查询和更新数据库中数据的方法。

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
经过前一节对Mybatis的使用会发现Mybatis其实最主要的是帮我们做了三件事：
1. 解析xml配置文件
2. 帮mapper（或者dao）层接口创建代理对象
3. 封装JDBC操作数据库

### 3.1 构思
首先java本身是不认识xml文件的，每一个xml文件都要解析到一个java类生成实例对象经过编译去执行。所以首先需要创建以下四个文件：

* Configuration：来解析核心配置文件mybatis-config.xml
* Mapper：来解析映射配置文件（如：UserMapper.xml）
* CoreConfigBuilder：提供解析核心配置文件的工具类
* MapperConfigBuilder：提供解析mapper映射配置文件（如：UserMapper.xml）的工具类

然后创建Mybatis的核心接口SqlSession，以及其实现类DefaultSqlSession。他的主要作用是生成mapper代理对象、操作数据库以及对事务的控制（事务暂时不考虑）。

### 3.2 代码实现

观察mybatis-config.xml结构，其主要节点包括数据源dataSource的配置以及其属性driver、url、username和password；以及mappers集合。根据其节点数据结构创建Configuration类：

```
// Configuration.java

public class Configuration {
    private String url;
    private String driver;
    private String username;
    private String password;
    private Map<String, Mapper> mappers = new HashMap<String, Mapper>();

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public String getDriver() {
        return driver;
    }

    public void setDriver(String driver) {
        this.driver = driver;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public Map<String, Mapper> getMappers() {
        return mappers;
    }

    public void setMappers(Map<String, Mapper> mappers) {
        this.mappers = mappers;
    }
}

```

观察UserMapper.xml，主要节点包括：
* \<mapper>标签以及其属性
    * namespace：userMapper接口的全限定路径；
* \<select>标签以及其属性
  * id：userMapper接口里面的方法名（getAllUser）
  * resultType：getAllUser方法的返回值类型
  * 标签内容：要执行的SQL语句
  
根据其结构创建Mapper类：
```
public class Mapper {
    private String sqlQueryString;
    private String resultType;

    public String getSqlQueryString() {
        return sqlQueryString;
    }

    public void setSqlQueryString(String sqlQueryString) {
        this.sqlQueryString = sqlQueryString;
    }

    public String getResultType() {
        return resultType;
    }

    public void setResultType(String resultType) {
        this.resultType = resultType;
    }
}
```

工具类CoreConfigBuilder：
```
public class CoreConfigBuilder {
    public static Configuration loadConfiguration(InputStream in) {

        try {
            Configuration config = new Configuration();

            // dom4j读取配置文件
            SAXReader reader = new SAXReader();
            Document document = reader.read(in);

            // 获取根节点
            Element root = document.getRootElement();

            // 获取数据源配置节点 dataSource
            List<Element> dataSourceList = root.selectNodes("//dataSource");
            Element dataSource = dataSourceList.get(0);
            // 将xml中dataSource中配置的属性值解析到config中
            List<Element> propertyList = dataSource.elements("property");
            for(Element property: propertyList) {
                String propertyName = property.attributeValue("name");
                if("driver".equals(propertyName)) {
                    config.setDriver(property.attributeValue("value"));
                }
                if("url".equals(propertyName)) {
                    config.setUrl(property.attributeValue("value"));
                }
                if("username".equals(propertyName)) {
                    config.setUsername(property.attributeValue("value"));
                }
                if("password".equals(propertyName)) {
                    config.setPassword(property.attributeValue("value"));
                }
            }
            // 获mappers节点
            Element mappers = root.element("mappers");
            // mappers下可能有多个mapper节点
            List<Element> mapperList = mappers.elements("mapper");
            
            for(Element mapper: mapperList) {
                // mapper节点的resource属性值 为每一个mapper映射配置文件的地址
                String mapperPath = mapper.attributeValue("resource");
                // 使用MapperConfigBuilder工具类去解析
                MapperConfigBuilder mapperConfigBuilder = new MapperConfigBuilder();
                Map<String, Mapper> mapperMap =  mapperConfigBuilder.mapperConfigLoad(mapperPath);
                config.setMappers(mapperMap);
            }
            return config;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }


}

```

工具类MapperConfigBuilder：
```
public class MapperConfigBuilder {
    public Map<String, Mapper> mapperConfigLoad(String mapperPath) throws IOException {
        InputStream inputStream = null;
        try{
            Map<String, Mapper> mapperMap = new HashMap<>();

            inputStream = Resources.getResourceAsStream(mapperPath);

            SAXReader reader = new SAXReader();
            Document document = reader.read(inputStream);

            Element root = document.getRootElement();

            // namespace 对应UserMapper接口的地址
            String namespace = root.attributeValue("namespace");

            // 获取select标签
            Element selectElement = root.element("select");

            // id对应UserMapper接口的方法名
            String methodName = selectElement.attributeValue("id");
            // resultType对应方法的返回值类型
            String resultType = selectElement.attributeValue("resultType");
            // 获取要执行的sql语句
            String queryString = selectElement.getText();

            String key = namespace + "." + methodName;

            Mapper mapper = new Mapper();
            mapper.setResultType(resultType);
            mapper.setSqlQueryString(queryString);

            mapperMap.put(key, mapper);

            return mapperMap;
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException(e);
        } finally {
            inputStream.close();
        }

    }
}

```

核心接口SqlSession：
```
public interface SqlSession {

    <T> T getMapper(Class<T> clazz);

    void close();
}

```
SqlSession实现类DefaultSqlSession：
```
public class DefaultSqlSession implements SqlSession {
    private Configuration config;
    private Connection conn;

    public DefaultSqlSession(Configuration config) {
        this.config = config;
        this.conn = this.getConnection(config);
    }

    @Override
    public <T> T getMapper(Class<T> clazz) {
        // 生成代理对象
        return (T) Proxy.newProxyInstance(clazz.getClassLoader(), new Class[] {clazz}, new MapperProxy(config.getMappers(), this.conn));
    }

    @Override
    public void close() {
        try {
            this.conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    private Connection getConnection(Configuration config) {
        try{
            String driver = config.getDriver();
            String url = config.getUrl();
            String username = config.getUsername();
            String password = config.getPassword();

            Class.forName(driver);

            return DriverManager.getConnection(url, username, password);

        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}

```

动态代理InvocationHandler的实现类MapperProxy：
```
package com.mybatis.mini_batis.proxy;

import com.mybatis.mini_batis.Mapper;

import java.beans.PropertyDescriptor;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.ResultSetMetaData;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

public class MapperProxy implements InvocationHandler {
    private Map<String, Mapper> mapperMap;
    private Connection conn;

    public MapperProxy(Map<String, Mapper> mapperMap, Connection conn) {
        this.mapperMap = mapperMap;
        this.conn = conn;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        String className = method.getDeclaringClass().getName();
        String methodName = method.getName();

        String key = className + "." + methodName;

        Mapper mapper = mapperMap.get(key);

        if(mapper == null) {
            throw new IllegalArgumentException("参数有误，找不到Mapper");
        }

        // 开始执行数据库查询操作 把查询的结果集绑定到实体类集合 并返回实体类集合
        // 以下均为JDBC操作
        PreparedStatement pstm;
        ResultSet res;
        try{
            String sqlQuery = mapper.getSqlQueryString();   // 要执行的sql语句
            String resultType = mapper.getResultType(); // 返回类型
            Class domain = Class.forName(resultType);

            pstm = conn.prepareStatement(sqlQuery);
            // 执行sql语句  获取结果集
            res = pstm.executeQuery();

            List<Object> resultList = new ArrayList<>();

            while(res.next()) {
                // 创建实体类对象
                Object obj = domain.newInstance();

                // 获取结果集的元信息
                ResultSetMetaData rsmd = res.getMetaData();

                // 获取列数
                int colunmCount = rsmd.getColumnCount();

                for(int i = 1; i <= colunmCount; i++) {
                    // 获取列的名字
                    String columnName = rsmd.getColumnName(i);
                    // 根据列名 获取列的值
                    Object columnValue = res.getObject(columnName);
                    String type = columnValue.getClass().toString();
                    PropertyDescriptor pd = new PropertyDescriptor(columnName, domain);

                    Method writeMethod = pd.getWriteMethod();
                    writeMethod.invoke(obj, columnValue);
                }
                resultList.add(obj);
            }
            return resultList;

        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

}

```

至此迷你版的mybatis就完成了，测试一下：
修改pom.xml:
```
<dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
    <!-- 去除mybatis的jar包 -->
    <!--<dependency>-->
      <!--<groupId>org.mybatis</groupId>-->
      <!--<artifactId>mybatis</artifactId>-->
      <!--<version>3.2.8</version>-->
    <!--</dependency>-->
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.15</version>
    </dependency>

    <!-- dom4j 帮助解析xml的工具 -->
    <dependency>
      <groupId>dom4j</groupId>
      <artifactId>dom4j</artifactId>
      <version>1.6.1</version>
    </dependency>
    <!-- dom4j依赖的jar包 -->
    <dependency>
      <groupId>jaxen</groupId>
      <artifactId>jaxen</artifactId>
      <version>1.1.6</version>
    </dependency>
```

修改MyTest:
```
public class MybatisTest {
    @Test
    public void mytest() {
        try{
            // 读取配置文件
            InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
            
            // 解析核心配置文件
            Configuration config = CoreConfigBuilder.loadConfiguration(inputStream);
            // 获取mybatis核心接口实现类
            SqlSession sqlSession = new DefaultSqlSession(config);

            // 使用 SqlSession 创建 Mapper 的代理对象
            UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

            // 使用代理对象查询
            List<User> list = userMapper.getAllUser();

            for(User user: list) {
                System.out.println(user);
            }

            // 释放资源
            sqlSession.close();
            inputStream.close();

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

事实上Mybatis不止xml配置使用方式，还支持注解配置的方式，上述示例只展示了查表select的操作，还有更多更复杂的操作。无论哪种方式，Mybatis的工作原理都是不变的，更多Mybatis使用参见[Mybatis官方文档](https://mybatis.org/mybatis-3/)，或者[中文文档](https://mybatis.org/mybatis-3/zh/)

