# 注解和反射
## 注解
##### 注解的作用
* 不是程序本身，可以对程序作出解释
* 可以被其他程序（如编译器）读取
##### 内置注解

* @Override重写；
* @Deprecated 已过期，表示方法是不被建议使用的；
* @SuppressWarnings 压制警告，抑制警告
##### 元注解：作用就是负责解释其他注解的注解
* @Target：描述注解的使用范围
* @Retention：表示注解在哪个生命周期有效
* @Document: 说明是否将注解生成在javadoc中
* @Inherited：说明子类可以继承父类中该注解
  
##### 自定义注解
java中使用@interface声明一个注解，使用@interface注解时，会自动继承java.lang.annotation.Annotation接口。
自定义一个注解demo：
```
public class test {
    @MyAnnotation(arg1 = "张三")
    public void test(){

    }
}
// 规定该自定义注解用于类和方法
@Target({ElementType.TYPE, ElementType.METHOD})
// 规定该注解在源码级生效
@Retention(RetentionPolicy.RUNTIME)
@interface MyAnnotation{
    // 注解的参数  参数类型+参数名()
    String arg1();
    // default 提供注解默认值  使用的时候可以不传，如果没有默认值必须传
    String arg2() default "李四";
}
```
## 反射
学习Java反射需要先了解Java中普通类、Class特殊类、Class示例对象、Object对象他们的关系。有一点像Js中原型链一样很绕，但是跟原型链又不尽一样。只要了解了他们的关系以及类的加载原理，反射还是很容易学会的。
### 动态语言与静态语言
##### 动态语言
动态语言（弱类型语言）是运行时才确定数据类型的语言，变量在使用之前无需申明类型，通常变量的值是被赋值的那个值的类型。比如Php、Asp、JavaScript、Python、Perl等等
##### 静态语言
静态语言（强类型语言）是编译时变量的数据类型就可以确定的语言，大多数静态语言要求在使用变量之前必须生命数据类型。比如Java、C、C++、C#等。

Java不是动态语言，但是Java可以利用反射机制获得类似动态语言特性，所以Java具有一定的动态性，Java可以称之为准动态语言。Java的动态性让编程更加灵活。
### Class类
##### 什么是类
类是面向对象编程语言的一个重要概念，它是对一项事物的抽象概括，可以包含该事物的一些属性定义，以及操作属性的方法。面向对象编程中，我们都是以类来编码。ES6中也有类的概念，应该不陌生。
##### 什么是Class类
注意C是大写的，在java中小写的class是关键字，用来声明一个类的。Class是在java.lang包下真实存在的一个类，在java.lang下有这么一个Class.java文件，它跟普通的类一样，是一个实实在在的类。
在Java里，所有的类的根源都是Object类，Class类也不例外，它是继承自Object的一个特殊的类，Class类没有公共的构造方法，Class对象是在类加载的时候由Java虚拟机以及通过调用类加载器中的 defineClass 方法自动构造的，因此不能显式地声明一个Class对象。
根据类的定义，类是对一项事物的抽象概括。Class类则是对其他类的一个抽象概括。其内部可以记录类的成员、接口等信息，也就是在Java里，Class是一个用来表示类的类。
##### Class类的特点
* Class本身是一个类
* Class实例对象只能由系统创建，我们无法创建
* 一个类加载后在JVM中只有一个Class实例
* 一个Class对象对应一个加载到JVM中的.class文件
* 通过Class对象可以完整的得到一个类的结构
* Class类是反射的根源，针对任何想动态加载、运行的类，唯有先获得相应的Class对象
##### 类的加载过程
我们编写的类代码，在经过编译器编译之后，会为每个类生成对应的.class文件，这个就是JVM可以加载执行的字节码。
运行时期间，当我们需要实例化任何一个类时，JVM会首先尝试看看在内存中是否有这个类，如果有，那么会直接创建类实例；如果没有，那么就会根据类名去加载这个类，当加载一个类，或者当加载器(class loader)的defineClass()被JVM调用，便会为这个类产生一个Class对象（一个Class类的实例），用来表达这个类，该类的所有实例都共同拥有着这个Class对象，而且是唯一的。
类的加载过程主要分为以下几步：
* 类的加载
将类的.class文件读入内存，并将这些静态数据转化成方法区的运行时数据结构，然后创建一个代表这个类的Class对象，此过程由类加载器完成
* 类的链接：将类的二进制数据合并到JRE中
  * 验证：确保加载的类信息符合JVM规范，没有安全方面问题
  * 准备：正式为类变量分配内存，并设置类变量默认初始值，这些内存都将在方法区中分配
  * 解析：虚拟机常量池内的符号引用（常量名）替换为直接引用（地址）的过程

