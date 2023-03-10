# Java 中的类装入器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-classloaders>

## 1。级装载器简介

类加载器负责**在运行时将 Java 类动态加载到 JVM** **(Java 虚拟机)。**它们也是 JRE (Java 运行时环境)的一部分。因此，由于有了类加载器，JVM 不需要知道底层文件或文件系统就可以运行 Java 程序。

此外，这些 Java 类不是一次加载到内存中，而是在应用程序需要时加载。这就是类装入器发挥作用的地方。它们负责将类加载到内存中。

在本教程中，我们将讨论不同类型的内置类加载器以及它们是如何工作的。然后我们将介绍我们自己的定制实现。

## 延伸阅读:

## [理解 Java 中的内存泄漏](/web/20221103080026/https://www.baeldung.com/java-memory-leaks)

Learn what memory leaks are in Java, how to recognize them at runtime, what causes them, and strategies for preventing them.[Read more](/web/20221103080026/https://www.baeldung.com/java-memory-leaks) →

## [ClassNotFoundException vs NoClassDefFoundError](/web/20221103080026/https://www.baeldung.com/java-classnotfoundexception-and-noclassdeffounderror)

Learn about the differences between ClassNotFoundException and NoClassDefFoundError.[Read more](/web/20221103080026/https://www.baeldung.com/java-classnotfoundexception-and-noclassdeffounderror) →

## 2。内置类装载机的种类

让我们从学习如何使用各种类装入器装入不同的类开始:

```java
public void printClassLoaders() throws ClassNotFoundException {

    System.out.println("Classloader of this class:"
        + PrintClassLoader.class.getClassLoader());

    System.out.println("Classloader of Logging:"
        + Logging.class.getClassLoader());

    System.out.println("Classloader of ArrayList:"
        + ArrayList.class.getClassLoader());
}
```

执行时，上述方法会打印:

```java
Class loader of this class:[[email protected]](/web/20221103080026/https://www.baeldung.com/cdn-cgi/l/email-protection)
Class loader of Logging:[[email protected]](/web/20221103080026/https://www.baeldung.com/cdn-cgi/l/email-protection)
Class loader of ArrayList:null
```

正如我们所见，这里有三种不同的类装入器:应用程序、扩展和引导程序(显示为`null`)。

应用程序类加载器加载包含示例方法的类。应用程序或系统类加载器在类路径中加载我们自己的文件。

接下来，扩展类加载器加载`Logging`类。**扩展类加载器加载标准核心 Java 类的扩展类。**

最后，引导类装入器装入`ArrayList`类。引导或原始类装入器是所有其它类装入器的父类。

然而，我们可以看到对于`ArrayList,`,它在输出中显示`null`。**这是因为 bootstrap 类加载器是用本机代码编写的，而不是 Java，所以它不会显示为 Java 类。**因此，引导类装入器的行为会因 JVM 而异。

现在让我们更详细地讨论每一个类装入器。

### 2.1。自举类加载器

Java 类由`java.lang.ClassLoader`的实例加载。然而，类装入器本身就是类。那么问题来了，谁装载了`java.lang.ClassLoader`本身`?`

这就是引导或原始类装入器发挥作用的地方。

它主要负责加载 JDK 内部类，通常是位于`$JAVA_HOME/jre/lib` 目录下的`rt.jar`和其他核心库。此外，**引导类加载器充当所有其他`ClassLoader`实例**的父类。

**这个引导类加载器是核心 JVM 的一部分，用本机代码编写，**如上面的例子所指出的。不同的平台可能有这种特定的类装入器的不同实现。

### 2.2。扩展类加载器

**扩展类加载器是引导类加载器的子类，负责加载标准核心 Java 类**的扩展，以便它们可用于平台上运行的所有应用。

扩展类加载器从 JDK 扩展目录加载，通常是`$JAVA_HOME/lib/ext`目录，或者任何其他在`java.ext.dirs`系统属性中提到的目录。

### 2.3。系统类加载器

另一方面，系统或应用程序类加载器负责将所有应用程序级的类加载到 JVM 中。**它加载在 classpath 环境变量中找到的文件，`-classpath,`或`-cp`命令行选项**。它也是扩展类加载器的一个孩子。

## 3。类装入器是如何工作的？

类别载入器是 Java 运行时环境的一部分。当 JVM 请求一个类时，类装入器试图定位该类，并使用完全限定的类名将类定义装入运行时。

**`java.lang.ClassLoader.loadClass()`方法负责将类定义加载到运行时**。它尝试基于完全限定名加载类。

如果类还没有被加载，它将请求委托给父类加载器。这个过程递归地发生。

最终，如果父类装入器没有找到该类，那么子类将调用`java.net.URLClassLoader.findClass()`方法在文件系统本身中寻找类。

如果最后一个子类装入器也不能装入类，它抛出[T0 或者`java.lang.ClassNotFoundException.`T3](/web/20221103080026/https://www.baeldung.com/java-classnotfoundexception-and-noclassdeffounderror)

让我们来看一个抛出`ClassNotFoundException`时的输出示例:

```java
java.lang.ClassNotFoundException: com.baeldung.classloader.SampleClassLoader    
    at java.net.URLClassLoader.findClass(URLClassLoader.java:381)    
    at java.lang.ClassLoader.loadClass(ClassLoader.java:424)    
    at java.lang.ClassLoader.loadClass(ClassLoader.java:357)    
    at java.lang.Class.forName0(Native Method)    
    at java.lang.Class.forName(Class.java:348)
```

如果我们仔细查看从调用`java.lang.Class.forName()`开始的事件序列，我们可以看到它首先试图通过父类加载器加载类，然后`java.net.URLClassLoader.findClass()`寻找类本身。

当它仍然没有找到类时，它抛出一个`ClassNotFoundException.`

现在让我们研究一下类装入器的三个重要特性。

### 3.1。代表团模式T3

类装入器遵循委托模型，其中**在请求查找类或资源时，`ClassLoader`实例将把类或资源的搜索委托给父类装入器**。

假设我们请求将一个应用程序类加载到 JVM 中。系统类装入器首先将该类的装入委托给它的父扩展类装入器，后者又将它委托给引导类装入器。

只有当引导程序和扩展类装入器装入类不成功时，系统类装入器才试图自己装入类。

### 3.2。独特类

作为委托模型的结果，很容易确保**唯一的类，因为我们总是试图向上委托**。

如果父类装入器不能找到该类，那么当前实例将尝试自己找到它。

### 3.3。能见度

此外，**子类装入器对于由它们的父类装入器**装入的类是可见的。

例如，由系统类加载器加载的类可以看到由扩展和引导类加载器加载的类，但反之则不然。

为了说明这一点，如果类 A 由应用程序类加载器加载，而类 B 由扩展类加载器加载，那么就应用程序类加载器加载的其他类而言，A 类和 B 类都是可见的。

然而，类 B 是扩展类装入器装入的其他类唯一可见的类。

## 4。自定义类加载器

对于文件已经在文件系统中的大多数情况，内置的类加载器已经足够了。

然而，在我们需要从本地硬盘或网络加载类的场景中，我们可能需要使用定制的类加载器。

在这一节中，我们将介绍定制类装入器的一些其他用例，并演示如何创建一个。

### 4.1.定制类装入器用例

定制的类装入器不仅仅是在运行时装入类。一些使用案例可能包括:

1.  帮助修改现有的字节码，例如编织代理
2.  创建动态适应用户需求的类，例如在 JDBC，不同驱动程序实现之间的切换是通过动态类加载来完成的。
3.  实现类版本控制机制，同时为具有相同名称和包的类加载不同的字节码。这可以通过 URL 类加载器(通过 URL 加载 jar)或定制类加载器来完成。

下面是定制类装入器可能派上用场的更具体的例子。

例如，浏览器使用自定义的类加载器从网站加载可执行内容。浏览器可以使用不同的类加载器从不同的网页加载小程序。applet viewer 用于运行 applet，它包含一个`ClassLoader`来访问远程服务器上的网站，而不是在本地文件系统中查找。

然后，它通过 HTTP 加载原始字节码文件，并将它们转换成 JVM 中的类。即使这些**小程序有相同的名字，如果由不同的类装入器**装入，它们也被认为是不同的组件。

既然我们已经理解了为什么定制类加载器是相关的，让我们实现一个子类`ClassLoader`来扩展和总结 JVM 如何加载类的功能。

### 4.2.创建我们的自定义类加载器

为了便于说明，假设我们需要使用自定义的类装入器从文件中装入类。

**我们需要扩展`ClassLoader`类并覆盖`findClass()`方法:**

```java
public class CustomClassLoader extends ClassLoader {

    @Override
    public Class findClass(String name) throws ClassNotFoundException {
        byte[] b = loadClassFromFile(name);
        return defineClass(name, b, 0, b.length);
    }

    private byte[] loadClassFromFile(String fileName)  {
        InputStream inputStream = getClass().getClassLoader().getResourceAsStream(
                fileName.replace('.', File.separatorChar) + ".class");
        byte[] buffer;
        ByteArrayOutputStream byteStream = new ByteArrayOutputStream();
        int nextValue = 0;
        try {
            while ( (nextValue = inputStream.read()) != -1 ) {
                byteStream.write(nextValue);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        buffer = byteStream.toByteArray();
        return buffer;
    }
}
```

在上面的例子中，我们定义了一个自定义的类装入器，它扩展了默认的类装入器，并从指定的文件中装入一个字节数组。

### 5.理解`java.lang.ClassLoader`

让我们讨论一些来自`java.lang.ClassLoader`类的基本方法，以便更清楚地了解它是如何工作的。

### 5.1.`loadClass()`法

```java
public Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
```

该方法负责加载给定了名称参数的类。name 参数是指完全限定的类名。

Java 虚拟机调用`loadClass()`方法来解析类引用，将 resolve 设置为`true`。然而，解析一个类并不总是必要的。**如果我们只需要确定该类是否存在，那么 resolve 参数被设置为`false`。**

这个方法作为类装入器的入口点。

我们可以试着从`java.lang.ClassLoader:`的源代码中了解`loadClass()`方法的内部工作原理

```java
protected Class<?> loadClass(String name, boolean resolve)
  throws ClassNotFoundException {

    synchronized (getClassLoadingLock(name)) {
        // First, check if the class has already been loaded
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    c = findClass(name);
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```

方法的默认实现按以下顺序搜索类:

1.  调用`findLoadedClass(String)`方法来查看该类是否已经加载。
2.  调用父类加载器上的`loadClass(String)`方法。
3.  调用`findClass(String)`方法来查找该类。

### 5.2.`defineClass()`法

```java
protected final Class<?> defineClass(
  String name, byte[] b, int off, int len) throws ClassFormatError
```

此方法负责将字节数组转换为类的实例。在我们使用这个类之前，我们需要解决它。

如果数据不包含有效的类，它抛出一个`ClassFormatError.`

此外，我们不能覆盖这个方法，因为它被标记为 final。

### 5.3.`findClass()`法

```java
protected Class<?> findClass(
  String name) throws ClassNotFoundException
```

此方法查找以完全限定名作为参数的类。我们需要在定制的类装入器实现中覆盖这个方法，它遵循装入类的委托模型。

另外，如果父类装入器找不到请求的类，`loadClass()`就会调用这个方法。

如果没有类装入器的父类找到该类，默认实现抛出一个`ClassNotFoundException`。

### 5.4.`getParent()`法

```java
public final ClassLoader getParent()
```

该方法返回委托的父类加载器。

一些实现，比如之前在第 2 节中看到的，使用`null`来表示引导类加载器。

### 5.5.`getResource()`法

```java
public URL getResource(String name)
```

此方法尝试查找具有给定名称的资源。

它将首先委托给资源的父类加载器。**如果父级是`null`，则搜索虚拟机内置的类加载器的路径。**

如果失败，那么该方法将调用`findResource(String)`来查找资源。指定为输入的资源名称可以是类路径的相对名称，也可以是绝对名称。

它返回一个用于读取资源的 URL 对象，或者如果找不到资源或者调用者没有足够的权限返回资源，则返回 null。

需要注意的是，Java 从类路径加载资源。

最后，**Java 中的资源加载被认为是独立于位置的，**因为只要环境被设置为查找资源，代码在哪里运行并不重要。

## 6.上下文类加载器

一般来说，上下文类加载器提供了一种替代 J2SE 引入的类加载委托方案的方法。

正如我们之前所学的，JVM 中的类装入器遵循层次模型，因此除了引导类装入器之外，每个类装入器都有一个父类。

然而，有时当 JVM 核心类需要动态加载应用程序开发人员提供的类或资源时，我们可能会遇到问题。

例如，在 JNDI，核心功能由`rt.jar.`中的引导类实现，但是这些 JNDI 类可能加载由独立供应商实现的 JNDI 提供者(部署在应用程序类路径中)。这个场景要求引导类装入器(父类装入器)装入一个对应用程序装入器(子类装入器)可见的类。

J2SE 代表团在这里不起作用，为了解决这个问题，我们需要找到类加载的替代方法。这可以通过使用线程上下文加载器来实现。

`java.lang.Thread` 类有一个方法 **`getContextClassLoader(),` ，它返回特定线程**的`ContextClassLoader`。`ContextClassLoader`由线程的创建者在加载资源和类时提供。

如果没有设置这个值，那么它默认为父线程的类装入器上下文。

## 7。结论

类装入器对于执行 Java 程序是必不可少的。在本文中，我们对它们进行了很好的介绍。

我们讨论了不同类型的类装入器，即引导、扩展和系统类装入器。Bootstrap 充当所有这些类的父类，负责加载 JDK 内部类。另一方面，扩展和系统分别从 Java 扩展目录和类路径加载类。

我们还了解了类装入器是如何工作的，并研究了一些特性，比如委托、可见性和惟一性。然后我们简要地解释了如何创建一个定制的类装入器。最后，我们介绍了上下文类装入器。

和往常一样，这些例子的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221103080026/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-jvm)