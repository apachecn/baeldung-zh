# Java system . getproperty vs system . getenv

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-system-get-property-vs-system-getenv>

## 1.介绍

当在 Java 应用程序中时，包`java.lang`被自动导入。这个包包含了很多常用的类，从`NullPointerException`到`Object`、`Math`、`String`。

`java.lang.System`类是一个`final`类，这意味着我们不能子类化它，因此，所有的方法都是`static`。

在本教程中，我们将探索两种`System`方法之间的差异，用于**读取系统属性和环境变量。**这些方法是`getProperty`和`getenv`。

## 2.使用`System.getProperty()`

Java 平台使用一个`Properties`对象来提供**关于本地系统和配置的信息，**，我们称之为**系统属性**。

系统属性包括诸如当前用户、Java 运行时的当前版本和文件路径名分隔符等信息。

在下面的代码中，我们使用`System.getProperty(“log_dir”)` 来读取属性`log_dir`的值。我们还使用默认值参数，所以如果属性不存在，`getProperty`返回`/` tmp `/log`:

```java
String log_dir = System.getProperty("log_dir","/tmp/log"); 
```

为了在运行时更新系统属性，我们使用了`System.setProperty`方法:

```java
System.setProperty("log_dir", "/tmp/log");
```

我们可以使用`propertyName`命令行参数将我们自己的属性或配置值传递给应用程序:

```java
java -jar jarName -DpropertyName=value
```

我们在 app.jar 中将 foo 的属性设置为 bar 值:

```java
java -jar app -Dfoo="bar"
```

**`System.getProperty`总会返回一个`String`。**

## 3.使用`System.getenv()`

**环境变量是键/值对，就像`Properties.`** 许多操作系统使用环境变量将配置信息传递给应用程序。

设置环境变量的方式因操作系统而异。例如，在 Windows 中，我们使用控制面板中的系统实用程序，而在 Unix 中，我们使用 shell 脚本。

**创建流程时，默认继承其父流程的克隆环境。**

以下代码片段说明了如何使用 lambda 表达式打印所有环境变量:

```java
System.getenv().forEach((k, v) -> {
    System.out.println(k + ":" + v);
}); 
```

**`getenv()`返回一个只读的`Map.`** 试图给地图添加值时抛出一个`UnsupportedOperationException`。

要获得单个变量，我们可以用变量名调用`getenv`:

```java
String log_dir = System.getenv("log_dir");
```

另一方面，我们可以从我们的应用程序创建另一个流程，并向其环境添加新的变量。

要在 Java 中创建新流程，我们可以使用`ProcessBuilder`类，它有一个名为`environment`的方法。这个方法返回一个`Map,`，但是这次地图不是只读的，这意味着我们可以向它添加元素:

```java
ProcessBuilder pb = new ProcessBuilder(args);
Map<String, String> env = pb.environment();
env.put("log_dir", "/tmp/log");
Process process = pb.start();
```

## 4.差异

虽然两者本质上都是为`String`键提供`String`值的映射，但是让我们来看看一些不同之处:

1.  我们可以在运行时更新属性，而环境变量是操作系统变量的不可变副本。
2.  属性只包含在 Java 平台中，而环境变量在操作系统级别是全局的，可用于同一台机器上运行的所有应用程序。
3.  在打包应用程序时，属性必须存在，但是我们几乎可以在任何时候在操作系统上创建环境变量。

## 5.结论

虽然在概念上相似，但是属性和环境变量的应用是完全不同的。

两者之间的选择往往是一个范围的问题。使用环境变量，同一应用程序可以部署到多台机器上运行不同的实例，并且可以在操作系统级别进行配置，甚至可以在 AWS 或 Azure 控制台中进行配置。这消除了重新构建应用程序来更新配置的需要。

永远记住`getProperty`遵循骆驼案例惯例，而`getenv`不遵循。