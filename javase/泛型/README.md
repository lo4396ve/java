# 泛型


泛型是一种类似”模板代码“的技术。如果了解TypeScript，对于泛型应该不会陌生。不了解TypeScript也没关系，泛型一样可以学得会。

### 为什么需要泛型
> **个人理解**：在JavaScript弱类型语言中，声明变量不需要指定变量的类型，变量的类型取决于对赋值的数据类型。
而Java是一种强类型语言，通常在声明的时候就要指定变量的类型(比如int a = 0)，强类型的优点是保证类型正确，不合法的类型不会通过编译，一定程度上保障了代码的规范性和安全性。缺点就是不够灵活。



##### 没有泛型的时候，根据ArrayList的特性应该是这样定义的：
```
public class ArrayList {
    /* 
    使用Object类型定义数组来存放各种数据，因为Object类是所有java类(Object类除外)的终极父类
    */
    private Object[] array;
    private int size;
    public void add(Object e) {...}
    public void remove(int index) {...}
    public Object get(int index) {...}
}
...
ArrayList list = new ArrayList();
list.add("Hello");
// 获取的是Object类型，必须强制转型为String，如果不知道list里面放的是什么数据类型就会很容易出现错误转型。
String first = (String) list.get(0);

```


##### 把ArrayList变成一种模板：ArrayList\<E>：
```
// E可以是任何一种类型，
public class ArrayList<E> {
    private E[] array;
    private int size;
    public void add(E e) {...}
    public void remove(int index) {...}
    public E get(int index) {...}
}

...

// 只可以存放String的ArrayList
ArrayList<String> strList = new ArrayList<String>();
strList.add("xxx"); // OK
String s = strList.get(0); // OK
strList.add(new Integer(111)); // compile error!
Integer n = strList.get(0); // compile error!

```

可以针对不同的类型创建不同的ArrayList，获取元素就不用强制类型转换了，因为strList只可以存放String类型的数据。

#### 向上转型
实际上ArrayList的源码是这样的：
```
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable
  {
    private int size;
    ...
  }
```
ArrayList\<E>实现了List\<E>接口，它可以向上转型为List\<E>：
```
// ArrayList<E>可以向上转型为List<E>
List<Integer> list = new ArrayList<Integer>();  // OK
list.add(1);
Integer i = list.get(0);    // OK
```

但是ArrayList\<Integer>不可以向上转型为ArrayList\<Number>。假设可以这样做：
```
ArrayList<Integer> integerList = new ArrayList<Integer>();
// 添加一个Integer：
integerList.add(new Integer(1));
// “向上转型”为ArrayList<Number>：
ArrayList<Number> numberList = integerList;
// 添加一个Float，因为Float也是Number：
numberList.add(new Float(1.1));
// 从ArrayList<Integer>获取索引为1的元素（即添加的Float）：
Integer n = numberList.get(1); // ClassCastException! 因为索引1元素是一个Float类型数据，Float不可以转型为Integer。
```

### 使用的泛型
#### 泛型类
```
class MyObject<K, V> {
    private K key;
    private V value;

    public MyObject(K k, V v) {
        this.key = k;
        this.value = v;
    }
    ...
    get and set
    ...
}
```

#### 泛型接口
```
interface MyInter<E> {
    public E getData();
}
```
泛型接口的实现分为指定接口泛型类型和不指定泛型接口类型，如果不指定泛型接口，那么实现类也必须是一个泛型类，如果指定泛型接口类型，实现类可以试泛型类也可以不是泛型类。

* 指定泛型接口类型，且实现类不是泛型类
    ```
    public class MyInterImpl implements MyInter<Integer> {

        @Override
        public Integer getData() {
            return null;
        }
    }
    ```
* 指定泛型接口类型，且实现类是泛型类
    ```
    public class MyInterImpl<E> implements MyInter<String> {
        private E e;

        @Override
        public E getData() {
            return null;
        }

        public E getE() {
            return e;
        }
    }
    ```
    这时候实现类MyInterImpl\<E>的泛型只是给调用改实现类的时候使用的，跟泛型接口MyInter\<E>没有任何关系
