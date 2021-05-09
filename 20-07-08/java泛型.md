## 泛型

### 概念

从java5后,java引入了**参数化类型(parameterized type)**的概念,允许程序在创建集合是指定集合元素的类型.例如`List<String>list`.java的参数化类型被成为**泛型(Grneric)**

泛型的一个设计原则就是:只要代码在编译时没有出现警告,就不会遇到ClassCastException

```java
    public static void main(String[] args) throws Exception {
        // 创建一个只能保存字符串的List集合
       List<String>list = new ArrayList<>();
       list.add("aa");
       list.add("bb");
    }
```

### Java7泛型的**菱形**语法

在Java7以前,如果使用带泛型的接口,类定义变量,那么调用构造器创建对象时构造器后面也必须带泛型,这样就显得多余了.

例如:

```java
// 创建的对象构造器后面也得加泛型
List<String>list = new ArrayList<String>();
Map<String,Object>map = new HashMap<String,Object>();
```

Java7开始,Java允许在构造器后不需要完整的泛型信息,只要给出一对尖括号(\<\>)就行,java可以推断尖括号里因该是什么泛型信息.

例如:

```java
List<String>list = new ArrayList<>();
Map<String,Object>map = new HashMap<>();
```

### 使用泛型

#### 一. 定义泛型接口,类

java5改写后的List接口,Iterator接口,Map代码片段

```java
// list接口
// 定义接口时指定一个类型形参,该形参名为E
public interface List<E> extends Collection<E> {
    // 在该接口中,E可以作为一种类型使用
    boolean add(E e);
    Iterator<E> iterator(); // ①注意这里的返回类型
}
// Iterator接口
public interface Iterator<E> {
    E next();
}
// Map接口
public interface Map<K, V> {
    V put(K key, V value);
    Set<K> keySet(); // ②注意这里的返回类型
}
```

泛型的实质: 允许在定义接口,类停驶声明类型形参,类型形参在整个接口,类内可以当成类型使用,几乎所有可使用普通类型的地方都可以使用这种类型形参.

除此之外,①②处声明的返回类型时Iterator\<E\>,Set\<K\>,这表明Set\<K\>是一种特殊的数据类型,可以认为是Set类型的字类.

通过这种方式,虽然程序只定义了一个List\<E\>接口,但实际使用时可以产生无数的List\<E\>接口,只要为E传入不同的类型实参,系统就会多出一个新的List子接口.(这里说的子接口并不存在,只是一种方便理解的说法)

#### 二. 从泛型类派生子类

```java
// 如果写死父类泛型则没问题
public class A extends Apple<String>{}
// 或者不指定泛型类型
public class A extends Apple{}
// 还有一种写法
public class A<T> extends Apple<T>{}
// 错误写法: 定义类A继承Apple类,Apple类不能跟类型新参
public class A extends Apple<T>{}
```

以上写法,接口同理

#### 三.并不存在泛型类

上面说可以把ArrayList\<String\>看成ArrayList的字类,但是系统不会创建ArrayList\<String\>的class文件,而且也不会当成别的类型处理.

```java
public static void main(String[] args) throws Exception {
        // 声明两个泛型不同的list
        List<String> list = new ArrayList<>();
        List<Integer>list2 = new ArrayList<>();
	    // 结果为true表示类型没变
        boolean isSame = list.getClass() == list2.getClass();// true
    }
```

由于系统中并不会真正的生成泛型类,所以instanceof运算符后面不能有泛型类.

例如:

```java
public static void main(String[] args) throws Exception {
        Collection<String> c = new ArrayList<>();
        // 这样写就错了
        boolean flag = c instanceof ArrayList<String>;
        // 可以这样写
        boolean flag2 = c instanceof ArrayList;
    }
```

#### 四. 类型统配符

首先看一段代码:

```java
    void show(List list) {
        for (int i = 0; i < list.size(); i++) {
            System.out.println(list.get(i));
        }
    }
```

这是一段普通的遍历List集合的代码.

