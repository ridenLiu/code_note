## 获取当前项目jar包所在文件夹的位置

```java
    public static String currentJarPath() {
        // 类所在的jar包的绝对路径
        String jarWholePath = JarReader.class.getProtectionDomain().getCodeSource().getLocation().getPath();
        try {
            // 对路径进行转码,确保中文不乱码
            jarWholePath = java.net.URLDecoder.decode(jarWholePath, "UTF-8");
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        // jar包所在文件夹的位置
        String jarPath = new File(jarWholePath).getParentFile().getAbsolutePath();
        return jarPath;
    }
```

## 使用java.net.URL对象通过Jar URL协议访问jar中的文件

### Jar URL作用

**作用：** Jar包中资源文件的路径表示

### Jar URL分类

1. Jar file（Jar包本身）：jar:http://www.foo.com/bar/baz.jar!

2. Jar entry（Jar包中某个资源文件）：jar:http://www.foo.com/bar/baz.jar!/COM/foo/Quux.class

3. Jar directory（Jar包中某个目录）：jar:http://www.foo.com/bar/baz.jar!/COM/foo/

### 实例

```java
		String path = "jar:file:/D:/chart-tools.jar!/BOOT-INF/classes/localJSON.properties"
        URL u = new URL(path);
        JarURLConnection conn = (JarURLConnection) u.openConnection();
        InputStreamReader reader = new InputStreamReader(conn.getInputStream(), StandardCharsets.UTF_8);
        char[] b = new char[1000];
        reader.read(b, 0, b.length);
        System.out.println(new String(b));
```

