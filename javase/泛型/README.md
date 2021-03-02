# 泛型
### 什么是泛型

  泛型是一种类似”模板代码“的技术。如果了解TypeScript，对于泛型应该不会陌生。不了解TypeScript也没关系，泛型一样可以学得会。


> **个人理解：**
如果在JavaScript弱类型语言中，声明一个变量可以直接用JavaScript提供的关键(var与ES6提供的let,const)，比如var a = xxx; 至于变量a是String类型还是Number或者其他类型，取决于对a的赋值xxx,xxx是String类型则a是就是String类型。
而Java是一种强类型语言，通常无论是定义一个变量还是一个方法，都要指定变量的类型(比如int a = 0)，方法的返回值的类型和方法的参数类型（比如 public String getName(String name)）。
强类型的优点是不合法的代码无法通过编译，不会导致这种非法代码部署到业务环境中，一定程度上保障了代码的规范性和安全性。但是缺点就是不够灵活。

##### 泛型的诞生

1. 假如没有泛型的时候，根据ArrayList的特性（可以存储任何数据类型）应该是这样定义的：
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
    ```
    针对上面定义的ArrayList，应该这样使用：
    ```
    ArrayList list = new ArrayList();
    list.add("Hello");
    // 获取到Object，必须强制转型为String:
    String first = (String) list.get(0);
    ```
    这是我们前提知道list.get(0)是一种String类型，如果不知道list里面放的是什么数据类型就会很容易出现错误转型。

2. 如果把ArrayList变成一种模板：ArrayList<E>，则应该这样定义：
    ```
    public class ArrayList<E> {
        private E[] array;
        private int size;
        public void add(E e) {...}
        public void remove(int index) {...}
        public E get(int index) {...}
    }
    ```
    E可以是任何一种类型，再使用ArrayList则应该这样使用：
    ```
    // 只可以存放String的ArrayList
    ArrayList<String> strList = new ArrayList<String>();
    // 只可以存放Float的ArrayList:
    ArrayList<Float> floatList = new ArrayList<Float>();

    strList.add("xxx"); // OK
    String s = strList.get(0); // OK
    strList.add(new Integer(111)); // compile error!
    Integer n = strList.get(0); // compile error!
    ```
    现在就可以针对不同的类型创建不同的ArrayList，再取ArrayList里面的元素，就不用做强制类型转换了，因为strList只可以存放String类型的数据，floatList只可以存放Float类型的数据。
##### 向上转型
实际上定义ArrayList的代码是这样的：
```
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable
  {
    private int size;
    ...
  }
```
ArrayList<E>实现了List<E>接口，它可以向上转型为List<E>：
```
// ArrayList<E>可以向上转型为List<E>
List<Integer> list = new ArrayList<Integer>();
list.add(111);
Integer i = list.get(0);
```

但是ArrayList<Integer>不可以向上转型为ArrayList<Number>。假设可以这样做：
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

### 擦拭法
Java语言的泛型实现方式是擦拭法，擦拭法是指虚拟机对泛型其实一无所知，所有的工作都是编译器做的。Java中的泛型仅仅是给编译器javac使用的，确保数据安全性和免去类型强制转换，一旦编译完成，所有和泛型相关的类型全部擦除。
Java的泛型是由编译器在编译时实行的，编译器内部永远把所有类E视为Object处理，但是，在需要转型的时候，编译器会根据E的类型自动为我们实行安全地强制转型。

如果定义一个泛型类MyClass<E>，我们和编译器看到的代码是这样的：
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
1. 泛型<E>不能是基本类型（例如int），因为实际类型是Object，Object类型无法持有基本类型
    ```
    ArrayList<int> list = new ArrayList<int>(); // compile error!
    ```
2. 无法获取带泛型实例的Class

    由于擦拭法，ArrayList<E>中E是Object，所以ArrayList<String>和ArrayList<Integer>经过编译都会变成ArrayList<Object>，所以假设能获取泛型的Class，则对ArrayList<String>和ArrayList<Integer>获取class时，获取的是同一个class。
    假设能获取泛型的Class：
    ```
    ArrayList<String> strList = new ArrayList<String>();
    ArrayList<Integer> integerList = new ArrayList<Integer>();
    strList.getClass() == integerList.getClass();  // 如果可以获取他们的class，那么打印结果为：true
    ```
    事实上strList.getClass()就会编译错误。
3. 无法获取带泛型的Class
   无法获取ArrayList<String>.class，如果这么写编译器会报错，只能获取ArrayList.class，所以也无法判断带泛型的类型，比如:
   ```
    ArrayList<String> strlist = new ArrayList<String>();
    System.out.println(strlist instanceof ArrayList<String>); // 这样写编译器会报错
    System.out.println(strlist instanceof ArrayList); // 这样写是合法的 且打印结果为：true
   ```

4. 不能直接实例化E类型，要实例化E类型必须借助额外的Class<E>参数
   
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
虽然Integer是Number的子类，前面向上转型可知ArrayList<Integer>不可以向上转型为ArrayList<Number>，所以ArrayList<Integer>并不是ArrayList<Number>的子类。

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
然而当使用这个方法的时候会发现，不管传ArrayList<Integer>类型的变量还是
ArrayList<Long>、ArrayList<Float>都会报错，因为它们三个不是ArrayList<Number>的子类，它们三个都无法向上转型为ArrayList<Number>。
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

而实际上这种场景并不少见，要想解决这种问题的办法就是使用extends通配符ArrayList<? extends Number>，这样就可以接受所有Number或者Number子类的泛型（比如ArrayList<Integer>）。

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

由于ArrayList<? extends Number>可以接受Number或者其子类，而无法确定到底是Integer类型还是Float类型或者是Long类型，所以当获取ArrayList<? extends Number>里面的的元素类型也应该是Number类型：
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

这时会发现args.set(i, zero)这句代无法通过编译报错。代码看起来是没有问题的，但为什么会报错？

当前代码"看起来"是没有问题，传给setItem方法的intList是ArrayList<Integer>类型，setItem是可以接受ArrayList<Integer>类型的，传给args.set的zero也是Integer类型，一切"看起来"都是正常的。

但是不要忘了，setItem(ArrayList<? extends Number> args)除了可以接受ArrayList<Integer>类型，也可以接受ArrayList<Float>类型，如果传给setItem的参数args是ArrayList<Float>类型，观察以下代码：

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
再执行args.set(i, zero)就会出现把Integer类型的数据放到ArrayList<Float>中，而ArrayList<Float>是不会接受一个Integer类型的。所以为了避免这种情况发生，编译器是无法让他们通过编译的。
但有一个例外就是给set方法传null，如果改成args.set(i, null)就可以正常编译和执行了。

总结就是在使用<? extends Number>时，只能确定"?"是Number或者Number的子类，但无法确定"?"到底是哪种类型，所以当对这种类型数据使用set方法时，也无法确定该给set方法传一个什么类型的数据，所以不允许对<? extends Number>类型调用set方法（除非传null）。