但是由于List是一个有泛型的接口,这里使用List时没有使用泛型(不用泛型其实都可以,但你都看到这了,那就考虑用泛型呗),需要考虑添加泛型.

问题又来了,这里集合形参的类型是不确定的,那鲁迅就说过了: 类型不确定?就用Object

使用泛型的代码:

```java
    void show(List<Object> list) {
        for (int i = 0; i < list.size(); i++) {
            System.out.println(list.get(i));
        }
    }
```

好了那就开用吧:

```java
    public static void main(String[] args) throws Exception {
        MainTest m = new MainTest();
        List<String>list= new ArrayList<>();
        list.add("aaa");
        list.add("bbb");
        list.add("ccc");
        m.show(list); // 这里报错了
    }
```

上面使用发生编译错误: 

`java: 不兼容的类型: java.util.List<java.lang.String>无法转换为java.util.List<java.lang.Object>`

这表明List\<String\>对象不能当成List\<Object\>对象使用,也就是说,List\<String\>类并不是List\<Object\>类的字类.

实际上如果A是B的父类,而G是具有泛型声明的类或接口,G\<B\>并不是G\<A\>的字类,这一点非常值得注意,因为这和大部分人的习惯认为是不同的.



与数组进行对比,先看下数组是如何工作的.在数组中可以直接把Integer[]数组赋给一个Number[]变量.但是这是不安全的.

实例代码:

```java
    public static void main(String[] args) throws Exception {
        Number[]arr = new Integer[10];
        arr[0] = 0.5;
        //运行报错: java.lang.ArrayStoreException: java.lang.Double
    }
```

这里Java允许Integer[]数组赋值给Number[]是一种不安全的设计,因次才Java在泛型设计时进行了改进,它不在允许类似List\<Integer\>对象赋值给List\<Number\>的情况发生.

例如:下面的代码将导致编译错误:

```java
public static void main(String[] args) throws Exception {
    List<Integer>list2 = new ArrayList<>();
    // 编译错误:不兼容的类型: java.util.List<java.lang.Integer>无法转换为java.util.List<java.lang.Number>
    List<Number>list = list2;
}
```

好了说了一大堆,那么到底怎样才能处理**形参类型不确定**的问题呢.答案是: **使用类型通配符号**

##### 使用类型通配符

类型统配符就是一个问号(?),将一个问号作为类型实参传给List集合.

上面的show方法修改如下:

```java
    void show(List<?> list) {
        for (int i = 0; i < list.size(); i++) {
            System.out.println(list.get(i));
        }
    }
```

现在不管任何类型的List调用它,都是OK的,其类型是Object,这永远是安全的.

上面程序中使用的List\<?\>,这种写法可以使用于任何支持泛型声明的接口和类,比如Set\<?\>,Map\<?,?\>等.

**注意:**这种带统配符的List仅表示它是各种泛型List的父类,并不能把元素添加到其中.例如,下面的代码将会引起编译错误:

```java
    public static void main(String[] args) throws Exception {
        List<?>list= new ArrayList<>();
        // 编译错误: 不兼容的类型: java.lang.String无法转换为capture#1, 共 ?
        list.add("aaa");
    }
```

因为程序无法确定list集合中的元素,所以不能向其中添加对象.

更具前面的List\<T\>接口定义的代码可知,add()方法有类型参数T作为元素类型,所以传给add的参数必须是T类的对象.但因为该例中不知道T是什么类型,所以程序无法将对象*丢进*该集合.唯一的例外是null,因为它是所以类型的实例.

另一方面,List<?>返回的对象都是Object类型的.

##### 设定类定统配符的上限

当直接使用List\<?\>这种形式时,即表明这个List集合可以是任何泛型List的父类.但还有一种特殊的情形,程序不希望这个List\<?\>是任何泛型List的父类,只希望它代表某一类泛型List的父类.比如希望这个集合中的元素都是Number字类类型的

再次修改show方法:

