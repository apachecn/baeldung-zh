# 从 JSON 文件加载 Spring Boot 属性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-json-properties>

## 1。简介

使用外部配置属性是一种非常常见模式。

而且，最常见的问题之一是在多种环境中改变我们的应用程序的行为的能力——例如开发、测试和生产——而不必改变部署工件。

在本教程中，**我们将关注** **如何在 Spring Boot 应用程序中从 JSON 文件加载属性**。

## 2。在 Spring Boot 装载房产

Spring 和 Spring Boot 为加载外部配置提供了强大的支持——你可以在本文的[中找到对基础知识的概述。](/web/20220630133227/https://www.baeldung.com/properties-with-spring)

由于**这种支持主要集中在`.properties`和。`yml`文件——使用`JSON`通常需要额外的配置**。

我们将假设基本特性是众所周知的——在这里，我们将关注于`JSON`特定的方面。

## 3。通过命令行加载属性

我们可以在命令行中以三种预定义的格式提供`JSON`数据。

首先，我们可以在一个`UNIX` shell 中设置环境变量`SPRING_APPLICATION_JSON`:

```java
$ SPRING_APPLICATION_JSON='{"environment":{"name":"production"}}' java -jar app.jar
```

提供的数据将被填充到 Spring `Environment`中。在这个例子中，我们将得到一个值为“production”的属性`environment.name` 。

同样，我们可以将我们的`JSON`作为一个`System property, `来加载，例如:

```java
$ java -Dspring.application.json='{"environment":{"name":"production"}}' -jar app.jar
```

最后一个选项是使用一个简单的命令行参数:

```java
$ java -jar app.jar --spring.application.json='{"environment":{"name":"production"}}'
```

对于最后两种方法， `spring.application.json`属性将使用未解析的给定数据`String`来填充。

这些是将`JSON`数据加载到我们的应用程序中的最简单的选项。**这种简约方法的缺点是缺乏可扩展性。**

在命令行中加载大量数据可能会很麻烦并且容易出错。

## 4。通过`PropertySource`注释加载属性

Spring Boot 提供了一个强大的生态系统来通过注释创建配置类。

首先，我们用一些简单的成员定义一个配置类:

```java
public class JsonProperties {

    private int port;

    private boolean resend;

    private String host;

   // getters and setters

}
```

我们可以在一个外部文件中提供标准`JSON`格式的数据(姑且称之为`configprops.json`):

```java
{
  "host" : "[[email protected]](/web/20220630133227/https://www.baeldung.com/cdn-cgi/l/email-protection)",
  "port" : 9090,
  "resend" : true
}
```

现在我们必须将 JSON 文件连接到配置类:

```java
@Component
@PropertySource(value = "classpath:configprops.json")
@ConfigurationProperties
public class JsonProperties {
    // same code as before
}
```

我们在类和 JSON 文件之间有一个松散的耦合。这种连接基于字符串和变量名。因此，我们没有编译时检查，但我们可以通过测试来验证绑定。

因为字段应该由框架填充，所以我们需要使用集成测试。

对于极简设置，我们可以定义应用程序的主要入口点:

```java
@SpringBootApplication
@ComponentScan(basePackageClasses = { JsonProperties.class})
public class ConfigPropertiesDemoApplication {
    public static void main(String[] args) {
        new SpringApplicationBuilder(ConfigPropertiesDemoApplication.class).run();
    }
}
```

现在我们可以创建我们的集成测试了:

```java
@RunWith(SpringRunner.class)
@ContextConfiguration(
  classes = ConfigPropertiesDemoApplication.class)
public class JsonPropertiesIntegrationTest {

    @Autowired
    private JsonProperties jsonProperties;

    @Test
    public void whenPropertiesLoadedViaJsonPropertySource_thenLoadFlatValues() {
        assertEquals("[[email protected]](/web/20220630133227/https://www.baeldung.com/cdn-cgi/l/email-protection)", jsonProperties.getHost());
        assertEquals(9090, jsonProperties.getPort());
        assertTrue(jsonProperties.isResend());
    }
}
```

因此，该测试将产生一个错误。即使加载`ApplicationContext`也会失败，原因如下:

```java
ConversionFailedException: 
Failed to convert from type [java.lang.String] 
to type [boolean] for value 'true,'
```

加载机制通过`PropertySource`注释成功地将类与 JSON 文件连接起来。但是`resend`属性的值被计算为“`true,”` ”(带逗号)，不能转换为布尔值。

**因此，我们必须在加载机制中注入一个 JSON 解析器。**幸运的是，Spring Boot 附带了杰克森库，我们可以通过`PropertySourceFactory`使用它。

## 5。使用`PropertySourceFactory` 解析 JSON

**我们必须为客户`PropertySourceFactory`提供解析 JSON 数据的能力:**

```java
public class JsonPropertySourceFactory 
  implements PropertySourceFactory {

    @Override
    public PropertySource<?> createPropertySource(
      String name, EncodedResource resource)
          throws IOException {
        Map readValue = new ObjectMapper()
          .readValue(resource.getInputStream(), Map.class);
        return new MapPropertySource("json-property", readValue);
    }
}
```

我们可以提供这个工厂来加载我们的配置类。为此，我们必须引用来自`PropertySource`注释的工厂:

```java
@Configuration
@PropertySource(
  value = "classpath:configprops.json", 
  factory = JsonPropertySourceFactory.class)
@ConfigurationProperties
public class JsonProperties {

    // same code as before

}
```

因此，我们的测试将通过。此外，这个属性源工厂也将很乐意解析列表值。

所以现在我们可以用一个列表成员(以及相应的 getters 和 setters)来扩展我们的配置类:

```java
private List<String> topics;
// getter and setter
```

我们可以在 JSON 文件中提供输入值:

```java
{
    // same fields as before
    "topics" : ["spring", "boot"]
}
```

我们可以用一个新的测试用例轻松测试列表值的绑定:

```java
@Test
public void whenPropertiesLoadedViaJsonPropertySource_thenLoadListValues() {
    assertThat(
      jsonProperties.getTopics(), 
      Matchers.is(Arrays.asList("spring", "boot")));
}
```

### 5.1。嵌套结构

处理嵌套的 JSON 结构不是一件容易的事情。作为更健壮的解决方案，Jackson 库的映射器将把嵌套数据映射到一个`Map. `

所以我们可以用 getters 和 setters 向我们的`JsonProperties`类添加一个`Map`成员:

```java
private LinkedHashMap<String, ?> sender;
// getter and setter
```

在 JSON 文件中，我们可以为这个字段提供一个嵌套的数据结构:

```java
{
  // same fields as before
   "sender" : {
     "name": "sender",
     "address": "street"
  }
}
```

现在，我们可以通过地图访问嵌套数据:

```java
@Test
public void whenPropertiesLoadedViaJsonPropertySource_thenNestedLoadedAsMap() {
    assertEquals("sender", jsonProperties.getSender().get("name"));
    assertEquals("street", jsonProperties.getSender().get("address"));
}
```

## 6。`ContextInitializer`使用自定义

**如果我们想对属性的加载有更多的控制，我们可以使用 custom `ContextInitializers`。**

这种手动方法更加繁琐。但是，结果是，我们将完全控制数据的加载和解析。

我们将像以前一样使用相同的 JSON 数据，但是我们将加载到不同的配置类中:

```java
@Configuration
@ConfigurationProperties(prefix = "custom")
public class CustomJsonProperties {

    private String host;

    private int port;

    private boolean resend;

    // getters and setters

}
```

注意，我们不再使用`PropertySource`注释了。但是在`ConfigurationProperties`注释中，我们定义了一个前缀。

在下一节中，我们将研究如何将属性加载到`‘custom'`名称空间中。

### 6.1。将属性加载到自定义名称空间

为了给上面的 properties 类提供输入，我们将从 JSON 文件加载数据，在解析之后，我们将用`MapPropertySources:`填充 Spring `Environment`

```java
public class JsonPropertyContextInitializer
 implements ApplicationContextInitializer<ConfigurableApplicationContext> {

    private static String CUSTOM_PREFIX = "custom.";

    @Override
    @SuppressWarnings("unchecked")
    public void 
      initialize(ConfigurableApplicationContext configurableApplicationContext) {
        try {
            Resource resource = configurableApplicationContext
              .getResource("classpath:configpropscustom.json");
            Map readValue = new ObjectMapper()
              .readValue(resource.getInputStream(), Map.class);
            Set<Map.Entry> set = readValue.entrySet();
            List<MapPropertySource> propertySources = set.stream()
               .map(entry-> new MapPropertySource(
                 CUSTOM_PREFIX + entry.getKey(),
                 Collections.singletonMap(
                 CUSTOM_PREFIX + entry.getKey(), entry.getValue()
               )))
               .collect(Collectors.toList());
            for (PropertySource propertySource : propertySources) {
                configurableApplicationContext.getEnvironment()
                    .getPropertySources()
                    .addFirst(propertySource);
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

正如我们所见，这需要一点相当复杂的代码，但这是灵活性的代价。在上面的代码中，我们可以指定自己的解析器，并决定如何处理每个条目。

在本演示中，我们只是将属性放入一个自定义的名称空间中。

要使用这个初始化器，我们必须将它连接到应用程序。对于生产使用，我们可以将其添加到`SpringApplicationBuilder`:

```java
@EnableAutoConfiguration
@ComponentScan(basePackageClasses = { JsonProperties.class,
  CustomJsonProperties.class })
public class ConfigPropertiesDemoApplication {
    public static void main(String[] args) {
        new SpringApplicationBuilder(ConfigPropertiesDemoApplication.class)
            .initializers(new JsonPropertyContextInitializer())
            .run();
    }
}
```

另外，请注意，`CustomJsonProperties`类已经被添加到了`basePackageClasses`中。

对于我们的测试环境，我们可以在`ContextConfiguration`注释中提供我们的定制初始化器:

```java
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = ConfigPropertiesDemoApplication.class, 
  initializers = JsonPropertyContextInitializer.class)
