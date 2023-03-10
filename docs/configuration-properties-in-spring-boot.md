# Spring Boot @ configuration properties 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/configuration-properties-in-spring-boot>

## 1。简介

Spring Boot 有许多有用的特性，包括**外部化的配置和容易访问属性文件**中定义的属性。一个早期的[教程](/web/20220529014431/https://www.baeldung.com/properties-with-spring)描述了各种各样的方法。

我们现在将更详细地研究 *@ConfigurationProperties* 注释。

## 延伸阅读:

## [Spring @ Value 快速指南](/web/20220529014431/https://www.baeldung.com/spring-value-annotation)

Learn to use the Spring @Value annotation to configure fields from property files, system properties, etc.[Read more](/web/20220529014431/https://www.baeldung.com/spring-value-annotation) →

## [弹簧和 Spring Boot 的特性](/web/20220529014431/https://www.baeldung.com/properties-with-spring)

Tutorial for how to work with properties files and property values in Spring.[Read more](/web/20220529014431/https://www.baeldung.com/properties-with-spring) →

## 2。设置

本教程使用一个相当标准的设置。我们首先在我们的 *pom.xml* 中添加[*spring-boot-starter-parent*](https://web.archive.org/web/20220529014431/https://search.maven.org/search?q=a:spring-boot-starter-parent%20AND%20g:org.springframework.boot)作为父节点:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.0</version>
    <relativePath/>
</parent>
```

为了能够验证文件中定义的属性，我们还需要一个 JSR-303 的实现，而[*hibernate-validator*](https://web.archive.org/web/20220529014431/https://search.maven.org/search?q=a:hibernate-validator%20AND%20g:org.hibernate)就是其中之一。

让我们将它添加到我们的 pom.xml 中:

```java
<dependency>
   <groupId>org.hibernate</groupId>
   <artifactId>hibernate-validator</artifactId>
   <version>6.0.16.Final</version>
</dependency> 
```

【Hibernate Validator 入门”页面有更多的细节。

## 3。简单属性

官方文档建议我们将配置属性隔离到单独的 POJOs 中。

所以让我们从这样开始:

```java
@Configuration
@ConfigurationProperties(prefix = "mail")
public class ConfigProperties {

    private String hostName;
    private int port;
    private String from;

    // standard getters and setters
}
```

我们使用 *@Configuration* 以便 Spring 在应用程序上下文中创建一个 Spring bean。

**`@ConfigurationProperties`最适用于具有相同前缀的分层属性；**因此，我们加一个前缀`mail`。

Spring 框架使用标准的 JavaBean setter，所以我们必须为每个属性声明 setter。

注意:如果我们在 POJO 中不使用 *@Configuration* ，那么我们需要在主 Spring 应用程序类中添加`@EnableConfigurationProperties(ConfigProperties.class)`来将属性绑定到 POJO 中:

```java
@SpringBootApplication
@EnableConfigurationProperties(ConfigProperties.class)
public class EnableConfigurationDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(EnableConfigurationDemoApplication.class, args);
    }
}
```

就是这样！ **Spring 将自动绑定我们的属性文件中定义的前缀为 *mail* 并且与`ConfigProperties`类**中的一个字段同名的任何属性。

Spring 对绑定属性使用了一些宽松的规则。因此，以下变体都被绑定到属性 *hostName* :

```java
mail.hostName
mail.hostname
mail.host_name
mail.host-name
mail.HOST_NAME 
```

因此，我们可以使用下面的属性文件来设置所有的字段:

```java
#Simple properties
[[email protected]](/web/20220529014431/https://www.baeldung.com/cdn-cgi/l/email-protection)
mail.port=9000
[[email protected]](/web/20220529014431/https://www.baeldung.com/cdn-cgi/l/email-protection) 
```

### 3.1.Spring Boot 2.2

**从 [Spring Boot 2.2](https://web.archive.org/web/20220529014431/https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.2-Release-Notes#configurationproperties-scanning) 开始，Spring 通过类路径扫描**找到并注册`@ConfigurationProperties` 类。因此，**没有必要用`@Component`** `(and other meta-annotations like @Configuration),` **或者甚至用`@EnableConfigurationProperties:`** 来注释这样的类

```java
@ConfigurationProperties(prefix = "mail") 
public class ConfigProperties { 

    private String hostName; 
    private int port; 
    private String from; 

    // standard getters and setters 
} 
```

由`@SpringBootApplication` 启用的类路径扫描器找到了`ConfigProperties` 类，即使我们没有用`@Component.`注释这个类

此外，我们可以使用**和`[@ConfigurationPropertiesScan](https://web.archive.org/web/20220529014431/https://docs.spring.io/spring-boot/docs/2.2.0.RELEASE/api/org/springframework/boot/context/properties/ConfigurationPropertiesScan.html)` 注释来扫描配置属性类的定制位置:**

```java
@SpringBootApplication
@ConfigurationPropertiesScan("com.baeldung.configurationproperties")
public class EnableConfigurationDemoApplication { 

    public static void main(String[] args) {   
        SpringApplication.run(EnableConfigurationDemoApplication.class, args); 
    } 
}
```

这样，Spring 将只在`com.baeldung.properties` 包中寻找配置属性类。

## 4。嵌套属性

**我们可以在`Lists, Maps,`和`Classes.`** 中嵌套属性

让我们为一些嵌套属性创建一个新的`Credentials`类:

```java
public class Credentials {
    private String authMethod;
    private String username;
    private String password;

    // standard getters and setters
}
```

我们还需要更新`ConfigProperties`类以使用`List,`和`Map`，并且更新`Credentials`类:

```java
public class ConfigProperties {

    private String host;
    private int port;
    private String from;
    private List<String> defaultRecipients;
    private Map<String, String> additionalHeaders;
    private Credentials credentials;

    // standard getters and setters
}
```

以下属性文件将设置所有字段:

```java
#Simple properties
[[email protected]](/web/20220529014431/https://www.baeldung.com/cdn-cgi/l/email-protection)
mail.port=9000
[[email protected]](/web/20220529014431/https://www.baeldung.com/cdn-cgi/l/email-protection)

#List properties
mail.defaultRecipients[0][[email protected]](/web/20220529014431/https://www.baeldung.com/cdn-cgi/l/email-protection)
mail.defaultRecipients[1][[email protected]](/web/20220529014431/https://www.baeldung.com/cdn-cgi/l/email-protection)

#Map Properties
mail.additionalHeaders.redelivery=true
mail.additionalHeaders.secure=true

#Object properties
mail.credentials.username=john
mail.credentials.password=password
mail.credentials.authMethod=SHA1
```

## 5。在`@Bean`方法上使用`@ConfigurationProperties`

**我们也可以在`@Bean`注释的方法上使用`@ConfigurationProperties`注释。**

当我们想要将属性绑定到不受我们控制的第三方组件时，这种方法可能特别有用。

让我们创建一个简单的`Item`类，我们将在下一个例子中使用:

```java
public class Item {
    private String name;
    private int size;

    // standard getters and setters
}
```

现在让我们看看如何在一个`@Bean`方法上使用`@ConfigurationProperties`来将外部化的属性绑定到`Item`实例:

```java
@Configuration
public class ConfigProperties {

    @Bean
    @ConfigurationProperties(prefix = "item")
    public Item item() {
        return new Item();
    }
}
```

因此，任何带有条目前缀的属性都将被映射到由 Spring 上下文管理的`Item`实例。

## 6。属性验证

**`@ConfigurationProperties`使用 JSR-303 格式提供属性验证。**这允许各种各样的整洁的东西。

例如，让我们将`hostName`属性设为强制属性:

```java
@NotBlank
private String hostName;
```

接下来，让我们将`authMethod`属性的长度设为 1 到 4 个字符:

```java
@Length(max = 4, min = 1)
private String authMethod;
```

然后是从 1025 到 65536 的`port`属性:

```java
@Min(1025)
@Max(65536)
private int port; 
```

最后，`from`属性必须匹配电子邮件地址格式:

```java
@Pattern(regexp = "^[a-z0-9._%+-][[email protected]](/web/20220529014431/https://www.baeldung.com/cdn-cgi/l/email-protection)[a-z0-9.-]+\\.[a-z]{2,6}$")
private String from; 
```

这有助于我们减少代码中大量的*if–else*条件，使代码看起来更加简洁明了。

**如果这些验证中的任何一个失败，那么主应用程序将无法启动，并出现*IllegalStateException***。

Hibernate 验证框架使用标准的 JavaBean getter 和 setter，因此为每个属性声明 getter 和 setter 非常重要。

## 7。属性转换

`@ConfigurationProperties`支持将属性绑定到对应 beans 的多种类型的转换。

### 7.1。`Duration`

我们将从查看将属性转换成`Duration` 对象`.`开始

这里我们有两个类型为`Duration`的字段:

```java
@ConfigurationProperties(prefix = "conversion")
public class PropertyConversion {

    private Duration timeInDefaultUnit;
    private Duration timeInNano;
    ...
}
```

这是我们的属性文件:

```java
conversion.timeInDefaultUnit=10
conversion.timeInNano=9ns
```

结果，字段`timeInDefaultUnit`将具有 10 毫秒的值，并且`timeInNano`将具有 9 纳秒的值。

**支持的单位是`ns, us, ms, s, m, h`和`d`，分别代表纳秒、微秒、毫秒、秒、分钟、小时和天。**

默认单位是毫秒，这意味着如果我们不在数值旁边指定一个单位，Spring 会将该值转换为毫秒。

我们也可以使用`@DurationUnit:`覆盖默认单位

```java
@DurationUnit(ChronoUnit.DAYS)
private Duration timeInDays;
```

这是相应的属性:

```java
conversion.timeInDays=2
```

### 7.2。`DataSize`

**同样，Spring Boot `@ConfigurationProperties`支持`DataSize`类型转换。**

让我们添加三个类型为`DataSize`的字段:

```java
private DataSize sizeInDefaultUnit;

private DataSize sizeInGB;

@DataSizeUnit(DataUnit.TERABYTES)
private DataSize sizeInTB;
```

这些是相应的属性:

```java
conversion.sizeInDefaultUnit=300
conversion.sizeInGB=2GB
conversion.sizeInTB=4
```

**在这种情况下，`sizeInDefaultUnit`值将是 300 字节，因为默认单位是字节。**

支持的单位是`B, KB, MB, GB`和`TB.` ，我们也可以使用`@DataSizeUnit.`覆盖默认单位

### 7.3。`Converter`风俗

我们还可以添加我们自己的定制`Converter`来支持将属性转换成特定的类类型。

让我们添加一个简单的类`Employee`:

```java
public class Employee {
    private String name;
    private double salary;
}
```

然后，我们将创建一个自定义转换器来转换该属性:

```java
conversion.employee=john,2000
```

我们将把它转换成类型为`Employee`的文件:

```java
private Employee employee;
```

我们将需要实现`Converter`接口，然后**使用`@ConfigurationPropertiesBinding`注释来注册我们的自定义** `**Converter**:`

```java
@Component
@ConfigurationPropertiesBinding
public class EmployeeConverter implements Converter<String, Employee> {

    @Override
    public Employee convert(String from) {
        String[] data = from.split(",");
        return new Employee(data[0], Double.parseDouble(data[1]));
    }
}
```

## 8.不可变的`@ConfigurationProperties`绑定

从 Spring Boot 2.2 开始，**我们可以使用`@ConstructorBinding`注释来绑定我们的配置属性**。

这实质上意味着`@ConfigurationProperties`注释的类现在可能是[不可变的](/web/20220529014431/https://www.baeldung.com/java-immutable-object)。

```java
@ConfigurationProperties(prefix = "mail.credentials")
@ConstructorBinding
public class ImmutableCredentials {

    private final String authMethod;
    private final String username;
    private final String password;

    public ImmutableCredentials(String authMethod, String username, String password) {
        this.authMethod = authMethod;
        this.username = username;
        this.password = password;
    }

    public String getAuthMethod() {
        return authMethod;
    }

    public String getUsername() {
        return username;
    }

    public String getPassword() {
        return password;
    }
}
```

正如我们所见，当使用 `@ConstructorBinding,` 时，我们需要向构造函数提供我们想要绑定的所有参数。

注意 `ImmutableCredentials` 的所有字段都是最终的。此外，没有 setter 方法。

此外，需要强调的是 **要使用构造函数绑定，我们需要用 `@EnableConfigurationProperties` 或 `with @ConfigurationPropertiesScan`** `.`显式启用我们的配置类

## 9.Java 16 `record` s

Java 16 引入了`record `类型作为 [JEP 395](https://web.archive.org/web/20220529014431/https://openjdk.java.net/jeps/395) 的一部分。记录是充当不可变数据的透明载体的类。这使他们成为配置持有者和 dto 的完美候选人。事实上，**我们可以在 Spring Boot** 将 Java 记录定义为配置属性。例如，前面的示例可以重写为:

```java
@ConstructorBinding
@ConfigurationProperties(prefix = "mail.credentials")
public record ImmutableCredentials(String authMethod, String username, String password) {
}
```

显然，与那些吵闹的 getters 和 setters 相比，它更简洁。

此外，截至 [Spring Boot 2.6](https://web.archive.org/web/20220529014431/https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.6.0-M2-Release-Notes#records-and-configurationproperties) 、**对于单构造者记录，我们可以去掉`@ConstructorBinding `注释**。但是，如果我们的记录有多个构造函数，那么`@ConstructorBinding`仍然应该用来标识用于属性绑定的构造函数。

## 10。结论

在本文中，我们探索了 *@ConfigurationProperties* 注释，并强调了它提供的一些有用的特性，比如宽松绑定和 Bean 验证。

像往常一样，代码可以在 Github 上的[处获得。](https://web.archive.org/web/20220529014431/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-properties)