* 类的初始化
  * JVM执行类构造器<clinit>()方法过程， 该方法是由编译期自动收集类中的所有类变量的赋值动作和静态代码块中的语句合并。
  * 当初始化一个类如果其父类还没有初始化，先初始化父类
  * 虚拟机会保证一个类的<clinit>()方法在多线程环境中被正确加锁和同步
JVM负责对类进行初始化
感兴趣的话可以查找java类加载更具体的介绍。

### 类加载器
##### 类加载器的作用
将类的.class文件读入内存，并将这些静态数据转化成方法区的运行时数据结构，然后创建一个代表这个类的Class对象
##### 类缓存
一旦某个类被加载到类加载器中，将维持加载（缓存）一段时间，JVM可以回收这些Class对象。
##### 类加载器分类
* 引导类加载器
使用C++编写的，是JVM自带的类加载器，负责加载Java核心类库。一般我们无法拿到这个类。
* 扩展类加载器
负责jre/lib/ext目录下jar包或者-D java.ext.dirs指定目录下的jar包装入工作库
* 系统类加载器
负责java -classpath 或者-D java.class.path所指目录下的类和jar包装入工作裤，是最常用的加载器 

这三个加载器存在继承关系：
引导类加载器(根加载器)是扩展类加载器的父类
扩展类加载器是系统类记载器的父类

##### 类加载器demo
```
public class Test {
    public static void main(String[] args) throws ClassNotFoundException {

        // 当前Test类的类加载器
        ClassLoader testLoader = Class.forName("com.ssm.dao.Test").getClassLoader();
        System.out.println("testLoader"+ testLoader);   // sun.misc.Launcher$AppClassLoader@18b4aac2

        // JDK内置类的类加载器
        ClassLoader objectLoader = Class.forName("java.lang.Object").getClassLoader();  
        System.out.println("objectLoader" + objectLoader); // 因为无法获取引导类加载器，所以打印null
        // 系统类可以加载的路径
        System.out.println(System.getProperty("java.class.path"));
        /*
         会打印可以加载的所有目录，包括java核心类库路径，本项目路径，以及maven下各种包的路径
         /Library/Java/JavaVirtualMachines/jdk1.8.0_201.jdk/Contents/Home/jre/lib/charsets.jar:
         /Library/Java/JavaVirtualMachines/jdk1.8.0_201.jdk/Contents/Home/jre/lib/deploy.jar:
         /Library/Java/JavaVirtualMachines/jdk1.8.0_201.jdk/Contents/Home/jre/lib/ext/cldrdata.jar:
         /Library/Java/JavaVirtualMachines/jdk1.8.0_201.jdk/Contents/Home/jre/lib/ext/dnsns.jar:
         ...等java核心类库:
         /Users/username/work-space/target/classes:
         /usr/local/apache-maven-3.6.1/respository/org/hibernate/hibernate-validator/5.2.4.Final/hibernate-validator-5.2.4.Final.jar:
         */
    }
}
```
### Object类

Object类存储在java.lang包中，是所有java类(Object类和接口除外)的终极父类。创建一个类时，如果没有明确继承一个父类，那么它就会自动继承 Objec。

