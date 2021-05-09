## 函数式接口(FunctionInterface)

#### 问题: Consumer和BiConsumer都是java.util.function包下的两个功能性(函数式)接口(FunctionInterface)

#### 一. 关于功能性接口

所谓函数式接口，指的是只有一个抽象方法的接口。

函数式接口可以被隐式转换为Lambda表达式。

函数式接口可以用**@FunctionalInterface**注解标识。

函数式接口主要是完成一种把方法体(函数)作为参数传入一个方法的功能.

##### JDK1.8之前就出现了一些符合函数式接口定义的接口：

```java
java.lang.Runnable
java.util.concurrent.Callable
java.security.PrivilegedAction
java.util.Comparator
java.io.FileFilter
java.nio.file.PathMatcher
java.lang.reflect.InvocationHandler
java.beans.PropertyChangeListener
java.awt.event.ActionListener
javax.swing.event.ChangeListener
```

##### JDK1.8之后，又添加了一组函数式接口：

```java
java.util.function.*
```

这个路径下有一大堆接口，都是函数式接口，代表了接口调用的各种不同应用场景。

另外，在JDK1.8开始，之前就有的函数式接口(比如Runnable接口)也都添加了@FunctionalInterface注解。

#### 二.关于@FunctionInterface注解

java1.8添加的.其实作用就是提示其他人或者编译器，该接口是函数式接口，该接口内只能有一个抽象方法(默认方法不限)。当某些不知道的人 在该接口中又添加了方法时，编译器会有警告。所以，@FunctionInterface接口仅仅只是提示作用。

#### 三.java.util.function下的函数式接口的例子

##### 1.Consumer\<T\>

接口的唯一方法是:

```java
void accept(T t);
```

这是一个单参数，无返回值的方法，参数是泛型类。这个接口被称为消费型接口，因为没有返回值，接口里面干了什么和调用方没什么关系。

iterator接口中的forEach就使用的该接口

```java
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
```

##### 2. BiConsumer\<T\>

和上面的Consumer很像,不过是接收了两个参数,这个适合处理键值对数据

```java
 void accept(T t, U u);
```

java8中Map中新添加的forEach方法使用了BiConsumer:

```java
    default void forEach(BiConsumer<? super K, ? super V> action) {
        Objects.requireNonNull(action);
        for (Map.Entry<K, V> entry : entrySet()) {
            K k;
            V v;
            try {
                k = entry.getKey();
                v = entry.getValue();
            } catch (IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }
            action.accept(k, v);
        }
    }
```

##### 3. Supplier\<T\>

接口的唯一抽象方法是:

```java
 T get();
```

这是一个无参数，有返回值的方法，返回值类型是泛型类。这个接口被称作供给型接口.

个人认为这个接口就是让用户能够处理自己要返回的对象,或者进行阻塞

在Objects.requireNonNull方法中使用了:

本方法允许将消息的创建延迟，直到空检查结束之后

```java
    public static <T> T requireNonNull(T obj, Supplier<String> messageSupplier) {
        if (obj == null)
            throw new NullPointerException(messageSupplier == null ? null : messageSupplier.get());
        return obj;
    }
```

