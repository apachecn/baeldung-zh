# @PropertySource 拥有 Spring Boot 的 YAML 文件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-yaml-propertysource>

## 1.概观

在这个快速教程中，我们将展示如何在 Spring Boot 使用 `@PropertySource` 注释读取 YAML 属性文件。

## 2.`@PropertySource`和`YAML`格式

Spring Boot 非常支持外部化配置。此外，可以使用不同的方式和格式来读取开箱即用的 Spring Boot 应用程序中的属性。

但是，**默认情况下，** **`@PropertySource`** **不加载 YAML 文件**。这个事实在[官方文档](https://web.archive.org/web/20220629003347/https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-external-config-yaml-shortcomings)中有明确提及。

所以，如果我们想在应用程序中使用 `@PropertySource` 注释，我们需要坚持使用标准的 `properties` 文件。或者我们可以自己实现缺失的拼图块！

## 3.自定义`PropertySourceFactory`

从 Spring 4.3 开始， `@PropertySource` 附带了 `factory` 属性。我们可以利用它来为**提供我们自定义的****`PropertySourceFactory`****的实现，从而将处理 YAML 的文件处理**。

这比听起来容易多了！让我们来看看如何做到这一点:

```java
public class YamlPropertySourceFactory implements PropertySourceFactory {

    @Override
    public PropertySource<?> createPropertySource(String name, EncodedResource encodedResource) 
      throws IOException {
        YamlPropertiesFactoryBean factory = new YamlPropertiesFactoryBean();
        factory.setResources(encodedResource.getResource());

        Properties properties = factory.getObject();

        return new PropertiesPropertySource(encodedResource.getResource().getFilename(), properties);
    }
}
```

我们可以看到，实现一个单独的 [`createPropertySource`](https://web.archive.org/web/20220629003347/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/support/PropertySourceFactory.html#createPropertySource-java.lang.String-org.springframework.core.io.support.EncodedResource-) 方法就足够了。

在我们的自定义实现中，首先，**我们使用了** **[`YamlPropertiesFactoryBean`](https://web.archive.org/web/20220629003347/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/config/YamlPropertiesFactoryBean.html) 将 YAML 格式的资源转换为`java.util.Properties`** **对象**。

然后，我们简单地返回了一个 [`PropertiesPropertySource`](https://web.archive.org/web/20220629003347/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/core/env/PropertiesPropertySource.html) 的新实例，这是一个允许 Spring 读取解析的属性的包装器。

## 4.`@PropertySource`和`YAML`在动作

现在让我们把所有的部分放在一起，看看如何在实践中使用它们。

首先，让我们创建一个简单的 YAML 文件—`foo.yml`:

```java
yaml:
  name: foo
  aliases:
    - abc
    - xyz
```

接下来，让我们用`@ConfigurationProperties`创建一个属性类，并使用我们的自定义`YamlPropertySourceFactory:`

```java
@Configuration
@ConfigurationProperties(prefix = "yaml")
@PropertySource(value = "classpath:foo.yml", factory = YamlPropertySourceFactory.class)
public class YamlFooProperties {

    private String name;

    private List<String> aliases;

    // standard getter and setters
}
```

最后，**让我们验证属性是否被正确注入**:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class YamlFooPropertiesIntegrationTest {

    @Autowired
    private YamlFooProperties yamlFooProperties;

    @Test
    public void whenFactoryProvidedThenYamlPropertiesInjected() {
        assertThat(yamlFooProperties.getName()).isEqualTo("foo");
        assertThat(yamlFooProperties.getAliases()).containsExactly("abc", "xyz");
    }
}
```

## 5.结论

综上所述，在这个快速教程中，我们首先展示了创建一个自定义 `PropertySourceFactory` 是多么容易。之后，我们介绍了如何使用 `factory` 属性将这个自定义实现传递给 `@PropertySource` 。

因此，**我们能够成功地将 YAML 属性文件加载到我们的 Spring Boot 应用程序**中。

像往常一样，GitHub 上的所有代码示例[都可用。](https://web.archive.org/web/20220629003347/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-properties-2)