在Object类中定义了一个getClass方法，该方法会被所有的子类继承。getClass方法返回值类型是一个Class类，此类是java反射的源头。

### 什么是反射（Reflection）
前面了解了类的加载过程，知道了一个类加载完后在堆内存的方法中会产生一个Class类型的对象（一个类只有一个Class对象），这个对象就包含了完整的类的结构信息。就像一面镜子一样可以通过这个对象看到类的结构，所以称之为：反射。

### 反射常用的API
* static Class.forName(String name); 返回指定类名的的Class对象
* newInstance(); 通过反射创建一个对象，调用默认构造方法创建并返回Class对象的实例
* getName(); 返回Class对象表示的类的名字
* getSuperClass(); 返回当前Class对象的父类的Class对象
* getInterfaces(); 返回这个对象所实现的所有接口
* getClassLoader(); 返回该类的类加载器
* getConstructors(); 返回一个包含某些Constructor对象的数组
* getMethod(String name, Class<?>... parameterTypes)的作用是获得对象所声明的公开方法。该方法的第一个参数name是要获得方法的名字，第二个参数parameterTypes是按声明顺序标识该方法形参类型。
* getField(); 返回类的公共成员字段或由此Class对象表示的接口
* ...

### 如何获取Class类的实例
* 如果已知具体的类，可以通过类的class属性获取
```
Class clazz = User.class;
```
* 如果已知某个类的实例，可以通过实例的getClass()方法获取
```
User user = new User();
Class clazz = user.getClass();
```
* 如果已知一个类的全类名，可以通过Class类的静态方法forName()获取
```
Class clazz = Class.forName("com.xxx");
```
* 内置基本数据类型直接使用Type属性获取
```
Class clazz = Integer.TYPE;
```
* 通过类加载器获取
### 一个简单的实例
```
package com.ssm.dao;

public class Test {
    public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
        // 通过反射获取类的Class对象
        Class c1  = Class.forName("com.xxx.User");  // User的完整地址

        // 构造一个对象 newInstance()会调用无参构造方法
        User user = (User) c1.newInstance();

        // 获取有参构造器 通过有参构造方法创建一个对象
        Constructor constructor = c1.getDeclaredConstructor(String.class);
        User user2 = (User) constructor.newInstance("王二");

        // 获取类的Methos方法  并调用方法
        Method setName = c1.getDeclaredMethod("setName", String.class);
        // invoke 就像js里的call方法
        setName.invoke(user, "张三");
        System.out.println(user.getName()); // 张三

        User user3 = (User) c1.newInstance();
        Field name = c1.getDeclaredField("name");
        // name是private私有属性 可以通过setAccessible(true)关闭程序安全监测
        name.setAccessible(true);
        name.set(user3, "李四");
        System.out.println(user3.getName());    // 李四
    }
}

class User {
    private String name;

    public User() {}

    public User(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                '}';
    }
}
```


### 反射操作泛型（了解即可）

Java语言的泛型实现方式是擦拭法，擦拭法是指虚拟机对泛型其实一无所知，所有的工作都是编译器做的。Java中的泛型仅仅是给编译器javac使用的，确保数据安全性和免去类型强制转换，一旦编译完成，所有和泛型相关的类型全部擦除。

Java为了通过反射操作这些类型，新增了以下几个类：
* TypeVariable：是各种类型变量的公共父接口
* ParameterizedType: 表示一种参数化的类型，比如Collection<String>中的String
* GenericArrayType: 表示一种元素类型是参数化类型或者类型变量的数组类型
* WildcardType: 代表一种通配符类型表达式，比如?, ? extends Number, ? super Integer【wildcard是一个单词：就是“通配符”】
  
