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
感兴趣的话可以查找java类加载更具体的介绍。
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
### 体验反射
```
package com.ssm.dao;

public class Test {
    public static void main(String[] args) throws ClassNotFoundException {
        // 通过反射获取类的Class对象
        Class c1  = Class.forName("com.ssm.dao.User");
        Class c2  = Class.forName("com.ssm.dao.User");
        System.out.println(c1); // class com.ssm.dao.User
        System.out.println(c1 == c2); // 一个类在内存中只有一个Class对象，所以打印结果为true
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


### 什么是Class类


##### 反射机制
##### 理解Class类并获取Class实例
##### 类的加载与ClassLoader
##### 创建运行时类的对象
##### 获取运行时类的完整结构
##### 调用运行时类的指定结构
##### 反射机制


