# 如何将一个属性值注入到一个不由 Spring 管理的类中？

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/inject-properties-value-non-spring-class>

## 1。概述

按照设计，用 `@Repository, @Service, @Controller`标注的类等等。由 Spring 管理，注射配置简单自然。不那么简单的是**将配置传递给不是由 Spring 直接管理的类。**

在这种情况下，我们可以使用基于`ClassLoader-`的配置加载，或者简单地在另一个 bean 中实例化我们的类，并手动设置所需的参数——这是建议的选项，因为配置条目不需要专门存储在`*.properties`文件中。

在这篇简短的文章中，我们将讨论用 Java `ClassLoader`加载`*.properties`文件的主题，以及将 Spring 已经[加载的配置注入到一个非托管类中。](/web/20220926201346/https://www.baeldung.com/properties-with-spring)

## 2。用类加载器加载配置

简单地说，`*.properties`文件是保存一些配置信息的资源文件。我们可以使用 Java `ClassLoader`来做同样的事情，而不是使用支持自动应用配置加载的第三方实现，例如在 Spring 中实现的。

我们将创建一个容器对象来保存在`resourceFileName`中定义的`Properties`。为了用配置填满容器，我们将使用一个`ClassLoader`。

让我们定义实现`loadProperties(String resourceFileName)`方法的`PropertiesLoader`类:

```
public class PropertiesLoader {

    public static Properties loadProperties(String resourceFileName) throws IOException {
        Properties configuration = new Properties();
        InputStream inputStream = PropertiesLoader.class
          .getClassLoader()
          .getResourceAsStream(resourceFileName);
        configuration.load(inputStream);
        inputStream.close();
        return configuration;
    }
}
```

每个`Class`对象包含一个对实例化它的`ClassLoader`的引用；这是一个主要负责加载类的对象，但是在本教程中，我们将使用它来加载资源文件，而不是普通的 Java 类。`ClassLoader`正在类路径上寻找`resourceFileName`。

之后，我们通过`getResourceAsStream` API 将资源文件作为`InputStream`加载。

在上面的例子中，我们定义了一个配置容器，它可以使用`load(InputStream)` API 解析`resourceFileName`。

load 方法实现了对`*.properties`文件的解析，并支持将`“:”`或`“=”`字符作为分隔符。此外，新行开头使用的`“#”`或`“!”`字符都是注释标记，会导致该行被忽略。

最后，让我们从配置文件中读取已定义配置条目的确切值:

```
String property = configuration.getProperty(key);
```

## 3。带弹簧的装载配置

第二个解决方案是利用 Spring Spring 特性来处理一些文件的低级加载和处理。

让我们定义一个`Initializer`来保存初始化自定义类所需的配置。在`Bean`初始化期间，框架将从`*.properties`配置文件中加载所有标注有`@Value`的字段:

```
@Component
public class Initializer {

    private String someInitialValue;
    private String anotherManagedValue;

    public Initializer(
      @Value("someInitialValue") String someInitialValue,
      @Value("anotherValue") String anotherManagedValue) {

        this.someInitialValue = someInitialValue;
        this.anotherManagedValue = anotherManagedValue;
    }

    public ClassNotManagedBySpring initClass() {
        return new ClassNotManagedBySpring(
          this.someInitialValue, this.anotherManagedValue);
    }
}
```

`Initializer`现在可以负责实例化`ClassNotManagedBySpring` `.`

现在我们将简单地访问我们的`Initializer`实例并在其上运行`initClass()`方法来处理我们的自定义`ClassNotManagedBySpring`的实例化:

```
ClassNotManagedBySpring classNotManagedBySpring = initializer.initClass();
```

一旦我们有了对`Initializer`的引用，我们就能够实例化我们的自定义`ClassNotManagedBySpring.`

## 4。总结

在这个快速教程中，我们主要关注将属性读入一个非 Spring Java 类。

和往常一样，在 GitHub 上可以找到一个示例实现[。](https://web.archive.org/web/20220926201346/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-properties-2)