# Spring Boot 3 和 Spring Framework 6.0–新功能

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-3-spring-6-new>

## 1.概观

距离《Spring Boot 3》的上映只有 3 个月了。Spring Framework 6.0 可能会在 Spring Boot 3 之前不久出现。所以现在是了解新内容的好时机。

## 2.Java 17

虽然之前已经有对 Java 17 的支持，但是现在这个 LTS 版本得到了基线。

当从 LTS 版本 11 迁移时，Java 开发人员受益于新的语言特性。因为在本文中，Java 本身并不是主题，所以我们只列出对 Spring Boot 开发者来说最重要的新特性。我们可以在 Java [17](/web/20220915070649/https://www.baeldung.com/java-17-new-features) 、 [16](/web/20220915070649/https://www.baeldung.com/java-16-new-features) 、 [15](/web/20220915070649/https://www.baeldung.com/java-15-new) 、 [14](/web/20220915070649/https://www.baeldung.com/java-14-new-features) 、 [13](/web/20220915070649/https://www.baeldung.com/java-13-new-features) 和 [12](/web/20220915070649/https://www.baeldung.com/java-12-new-features) 的单独文章中找到任何细节。

### 2.1.记录

引入 Java 记录( [JEP 395](https://web.archive.org/web/20220915070649/https://openjdk.java.net/jeps/395) ，参见 [Java 14 记录关键字](/web/20220915070649/https://www.baeldung.com/java-record-keyword))的目的是作为一种快速创建数据载体类的方法，即目标只是包含数据并在模块之间传送数据的类，也称为 POJOs(普通旧 Java 对象)和 dto(数据传输对象)。

我们可以很容易地创建不可变的 dto:

```java
public record Person (String name, String address) {}
```

目前，我们在将它们与 [Bean 验证](https://web.archive.org/web/20220915070649/https://github.com/spring-projects/spring-framework/issues/27868)组合时需要小心，因为验证约束在构造函数参数上不受支持，例如，当实例在 JSON 反序列化(Jackson)上创建并作为参数放入控制器的方法时。

### 2.2.文本块

使用 [JEP 378](https://web.archive.org/web/20220915070649/https://openjdk.java.net/jeps/378) ，现在可以创建多行文本块，而无需在换行符上连接字符串:

```java
String textBlock = """
Hello, this is a
multi-line
text block.
""";
```

### 2.3.切换表达式

Java 12 引入了开关表达式( [JEP 361](https://web.archive.org/web/20220915070649/https://openjdk.java.net/jeps/361) )，它(像所有表达式一样)计算单个值，并且可以在语句中使用。代替组合嵌套的`if`–`else`-操作符(`?:`)，我们现在可以使用`switch`–`case`-构造:

```java
DayOfWeek day = DayOfWeek.FRIDAY;
int numOfLetters = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY                -> 7;
    case THURSDAY, SATURDAY     -> 8;
    case WEDNESDAY              -> 9;
};
```

### 2.4.模式匹配

模式匹配在[琥珀项目](https://web.archive.org/web/20220915070649/https://openjdk.org/projects/amber/)中得到阐述，并在 Java 语言中找到了自己的道路。在 Java 语言的情况下，它们可以帮助简化`instanceof`评估的代码。

我们可以用`instanceof`直接使用它们:

```java
if (obj instanceof String s) {
    System.out.println(s.toLowerCase());
}
```

我们也可以在`switch`–`case`语句中使用它:

```java
static double getDoubleUsingSwitch(Object o) {
    return switch (o) {
        case Integer i -> i.doubleValue();
        case Float f -> f.doubleValue();
        case String s -> Double.parseDouble(s);
        default -> 0d;
    };
}
```

### 2.5.密封的类和接口

密封类可以通过指定允许的子类来限制继承:

```java
public abstract sealed class Pet permits Dog, Cat {}
```

我们可以在 Java 中的[密封类和接口中找到更多细节。](/web/20220915070649/https://www.baeldung.com/java-sealed-classes-interfaces)

## 3\. Jakarta EE 9

这个最重要的突破性变化可能是从 Java EE 到 Jakarta EE9 的跳跃，其中包名称空间从`javax.*`变成了`jakarta.*`。因此，每当我们直接使用 Java EE 中的类时，我们都需要调整代码中的所有导入。

例如，当我们访问 Spring MVC 控制器中的`HttpServletRequest`对象时，我们需要替换:

```java
import javax.servlet.http.HttpServletRequest;
```

使用:

```java
import jakarta.servlet.http.HttpServletRequest;
```

当然，我们不必经常使用 Servlet API 的类型，但是如果我们使用 bean 验证和 JPA，这是不可避免的。

当我们使用依赖于 Java/Jakarta EE 的外部库时，我们应该注意(例如，我们必须使用 Hibernate Validator 7+，Tomcat 10+和 Jetty 11+)。

## 4.进一步的依赖性

Spring Framework 6 和 Spring Boot 3 需要以下最低版本:

*   锅内 1.7+
*   Lombok 1.18.22+ ( [JDK17 支持](https://web.archive.org/web/20220915070649/https://github.com/projectlombok/lombok/issues/2898))
*   Gradle 7.3+

## 5.要点

两个至关重要的话题受到了特别关注:`Native Executables`和`Observability`。总体意味着:

*   Spring 框架引入了核心抽象
*   投资组合项目始终与它们相集成
*   Spring Boot 提供自动配置

### 5.1.本机可执行文件

构建本机可执行文件并将其部署到 GraalVM 会获得更高的优先级。因此 [Spring Native](/web/20220915070649/https://www.baeldung.com/spring-native-intro) initiative 是[移入 Spring proper](https://web.archive.org/web/20220915070649/https://spring.io/blog/2022/03/22/initial-aot-support-in-spring-framework-6-0-0-m3) 。

对于 AOT 一代，没有必要包含单独的插件，我们可以只使用一个[的新目标](https://web.archive.org/web/20220915070649/https://docs.spring.io/spring-boot/docs/3.0.0-M3/maven-plugin/reference/htmlsingle/#aot)的`spring-boot-maven-plugin`:

```java
mvn spring-boot:aot-generate
```

本地提示也将成为 Spring 核心的一部分。这个[的测试基础设施将在里程碑 5(6 . 0 . 0-M5)](https://web.archive.org/web/20220915070649/https://github.com/spring-projects/spring-framework/issues/27981)时可用。

### 5.2.可观察性

Spring 6 引入了 Spring Observability——一个基于 [Spring Cloud Sleuth](https://web.archive.org/web/20220915070649/https://spring.io/projects/spring-cloud-sleuth) 的新项目。更多的是为了用[千分尺](https://web.archive.org/web/20220915070649/https://micrometer.io/)高效地记录应用度量，并通过 [OpenZipkin](https://web.archive.org/web/20220915070649/https://zipkin.io/) 或 [OpenTelemetry](https://web.archive.org/web/20220915070649/https://opentelemetry.io/) 等提供者实现追踪。

Spring Observability 优于以前基于代理的 Observability，因为它在本机编译的 Spring 应用程序中无缝工作，可以更有效地提供更好的信息。

## 6.Spring Web MVC 中较小的变化

最重要的新特性之一是对 RFC7807 (问题细节标准)的[支持。不需要像](https://web.archive.org/web/20220915070649/https://github.com/spring-projects/spring-framework/issues/27052)[扎兰多问题](https://web.archive.org/web/20220915070649/https://github.com/zalando/problem)那样包含单独的库。

另一个较小的变化是 [HttpMethod](https://web.archive.org/web/20220915070649/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/HttpMethod.html) 不再是一个 enum，而是一个允许为扩展的 HTTP 方法创建实例的类，例如 WebDAV 定义的方法:

```java
HttpMethod lock = HttpMethod.valueOf("LOCK");
```

至少放弃了一些过时的基于 servlet 的集成，如 Commons FileUpload(我们应该使用 [`StandardServletMultipartResolver`](https://web.archive.org/web/20220915070649/https://www.logicbig.com/tutorials/spring-framework/spring-web-mvc/file-upload-servlet-resolver.html) 上传多部分文件)、Tiles 和 FreeMarker JSP 支持(我们应该使用 [FreeMarker 模板视图](/web/20220915070649/https://www.baeldung.com/freemarker-in-spring-mvc-tutorial))。

## 7.迁移项目

有一些我们应该知道的关于项目迁移的提示。建议的步骤是:

1.  迁移到[Spring Boot 2.7](https://web.archive.org/web/20220915070649/https://spring.io/blog/2022/05/19/spring-boot-2-7-0-available-now)(Spring Boot 3 发布时会有基于 Spring Boot 2.7 的迁移指南)
2.  检查不推荐使用的代码和[遗留配置文件处理](https://web.archive.org/web/20220915070649/https://spring.io/blog/2020/08/14/config-file-processing-in-spring-boot-2-4)——在新的主要版本中它被删除了
3.  迁移到 Java 17
4.  检查第三方项目以获得 Jakarta EE 9 兼容版本
5.  由于 Spring Boot 3 还没有发布，我们可以尝试使用当前的里程碑来测试迁移

## 8.结论

正如我们已经看到的，迁移到 Spring Boot 3 和 Spring 6 也是迁移到 Java 17 和 Jakarta EE 9。如果我们非常重视可观察性和本机可执行文件，我们将从即将到来的主要版本中受益最多。

和往常一样，GitHub 上的所有代码[都是可用的。](https://web.archive.org/web/20220915070649/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-3)