public class JsonPropertiesIntegrationTest {

    // same code as before

}
```

在自动连接我们的`CustomJsonProperties`类之后，我们可以从自定义名称空间测试数据绑定:

```java
@Test
public void whenLoadedIntoEnvironment_thenFlatValuesPopulated() {
    assertEquals("[[email protected]](/web/20220630133227/https://www.baeldung.com/cdn-cgi/l/email-protection)", customJsonProperties.getHost());
    assertEquals(9090, customJsonProperties.getPort());
    assertTrue(customJsonProperties.isResend());
}
```

### 6.2。展平嵌套结构

Spring 框架提供了一个强大的机制来将属性绑定到对象成员中。此功能的基础是属性中的名称前缀。

如果我们扩展我们的自定义`ApplicationInitializer`来将`Map`值转换成名称空间结构，那么框架可以将我们的嵌套数据结构直接加载到相应的对象中。

增强的`CustomJsonProperties`级:

```java
@Configuration
@ConfigurationProperties(prefix = "custom")
public class CustomJsonProperties {

   // same code as before

    private Person sender;

    public static class Person {

        private String name;
        private String address;

        // getters and setters for Person class

   }

   // getters and setters for sender member

}
```

增强型`ApplicationContextInitializer`:

```java
public class JsonPropertyContextInitializer 
  implements ApplicationContextInitializer<ConfigurableApplicationContext> {

    private final static String CUSTOM_PREFIX = "custom.";

    @Override
    @SuppressWarnings("unchecked")
    public void 
      initialize(ConfigurableApplicationContext configurableApplicationContext) {
        try {
            Resource resource = configurableApplicationContext
              .getResource("classpath:configpropscustom.json");
            Map readValue = new ObjectMapper()
              .readValue(resource.getInputStream(), Map.class);
            Set<Map.Entry> set = readValue.entrySet();
            List<MapPropertySource> propertySources = convertEntrySet(set, Optional.empty());
            for (PropertySource propertySource : propertySources) {
                configurableApplicationContext.getEnvironment()
                  .getPropertySources()
                  .addFirst(propertySource);
            }
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }

    private static List<MapPropertySource> 
      convertEntrySet(Set<Map.Entry> entrySet, Optional<String> parentKey) {
        return entrySet.stream()
            .map((Map.Entry e) -> convertToPropertySourceList(e, parentKey))
            .flatMap(Collection::stream)
            .collect(Collectors.toList());
    }

    private static List<MapPropertySource> 
      convertToPropertySourceList(Map.Entry e, Optional<String> parentKey) {
        String key = parentKey.map(s -> s + ".")
          .orElse("") + (String) e.getKey();
        Object value = e.getValue();
        return covertToPropertySourceList(key, value);
    }

    @SuppressWarnings("unchecked")
    private static List<MapPropertySource> 
       covertToPropertySourceList(String key, Object value) {
        if (value instanceof LinkedHashMap) {
            LinkedHashMap map = (LinkedHashMap) value;
            Set<Map.Entry> entrySet = map.entrySet();
            return convertEntrySet(entrySet, Optional.ofNullable(key));
        }
        String finalKey = CUSTOM_PREFIX + key;
        return Collections.singletonList(
          new MapPropertySource(finalKey, 
            Collections.singletonMap(finalKey, value)));
    }
}
```

**因此，我们嵌套的 JSON 数据结构将被加载到一个配置对象:**

```java
@Test
public void whenLoadedIntoEnvironment_thenValuesLoadedIntoClassObject() {
    assertNotNull(customJsonProperties.getSender());
    assertEquals("sender", customJsonProperties.getSender()
      .getName());
    assertEquals("street", customJsonProperties.getSender()
      .getAddress());
}
```

## 7。结论

Spring Boot 框架提供了一种通过命令行加载外部 JSON 数据的简单方法。在需要的情况下，我们可以通过正确配置的`PropertySourceFactory`来加载 JSON 数据。

尽管加载嵌套属性是可以解决的，但是需要格外小心。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220630133227/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-properties-3)