* 不指定泛型接口类型，实现类也必须是泛型类
    ```
    public class MyInterImpl<E> implements MyInter<E> {

        @Override
        public E getData() {
            return null;
        }
    }
    ```

#### 泛型方法
泛型方法可以存在于泛型类中，也可以在普通类中定义泛型方法。泛型方法规则：

```
修饰符  <T> 返回值 方法名(参数...){}
public <T>   T   myFun(ArrayList<T> list) {}

/*
* 修饰符：控制访问权限
* <T>:（重要）表示声明此方法为一个泛型方法
* 返回值：方法的返回值类型，可以是任何类型或者返回T类型
* 方法名：方法的名字
* 参数也可以是一个泛型类型，比如ArrayList<String>)或者ArrayList<T>)
*/
```

在普通类中声明泛型方法
```
public class Demo {
    // 声明泛型方法
    public <T> T fn(ArrayList<T> list) {
        for (T t : list) {
            System.out.println(t);
        }
        return list.get(0);
    }
    public static void main(String[] args) {
        Demo demo = new Demo();
        ArrayList<String> strList = new ArrayList<String>();
        strList.add("hello");
        strList.add("world");
        String res = demo.fn(strList);
    }
}
```

在泛型类中定义泛型方法
```
public class Demo<E> {
    // 声明泛型方法
    public <T> T fn(ArrayList<T> list) {
        for (T t : list) {
            System.out.println(t);
        }
        return list.get(0);
    }

    public static void main(String[] args) {

        ArrayList<Integer> list = new ArrayList<Integer>();
        list.add(1);

        Demo<String> demo = new Demo<String>();
        demo.fn(list);
    }
}
```

为什么需要泛型方法？观察以下示例：
```
public class Demo<E> {

    // 泛型方法
    public <T> void fn1(ArrayList<T> list) {
        for (T t : list) {
            System.out.println(t);
        }
    }
    // fn2不是泛型方法，是接受一个ArrayList<E>类型参数的普通方法
    public void fn2(ArrayList<E> list) {
        for (E e : list) {
            System.out.println(e);
        }
    }
    // fn3也不是泛型方法，是接受一个指定泛型类型ArrayList<Integer>参数的普通方法
    public void fn3(ArrayList<Integer> list) {
        for (Object o : list) {
            System.out.println(o);
        }
    }

    public static void main(String[] args) {
        ArrayList<String> strList = new ArrayList<String>();
        strList.add("hello");
        ArrayList<Integer> intList = new ArrayList<Integer>();
        intList.add(1);

        Demo<String> demo = new Demo<String>();
        demo.fn1(strList);
        demo.fn1(intList);

        demo.fn2(strList);
        demo.fn2(intList);  // error

        demo.fn3(strList);  // error
        demo.fn3(intList);

    }
}
```
泛型类实例化要指定泛型类型，示例中实例化Demo类时指定了Demo\<String>。
fn2的参数ArrayList\<E>受Demo\<String>实例化限制，只能接受ArrayList\<String>类型参数。
fn3指定了参数类型ArrayList\<Integer>，所以只接受ArrayList\<Integer>类型参数。
fn1泛型方法就没有这些局限性了，取决于参数传的类型，使用更加灵活。


### 擦拭法
Java语言的泛型实现方式是擦拭法，擦拭法是指虚拟机对泛型其实一无所知，所有的工作都是编译器做的。Java中的泛型仅仅是给编译器javac使用的，确保数据安全性和免去类型强制转换，一旦编译完成，所有和泛型相关的类型全部擦除。
Java的泛型是由编译器在编译时实行的，编译器内部永远把所有类E视为Object处理，但是，在需要转型的时候，编译器会根据E的类型自动为我们实行安全地强制转型。

如果定义一个泛型类MyClass\<E>，我们和编译器看到的代码是这样的：
```
public class MyClass<E> {
  private E arg;
  public MyClass(E arg) {
    this.arg = arg;
  }
  public E getArg() {
    return this.arg;
  }
  ...
}
// 使用MyClass
MyClass<String> myclass = new MyClass<String>("xxx");
String s = myclass.getArg();
```

