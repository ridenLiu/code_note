## Objects 与 Object 区别

**Object** 是 Java 中所有类的基类，位于java.lang包。

**Objects** 是 Object 的工具类，位于java.util包。它从jdk1.7开始才出现，被final修饰不能被继承，拥有私有的构造函数。

它由一些静态的实用方法组成，这些方法是null-save（空指针安全的）或null-tolerant（容忍空指针的），用于计算对象的hashcode、返回对象的字符串表示形式、比较两个对象。

### Objects中的方法

#### equals

Object.equals方法容易抛出空指针异常,以前常常把常量作为方法调用者(类似:"aaa".equals(str);)来避免NPE

Objects中的equals是null-save的,则不必考虑这个问题.

源码:

```java
    public static boolean equals(Object a, Object b) {
        return (a == b) || (a != null && a.equals(b));
    }
```

#### deepEquals

顾名思义，深度比较两个对象。

当参数是数组对象，其方法内部采用的是Arrays.deepEquals0方法的算法。

使用Objects.deepEquals方法有个好处，当我们在写业务代码时，可以直接使用此方法来判断两个复杂类型，

比如使用了泛型的列表对象List<T>、或者通过反射得到的对象，不清楚对象的具体类型。

源码如下：

```java
public static boolean deepEquals(Object a, Object b) {
        if (a == b)
            return true;
        else if (a == null || b == null)
            return false;
        else
            return Arrays.deepEquals0(a, b);
    }
```

说明下Arrays.deepEquals0方法：

   如果参数是Object类型的数组，则调用Arrays.deepEquals方法，在参数数组的循环中，递归调用deepEquals0，直到出现不相同的元素，或者循环结束；

   如果参数是基本类型的数组，则根据该类型调用Arrays.equals方法。Arrays工具类依照八种基本类型对equals方法做了重载。

#### hashCode

返回一个整型数值，表示该对象的哈希码值。若参数对象为空，则返回整数0；若不为空，则直接调用了Object.hashCode方法。

源码如下：

```java
public static int hashCode(Object o) {
        return o != null ? o.hashCode() : 0;
    }
```

Object支持hashCode方法是为了提高哈希表（例如java.util.HashMap 提供的哈希表）的性能。

以集合Set为例，当新加一个对象时，需要判断现有集合中是否已经存在与此对象相等的对象，如果没有hashCode()方法，需要将Set进行一次遍历，并逐一用equals()方法判断两个对象是否相等，此种算法时间复杂度为o(n)。通过借助于hasCode方法，先计算出即将新加入对象的哈希码，然后根据哈希算法计算出此对象的位置，直接判断此位置上是否已有对象即可。

（注：Set的底层用的是Map的原理实现）

#### hash

为一系列的输入值生成哈希码，该方法的参数是可变参数。

源码如下：

```java
public static int hash(Object... values) {
        return Arrays.hashCode(values);
    }
```

它是将所有的输入值都放到一个数组，然后调用Arrays.hashCode(Object[])方法来实现哈希码的生成。

对于当一个对象包含多个成员，重写Object.hashCode方法时，hash方法非常有用。

举个Java源码中的例子：

java.lang.invoke.MemberName 类，该类有Class<?> clazz、String name、Object type、int flags、Object resoulution这几个成员变量，

该类的hashCode方法如下：

```java
    @Override
    public int hashCode() {
        return Objects.hash(clazz, getReferenceKind(), name, getType());
    }
```

 警告:**当提供的参数是一个对象的引用，返回值不等于该对象引用的散列码。**这个值可以通过调用hashCode方法来计算。

####  toString

#### 　　***toString(Object o)***

　　返回指定对象的字符串表示形式。如果参数为空对象null，则返回字符串“null”。

　　源码:

```java
    public static String toString(Object o) {
        return String.valueOf(o);
    }
```

　　String.valueOf(Object obj)源码:

```java
    public static String valueOf(Object obj) {
        return (obj == null) ? "null" : obj.toString();
    }
```

　　Object.toString()源码:

```java
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
```

#### 　　***toString(Object o, String nullDefault)***

返回指定对象的字符串表示形式。如果参数为空对象null，则返回第二个参数nullDefault所指定的对象。

源码:

```java
public static String toString(Object o, String nullDefault) {
        return (o != null) ? o.toString() : nullDefault;
    }
```

#### compare

如果两个参数相同则返回整数0。因此，如果两个参数都为空对象null，也是返回整数0。

注意：如果其中一个参数是空对象null，是否会抛出空指针异常NullPointerException取决于排序策略，如果有的话，则由Comparator来决定空值null。

源码:

```java
public static <T> int compare(T a, T b, Comparator<? super T> c) {
        return (a == b) ? 0 :  c.compare(a, b);
    }
```

#### requireNonNull

　　**requireNonNull(T obj)**

　　检查指定类型的对象引用不为空null。当参数为null时，抛出空指针异常。设计这个方法主要是为了在方法、构造函数中做参数校验。

　　源码如下:

```java
    public static <T> T requireNonNull(T obj) {
        if (obj == null)
            throw new NullPointerException();
        return obj;
    }
```

​		**requireNonNull(T obj, String message)**　

　　该方法是requireNonNull的重载方法，当被校验的参数为null时，根据第二个参数message抛出自定义的异常消息。

　　源码如下：

```java
public static <T> T requireNonNull(T obj, String message) {
        if (obj == null)
            throw new NullPointerException(message);
        return obj;
    }
```

​		**requireNonNull(T obj, Supplier\<String\> messageSupplier)**

　检查指定的对象引用不为空null，如果是空，抛出自定义的空指针异常。从jdk1.8开始。

　　与requireNonNull(Object, String)方法不同，本方法允许将消息的创建延迟，直到空检查结束之后。

　　虽然在非空例子中这可能会带来性能优势， 但是决定调用本方法时应该小心，创建message supplier的开销低于直接创建字符串消息。　　

　　源码如下：

```java
public static <T> T requireNonNull(T obj, Supplier<String> messageSupplier) {
        if (obj == null)
            throw new NullPointerException(messageSupplier.get());
        return obj;
    }
```

#### isNull

判空方法，如果参数为空则返回true。从jdk1.8开始。

源码如下：

```java
 public static boolean isNull(Object obj) {
        return obj == null;
    }
```

apiNote: 该方法的存在是用于java.util.function.Predicate类，filter(Objects::isNull)。

来看下Predicate类中，使用到本方法的代码： 

```java
static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)  
                ? Objects::isNull   // 双冒号，代表方法引用。
                : object -> targetRef.equals(object); // 此处为lambda表达式。接收object对象，返回参数targetRef与该对象的比较结果。
    }
```

#### nonNull

判断非空方法，如果参数不为空则返回true。从jdk1.8开始。

源码如下：

```java
    public static boolean nonNull(Object obj) {
        return obj != null;
    }
```

apiNote: 该方法的存在是用于java.util.function.Predicate类，filter(Objects::nonNull)。