```java
    void show(List<? extends Number> list) {
        for (int i = 0; i < list.size(); i++) {
            System.out.println(list.get(i));
        }
    }
```

调用

```java
    public static void main(String[] args) throws Exception {
        List<Integer> list = new ArrayList<>();
        List<String> list2 = new ArrayList<>();
        MainTest m = new MainTest();
        m.show(list);
        m.show(list2);// 编译错误
        // 报错: 不兼容的类型: java.util.List<java.lang.String>无法转换为java.util.List<? extends java.lang.Number>
    }
```

##### 设定类定统配符的下限

```java
public static <T> void copy(List<? super T> dest, List<? extends T> src) {}
```

使用? super T表示该类型需要是T类型,或者T类型的父类

##### 设定类型形参上限

Java泛型不仅允许在使用统配符形参时设定上限,而且可以在定义类型新参时设定上限,用于表示传给该类型形参的实际类型是指定类型的类型,或者是其字类.

实例:

```java
class Apple<T extends Number>{
    T col;
    public static void main(String[] args) {
        // 只要泛型是Number或者Number子类型都没问题
        Apple<Number>a1 = new Apple<>();
        Apple<Integer>a1 = new Apple<>();
        Apple<Double> a2= new Apple<>();
        // 编译错误,String不是Number类型或其字类
        Apple<String> a3 = new Apple<>();
    }
}
```

还有一种特殊的情况,就是要为类型形参设置多个上限(至多一个父类上限,可以有多个接口上限).表明该类型形参必须是其父类的字类(父类本身也行),并且实现了多个上限接口.

实例: 

```java
class Apple<T extends Number & java.lang.Comparable>{
    T col;

    public static void main(String[] args) {
        Apple<Integer>a1 = new Apple<>();
        Apple<Double> a2= new Apple<>();
        // String类型虽然实现了Comparable接口,但不是Number类型
        Apple<String> a3 = new Apple<>();
    }
}
```

指定多个上限时,类上限必须是第一位,按上面的实例,类型必须首先是Number类型.

#### 五.泛型方法

上面说了在定义类接口时可以使用类型形参,在该类的方法定义和成员变量定义,接口丶方法定义中,这类类型形参可以被当成不同类型来用.在另外一些情况下,定义类,接口时没有使用类型形参,但定义方法时想自己定义类型形参,也是可以的,Java5提供了泛型方法的支持.

##### 定义泛型方法

先看一段代码:

```java
    static void fromArrayToCollection(Object[] arr, Collection<Object> c) {
        for (Object item : arr) {
            c.add(item);
        }
    }
```

上面这段代码乍一看没问题,但其实Collection\<Object\>使得arr数组只能是Object类型,其他任何类型都不行(例如其子类String,Integer).

```java
String[]arr = new String[]{"aaa","bbb"};
List<String>strList = new ArrayList<>();
// 下面代码编译错误: Collection<Object>对象不能当成Collection<Object>使用
fromArrayToCollection(arr,strList);
```

这个问题上面讲类型统配符不是说过么?直接把Collection\<Object\>改成Collection\<?\>不就行了么.但其实不行,因为Java不允许把对象放进一个未知类型的集合中.

为了解决这个问题,可以使用Java5提供的泛型方法(Generic Method).所谓泛型方法,就是在声明方法时定义一个或多个类型形参.

泛型方法的用法格式如下:

`修饰符<T,S>返回值类型 方法名(形参列表){}`

具体使用代码:

```java
    // 修饰符<T,S>返回值类型 方法名(形参列表){}
    static <T> void fromArrayToCollection(T[] arr, Collection<T> c) {
        for (T item : arr) {
            c.add(item);
        }
    }
```

调用:

```java
	public static void main(String[] args) throws Exception {
        String[]arr = {"朴朴乐","朴人勇","朴一生"};
        List<String>list= new ArrayList<>();
        fromArrayToCollection(arr,list);
        System.out.println(list);// [朴朴乐, 朴人勇, 朴一生]
    }
```