编译器会擦除所有的泛型相关的类型，编译器内部永远把所有类E视为Object处理，所以编译后交付JVM执行的代码则是这样的：
```
public class MyClass {
  private Object arg;
  public MyClass(Object arg) {
    this.arg = arg;
  }
  public Object getArg() {
    return this.arg;
  }
  ...
}
// 使用MyClass
MyClass myclass = new MyClass("xxx");
String s = (String) myclass.getArg();
```

基于擦拭法的特性，泛型有以下几种限制：
1. **泛型\<E>不能是基本类型（例如int），因为实际类型是Object，Object类型无法持有基本类型**
    ```
    ArrayList<int> list = new ArrayList<int>(); // compile error!
    ```
2. **无法获取带泛型实例的Class**

    由于擦拭法，ArrayList\<E>中E是Object，所以ArrayList\<String>和ArrayList\<Integer>经过编译都会变成ArrayList\<Object>，所以假设能获取泛型的Class，则对ArrayList\<String>和ArrayList\<Integer>获取class时，获取的是同一个class。
    假设能获取泛型的Class：
    ```
    ArrayList<String> strList = new ArrayList<String>();
    ArrayList<Integer> integerList = new ArrayList<Integer>();
    strList.getClass() == integerList.getClass();  
    // 如果可以获取他们的class，那么打印结果为：true
    // 事实上strList.getClass()就会编译错误。
    ```
    
3. **无法获取带泛型的Class**
   无法获取ArrayList\<String>.class，这么写代码也不会通过编译，只能获取ArrayList.class，所以也无法判断带泛型的类型，比如:
   ```
    ArrayList<String> strlist = new ArrayList<String>();
    System.out.println(strlist instanceof ArrayList<String>); // 这样写编译器会报错
    System.out.println(strlist instanceof ArrayList); // 这样写是合法的 且打印结果为：true
   ```

4. **不能直接实例化E类型**
   
   ```
    // 定义带泛型的类MyClass<E>
    public class MyClass<E> {
      private E arg;
      public MyClass(E arg) {
        /* 
        new E()编译器会报错，因为擦拭法，new E()实际上执行的是new Object()。
        这样在使用MyClass<E>类时不管是new MyClass<String>还是new MyClass<Integer>都变成Object，编译器是不允许这样做的
        */
        arg = new E();  
      }
      public E getArg() {
        return this.arg;
      }
    
    }
   ```

   要想实例化E类型，可以借助Class<E>通过反射实例化E类型
   ```
    class MyClass<E>{
        private E arg;
        public MyClass(Class<E> e) throws IllegalAccessException, InstantiationException {
            arg = e.newInstance();
        }
        ...
    }
   ```

### 通配符
#### extends通配符

##### 为什么需要extends通配符
虽然Integer是Number的子类，现在已经知道ArrayList\<Integer>不可以向上转型为ArrayList\<Number>，所以ArrayList\<Integer>并不是ArrayList\<Number>的子类。

假如有这么个场景，定义了一个方法，用来计算一个ArrayList列表里面所有元素的和，已知列表每个元素都是Number类型，应该这样定义这个方法：

```
public static Number reduce(ArrayList<Number> args) {
    int res = 0;
    for(int i = 0; i < args.size(); i++) {
        Number item = args.get(i);
        res = res + item.intValue();
    }
    return res;
}
```
然而当使用这个方法的时候，不管传ArrayList\<Integer>还是ArrayList\<Float>都会报错，因为它们不是ArrayList\<Number>的子类，无法向上转型为ArrayList\<Number>。

完整示例：

```
public class Demo {
    public static void main(String[] args) {
        ArrayList<Integer> intList = new ArrayList<Integer>();
        intList.add(1);
        intList.add(2);
        intList.add(3);
        /*
          reduce(intList)会报错，因为ArrayList<Integer>不是ArrayList<Number>子类，
          所以reduce(ArrayList<Number> args)不接受参数类型ArrayList<Integer>
        */
        Number n = reduce(intList); // error
        System.out.println(n);
    }

    public static Number reduce(ArrayList<Number> args) {
        int res = 0;
        for(int i = 0; i < args.size(); i++) {
            Number item = args.get(i);
            res = res + item.intValue();
        }
        return res;
    }
}
```
上面代码只有传ArrayList\<Number>不会报错。为了能接受ArrayList\<Integer>、ArrayList\<Float>等类型，把reduce方法参数改成extends通配符ArrayList<? extends Number>方式，就可以接受ArrayList\<Number的子类>了。

