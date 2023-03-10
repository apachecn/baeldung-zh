# Java 中的 NoSuchMethodError

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-nosuchmethod-error>

## 1.概观

在本教程中，我们将看看`java.lang.NoSuchMethodError` 和一些处理它的方法。

## 2.`NoSuchMethodError`

顾名思义，**`NoSuchMethodError`发生在某个特定方法没有找到**的时候。此方法可以是实例方法，也可以是静态方法。

在大多数情况下， **我们能够在编译时捕捉到这个错误。因此**，这不是一个大问题。然而，**有时候会在运行时抛出**，然后找到它就变得有点困难了。根据 [Oracle 文档](https://web.archive.org/web/20220630131738/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/NoSuchMethodError.html)，如果一个类被不兼容地更改，这个错误可能会在运行时发生。

因此，我们可能会在下列情况下遇到此错误。首先，**如果我们只对我们的代码进行部分重新编译**。其次，如果**版本与我们应用程序中的依赖项**不兼容，比如外部 jar。

注意`NoSuchMethodError`继承树包括`[IncompatibleClassChangeError](https://web.archive.org/web/20220630131738/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/IncompatibleClassChangeError.html) and [LinkageError](https://web.archive.org/web/20220630131738/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/LinkageError.html).`这些错误与编译后不兼容的类变化有关。

## 3.`NoSuchMethodError`的例子

让我们通过一个例子来看看这个错误。为此，我们将创建两个类。第一个是`SpecialToday` ，它将列出餐馆当天的特色菜:

```java
public class SpecialToday {
    private static String desert = "Chocolate Cake";

    public static String getDesert() {
        return desert;
    }
}
```

第二个类`MainMenu`从`SpecialsToday:` 调用方法

```java
public class MainMenu {
    public static void main(String[] args) {
        System.out.println("Today's Specials: " + getSpecials());
    }

    public static String getSpecials() {
        return SpecialToday.getDesert();
    }
}
```

这里的输出将是:

```java
Today's Specials: Chocolate Cake
```

接下来，我们将删除`SpecialToday` 中的方法 getDesert()，只重新编译这个更新后的类。这一次，当我们运行我们的`MainMenu,` 时，我们注意到以下运行时错误:

```java
Exception in thread "main" java.lang.NoSuchMethodError: SpecialToday.getDesert()Ljava/lang/String;
```

## 4.如何处理`NoSuchMethodError`

现在让我们看看如何处理这个问题。对于上面的代码，让我们**做一个完整的清理编译，**包括这两个类。我们会注意到在我们编译的时候错误会被发现。如果我们使用像 Eclipse ，**这样的 IDE，它会被更早地检测到，只要我们更新`SpecialsToday`，**。

因此，如果我们的应用程序遇到这个错误，作为第一步，我们将进行一次完全干净的编译。对于 maven，我们将运行`mvn clean install` 命令。

有时问题在于我们应用程序的外部依赖性。在这种情况下，我们将首先**检查由类路径加载器拉出的构建路径中 jar**的顺序。我们将跟踪并更新不一致的 jar。

然而，如果我们在运行时仍然遇到这个错误，我们将不得不更深入地挖掘。我们必须**确保编译时和运行时类和 jar 有相同的版本**。为此，我们可以**用-verbose: class 选项**运行应用程序来检查加载的类。我们可以如下运行该命令:

```java
$ java -verbose:class com.baeldung.exceptions.nosuchmethoderror.MainMenu
[0.014s][info][class,load] opened: /usr/lib/jvm/java-11-openjdk-amd64/lib/modules
[0.015s][info][class,load] opened: /usr/share/java/java-atk-wrapper.jar
[0.028s][info][class,load] java.lang.Object source: shared objects file
[0.028s][info][class,load] java.io.Serializable source: shared objects file
```

在运行时，使用关于加载到各个 jar 中的所有类的信息，我们可以跟踪不兼容的依赖关系。

我们还应该**确保在两个或更多的 jar 中没有重复的类**。在大多数情况下，maven 将直接帮助控制冲突的依赖关系。此外，我们可以运行`mvn dependency: tree`命令来获得项目的依赖树，如下所示:

```java
$ mvn dependency:tree
[INFO] Scanning for projects...
[INFO]
[INFO] -------------< com.baeldung.exceptions:nosuchmethoderror >--------------
[INFO] Building nosuchmethoderror 0.0.1-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO]
[INFO] --- maven-dependency-plugin:2.8:tree (default-cli) @ nosuchmethoderror ---
[INFO] com.baeldung.exceptions:nosuchmethoderror:jar:0.0.1-SNAPSHOT
[INFO] \- org.junit:junit-bom:pom:5.7.0-M1:compile
```

我们可以在这个命令生成的列表中检查库及其版本。此外，我们还可以使用 maven 标签来管理依赖关系。使用[`<exclusions>`标签](/web/20220630131738/https://www.baeldung.com/maven-version-collision)，我们可以排除有问题的依赖。使用[`<optional>`标签](/web/20220630131738/https://www.baeldung.com/maven-optional-dependency)，我们可以防止不必要的依赖被捆绑在 jar 或 war 中。

## 5.结论

在本文中，我们讨论了`NoSuchMethodError`。我们讨论了这个错误的原因以及处理它的方法。关于如何正确处理错误的更多细节，请参考我们关于[捕捉 Java 错误](/web/20220630131738/https://www.baeldung.com/java-error-catch)的文章。

和往常一样，本文中的代码可以在 GitHub 上获得。