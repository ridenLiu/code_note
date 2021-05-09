## 关于jdk1.7新增的AutoCloseable接口

功能:

在`JDK7` 中只要实现了`AutoCloseable` 或`Closeable` 接口的类或接口，都可以使用`try-with-resource` 来实现异常处理和资源关闭异常抛出顺序

以前释放资源的写法:

```java
    static String readFirstLingFromFile(String path) throws IOException {
        BufferedReader br = null;
        try {
            br = new BufferedReader(new FileReader(path));
            return br.readLine();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (br != null)
                br.close();
        }
        return null;
    }
```

利用AutoCloseable接口的写法:

```java
    static void testAutoCloseable() throws Exception {
        // 将资源是声明写在try后的括号里,在try语句快(try(){语句块...})执行完后,会自动关闭资源(调用close()方法)
        try (MyResource r = new MyResource()) {
            System.out.println("running");
            // 接下来java会自动调用资源的close()方法.
        }
        System.out.println("over");
    }
```