修改后的代码为：
```
public class Demo {
    public static void main(String[] args) {
        ArrayList<Integer> intList = new ArrayList<Integer>();
        intList.add(1);
        intList.add(2);
        intList.add(3);
        Number n = reduce(intList); // 正常执行
        System.out.println(n);
    }

    public static Number reduce(ArrayList<? extends Number> args) {
        int res = 0;
        for(int i = 0; i < args.size(); i++) {
            Number item = args.get(i);
            res = res + item.intValue();
        }
        return res;
    }
}
```

这种声明泛型的方式<? extends Number>的方式成为上界统通配符，也叫上限通配符，它把泛型E的类型的上限定在了Number，只接受Number或者其子类。

##### extends通配符的get方法

由于ArrayList<? extends Number>可以接受Number或者其子类，而无法确定到底是Integer类型还是Float类型或者是Long类型，所以获取ArrayList<? extends Number>的元素，类型也应该是Number类型：
```
Number item = args.get(0);  // 正常
Integer item = args.get(0); // error，因为无法确定获取的元素一定是Integer类型，如果元素是Float类型，会导致转型错误，为了避免这种错误发生，直接在编辑器中就会提示错误
```

##### extends通配符的set方法

如果有这种场景，需要把列表中的每个元素都变成0，则需要添加一个遍历列表并对每个元素设置值的方法：

```
public class Test05 {
    public static void main(String[] args) {
        ArrayList<Integer> intList = new ArrayList<Integer>();
        intList.add(1);
        intList.add(2);
        intList.add(3);
        
        setItem(intList)
    }

    public static Number reduce(ArrayList<? extends Number> args) {
        int res = 0;
        for(int i = 0; i < args.size(); i++) {
            Number item = args.get(i);
            res = res + item.intValue();
        }
        return res;
    }
    public static void setItem(ArrayList<? extends Number> args) {
        for(int i = 0; i < args.size(); i++) {
            Integer zero = 0;
            args.set(i, zero);  // error
        }

    }
}
```

发现args.set(i, zero)无法通过编译报错。代码看起来是没有问题的，但为什么会报错？

代码"看起来"是没有问题，传给setItem方法的intList是ArrayList\<Integer>类型，setItem是可以接受ArrayList\<Integer>类型的，传给args.set的zero也是Integer类型，一切"看起来"都是正常的。

请不要忘了，setItem(ArrayList<? extends Number> args)除了可以接受ArrayList\<Integer>类型，也可以接受ArrayList\<Float>类型，如果传给setItem的参数args是ArrayList\<Float>类型，观察以下代码：

```
public class Test05 {
    public static void main(String[] args) {
        ArrayList<Float> floatList = new ArrayList<Float>();
        floatList.add(new Float(0.1));
        floatList.add(new Float(0.2));
        floatList.add(new Float(0.3));
        setItem(floatList);
    }

    public static Number reduce(ArrayList<? extends Number> args) {
        int res = 0;
        for(int i = 0; i < args.size(); i++) {
            Number item = args.get(i);
            res = res + item.intValue();
        }
        return res;
    }
    public static void setItem(ArrayList<? extends Number> args) {
        for(int i = 0; i < args.size(); i++) {
            Integer zero = 0;
            args.set(i, zero);  // error 
        }

    }
}
```
再执行args.set(i, zero)就会出现把Integer类型的数据放到ArrayList\<Float>中，而ArrayList\<Float>是不会接受一个Integer类型的。所以为了避免这种情况发生，编译器是无法让他们通过编译的。
但有一个例外就是给set方法传null，如果改成args.set(i, null)就可以正常编译和执行了。

