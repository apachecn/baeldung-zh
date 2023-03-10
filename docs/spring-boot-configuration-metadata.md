# Spring Boot 配置元数据指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-configuration-metadata>

## 1。概述

当编写 Spring Boot 应用程序时，将配置属性映射到 Java beans 上是很有帮助的。然而，记录这些属性的最佳方式是什么呢？

在本教程中，我们将探索 [Spring Boot 配置处理器](https://web.archive.org/web/20221205195547/https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-configuration-metadata.html#configuration-metadata-annotation-processor)和[相关联的 JSON 元数据文件](https://web.archive.org/web/20221205195547/https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-configuration-metadata.html#configuration-metadata-format)，这些文件记录了每个属性的含义、约束等。

## 2。配置元数据

作为开发人员，我们开发的大多数应用程序在某种程度上都必须是可配置的。然而，通常，我们并不真正理解配置参数的作用，如果它有默认值，如果它被否决，有时，我们甚至不知道属性的存在。

为了帮助我们，Spring Boot 在一个 JSON 文件中生成了配置元数据，为我们提供了如何使用这些属性的有用信息。因此，**配置元数据是一个描述性文件，它包含与配置属性交互的必要信息。**

这个文件真正的好处是**ide 也可以读取它**，给我们 Spring 属性的自动完成，以及其他配置提示。

## 3。依赖性

为了生成这个配置元数据，我们将使用来自 [`spring-boot-configuration-processor`依赖项](https://web.archive.org/web/20221205195547/https://search.maven.org/search?q=spring-boot-configuration-processor)的配置处理器。

所以，让我们继续添加依赖关系为`optional`:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <version>2.1.6.RELEASE</version>
    <optional>true</optional>
</dependency>
```

这种依赖性将为我们提供一个 Java 注释处理器，在我们构建项目时调用。我们稍后会详细讨论这一点。

最好的做法是在 Maven 中添加一个依赖项作为`optional`，以防止`@ConfigurationProperties` 被应用到我们项目使用的其他模块。

## 4。配置属性示例

为了查看运行中的处理器，让我们假设有几个属性需要通过 Java bean 包含在我们的 Spring Boot 应用程序中:

```java
@Configuration
@ConfigurationProperties(prefix = "database")
public class DatabaseProperties {

    public static class Server {

        private String ip;
        private int port;

        // standard getters and setters
    }

    private String username;
    private String password;
    private Server server;

    // standard getters and setters
}
```

为此，我们要使用 [`@ConfigurationProperties`](/web/20221205195547/https://www.baeldung.com/configuration-properties-in-spring-boot) 标注。**配置处理器扫描带有此注释**的类和方法，以访问配置参数并生成配置元数据。

让我们将这些属性添加到一个属性文件中。在这种情况下，我们称之为`databaseproperties-test.properties`:

```java
#Simple Properties
database.username=baeldung
database.password=password
```

为了确保万无一失，我们还将添加一个测试来确保我们都准备好了:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = AnnotationProcessorApplication.class)
@TestPropertySource("classpath:databaseproperties-test.properties")
public class DatabasePropertiesIntegrationTest {

    @Autowired
    private DatabaseProperties databaseProperties;

    @Test
    public void whenSimplePropertyQueriedThenReturnsPropertyValue() 
      throws Exception {
        Assert.assertEquals("Incorrectly bound Username property", 
          "baeldung", databaseProperties.getUsername());
        Assert.assertEquals("Incorrectly bound Password property", 
          "password", databaseProperties.getPassword());
    }

}
```

我们还通过内部类`Server`添加了嵌套属性`database.server.id`和`database.server.port`。**我们应该添加内部类** `**Server**` **以及一个具有自己的 getter 和 setter 的字段** `**server**` **。**

在我们的测试中，让我们快速检查一下，以确保我们也可以成功地设置和读取嵌套属性:

```java
@Test
public void whenNestedPropertyQueriedThenReturnsPropertyValue() 
  throws Exception {
    Assert.assertEquals("Incorrectly bound Server IP nested property",
      "127.0.0.1", databaseProperties.getServer().getIp());
    Assert.assertEquals("Incorrectly bound Server Port nested property", 
      3306, databaseProperties.getServer().getPort());
}
```

好了，现在我们可以使用处理器了。

## 5。生成配置元数据

我们前面提到过，配置处理器生成一个文件——它使用注释处理。

所以，编译完我们的项目后，我们会看到一个名为`**spring-configuration-metadata.json**`**`**target/classes/META-INF**`**:**的**文件****

```java
{
  "groups": [
    {
      "name": "database",
      "type": "com.baeldung.autoconfiguration.annotationprocessor.DatabaseProperties",
      "sourceType": "com.baeldung.autoconfiguration.annotationprocessor.DatabaseProperties"
    },
    {
      "name": "database.server",
      "type": "com.baeldung.autoconfiguration.annotationprocessor.DatabaseProperties$Server",
      "sourceType": "com.baeldung.autoconfiguration.annotationprocessor.DatabaseProperties",
      "sourceMethod": "getServer()"
    }
  ],
  "properties": [
    {
      "name": "database.password",
      "type": "java.lang.String",
      "sourceType": "com.baeldung.autoconfiguration.annotationprocessor.DatabaseProperties"
    },
    {
      "name": "database.server.ip",
      "type": "java.lang.String",
      "sourceType": "com.baeldung.autoconfiguration.annotationprocessor.DatabaseProperties$Server"
    },
    {
      "name": "database.server.port",
      "type": "java.lang.Integer",
      "sourceType": "com.baeldung.autoconfiguration.annotationprocessor.DatabaseProperties$Server",
      "defaultValue": 0
    },
    {
      "name": "database.username",
      "type": "java.lang.String",
      "sourceType": "com.baeldung.autoconfiguration.annotationprocessor.DatabaseProperties"
    }
  ],
  "hints": []
}
```

接下来，让我们看看更改 Java beans 上的注释是如何影响元数据的。

### 5.1.关于配置元数据的附加信息

首先，让我们在`Server`上添加 JavaDoc 注释。

其次，让我们给`database.server.port`字段一个默认值，最后添加`@Min`和`@Max`注释:

```java
public static class Server {

    /**
     * The IP of the database server
     */
    private String ip;

    /**
     * The Port of the database server.
     * The Default value is 443.
     * The allowed values are in the range 400-4000.
     */
    @Min(400)
    @Max(800)
    private int port = 443;

    // standard getters and setters
}
```

如果我们现在检查`spring-configuration-metadata.json`文件，我们将看到这些额外的信息被反映出来:

```java
{
  "groups": [
    {
      "name": "database",
      "type": "com.baeldung.autoconfiguration.annotationprocessor.DatabaseProperties",
      "sourceType": "com.baeldung.autoconfiguration.annotationprocessor.DatabaseProperties"
    },
    {
      "name": "database.server",
      "type": "com.baeldung.autoconfiguration.annotationprocessor.DatabaseProperties$Server",
      "sourceType": "com.baeldung.autoconfiguration.annotationprocessor.DatabaseProperties",
      "sourceMethod": "getServer()"
    }
  ],
  "properties": [
    {
      "name": "database.password",
      "type": "java.lang.String",
      "sourceType": "com.baeldung.autoconfiguration.annotationprocessor.DatabaseProperties"
    },
    {
      "name": "database.server.ip",
      "type": "java.lang.String",
      "description": "The IP of the database server",
      "sourceType": "com.baeldung.autoconfiguration.annotationprocessor.DatabaseProperties$Server"
    },
    {
      "name": "database.server.port",
      "type": "java.lang.Integer",
      "description": "The Port of the database server. The Default value is 443.
        The allowed values are in the range 400-4000",
      "sourceType": "com.baeldung.autoconfiguration.annotationprocessor.DatabaseProperties$Server",
      "defaultValue": 443
    },
    {
      "name": "database.username",
      "type": "java.lang.String",
      "sourceType": "com.baeldung.autoconfiguration.annotationprocessor.DatabaseProperties"
    }
  ],
  "hints": []
}
```

我们可以用`database.server.ip`和`database.server.port `字段检查差异。事实上，额外的信息非常有用。因此，开发人员和 ide 更容易理解每个属性的作用。

我们还应该确保触发构建来获得更新的文件。在 Eclipse 中，如果我们选中`Build Automatically`选项，每个保存动作都会触发一个构建。在 IntelliJ 中，我们应该手动触发构建。

### 5.2。了解元数据格式

让我们仔细看看 JSON 元数据文件，并讨论它的组件。

`Groups`是用于对其他属性进行分组的高级项目，本身不指定值。在我们的例子中，我们有`database`组，它也是配置属性的前缀。我们还有一个`server` 组，它是通过内部类和组`ip`和`port`属性创建的。

`Properties`是我们可以为其指定值的配置项。这些属性在`.properties`或`.yml`文件中设置，可以有额外的信息，比如默认值和验证，正如我们在上面的例子中看到的。

`Hints`是帮助用户设置属性值的附加信息。例如，如果我们有一组属性允许值，我们可以提供每个值的描述。IDE 将为这些提示提供自动竞争帮助。

配置元数据上的每个组件都有自己的[属性](https://web.archive.org/web/20221205195547/https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-configuration-metadata.html)来更详细地解释配置属性。

## 6。结论

在本文中，我们研究了 Spring Boot 配置处理器及其创建配置元数据的能力。使用这些元数据使得与我们的配置参数进行交互变得更加容易。

我们给出了一个生成的配置元数据的例子，并详细解释了它的格式和组件。

我们还看到了 IDE 上的自动完成支持是多么有用。

和往常一样，本文中提到的所有代码片段都可以在我们的 GitHub 库中找到。**