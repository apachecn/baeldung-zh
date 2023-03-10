# 解决隐藏实用程序类公共构造函数声纳警告

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-sonar-hide-implicit-constructor>

## 1.概观

实用程序类只包含我们围绕特定主题组合在一起的**静态** **成员。因此，类本身是无状态的，而它们的成员包含可以跨几层重用的代码。**

在本教程中，我们将解释为什么静态代码分析器报告实用程序类不应该有公共构造函数。我们将通过实现私有构造函数来解决这个问题。此外，我们将探索哪些 Lombok 注释可以帮助我们生成一个。我们还将展示如何禁用这些警告。

最后，我们将评估一些用 Java 实现实用程序类的替代方法。

## 2.实用程序类别

与定义对象的类不同，**实用程序类不保存任何数据或状态。它们只包含行为**。实用工具只包含静态成员。它们的所有方法都是静态的，而数据只作为方法参数传递。

### 2.1.为什么是实用程序类？

在面向对象编程中，我们希望对我们的问题域进行建模，并将类似功能的系列组合在一起。

我们也可以选择**编写纯函数来模拟我们代码库中的常见行为，尤其是在使用函数式编程**的时候。与对象方法不同，这些纯函数不与任何对象的实例相关。然而，他们确实需要一个家。Java 没有专门的类型来存放一组函数，所以我们经常创建一个实用程序类。

Java 中流行的实用程序类的典型例子是来自`java.util`*的`Arrays` 和`Collections`以及来自`org.apache.commons.lang3` `.`的`StringUtils`*

 *### 2.2.用 Java 实现

Java 没有提供特殊的关键字或创建实用程序类的方法。因此，我们通常**创建一个作为普通 Java 类的实用程序类，但是只有静态成员**:

```java
public final class StringUtils {

    public static boolean isEmpty(String source) {
        return source == null || source.length() == 0;
    }

    public static String wrap(String source, String wrapWith) {
        return isEmpty(source) ? source : wrapWith + source + wrapWith;
    }
}
```

在我们的例子中，我们将实用程序类标记为`public`和`final`。实用程序通常是公开的，因为它们旨在跨多个层重用。

关键字`final`阻止子类化。由于**实用程序类不是为继承**设计的，我们不应该子类化它们。

### 2.3.公共构造函数警告

