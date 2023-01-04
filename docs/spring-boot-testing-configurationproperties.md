# 测试 Spring Boot @配置属性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-testing-configurationproperties>

## 1.概观

在之前的`[@ConfigurationProperties](/web/20220827110142/https://www.baeldung.com/configuration-properties-in-spring-boot),`指南中，我们学习了如何通过 Spring Boot 设置和使用`@ConfigurationProperties` 注释来处理外部配置。

在本教程中，**我们将讨论如何测试依赖于`@ConfigurationProperties`注释**的配置类，以确保我们的配置数据被正确加载并绑定到相应的字段。

## 2.属国

在我们的 Maven 项目中，我们将使用 [`spring-boot-starter`](https://web.archive.org/web/20220827110142/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter%22) 和 [`spring-boot-starter-test`](https://web.archive.org/web/20220827110142/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-test%22) 依赖项来启用核心 spring API 和 spring 的测试 API。此外，我们将使用 [`spring-boot-starter-validation`](https://web.archive.org/web/20220827110142/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter-validation%22) 作为 bean 验证依赖项:

```java
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.0</version>
</parent>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

## 3.绑定到用户定义的 POJOs 的属性

当使用外部化的配置时，**我们通常创建包含与匹配的配置属性**相对应的字段的 POJOs。我们已经知道，Spring 会自动将配置属性绑定到我们创建的 Java 类。

首先，让我们假设在一个名为`src/test/resources/server-config-test.properties`的属性文件中有一些服务器配置:

```java
server.address.ip=192.168.0.1
server.resources_path.imgs=/root/imgs
```

我们将定义一个对应于前面的属性文件的简单配置类:

```java
@Configuration
@ConfigurationProperties(prefix = "server")
public class ServerConfig {

    private Address address;
    private Map<String, String> resourcesPath;

    // getters and setters
}
```

还有相应的`Address`类型:

```java
public class Address {

    private String ip;

    // getters and setters
}
```

最后，我们将把`ServerConfig` POJO 注入到我们的测试类中，并验证它的所有字段都设置正确:

```java
@ExtendWith(SpringExtension.class)
@EnableConfigurationProperties(value = ServerConfig.class)
@TestPropertySource("classpath:server-config-test.properties")
public class BindingPropertiesToUserDefinedPOJOUnitTest {

    @Autowired
    private ServerConfig serverConfig;

    @Test
    void givenUserDefinedPOJO_whenBindingPropertiesFile_thenAllFieldsAreSet() {
        assertEquals("192.168.0.1", serverConfig.getAddress().getIp());

        Map<String, String> expectedResourcesPath = new HashMap<>();
        expectedResourcesPath.put("imgs", "/root/imgs");
        assertEquals(expectedResourcesPath, serverConfig.getResourcesPath());
    }
}
```

在本测试中，我们使用了以下注释:

*   `[@ExtendWith](/web/20220827110142/https://www.baeldung.com/junit-5-extensions)`–将 Spring 的 TestContext 框架与 JUnit5 集成
*   `[@EnableConfigurationProperties](/web/20220827110142/https://www.baeldung.com/spring-enable-config-properties)` –启用对`@ConfigurationProperties`bean(在本例中是`ServerConfig` bean)的支持
*   `[@TestPropertySource](/web/20220827110142/https://www.baeldung.com/spring-test-property-source)` –指定覆盖默认`application.properties`文件的测试文件

## 4.`@ConfigurationProperties`上`@Bean`的方法

**创建配置 beans 的另一种方式是在`@Bean`方法**上使用`@ConfigurationProperties`注释。

例如，下面的`getDefaultConfigs()`方法创建了一个`ServerConfig`配置 bean:

```java
@Configuration
public class ServerConfigFactory {

    @Bean(name = "default_bean")
    @ConfigurationProperties(prefix = "server.default")
    public ServerConfig getDefaultConfigs() {
        return new ServerConfig();
    }
}
```

正如我们所见，我们能够在`getDefaultConfigs()`方法上使用`@ConfigurationProperties`来配置`ServerConfig`实例，而不必编辑`ServerConfig`类本身。**这在处理访问受限的外部第三方类时特别有用。**

接下来，我们将定义一个示例外部属性:

```java
server.default.address.ip=192.168.0.2
```

最后，为了告诉 Spring 在加载`ApplicationContext`时使用`ServerConfigFactory`类(从而创建我们的配置 bean)，我们将向测试类添加`@ContextConfiguration`注释:

```java
@ExtendWith(SpringExtension.class)
@EnableConfigurationProperties(value = ServerConfig.class)
@ContextConfiguration(classes = ServerConfigFactory.class)
@TestPropertySource("classpath:server-config-test.properties")
public class BindingPropertiesToBeanMethodsUnitTest {

    @Autowired
    @Qualifier("default_bean")
    private ServerConfig serverConfig;

    @Test
    void givenBeanAnnotatedMethod_whenBindingProperties_thenAllFieldsAreSet() {
        assertEquals("192.168.0.2", serverConfig.getAddress().getIp());

        // other assertions...
    }
}
```

## 5.属性验证

为了在 Spring Boot、**启用 [bean 验证](/web/20220827110142/https://www.baeldung.com/javax-validation)，我们必须用`@Validated`** 注释顶级类。然后我们添加所需的`javax.validation`约束:

```java
@Configuration
@ConfigurationProperties(prefix = "validate")
@Validated
public class MailServer {

    @NotNull
    @NotEmpty
    private Map<String, @NotBlank String> propertiesMap;

    @Valid
    private MailConfig mailConfig = new MailConfig();

    // getters and setters
}
```

类似地，`MailConfig`类也有一些约束:

```java
public class MailConfig {

    @NotBlank
    @Email
    private String address;

    // getters and setters
}
```

通过提供有效的数据集:

```java
validate.propertiesMap.first=prop1
validate.propertiesMap.second=prop2
[[email protected]](/web/20220827110142/https://www.baeldung.com/cdn-cgi/l/email-protection)
```

应用程序将正常启动，我们的单元测试将通过:

```java
@ExtendWith(SpringExtension.class)
@EnableConfigurationProperties(value = MailServer.class)
@TestPropertySource("classpath:property-validation-test.properties")
public class PropertyValidationUnitTest {

    @Autowired
    private MailServer mailServer;

    private static Validator propertyValidator;

    @BeforeAll
    public static void setup() {
        propertyValidator = Validation.buildDefaultValidatorFactory().getValidator();
    }

    @Test
    void whenBindingPropertiesToValidatedBeans_thenConstrainsAreChecked() {
        assertEquals(0, propertyValidator.validate(mailServer.getPropertiesMap()).size());
        assertEquals(0, propertyValidator.validate(mailServer.getMailConfig()).size());
    }
}
```

相反，**如果我们使用无效的属性，Spring 会在启动时抛出一个`IllegalStateException`**。

例如，使用以下任何无效配置:

```java
validate.propertiesMap.second=
validate.mail_config.address=user1.test
```

将导致我们的应用程序失败，并显示以下错误消息:

```java
Property: validate.propertiesMap[second]
Value:
Reason: must not be blank

Property: validate.mailConfig.address
Value: user1.test
Reason: must be a well-formed email address
```

注意**我们在`mailConfig`字段上使用了`@Valid`来确保检查`MailConfig`约束，即使没有定义`validate.mailConfig.address`。**否则，Spring 会将`mailConfig`设置为`null`并正常启动应用程序。

## 6.属性转换

[Spring Boot 属性转换](/web/20220827110142/https://www.baeldung.com/configuration-properties-in-spring-boot#property-conversion)使我们能够将一些属性转换成特定的类型。

在这一节中，我们将从测试使用 Spring 内置转换的配置类开始。然后，我们将测试一个我们自己创建的自定义转换器。

### 6.1.Spring Boot 的默认转换

让我们考虑以下数据大小和持续时间属性:

```java
# data sizes
convert.upload_speed=500MB
convert.download_speed=10

# durations
convert.backup_day=1d
convert.backup_hour=8
```

**Spring Boot 会自动将这些属性绑定到在`PropertyConversion`配置类中定义的匹配的`DataSize`和`Duration`字段**:

```java
@Configuration
@ConfigurationProperties(prefix = "convert")
public class PropertyConversion {

    private DataSize uploadSpeed;

    @DataSizeUnit(DataUnit.GIGABYTES)
    private DataSize downloadSpeed;

    private Duration backupDay;

    @DurationUnit(ChronoUnit.HOURS)
    private Duration backupHour;

    // getters and setters
}
```

我们将检查转换结果:

```java
@ExtendWith(SpringExtension.class)
@EnableConfigurationProperties(value = PropertyConversion.class)
@ContextConfiguration(classes = CustomCredentialsConverter.class)
@TestPropertySource("classpath:spring-conversion-test.properties")
public class SpringPropertiesConversionUnitTest {

    @Autowired
    private PropertyConversion propertyConversion;

    @Test
    void whenUsingSpringDefaultSizeConversion_thenDataSizeObjectIsSet() {
        assertEquals(DataSize.ofMegabytes(500), propertyConversion.getUploadSpeed());
        assertEquals(DataSize.ofGigabytes(10), propertyConversion.getDownloadSpeed());
    }

    @Test
    void whenUsingSpringDefaultDurationConversion_thenDurationObjectIsSet() {
        assertEquals(Duration.ofDays(1), propertyConversion.getBackupDay());
        assertEquals(Duration.ofHours(8), propertyConversion.getBackupHour());
    }
}
```

### 6.2.定制转换器

现在让我们假设我们想要转换`convert.credentials`属性:

```java
convert.credentials=user,123
```

进入以下`Credential`类:

```java
public class Credentials {

    private String username;
    private String password;

    // getters and setters
}
```

为此，我们可以实现一个自定义转换器:

```java
@Component
@ConfigurationPropertiesBinding
public class CustomCredentialsConverter implements Converter<String, Credentials> {

    @Override
    public Credentials convert(String source) {
        String[] data = source.split(",");
        return new Credentials(data[0], data[1]);
    }
}
```

最后，我们将向`PropertyConversion`类添加一个`Credentials`字段:

```java
public class PropertyConversion {
    private Credentials credentials;
    // ...
}
```

在我们的`SpringPropertiesConversionUnitTest`测试类中，我们还需要添加`@ContextConfiguration`来注册 Spring 上下文中的自定义转换器:

```java
// other annotations
@ContextConfiguration(classes=CustomCredentialsConverter.class)
public class SpringPropertiesConversionUnitTest {

    //...

    @Test
    void whenRegisteringCustomCredentialsConverter_thenCredentialsAreParsed() {
        assertEquals("user", propertyConversion.getCredentials().getUsername());
        assertEquals("123", propertyConversion.getCredentials().getPassword());
    }
}
```

正如前面的断言所示， **Spring 已经使用我们的定制转换器将`convert.credentials`属性解析成一个`Credentials`实例**。

## 7.YAML 文件装订

对于分层配置数据， [YAML 配置](/web/20220827110142/https://www.baeldung.com/spring-yaml)可能更方便。YAML 还支持在同一文档中定义多个概要文件。

位于`src/test/resources/`下的以下`application.yml`定义了`ServerConfig`级的“测试”配置文件:

```java
spring:
  config:
    activate:
      on-profile: test
server:
  address:
    ip: 192.168.0.4
  resources_path:
    imgs: /etc/test/imgs
---
# other profiles
```

因此，以下测试将通过:

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(initializers = ConfigDataApplicationContextInitializer.class)
@EnableConfigurationProperties(value = ServerConfig.class)
@ActiveProfiles("test")
public class BindingYMLPropertiesUnitTest {

    @Autowired
    private ServerConfig serverConfig;

    @Test
    void whenBindingYMLConfigFile_thenAllFieldsAreSet() {
        assertEquals("192.168.0.4", serverConfig.getAddress().getIp());

        // other assertions ...
    }
}
```

关于我们使用的注释，有几点需要注意:

*   `@ContextConfiguration(initializers = ConfigDataApplicationContextInitializer.cla``ss)`–加载`application.yml`文件
*   `@ActiveProfiles(“test”)`–指定将在该测试中使用“测试”配置文件

最后请记住，**无论是`@ProperySource`还是`@TestProperySource`都不支持加载`.yml`文件**。**因此，我们应该始终将我们的 YAML 配置放在`application.yml`文件**中。

## 8.覆盖`@ConfigurationProperties`配置

有时，我们可能希望用另一个数据集覆盖由`@ConfigurationProperties`加载的配置属性，特别是在测试的时候。

正如我们在前面的例子中看到的，我们可以使用`@TestPropertySource(“path_to_new_data_set”)`用一个新的配置替换整个原始配置(在`/src/main/resources)`下)。

**或者，我们可以使用`@TestPropertySource`** 的`properties`属性有选择地替换一些原始属性。

假设我们想用另一个值覆盖先前定义的`validate.mail_config.address`属性。我们所要做的就是用`@TestPropertySource,`注释我们的测试类，然后通过`properties`列表给同一个属性赋一个新值:

```java
@TestPropertySource(properties = {"[[email protected]](/web/20220827110142/https://www.baeldung.com/cdn-cgi/l/email-protection)"})
```

因此，Spring 将使用新定义的值:

```java
assertEquals("[[email protected]](/web/20220827110142/https://www.baeldung.com/cdn-cgi/l/email-protection)", mailServer.getMailConfig().getAddress());
```

## 9.结论

在本文中，我们学习了如何测试不同类型的配置类，这些配置类利用`@ConfigurationProperties`注释来加载`.properties`和`.yml`配置文件。

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220827110142/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-testing)