与接口,类声明中定义的类型不同:

该泛型方法定义了一个T类型形参,这个T类型形参就可以在该方法内当成普通类型使用.与接口,类声明中定义的类型形参不同的是,方法声明的只能在方法内使用,而接口,类声明的类型形参则可以在整个接口,类中使用.

还有一点不同的就是,方法中的泛型参数无须显示传入实际类型参数.如上面程序所示,调用fromArrayToCollection时,不需要传入参数类型,但系统仍然可以知道类型形参的数据类型,因为编译器根据实参类型判断类型形参的值.

再看一段代码:

```java
    static <T> void copyCollection(Collection<T> from, Collection<T> to) {
        for (T item : from) {
            to.add(item);
        }
    }
```

调用:

```java
    public static void main(String[] args) throws Exception {
        List<Integer> list1 = new ArrayList<>();
        List<Double> list2 = new ArrayList<>();
        copyCollection(list1,list2);// 编译报错
        /*
  java: 无法将类 MainTest中的方法 copyCollection应用到给定类型;
  需要: java.util.Collection<T>,java.util.Collection<T>
  找到: java.util.List<java.lang.Integer>,java.util.List<java.lang.Double>
  原因: 推论变量T具有不兼容的等式约束条件java.lang.Double,java.lang.Integer
        */
    }
```

上面的只指定了一种类型,使用时却用了两种,报错是必然的.

##### 泛型方法与统配符的区别

我们用两种方式声明同一个方法(Collections.copy,有改动)

使用泛型方法:

```java
    public static <T, S extends T> void copy2(List<T> dest, List<S> src) {}
```

同时使用泛型方法和类型统配符:

```java
    public static <T> void copy(List<T> dest, List<? extends T> src) {}
```

后一个方法似乎更简洁,而且注意类型形参S,它仅仅使用了一次,其他参数的类型,方法返回值类型都不依赖与它,那么类型形参S就没有存在的必要了,既可以使用类型统配符代替S.

使用类型统配符还有一个好处:super关键子.

其实Colleciotns.copy的源方法声明(本人使用JDK11)代码是:

```java
public static <T> void copy(List<? super T> dest, List<? extends T> src) {}
```

代码很好理解,就是dest中元素可以实T类型的父类类型.

我本以为泛型方法声明也能使用super关键字,就像这样

```java
public static <T, A extends T,B super T> void copy2(List<B> dest, List<A> src) {}
```

但实际上不行,B super T是错误的.

#### 六.Java7的菱形语法与泛型构造器

正如泛型方法允许再方法签名中声明类型形参一样,Java也允许再构造器签名中声明类型形参,这样就产生了所谓的泛型构造器.

代码:

```java
class Foo {
    public <T> Foo(T t) {
        System.out.println(t);
    }
}
```

```java
    public static void main(String[] args) throws Exception {
        // 这两种写法一样
        new Foo("aaa");
        new <String>Foo("aaa");
        // 但是如果传入的类型形参和传入形参不一致就不行了
        new <String>Foo(11);// 编译错误: 不兼容的类型: int无法转换为java.lang.String
    }
```

**注意 类,接口上的类型形参和构造方法上的类型形参是两个东西.**

先声明一个泛型类:

```java
class Too<T> {
    public <E> Too(E e) {
        System.out.println(e);
    }
}
```

使用:

```java
// Foo类声明中的T是String类型
// 泛型构造器中声明的E类型是Integer类型
Too<String>t1 = new Too<>(5);
// 显示指定泛型构造器中的E类型是Integer类型,T是String类型
Too<String> t2 = new <Integer>Too<String>(1);
// 如果显式的指定E类型是Integer类型,就不能使用菱形语法了
// 也就是说得改成上面t2对象样,或者都不写就是t1对象那样.
Too<String> t3 = new <Integer>Too<>(1);// 编译错误
```

#### 七.泛型方法和方法重载

上面说过统配符可以设置上限和下限,从而允许一个类里包含如下两个方法

