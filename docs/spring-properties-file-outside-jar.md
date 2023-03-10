# jar 外部的 Spring 属性文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-properties-file-outside-jar>

## 1.概观

属性文件是我们可以用来存储特定于项目的信息的一种常用方法。理想情况下，我们应该将它放在包装的外部，以便能够根据需要对配置进行更改。

在这个快速教程中，我们将研究从一个 [Spring Boot 应用程序](/web/20220626072207/https://www.baeldung.com/properties-with-spring)的 jar 之外的位置加载属性文件的各种**方法。**

## 2.使用默认位置

按照惯例，Spring Boot 在四个预先确定的位置按照以下优先顺序查找外部化的配置文件— `application.properties` 或`application.yml` — :

*   当前目录的一个`/config`子目录
*   当前目录
*   类路径`/config`包
*   类路径根

因此，在当前目录的`/config`子目录下的`application.properties`中定义的**属性将被加载。**如果发生冲突，这也将覆盖其他位置的属性。

## 3.使用命令行

如果以上约定对我们不起作用，我们可以直接在命令行中**配置位置:**

```java
java -jar app.jar --spring.config.location=file:///Users/home/config/jdbc.properties
```

我们还可以传递应用程序将搜索文件的文件夹位置:

```java
java -jar app.jar --spring.config.name=application,jdbc --spring.config.location=file:///Users/home/config
```

最后，另一种方法是通过 [Maven 插件](/web/20220626072207/https://www.baeldung.com/spring-boot-command-line-arguments)运行 Spring Boot 应用程序。

这里，我们可以使用一个`-D`参数:

```java
mvn spring-boot:run -Dspring.config.location="file:///Users/home/jdbc.properties"
```

## 4.使用环境变量

现在让我们说，我们不能改变启动命令。

很棒的是， **Spring Boot 还会读取环境变量`SPRING_CONFIG_NAME`和`SPRING_CONFIG_LOCATION`** :

```java
export SPRING_CONFIG_NAME=application,jdbc
export SPRING_CONFIG_LOCATION=file:///Users/home/config
java -jar app.jar
```

请注意，仍然会加载默认文件。但是在属性冲突的情况下，特定于环境的属性文件优先于属性文件。

## 5.使用应用程序属性

正如我们所看到的，我们必须在应用程序启动之前定义`spring.config.name` 和`spring.config.location`属性，所以在`application.properties` 文件(或 YAML 的对应文件)中使用它们没有任何效果。

Spring Boot 修改了版本 2.4.0 中处理属性的方式。

除了这一变化，该团队还引入了一个新的属性，允许直接从应用程序属性中导入其他配置文件:

```java
spring.config.import=file:./additional.properties,optional:file:/Users/home/config/jdbc.properties
```

## 6.程序化

如果我们想要编程访问，我们可以注册一个`PropertySourcesPlaceholderConfigurer` bean:

```java
public PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
    PropertySourcesPlaceholderConfigurer properties = 
      new PropertySourcesPlaceholderConfigurer();
    properties.setLocation(new FileSystemResource("/Users/home/conf.properties"));
    properties.setIgnoreResourceNotFound(false);
    return properties;
}
```

这里我们使用了`PropertySourcesPlaceholderConfigurer` 从一个定制的位置加载属性。

## 7。从 Fat Jar 中排除文件

Maven Boot 插件会自动将`src/main/resources`目录中的所有文件包含到 jar 包中。

如果我们不想让文件成为 jar 的一部分，我们可以使用一个简单的配置来排除它:

```java
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
            <excludes>
                <exclude>**/conf.properties</exclude>
            </excludes>
        </resource>
    </resources>
</build>
```

在这个例子中，我们已经过滤掉了包含在结果 jar 中的`conf.properties`文件。

## 8.结论

这篇文章展示了 Spring Boot 框架本身如何为我们处理[外部化的配置](/web/20220626072207/https://www.baeldung.com/configuration-properties-in-spring-boot)。

通常，我们只需要将属性值放在正确的文件和位置。但是我们也可以使用 Spring 的 Java API 进行更多的控制。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220626072207/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-environment)