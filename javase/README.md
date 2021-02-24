# JAVA入门
既然学习java，看一下java的介绍，java语法这些最基础的东西一点都不过分。
看基础语法很枯燥，而且基础语法还很多，对于第一次接触java的童鞋，不用把java基础语法每个都看一遍，只需要看一些常用的重要的知识点。回想当初学习前端，想必很多人都是从html开始学起的，第一次看html文档也没有一次把所有的HTML标签都过一遍吧。平铺式的把基础语法全看一遍也不是好的学习习惯，学着后面的忘着前面的。

## java介绍
首先Java 面向对象程序设计语言，随着Java的发展，SUN给Java又分出了三个不同版本：
* Java SE：Java SE就是标准版，包含标准的JVM和标准库
* Java EE：Java EE是企业版，它只是在Java SE的基础上加上了大量的API和库（真正开发用的最多的就是Java EE）
* Java ME：是一个针对嵌入式设备的“瘦身版”

然后是java的运行环境。任何语言都需要运行环境，比如浏览器内核和node就是js的运行环境。同样java也需要运行环境。java运行环境涉及到一下几个概念：
* JVM：JVM是java虚拟机，我的理解就是用来编译java代码的
* JRE：JRE=JVM+Java基础类库，JRE就是java的运行环境。
* JDK：JDK=JRE+Java工具，一般开发环境直接安装JDK。

可能有的童鞋会疑惑为什么JRE是Java的运行环境，而不是JVM。可以想象Js的运行环境，要想把Js运行起来首先需要编译，其次还需要提供Js的基础类库，不然Object谁会认识。
## 环境安装
* 安装JDK
  参考文章[安装JDK](https://www.liaoxuefeng.com/wiki/1252599548343744/1280507291631649)
* IDE
  Eclipse,MyEclipse,IDEA都可以。
## 语法
对于首次接触java的同学，只给出建议，哪些Java语法是适合这个时候去看的，具体的语法有很多途径都可以学到，比如廖雪峰大神的[Java教程](https://www.liaoxuefeng.com/wiki/1252599548343744)
* 修饰符
* 变量：怎么定义一个变量。（良心的Java虽然提供了var关键之来定义变量，这个跟Js的var很像，但是不建议初学者用，既然学习Java多写点强类型语法没什么坏处）
* 数据类型：基本数据类型和引用类型
* 流程控制：if,for,while, do while,break,continue。这些很js基本一样，其实大多数语言这方便都差不多。
* 面向对象编程（重点）
* 异常处理
* 注解
* 泛型
* 集合：常用List，Map，Set

相信看完这些的你，已经对Java有一个最初的认识了，已经可以自己写一个Hello World了。
