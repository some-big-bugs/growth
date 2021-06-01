<h1> 全栈的自我修养: 0005 Java 包扫描实现和应用(Jar篇) </h1>

> It's not the altitude, it's the attitude.<br>
> 决定一切的不是高度而是态度。<br>

**Table of Contents**

- [依赖的 Jar](#依赖的-jar)
- [思路](#思路)
- [完整代码](#完整代码)
- [整合后代码](#整合后代码)


如果你曾经使用过 `Spring`, 那你已经配过 包扫描路径吧，那包扫描是怎么实现的呢？让我们自己写个包扫描

上篇文章中介绍了使用 `File` 遍历的方式去进行包扫描，这篇主要补充一下`jar`包的扫描方式，在我们的项目中一般都会去依赖一些其他`jar` 包，


比如添加 guava 依赖

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>28.2-jre</version>
</dependency>
```

我们再次运行上次的测试用例

```java
@Test
public void testGetPackageAllClasses() throws IOException, ClassNotFoundException {
    ClassScanner scanner = new ClassScanner("com.google.common.cache", true, null, null);
    Set<Class<?>> packageAllClasses = scanner.doScanAllClasses();
    packageAllClasses.forEach(it -> {
        System.out.println(it.getName());
    });
}
```

什么都没有输出

# 依赖的 Jar 
基于`Java` 的反射机制，我们很容易根据 `class` 去创建一个实例对象，但如果我们根本不知道某个包下有多少对象时，我们应该怎么做呢？

在使用`Spring`框架时，会根据包扫描路径来找到所有的 `class`, 并将其实例化后存入容器中。

在我们的项目中也会遇到这样的场景，比如某个包为 `org.example.plugins`, 这个里面放着所有的插件，为了不每次增减插件都要手动修改代码，我们可能会想到用扫描的方式去动态获知 `org.example.plugins` 到底有多少 class, 当然应用场景很有很多

# 思路

既然知道是采用了 `jar` , 那我们使用遍历 jar 的方式去处理一下 

```java

JarFile jar = ((JarURLConnection) url.openConnection()).getJarFile();
// 遍历jar包中的元素
Enumeration<JarEntry> entries = jar.entries();

while (entries.hasMoreElements()) {
  JarEntry entry = entries.nextElement();
  String name = entry.getName();
}
```

这里获取的name 格式为 `com/google/common/cache/Cache.class` 是不是和上篇的文件路径很像呀, 这里可以通过对 `name` 进行操作获取`包名`和 `class`


```java
// 获取包名
String jarPackageName = name.substring(0, name.lastIndexOf('/')).replace("/", ".");

// 获取 class 路径, 这样就能通过类加载进行加载了
String className = name.replace('/', '.');
className = className.substring(0, className.length() - 6);
```

# 完整代码

```java
private void doScanPackageClassesByJar(String basePackage, URL url, Set<Class<?>> classes)
    throws IOException, ClassNotFoundException {
  // 包名
  String packageName = basePackage;
  // 获取文件路径
  String basePackageFilePath = packageName.replace('.', '/');
  // 转为jar包
  JarFile jar = ((JarURLConnection) url.openConnection()).getJarFile();
  // 遍历jar包中的元素
  Enumeration<JarEntry> entries = jar.entries();
  while (entries.hasMoreElements()) {
    JarEntry entry = entries.nextElement();
    String name = entry.getName();
    // 如果路径不一致，或者是目录，则继续
    if (!name.startsWith(basePackageFilePath) || entry.isDirectory()) {
      continue;
    }
    // 判断是否递归搜索子包
    if (!recursive && name.lastIndexOf('/') != basePackageFilePath.length()) {
      continue;
    }

    if (packagePredicate != null) {
      String jarPackageName = name.substring(0, name.lastIndexOf('/')).replace("/", ".");
      if (!packagePredicate.test(jarPackageName)) {
        continue;
      }
    }

    // 判定是否符合过滤条件
    String className = name.replace('/', '.');
    className = className.substring(0, className.length() - 6);
    // 用当前线程的类加载器加载类
    Class<?> loadClass = Thread.currentThread().getContextClassLoader().loadClass(className);
    if (classPredicate == null || classPredicate.test(loadClass)) {
      classes.add(loadClass);
    }

  }
}
```

在结合上篇中 `File` 扫描方式就是完成的代码了


# 整合后代码

```java
package org.example;

import java.io.File;
import java.io.FileFilter;
import java.io.IOException;
import java.net.JarURLConnection;
import java.net.URL;
import java.net.URLDecoder;
import java.util.Enumeration;
import java.util.LinkedHashSet;
import java.util.Set;
import java.util.function.Predicate;
import java.util.jar.JarEntry;
import java.util.jar.JarFile;

/**
 * class 扫描器
 *
 * @author zhangyunan
 */
public class ClassScanner {

  private final String basePackage;
  private final boolean recursive;
  private final Predicate<String> packagePredicate;
  private final Predicate<Class> classPredicate;


  /**
   * Instantiates a new Class scanner.
   *
   * @param basePackage      the base package
   * @param recursive        是否递归扫描
   * @param packagePredicate the package predicate
   * @param classPredicate   the class predicate
   */
  public ClassScanner(String basePackage, boolean recursive, Predicate<String> packagePredicate,
    Predicate<Class> classPredicate) {
    this.basePackage = basePackage;
    this.recursive = recursive;
    this.packagePredicate = packagePredicate;
    this.classPredicate = classPredicate;
  }

  /**
   * Do scan all classes set.
   *
   * @return the set
   * @throws IOException            the io exception
   * @throws ClassNotFoundException the class not found exception
   */
  public Set<Class<?>> doScanAllClasses() throws IOException, ClassNotFoundException {

    Set<Class<?>> classes = new LinkedHashSet<Class<?>>();

    String packageName = basePackage;

    // 如果最后一个字符是“.”，则去掉
    if (packageName.endsWith(".")) {
      packageName = packageName.substring(0, packageName.lastIndexOf('.'));
    }

    // 将包名中的“.”换成系统文件夹的“/”
    String basePackageFilePath = packageName.replace('.', '/');

    Enumeration<URL> resources = Thread.currentThread().getContextClassLoader().getResources(basePackageFilePath);
    while (resources.hasMoreElements()) {
      URL resource = resources.nextElement();
      String protocol = resource.getProtocol();
      if ("file".equals(protocol)) {
        String filePath = URLDecoder.decode(resource.getFile(), "UTF-8");
        // 扫描文件夹中的包和类
        doScanPackageClassesByFile(classes, packageName, filePath);
      } else if ("jar".equals(protocol)) {
        doScanPackageClassesByJar(packageName, resource, classes);
      }
    }

    return classes;
  }

  private void doScanPackageClassesByJar(String basePackage, URL url, Set<Class<?>> classes)
    throws IOException, ClassNotFoundException {
    // 包名
    String packageName = basePackage;
    // 获取文件路径
    String basePackageFilePath = packageName.replace('.', '/');
    // 转为jar包
    JarFile jar = ((JarURLConnection) url.openConnection()).getJarFile();
    // 遍历jar包中的元素
    Enumeration<JarEntry> entries = jar.entries();
    while (entries.hasMoreElements()) {
      JarEntry entry = entries.nextElement();
      String name = entry.getName();
      // 如果路径不一致，或者是目录，则继续
      if (!name.startsWith(basePackageFilePath) || entry.isDirectory()) {
        continue;
      }
      // 判断是否递归搜索子包
      if (!recursive && name.lastIndexOf('/') != basePackageFilePath.length()) {
        continue;
      }

      if (packagePredicate != null) {
        String jarPackageName = name.substring(0, name.lastIndexOf('/')).replace("/", ".");
        if (!packagePredicate.test(jarPackageName)) {
          continue;
        }
      }

      // 判定是否符合过滤条件
      String className = name.replace('/', '.');
      className = className.substring(0, className.length() - 6);
      // 用当前线程的类加载器加载类
      Class<?> loadClass = Thread.currentThread().getContextClassLoader().loadClass(className);
      if (classPredicate == null || classPredicate.test(loadClass)) {
        classes.add(loadClass);
      }

    }
  }

  /**
   * 在文件夹中扫描包和类
   */
  private void doScanPackageClassesByFile(Set<Class<?>> classes, String packageName, String packagePath)
    throws ClassNotFoundException {
    // 转为文件
    File dir = new File(packagePath);
    if (!dir.exists() || !dir.isDirectory()) {
      return;
    }
    // 列出文件，进行过滤
    // 自定义文件过滤规则
    File[] dirFiles = dir.listFiles((FileFilter) file -> {
      String filename = file.getName();

      if (file.isDirectory()) {
        if (!recursive) {
          return false;
        }

        if (packagePredicate != null) {
          return packagePredicate.test(packageName + "." + filename);
        }
        return true;
      }

      return filename.endsWith(".class");
    });

    if (null == dirFiles) {
      return;
    }

    for (File file : dirFiles) {
      if (file.isDirectory()) {
        // 如果是目录，则递归
        doScanPackageClassesByFile(classes, packageName + "." + file.getName(), file.getAbsolutePath());
      } else {
        // 用当前类加载器加载 去除 fileName 的 .class 6 位
        String className = file.getName().substring(0, file.getName().length() - 6);
        Class<?> loadClass = Thread.currentThread().getContextClassLoader().loadClass(packageName + '.' + className);
        if (classPredicate == null || classPredicate.test(loadClass)) {
          classes.add(loadClass);
        }
      }
    }
  }
}

```

