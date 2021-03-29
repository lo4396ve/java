## 代理模式


什么是代理：房屋中介
### 静态代理
以租房找中介为例子
给租房这件事定义一个接口：
```
public interface Rent {
    public void rent(String name);
}
```
房子House:
```
class House implements Rent{
    private String houseName;
    public House(String name) {
        this.houseName = name;
    }
    @Override
    public void rent(String houseName) {
        System.out.println("房子:" + houseName + "被租出去了");
    }
}
```
中介proxy:
```
public class Proxy implements Rent{
  
  private House house;
  public Proxy() {}
  public Proxy(House house) {
    this.house = house;
  }

  // 中介实现租房方法
  public void rent (String houseName) {
    // 谈价格
    System.out.println("谈价格");
    // 租房
    house.rent(houseName);
    // 租房之后签合同
    System.out.println("签合同");
  }
 
}
```
租户Client：
```
public class Client{
  public static void main (String[] args) {
    // 寻找想租的房子
    String houseName = "别墅一号";
    House house = new House(houseName);
    // 寻找中介，把想租的房子告诉（传值）他
    Proxy proxy = new Proxy(house);
    // 执行中介的租房子方法，中介会帮客户谈价格和签合同
    proxy.rent(houseName);
  }
}
```
### 动态代理
静态代理的缺点：不同的场景需要定义不同的代理类，如果换成办卡业务场景，需要再定义一个办卡代理类（CardProxy）
动态代理特点：动态代理的代理类是动态生成的，不需要手动声明。
动态代理分为两大类：
#### 基于接口：JDK动态代理
    两个重要的工具：
    * Proxy类：动态代理的代理类是动态生成的，而Proxy则提供了创建动态代理类方法
    * InvocationHandler接口：只提供了一个invoke方法，并在该方法内利用反射执行被代理对象真实的方法。


动态代理只是帮助动态生成代理类（中介）。继续使用静态代理声明的Rent和House。

新建ProxyInvocationHandler类：
```
// 利用这个类自动创建代理类
public class MyInvocation implements InvocationHandler {
    // 声明被代理的目标对象（比如该业务场景下为House房子）
    private Object target;
    public void setTarget(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 附加操作
        System.out.println("附加操作");
        // 真正目的：执行被代理接口的方法
        Object result = method.invoke(target, args);
        return result;
    }
}
```
修改Client：
```
public class Client {
    public static void main(String[] args) {
        // 获取想租的房子
        String houseName = "别墅一号";
        House house = new House(houseName);

        // 获取InvocationHandler实例
        MyInvocation myInvocation = new MyInvocation();
        // 设置代理对象（想要租的房子）
        myInvocation.setTarget(house);

        // 使用newProxyInstance方法动态创建代理对象
        Rent proxy = (Rent) Proxy.newProxyInstance(house.getClass().getClassLoader(), house.getClass().getInterfaces(), myInvocation);

        // 由代理去执行租房子这件事
        proxy.rent(houseName);

    }
}
```
###### 注意：
* 第一个需要注意的就是newProxyInstance方法：
  ```
  /**
  * newProxyInstance: Returns an instance of a 
  * proxy class for the specified interfaces that
  * dispatches method invocations to the specified invocation handler.
  *
  *
  * @param classLoader: the class loader to define the proxy class
  * @param interfaces: the list of interfaces for the proxy class to implement
  * @param h: the invocation handler to dispatch method invocations to
  *
  */

  上面这段英文摘自newProxyInstance源码的注释。
  翻译过来就是newProxyInstance方法返回指定接口的代理类实例，该接口将方法调用分派给指定的调用处理程序。
  说简单点就是newProxyInstance用来调用处理程序，并返回结果的。

  第一个参数需要传一个classLoader类加载器，如果有了解过类加载器，可知我们编写的所有的类都是由系统类加载器加载的，所以只要是我们自己编写的类，获取他们的类加载器得到的都是同一个类加载器。所以这里传那个类的类加载器都无所谓。
  第二个参数是一个接口数组，来告诉newProxyInstance方法，在生成动态代理类的时候需要实现这些接口。
  第三个参数是InvocationHandler类型，用来将方法调用分派到的调用处理程序。其实就是调用在实现InvocationHandler接口时重写的invoke方法。
  ```
* 第二个需要注意的是InvocationHandler接口提供的invoke方法
  1. 动态创建代理类时使用的newProxyInstance方法第三个参数 h 接受一个InvocationHandler类型的实现类（myInvocation）。
  2. 执行proxy.rent(houseName)时，会先获取要执行的（Method）rent方法和参数houseName。
  3. 把第二步获取的（Method）rent方法和参数houseName 传给myInvocation的invoke()方法去执行。
  4. 在myInvocation的invoke方法里面添加额外的操作（谈价格，签合同等），最终method.invoke(target, args)利用反射去执行真正的rent方法
* 第三个需要注意的是method.invoke方法
  可以把method.invoke比作JavaScript提供的call方法。

总结：代理模式可以降低业务逻辑之间的耦合度，每一种设计模式都是一种思想，简单的demo无法展现它们真正的魅力，但很多优秀的开源代码都是各种设计模式的完美实现，比如强大的Spring。


#### 基于类：需要借助cglib工具
JDK Proxy代理的前提条件是被代理目标必须是某个接口的实现类，如果想直接代理一个普通的类，需要借助cglib实现
```
// 代理目标类Dog.java
public class Dog {
    public void speak() {
        System.out.println("汪汪");
    }
}
// MyInterceptor.java
public class MyInterceptor implements MethodInterceptor {
    @Override
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("前置操作");
        Object result = methodProxy.invokeSuper(o, args);
        System.out.println("后置操作");
        return result;
    }
}
// 测试 MyTest.java
public class MyTest {
    public static void main(String[] args) {
        Enhancer enhancer =new Enhancer();
        enhancer.setSuperclass(Dog.class);
        enhancer.setCallback(new MyInterceptor());
        Dog proxy = (Dog) enhancer.create();
        proxy.speak();

    }
}
// 输出结果：
前置操作
汪汪
后置操作
```
MethodInterceptor的作用类似JDK动态代理InvocationHandler接口，但其实现原理是不一样的。
<!-- #### 基于java字节码：借助javasist工具 -->