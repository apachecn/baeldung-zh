# 春天和 Spring Boot 的属性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/properties-with-spring>

## 1。概述

本教程将向**展示如何通过 Java 配置和`@PropertySource.`在 Spring** 中设置和使用属性

我们还将了解房地产在 Spring Boot 是如何运作的。

## 延伸阅读:

## [春天表情语言指南](/web/20221001115719/https://www.baeldung.com/spring-expression-language)

This article explores Spring Expression Language (SpEL), a powerful expression language that supports querying and manipulating object graphs at runtime.[Read more](/web/20221001115719/https://www.baeldung.com/spring-expression-language) →

## [配置 Spring Boot 网络应用](/web/20221001115719/https://www.baeldung.com/spring-boot-application-configuration)

Some of the more useful configs for a Spring Boot application.[Read more](/web/20221001115719/https://www.baeldung.com/spring-boot-application-configuration) →

## [Spring Boot @ configuration properties 指南](/web/20221001115719/https://www.baeldung.com/configuration-properties-in-spring-boot)

A quick and practical guide to @ConfigurationProperties annotation in Spring Boot.[Read more](/web/20221001115719/https://www.baeldung.com/configuration-properties-in-spring-boot) →

## 2。通过注释注册一个属性文件

Spring 3.1 还引入了新的`@PropertySource`注释作为向环境添加属性源的便利机制。

我们可以结合使用这个注释和`@Configuration`注释:

```java
@Configuration
@PropertySource("classpath:foo.properties")
public class PropertiesWithJavaConfig {
    //...
}
```

注册新属性文件的另一个非常有用的方法是使用占位符，它允许我们在运行时动态选择正确的文件:

```java
@PropertySource({ 
  "classpath:persistence-${envTarget:mysql}.properties"
})
...
```

### 2.1.定义多个物业位置

根据 Java 8 惯例，注释`@PropertySource`是可重复的[。因此，如果我们使用 Java 8 或更高版本，我们可以使用这个注释来定义多个属性位置:](https://web.archive.org/web/20221001115719/https://docs.oracle.com/javase/tutorial/java/annotations/repeating.html)

```java
@PropertySource("classpath:foo.properties")
@PropertySource("classpath:bar.properties")
public class PropertiesWithJavaConfig {
    //...
}
```

当然，**我们也可以使用`@PropertySources`注释并指定一个`@PropertySource`数组。**这适用于任何受支持的 Java 版本，而不仅仅是 Java 8 或更高版本:

```java
@PropertySources({
    @PropertySource("classpath:foo.properties"),
    @PropertySource("classpath:bar.properties")
})
public class PropertiesWithJavaConfig {
    //...
}
```

在这两种情况下，值得注意的是，在属性名冲突的情况下，最后读取的源优先。

## 3。使用/注入属性

**用 [`@Value`标注](/web/20221001115719/https://www.baeldung.com/spring-value-annotation)** 注入一个属性很简单:

```java
@Value( "${jdbc.url}" )
private String jdbcUrl;
```

**我们还可以为属性指定一个默认值:**

```java
@Value( "${jdbc.url:aDefaultUrl}" )
private String jdbcUrl;
```

Spring 3.1 中新增的`PropertySourcesPlaceholderConfigurer`**解析 bean 定义属性值和`@Value`注释**中的${…}占位符。

最后，我们可以使用`Environment` API 获得属性的值:

```java
@Autowired
private Environment env;
...
dataSource.setUrl(env.getProperty("jdbc.url"));
```

## 4。Spring Boot 的房产

在我们进入更高级的属性配置选项之前，让我们花点时间看看 Spring Boot 新的属性支持。

一般来说，**这种新的支持与标准的 Spring** 相比涉及的配置更少，这当然是 Boot 的主要目标之一。

### 4.1。`application.properties:`默认属性文件

Boot 将其典型的配置惯例应用于属性文件。这意味着**我们可以简单地将一个`application.properties`文件放到我们的`src/main/resources`目录中，它将被自动检测到**。然后我们可以像平常一样注入任何加载的属性。

因此，通过使用这个默认文件，我们不必显式地注册一个`PropertySource`或者甚至提供一个属性文件的路径。

如果需要，我们还可以使用环境属性在运行时配置不同的文件:

```java
java -jar app.jar --spring.config.location=classpath:/another-location.properties
```

从 [Spring Boot 2.3](https://web.archive.org/web/20221001115719/https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.3-Release-Notes#support-of-wildcard-locations-for-configuration-files) 、**开始，我们也可以为配置文件**指定通配符位置。

例如，我们可以将`spring.config.location `属性设置为`config/*/`:

```java
java -jar app.jar --spring.config.location=config/*/
```

这样，Spring Boot 将在我们的 jar 文件之外寻找与`config/*/ `目录模式匹配的配置文件。当我们有多个配置属性源时，这就很方便了。

从版本`2.4.0`，**开始，Spring Boot 支持使用多文档属性文件**，类似于[YAML 对](https://web.archive.org/web/20221001115719/https://yaml.org/spec/1.2/spec.html#id2760395)的设计:

```java
baeldung.customProperty=defaultValue
#---
baeldung.customProperty=overriddenValue
```

注意，对于属性文件，三划线符号前面有一个注释字符(`#`)。

### 4.2。特定于环境的属性文件

如果我们需要针对不同的环境，在引导中有一个内置的机制。

**我们可以简单地在`src/main/resources`目录下定义一个`application-environment.properties` 文件，然后用相同的环境名设置一个 Spring 概要文件。**

例如，如果我们定义一个“暂存”环境，这意味着我们必须定义一个`staging`概要文件，然后是`application-staging.properties`。

该环境文件将被加载，**将优先于默认属性文件。**注意，默认文件仍然会被加载，只是当有属性冲突时，特定于环境的属性文件优先。

### 4.3。特定于测试的属性文件

在测试应用程序时，我们可能还需要使用不同的属性值。

**Spring Boot 通过在测试运行**期间查看我们的`src/test/resources`目录来为我们处理这个问题。同样，默认属性仍可正常注入，但如果发生冲突，将被这些属性覆盖。

### 4.4。`@TestPropertySource`注解

如果我们需要对测试属性进行更细粒度的控制，那么我们可以使用`[@TestPropertySource](https://web.archive.org/web/20221001115719/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/context/TestPropertySource.html)`注释。

**这允许我们为一个特定的测试环境设置测试属性，优先于默认的属性源:**

```java
@RunWith(SpringRunner.class)
@TestPropertySource("/foo.properties")
public class FilePropertyInjectionUnitTest {

    @Value("${foo}")
    private String foo;

    @Test
    public void whenFilePropertyProvided_thenProperlyInjected() {
        assertThat(foo).isEqualTo("bar");
    }
}
```

如果我们不想使用文件，我们可以直接指定名称和值:

```java
@RunWith(SpringRunner.class)
@TestPropertySource(properties = {"foo=bar"})
public class PropertyInjectionUnitTest {

    @Value("${foo}")
    private String foo;

    @Test
    public void whenPropertyProvided_thenProperlyInjected() {
        assertThat(foo).isEqualTo("bar");
    }
}
```

**我们也可以使用`[@SpringBootTest](https://web.archive.org/web/20221001115719/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/context/SpringBootTest.html)`注释的`properties` 参数:**达到类似的效果

```java
@RunWith(SpringRunner.class)
@SpringBootTest(
  properties = {"foo=bar"}, classes = SpringBootPropertiesTestApplication.class)
public class SpringBootPropertyInjectionIntegrationTest {

    @Value("${foo}")
    private String foo;

    @Test
    public void whenSpringBootPropertyProvided_thenProperlyInjected() {
        assertThat(foo).isEqualTo("bar");
    }
}
```

### 4.5。分层属性

如果我们有分组在一起的属性，我们可以利用`[@ConfigurationProperties](https://web.archive.org/web/20221001115719/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/context/properties/ConfigurationProperties.html)`注释，它会将这些属性层次结构映射到 Java 对象图中。

让我们看一些用于配置数据库连接的属性:

```java
database.url=jdbc:postgresql:/localhost:5432/instance
database.username=foo
database.password=bar
```

然后让我们使用注释将它们映射到一个数据库对象:

```java
@ConfigurationProperties(prefix = "database")
public class Database {
    String url;
    String username;
    String password;

    // standard getters and setters
}
```

Spring Boot 再次应用了配置方法的惯例，在属性名和它们对应的字段之间自动映射。我们需要提供的只是属性前缀。

如果您想更深入地了解配置属性，可以看看我们的深度文章。

### 4.6。备选:YAML 文件

Spring 也支持 YAML 文件。

所有相同的命名规则都适用于特定于测试、特定于环境和默认的属性文件。唯一的区别是文件扩展名和对类路径中的 [SnakeYAML](https://web.archive.org/web/20221001115719/https://bitbucket.org/asomov/snakeyaml) 库的依赖。

**YAML 特别擅长分层物业存储**；以下属性文件:

```java
database.url=jdbc:postgresql:/localhost:5432/instance
database.username=foo
database.password=bar
secret: foo
```

与以下 YAML 文件同义:

```java
database:
  url: jdbc:postgresql:/localhost:5432/instance
  username: foo
  password: bar
secret: foo
```

同样值得一提的是，YAML 文件不支持`@PropertySource`注释，所以如果我们需要使用这个注释，它会限制我们使用属性文件。

另一个值得注意的地方是，在 2.4.0 版本中，Spring Boot 改变了从多文档 YAML 文件中加载属性的方式。以前，它们的添加顺序基于配置文件激活顺序。然而，在新版本中，框架遵循了我们之前为`.properties`文件指出的相同的排序规则；在文件中声明的较低的属性将简单地覆盖那些较高的属性。

此外，在这个版本中，概要文件不再能够从概要文件特定的文档中被激活，使得结果更加清晰和可预测。

### 4.7。导入附加配置文件

在版本 2.4.0 之前，Spring Boot 允许使用`spring.config.location`和`spring.config.additional-location `属性包含额外的配置文件，但是它们有一定的限制。例如，它们必须在启动应用程序之前定义(作为环境或系统属性，或者使用命令行参数)，因为它们在过程的早期使用。

在提到的版本中，**我们可以使用`application.properties `或`application.yml `文件中的`spring.config.import`属性来轻松地包含附加文件。**这个属性支持一些有趣的特性:

*   添加几个文件或目录
*   这些文件可以从类路径或外部目录加载
*   指示如果找不到文件，启动过程是否应该失败，或者它是否是一个可选文件
*   导入无扩展名文件

让我们看一个有效的例子:

```java
spring.config.import=classpath:additional-application.properties,
  classpath:additional-application[.yml],
  optional:file:./external.properties,
  classpath:additional-application-properties/
```

注意:为了清楚起见，我们在这里用换行符格式化了这个属性。

Spring 将把 imports 作为一个新文档，直接插入到 import 声明的下面。

### 4.8。命令行参数的属性

除了使用文件，我们还可以在命令行上直接传递属性:

```java
java -jar app.jar --property="value"
```

我们也可以通过系统属性做到这一点，这些属性是在`-jar` 命令之前而不是之后提供的:

```java
java -Dproperty.name="value" -jar app.jar
```

### 4.9。环境变量的属性

Spring Boot 还将检测环境变量，将其视为属性:

```java
export name=value
java -jar app.jar 
```

### 4.10。属性值的随机化

如果我们不想要决定论的属性值，我们可以使用`[RandomValuePropertySource](https://web.archive.org/web/20221001115719/https://docs.spring.io/spring-boot/docs/1.5.7.RELEASE/api/org/springframework/boot/context/config/RandomValuePropertySource.html)`来随机化属性值:

```java
random.number=${random.int}
random.long=${random.long}
random.uuid=${random.uuid}
```

### 4.11。其他类型的财产来源

Spring Boot 支持多种属性来源，实现了一个经过深思熟虑的排序，允许明智的覆盖。值得参考一下[官方文档](https://web.archive.org/web/20221001115719/https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html)，它超出了本文的范围。

## 5。配置使用生豆—`PropertySourcesPlaceholderConfigurer`

除了将属性放入 Spring 的便捷方法之外，我们还可以手动定义和注册属性配置 bean。

**使用`PropertySourcesPlaceholderConfigurer`可以让我们完全控制配置，缺点是更加冗长，而且大多数时候是不必要的。**

让我们看看如何使用 Java 配置来定义这个 bean:

```java
@Bean
public static PropertySourcesPlaceholderConfigurer properties(){
    PropertySourcesPlaceholderConfigurer pspc
      = new PropertySourcesPlaceholderConfigurer();
    Resource[] resources = new ClassPathResource[ ]
      { new ClassPathResource( "foo.properties" ) };
    pspc.setLocations( resources );
    pspc.setIgnoreUnresolvablePlaceholders( true );
    return pspc;
}
```

## 6。父子上下文中的属性

这个问题反复出现:当我们的 **web 应用程序有一个父上下文和一个子上下文**时会发生什么？父上下文可能有一些公共的核心功能和 bean，然后是一个(或多个)子上下文，可能包含特定于 servlet 的 bean。

在这种情况下，定义属性文件并将其包含在这些上下文中的最佳方式是什么？如何最好地从 Spring 中检索这些属性？

我们来简单分解一下。

如果文件是在父上下文中定义的**:**

*   `@Value` 在**子上下文中工作**:是

*   `@Value`在**父上下文**中工作:是

*   `environment.getProperty`在**子上下文中**:是

*   **父上下文**中的`environment.getProperty`:是

如果文件是在子上下文中定义的**:**

*   `@Value`在**子上下文中工作**:是

*   `@Value`在**父上下文中工作**:否

*   `environment.getProperty`在**子上下文中**:是

*   **父上下文**中的`environment.getProperty`:否

## 7 .**。结论**

本文展示了几个在 Spring 中使用属性和属性文件的例子。

和往常一样，支持这篇文章的全部代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221001115719/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-properties)