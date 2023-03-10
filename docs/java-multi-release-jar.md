# 多版本 Jar 文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-multi-release-jar>

## 1.概观

Java 在不断发展，并为 JDK 增加了新的特性。而且，如果我们想在我们的 API 中使用这些特性，那么下游依赖者就必须升级他们的 JDK 版本。

有时，为了保持兼容性，我们被迫等待使用新的语言特性。

不过，在本教程中，我们将了解多版本 jar(Mr jar)以及它们如何同时包含与不同 JDK 版本兼容的实现。

## 2.简单的例子

让我们来看看一个名为`DateHelper`的实用程序类，它有一个检查[闰年](/web/20220627180039/https://www.baeldung.com/java-leap-year)的方法。让我们假设它是使用 JDK 7 编写的，并构建为在 JRE 7+上运行:

```java
public class DateHelper {
    public static boolean checkIfLeapYear(String dateStr) throws Exception {
        logger.info("Checking for leap year using Java 1 calendar API ");

        Calendar cal = Calendar.getInstance();
        cal.setTime(new SimpleDateFormat("yyyy-MM-dd").parse(dateStr));
        int year = cal.get(Calendar.YEAR);

        return (new GregorianCalendar()).isLeapYear(year);
    }
}
```

将从我们的测试应用程序的`main`方法中调用`checkIfLeapYear`方法:

```java
public class App {
    public static void main(String[] args) throws Exception {
        String dateToCheck = args[0];
        boolean isLeapYear = DateHelper.checkIfLeapYear(dateToCheck);
        logger.info("Date given " + dateToCheck + " is leap year: " + isLeapYear);
    }
}
```

让我们快进到今天。

我们知道 Java 8 有一种更简洁的方法来解析日期。所以，我们想利用这一点，重写我们的逻辑。**为此，我们需要改用 JDK 8+。然而，这将意味着我们的模块将停止在 JRE 7 上工作，而它最初就是为 JRE 7 编写的。**

除非绝对必要，否则我们不希望这种情况发生。

## 3.多版本 Jar 文件

Java 9 中的解决方案是**保持原始类不变，而是使用新的 JDK 创建一个新版本，并将它们打包在一起**。在运行时，JVM(版本 9 或更高版本)将调用这两个版本中的任何一个**，给予 JVM 支持的最高版本**更多的优先权。

例如，如果一个 MRJAR 包含同一类的 Java 版本 7(默认)、9 和 10，那么 JVM 10+将执行版本 10，而 JVM 9 将执行版本 9。在这两种情况下，都不会执行默认版本，因为对于该 JVM 存在更合适的版本。

注意，类的新版本的**公共定义应该与原始版本的**完全匹配。换句话说，我们不允许添加任何新版本专有的新公共 API。

## 4.文件夹结构

由于 Java 中的类通过它们的名字直接映射到文件，所以不可能在同一位置创建新版本的`DateHelper`。因此，我们需要在单独的文件夹中创建它们。

让我们从在与`java`相同的级别创建一个文件夹`java9`开始。之后，让我们克隆`DateHelper.java`文件，保留它的包文件夹结构，并把它放在`java9:`

```java
src/
    main/
        java/
            com/
                baeldung/
                    multireleaseapp/
                        App.java
                        DateHelper.java
        java9/
            com/
                baeldung/
                    multireleaseapp/
                        DateHelper.java
```

**一些还不支持 MRJARs** 的 ide 可能会抛出重复`DateHelper.java`类的错误。

我们将在另一篇教程中讨论如何将它与 Maven 等构建工具集成。现在，让我们只关注基本面。

## 5.代码更改

让我们重写`java9`克隆类的逻辑:

```java
public class DateHelper {
    public static boolean checkIfLeapYear(String dateStr) throws Exception {
        logger.info("Checking for leap year using Java 9 Date Api");
        return LocalDate.parse(dateStr).isLeapYear();
    }
}
```

注意这里的**我们没有对克隆类的公共方法签名做任何改变，只是改变了内部逻辑。同时，我们没有添加任何新的公共方法。**

这非常重要，因为如果不遵循这两条规则，jar 创建将会失败。

## 6.Java 中的交叉编译

交叉编译是 Java 中的一个特性，可以编译文件以便在早期版本上运行。这意味着我们没有必要安装单独的 JDK 版本。

让我们用 JDK 9 或更高版本编译我们的类。

首先，编译 Java 7 平台的旧代码:

```java
javac --release 7 -d classes src\main\java\com\baeldung\multireleaseapp\*.java
```

其次，为 Java 9 平台编译新代码:

```java
javac --release 9 -d classes-9 src\main\java9\com\baeldung\multireleaseapp\*.java
```

`release`选项用于指示 Java 编译器和目标 JRE 的版本。

## 7.创建 MRJAR

最后，使用版本 9+创建 MRJAR 文件:

```java
jar --create --file target/mrjar.jar --main-class com.baeldung.multireleaseapp.App
  -C classes . --release 9 -C classes-9 .
```

**`release`选项后跟一个文件夹名，使得该文件夹的内容被打包到 jar 文件中，位于版本号值:**

```java
com/
    baeldung/
        multireleaseapp/
            App.class
            DateHelper.class
META-INF/
    versions/
        9/
            com/
                baeldung/
                    multireleaseapp/
                        DateHelper.class
    MANIFEST.MF
```

`MANIFEST.MF`文件的属性设置为让 JVM 知道这是一个 MRJAR 文件:

```java
Multi-Release: true
```

因此，JVM 在运行时加载适当的类。

旧的 JVM 会忽略新的属性，该属性表明这是一个 MRJAR 文件，并将其视为普通的 JAR 文件。

## 8.测试

最后，让我们用 Java 7 或 8 来测试我们的 jar:

```java
> java -jar target/mrjar.jar "2012-09-22"
Checking for leap year using Java 1 calendar API 
Date given 2012-09-22 is leap year: true
```

然后，让我们针对 Java 9 或更高版本再次测试这个 jar:

```java
> java -jar target/mrjar.jar "2012-09-22"
Checking for leap year using Java 9 Date Api
Date given 2012-09-22 is leap year: true
```

## 9.结论

在本文中，我们通过一个简单的例子看到了如何创建一个多版本 jar 文件。

和往常一样，多版本应用的代码库在 GitHub 上[可用。](https://web.archive.org/web/20220627180039/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-9-new-features)