让我们尝试使用一个流行的静态代码分析工具 [SonarQube](/web/20220625232723/https://www.baeldung.com/sonar-qube) 来分析我们的示例实用程序类。我们可以使用构建工具插件在 Java 项目上运行 SonarQube 分析，在本例中是 Maven:

```java
mvn clean verify sonar:sonar -Dsonar.host.url=http://localhost:9000 -Dsonar.login=XYXYXYXY
```

静态代码分析会产生大量代码气味。SonarQube 警告我们**隐藏我们的实用程序类**中的隐式公共构造函数:

[![](img/e259e5f1f94bf5ed7f5d0c4d93043604.png)](/web/20220625232723/https://www.baeldung.com/wp-content/uploads/2021/12/sonar_public_constructor_3-1.png)

虽然我们没有给我们的实用程序类添加构造函数，但是 Java 隐式地添加了一个默认的公共构造函数。因此，允许 API 用户创建它的实例:

```java
StringUtils utils = new StringUtils();
```

这是对我们的实用程序类的误用，因为它不是为实例化而设计的。因此，SonarQube 规则建议我们添加一个私有构造函数，以隐藏默认的公共构造函数。

## 3.添加私有构造函数

现在让我们通过在我们的实用程序类中添加一个私有构造函数来解决报告的代码味道。

### 3.1.默认私有构造函数

让我们向实用程序类添加一个不带参数的私有构造函数。我们将**永远不会真正使用这个私有构造函数**。因此，在调用异常时抛出异常是一个很好的做法:

```java
public final class StringUtils {

    private StringUtils() {
        throw new UnsupportedOperationException("This is a utility class and cannot be instantiated");
    }

    // public static methods
}
```

我们应该注意，私有构造函数也是不可测试的。因此，这种方法将在我们的代码覆盖度量中导致一行未覆盖的代码。

### 3.2.使用 Lombok 构造器

我们可以利用 **`NoArgsConstructor` [龙目](/web/20220625232723/https://www.baeldung.com/intro-to-project-lombok)标注来** **自动生成私有构造函数**:

```java
@NoArgsConstructor(access= AccessLevel.PRIVATE)
public final class StringUtils {

    // public static methods
}
```

这样，我们可以避免手动添加一行未覆盖的代码。

### 3.3.使用 Lombok 实用程序类

我们还可以使用 **`UtilityClass` Lombok 注释，将整个类标记为实用程序**:

```java
@UtilityClass
public class StringUtils {

    // public static methods
}
```

在这种情况下，Lombok 将自动:

*   生成引发异常的私有构造函数
*   将我们添加的任何显式构造函数标记为错误
*   标记班级`final`

我们应该注意到，此时,`UtilityClass`注释仍然是一个实验性的特性。

## 4.禁用警告

如果我们决定不遵循推荐的解决方案，我们还可以选择禁用公共构造函数警告。

### 4.1.抑制警告

让我们利用 Java 的`SuppressWarnings`注释来**禁用单个类级别的警告**:

```java
@SuppressWarnings("java:S1118")
public final class StringUtils {

    // public static methods
}
```

我们应该将正确的 SonarQube 规则 ID 作为值参数传递。我们可以在 SonarQube 服务器 UI 中找到它:

[![](img/2a7293da26f06a462bfd720599af9ebb.png)](/web/20220625232723/https://www.baeldung.com/wp-content/uploads/2021/12/sonar_rule_id_2.png)

### 4.2.停用规则

在 SonarQube 现成的质量配置文件中，我们无法禁用任何预定义的规则。因此，为了**在整个项目级别**上禁用警告，我们首先需要创建一个定制的质量配置文件:

[![](img/b610c081083292360bceed1624889b4d.png)](/web/20220625232723/https://www.baeldung.com/wp-content/uploads/2021/12/sonar_deactivate_rule_2.png)

在我们的定制质量概要文件中，我们可以搜索和停用任何预定义的 Java 规则。

## 5.替代实现

让我们看看除了使用类之外，如何创建实用程序的一些可能的替代方法。

### 5.1.静态接口方法

从 Java 8 开始，我们可以在接口中定义并**实现 [`static`方法](/web/20220625232723/https://www.baeldung.com/java-static-default-methods):**

```java
public interface StringUtils {

    static boolean isEmpty(String source) {
        return source == null || source.length() == 0;
    }

    static String wrap(String source, String wrapWith) {
        return isEmpty(source) ? source : wrapWith + source + wrapWith;
    }
} 
```

因为我们不能实例化接口，所以我们消除了实用程序类实例化的问题。然而，我们正在制造另一个问题。由于接口被设计成由其他类实现，API 用户可能会错误地实现这个接口。

此外，接口不能包含私有常量和静态初始值设定项。

### 5.2。静态枚举方法

枚举是托管实例的容器。然而，我们可以创建一个实用程序作为一个 **enum，其中没有实例，只包含静态方法**:

```java
public enum StringUtils {;

    public static boolean isEmpty(String source) {
        return source == null || source.length() == 0;
    }

    public static String wrap(String source, String wrapWith) {
        return isEmpty(source) ? source : wrapWith + source + wrapWith;
    }
}
```

由于我们不能实例化枚举类型，我们消除了实用程序类实例化的问题。另一方面，顾名思义，枚举类型是为创建实际的枚举而设计的，而不是实用程序类。

## 6.结论

在本文中，我们探索了**实用程序类，并解释了为什么它们不应该有公共构造函数**。

在示例中，我们介绍了手动实现私有构造函数和使用 Lombok 注释。接下来，我们看到了如何抑制和禁用相关的 SonarQube 警告。最后，我们看了使用接口和枚举创建实用程序的两种替代方法。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220625232723/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-methods)*