```
package com.ssm.dao;

import java.lang.reflect.*;
import java.util.List;
import java.util.Map;

public class Test {
    public static void main(String[] args) throws NoSuchMethodException {
        // f1
        Method method1 = Test.class.getMethod("f1", Map.class, List.class);
        Type[] genericParameterTypes = method1.getGenericParameterTypes();
        for(Type genericParameterType : genericParameterTypes) {
            // 获取到f1的两个参数Map<String, User>, List<User>
            System.out.println("ParameterType=" + genericParameterType);
            if(genericParameterType instanceof ParameterizedType) {
                // 获取上面两个参数（Map<String, User>, List<User>）的泛型参数（String, User）
                Type[] arguments = ((ParameterizedType) genericParameterType).getActualTypeArguments();
                for(Type argument : arguments) {
                    System.out.println(argument);
                }
            }
        }
        /*
        打印结果：
        ParameterType=java.util.Map<java.lang.String, com.ssm.dao.User>
        class java.lang.String
        class com.ssm.dao.User
        ParameterType=java.util.List<com.ssm.dao.User>
        class com.ssm.dao.User
         */
        
        // f2
        Method method2 = Test.class.getMethod("f2", null);
        // 获取f2方法的返回值类型（Map<String, User>）
        Type genericReturnType = method2.getGenericReturnType();
        if(genericReturnType instanceof ParameterizedType) {
            // 获取f2方法的返回值类型（Map<String, User>）的泛型参数（String, User）
            Type[] arguments = ((ParameterizedType) genericReturnType).getActualTypeArguments();
            for(Type argument : arguments) {
                System.out.println(argument);
            }
        }
        /*
        打印结果
        class java.lang.String
        class com.ssm.dao.User
         */
    }
    

    public void f1(Map<String, User> map, List<User> list) {
        System.out.println("f1");
    }

    public Map<String, User> f2() {
        System.out.println("f2");
        return null;
    }
}

class User {
    private String name;

    public User() {}


    public User(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                '}';
    }
}

```


### 反射操作注解
直接通过Class对象的getAnnotation就可以获取类的注解。
如果要获取类里面属性的注解，先用Class对象的getDeclaredField获取属性(Field)，然后再获取注解。
```
package com.ssm.dao;

import java.lang.annotation.*;
import java.lang.reflect.Field;

public class Test03 {
    public static void main(String[] args) throws NoSuchFieldException {
        Class c = Student.class;

        // 获取Student的注解@MyTable(tableName="db_student", host="localhost")
        MyTable myTable = (MyTable) c.getAnnotation(MyTable.class);
        // 打印注解@MyTable(tableName="db_student", host="localhost")的参数（tableName和host）
        System.out.println(myTable.tableName());
        System.out.println(myTable.host());
        /*
        打印结果：
        db_student
        localhost
         */

        // 获取Student类的name属性的注解@myField(columnName="xxx", type="xxx")
        Field field = c.getDeclaredField("name");
        MyField annotation = field.getAnnotation(MyField.class);
        System.out.println(annotation.columnName());
        System.out.println(annotation.type());
        /*
        打印结果：
        db_name
        varchar
         */

    }
}

@MyTable(tableName="db_student", host="localhost")
class Student {
    @MyField(columnName="db_id", type="int")
    private int id;
    @MyField(columnName="db_name", type="varchar")
    private String name;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                '}';
    }
}

// 假设定义一个用于连接数据库表和java实体类的注解
@Target(ElementType.TYPE)   // 定义该注解使用范围是TYPE
@Retention(RetentionPolicy.RUNTIME) // 定义该注解生命周期
@interface MyTable{
    String tableName(); // 用于表示数据库表名字
    String host();  // 用于表示数据库地址
}

// 假设定义一个用于连接数据库表字段和java实体类属性的注解
@Target(ElementType.FIELD)   // 定义该注解使用范围是TYPE
@Retention(RetentionPolicy.RUNTIME) // 定义该注解生命周期
@interface MyField{
    String columnName(); // 用于表示数据库表字段名
    String type();  // 用于表示数据库表字段的类型
}
```
##### 理解Class类并获取Class实例
##### 类的加载与ClassLoader
##### 创建运行时类的对象
##### 获取运行时类的完整结构
##### 调用运行时类的指定结构
##### 反射机制


