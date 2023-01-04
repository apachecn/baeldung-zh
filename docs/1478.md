# 以编程方式创建 JAR 文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jar-create-programatically>

## 1。简介

在这篇简短的文章中，我们将回顾以编程方式创建 jar 文件的过程。当编写软件时，最终我们需要将它部署到生产状态。在某些情况下，可以将类路径与单独的文件一起使用。通常，处理单个文件更方便。对于 Java 来说，标准的做法是使用 JAR、WAR 或 EAR 文件。

基本流程是编写清单，打开罐子，添加内容，最后关闭罐子。

## 2。剖析一个 Jar 文件

jar 文件是 ZIP 文件格式的扩展，包括一个清单文件。清单文件是特定于 JAR 文件的特殊文件，可能包含各种设置。其中有些是主类，可选数据(即作者、版本等。)，以及代码签名信息。

我们可以使用 zip 兼容工具，如 [WinRar](winrar.com) ，来查看和提取部分或全部档案。我们还可以包含一个 jars 或 libs 子目录，用于包含依赖 jars。因为 jar 是 zip 文件的扩展，所以我们可以包含任何文件或目录。

## 3。创建一个`JarTool`类

为了简化创建 JAR 文件的过程，我们创建了一个单独的、简单的旧 Java 对象(POJO)类来封装我们的操作。我们可能包括将条目放入清单文件、创建 JAR 文件、添加文件或目录。

我们还可以创建方法来执行从 JAR 中的删除，甚至向现有的 JAR 添加条目，尽管这些操作需要完全读取和重写 JAR。

### 3.1。罐子清单

为了创建 JAR 文件，我们必须首先开始清单:

```
public class JarTool {    
    private Manifest manifest = new Manifest();

    public void startManifest() {
        manifest.getMainAttributes().put(Attributes.Name.MANIFEST_VERSION, "1.0");
    }
}
```

如果我们希望 jar 是可执行的，我们必须设置主类:

```
public void setMainClass(String mainFQCN) {
    if (mainFQCN != null && !mainFQCN.equals("")) {
        manifest.getMainAttributes().put(Attributes.Name.MAIN_CLASS, mainFQCN);
    }
}
```

此外，如果我们想要指定附加属性，我们可以将它们添加到清单中，例如:

```
addToManifest("Can-Redefine-Classes", "true");
```

方法如下:

```
public void addToManifest(String key, String value) {
     manifest.getMainAttributes().put(new Attributes.Name(key), value);
}
```

### 3.2.打开罐子进行书写

清单完成后，我们现在可以将条目写入 JAR 文件。为此，我们必须首先打开罐子:

```
public JarOutputStream openJar(String jarFile) throws IOException {        
    return new JarOutputStream(new FileOutputStream(jarFile), manifest);
} 
```

### 3.3.向 Jar 中添加文件

向 JAR 中添加文件时，Java 使用 Solaris 风格的文件名，并使用正斜杠作为分隔符(/)。注意**我们可以添加任何类型的任何文件**，包括其他 JAR 文件或空目录。这对于包含依赖项来说非常方便。

另外，因为 JAR 文件是类路径的一种形式，**我们必须指定我们希望在 JAR** 中使用绝对路径的哪一部分。出于我们的目的，根路径将是我们项目的类路径。

理解了这一点，我们现在可以用这个方法完成我们的`JarTool`类:

```
public void addFile(JarOutputStream target, String rootPath, String source) 
  throws FileNotFoundException, IOException {
    String remaining = "";
    if (rootPath.endsWith(File.separator)) {
        remaining = source.substring(rootPath.length());
    } else {
        remaining = source.substring(rootPath.length() + 1);
    }
    String name = remaining.replace("\\","/");
    JarEntry entry = new JarEntry(name);
    entry.setTime(new File(source).lastModified());
    target.putNextEntry(entry);

    BufferedInputStream in = new BufferedInputStream(new FileInputStream(source));
    byte[] buffer = new byte[1024];
    while (true) {
        int count = in.read(buffer);
        if (count == -1) {
            break;
        }
        target.write(buffer, 0, count);
    }
    target.closeEntry();
    in.close();
}
```

## 4。一个工作实例

为了演示可执行 jar 的最低要求，我们将编写一个应用程序类，然后看看它是如何工作的:

```
public class Driver {
    public static void main(String[] args) throws IOException {
        JarTool tool = new JarTool();
        tool.startManifest();
        tool.addToManifest("Main-Class", "com.baeldung.createjar.HelloWorld");

        JarOutputStream target = tool.openJar("HelloWorld.jar");

        tool.addFile(target, System.getProperty("user.dir") + "\\src\\main\\java",
          System.getProperty("user.dir") + "\\src\\main\\java\\com\\baeldung\\createjar\\HelloWorld.class");
        target.close();
    }
}
```

HelloWorld 类是一个非常简单的类，只有一个 main()方法来打印文本:

```
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
} 
```

为了证明这是可行的，我们举了这个例子:

```
$ javac -cp src/main/java src/main/java/com/baeldung/createjar/HelloWorld.java
$ javac -cp src/main/java src/main/java/com/baeldung/createjar/JarTool.java
$ javac -cp src/main/java src/main/java/com/baeldung/createjar/Driver.java
$ java -cp src/main/java com/baeldung/createjar/Driver
$ java -jar HelloWorld.jar
Hello World! 
```

这里，我们编译了每个类，然后执行了`Driver`类，这将创建`HelloWorld` jar。最后，我们已经执行了 jar，这导致打印“Hello World”消息。

上面的命令应该从项目位置执行。

## 5.结论

在本教程中，我们看到了如何以编程方式创建一个 jar 文件，向其中添加文件，并最终执行它。

当然，代码可以在 GitHub 上找到。