总结就是在使用<? extends Number>时，只能确定"?"是Number或者Number的子类，但无法确定"?"到底是哪种类型，所以当对这种类型数据使用set方法时，也无法确定该给set方法传一个什么类型的数据，所以不允许对<? extends Number>类型调用set方法（除非传null）。

#### super通配符
##### 为什么需要super通配符
假设有这样的场景，把一个存放Number类型的ArrayList元素写入到磁盘文件：

```
public class Demo<E> {
    // 写入磁盘的方法
    public static void write(ArrayList<Number> list) {
        for (Object obj : list) {
            System.out.println(obj);
            // 模拟写入
        }
    }

    public static void main(String[] args) {
        // Number类型
        ArrayList<Number> numList = new ArrayList<Number>();
        Number num = 1;
        numList.add(num);
        write(numList);
    }
}
```
随着需求变更，现在需要写入其他类型（比如Object类型），显然上面的实例是不支持的。可以利用super通配符实现改功能：
```
public class Demo<E> {

    public static void write(ArrayList<? super Number> list) {
        for (Object obj : list) {
            System.out.println(obj);
            // 模拟写入
        }
    }

    public static void main(String[] args) {
        // Number类型
        ArrayList<Number> numList = new ArrayList<Number>();
        Number num = 1;
        numList.add(num);
        write(numList);

        // String类型
        ArrayList<Object> objList = new ArrayList<Object>();
        String str = "hello";
        objList.add(str);
        write(objList);
    }

}

```
\<? super Number>支持所有Number的父类，声明ArrayList\<Object>指定泛型类型是Object，Object是Number的父类，所以代码可以正常运行。
虽然String str = "hello"是String类型，ArrayList\<Object>指定了泛型类型Object，String类型可以向上转型为Object。

##### super通配符的set方法
假设需求变成在写入之前先判断元素是不是null，如果是null则把元素值设为0否则设为1，并且0和1都是Number类型，需要对write进一步扩展：
```
public static void write(ArrayList<? super Number> list) {
    for(int i = 0; i < list.size(); i++) {
        System.out.println(list.get(i));
        Number num;
        if(list.get(i) == null) {
            num = 0;
            list.set(i, num);
        }else {
            num = 1;
            list.set(i, num);
        }
        // 模拟写入
    }
}
```

参考ArrayList以及其set方法的源码：
```
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    ...
    public E set(int index, E element) {
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
    ...
}
```

已知write传入的是ArrayList<? super Number>类型，所以ArrayList的set方法签名实际上是：
```
public <? super Number> set(int index, <? super Number> element) {
    ...
}
```
所以set的第二个参数element接受Number类型。


##### super通配符的get方法

加入在write方法for循环中想获取元素：
```
public static void write(ArrayList<? super Number> list) {
    for(int i = 0; i < list.size(); i++) {
        Number item = list.get(i);  // error
        // 模拟写入
    }
}
```
Number item = list.get(i)会报编译错误。

参考ArrayList以及其get方法的源码：
```
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    ...
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }
    ...
}

```
已知write传入的是ArrayList<? super Number>类型，所以ArrayList的get方法签名实际上是：
```
public <? super Number> get(int index) {
    ...
}
```
get返回值类型是Number或者Number的父类（比如Object类型）,因此无法使用Number类型来接收get()的返回值。
唯一可以接受get()返回值的是Object类型：

```
Object item = list.get(i);  // OK
```

#### 对比extends和super通配符
\<? extends E>类型和<? super E>类型的区别在于：
* \<? extends E>允许调用读方法E get()获取E的引用，但不允许调用写方法set(E)传入E的引用（传入null除外）
* \<? super E>允许调用写方法set(E)传入E的引用，但不允许调用读方法E get()获取E的引用（获取Object除外）

一个是允许读不允许写，另一个是允许写不允许读。

#### 无限定通配符
无限定通配符，即只定义一个?。无线定通配符不允许调用set(E)方法并传入引用（null除外），也不允许调用E get()方法并获取E引用（除非用Object引用）。既不能读，也不能写，只能做一些null判断。

无限定通配符有一个独特的特点，就是：ArrayList<?>是所有ArrayList<E>的超类。