```java
    public static <T> void copy(List<? super T> dest, List<? extends T> src) {}

    public static <T> void copy(T dest, List<? extends T> src) {}
```

根据泛型的规则,上面两个方法的签名确实不一样,当时这种区别不是很明显:

两个参数都是Collection对象,第一个集合里的元素都是第二个集合里的元素的父类

#### 八. Java8改进的类型推断

java8改进了泛型方法的类型推断能力:

1. 可以通过调用方法的上下文推断类型参数的目标类型
2. 可在方法调用链中,将推断得到的类型参数传递到最后一个方法

声明一个泛型类,注意类型参数E和T的区别

```java
class Apple<E> {
    public static <T> Apple<T> nil() {
        return null;
    }

    public static <T> Apple<T> cons(T head, Apple<T> tail) {
        return null;
    }

    E head() {
        return null;
    }
}
```

使用:

```java
    public static void main(String[] args) throws Exception {
        // 可以通过方法赋值的目标来推断类型参数(T)为String
        Apple<String> a1 = Apple.nil();
        // 这个写法和上面的写法效果是一样的
        Apple<String> a2 = Apple.<String>nil();
        // 下面第一个参数是Integer类型,java推断出类型参数(T)是Integer
        // 那么第二个参数类型就是Apple<Integer>类型,于是Apple.nil()中的参数类型(T)也变为Integer了
        Apple.cons(21, Apple.nil());
        // 下面写法和上面效果一样
        Apple.cons(21, Apple.<Integer>nil());
    }
```

虽然Java8泛型推断能力增强,但也不是万能的,例如下面的代码:

```java
// 编译错误:不兼容的类型: java.lang.Object无法转换为java.lang.String
String s = Apple.nil().head();
```

因次需要指定参数类型,即改成如下形式:

```java
String s = Apple.<String>nil().head();
```

#### 九.擦除与转换

再严格的泛型代码里,带泛型声明的类总应该带着类型参数.但为了与老Java代码保持一致,也允许再使用带泛型声明的类时不指定实际的参数.

如果没有为这个泛型类指定实际的类型参数,则该类型默认是声明该类型参数时指定的第一个上限类型.

比如把List\<Stirng\>转换未List,则改List则该List对集合的类型检查变成了类型参数的上限(即Object).

下面程序示范了这种擦除:

```java
class Apple<T extends Number> {
    T size;

    public Apple() {
    }
    public Apple(T size) {
        this.size = size;
    }
    public void setSize(T size) {
        this.size = size;
    }
    public T getSize() {
        return this.size;
    }
}
```

```java
    public static void main(String[] args) throws Exception {
        Apple<Integer> a= new Apple<>();
        // 这是返回的是Integer类型
        Integer size1 = a.getSize();
        // 把a对象赋值给Apple变量,丢失尖括号里的类型信息
        Apple b = a;
        // 编译错误:不兼容的类型: java.lang.Number无法转换为java.lang.Integer
        Integer size2 = b.getSize();
    }
```

还有一种泛型转换的特殊情况.比如从逻辑上看List\<Stirng\>是List的字类,如果直接把Lsit对象赋值给一个List\<String\>对象应该引起编译错误,但实际上不会,可以直接把List对象赋值给Lsit\<Stirng\>对象.

```java
    public static void main(String[] args) throws Exception {
        List<Integer> l1 = new ArrayList<>();
        l1.add(1);
        l1.add(3);
        List l2 = l1;
        // 下面代码,只会引起'未经检查的转换'编译警告,但不会报错.
        List<String> l3 = l2;
        // 但只要访问l3中的元素,就会发生运行时异常:java.lang.ClassCastException: class java.lang.Integer cannot be cast to class java.lang.String
        System.out.println(l3.get(0));
    }
```

错报错信息上可以看出,底层java就是对元素进行了强转,和下面的代码一样:

```java
List li = new ArrayList();
li.add(2);
System.out.println((String)li.get(0));
```



