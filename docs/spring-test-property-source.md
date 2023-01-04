# @TestPropertySource 快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-test-property-source>

## 1.概观

Spring 带来了许多特性来帮助我们测试代码。有时我们需要使用特定的配置属性，以便在我们的测试用例中建立期望的场景。

在这些情况下，**我们可以利用`@TestPropertySource`注释。使用这个工具，我们可以定义比项目中使用的任何其他资源优先级更高的配置源。**

因此，在这个简短的教程中，我们将看到使用这个注释的例子。此外，我们将分析它的默认行为和它支持的主要属性。

要了解更多关于 Spring Boot 考试的信息，我们建议看看我们的[‘Spring Boot 考试’教程](/web/20220703152918/https://www.baeldung.com/spring-boot-testing)。

## 2.属国

在我们的项目中包含所有需要的库的最简单的方法是在我们的`pom.xml`文件中添加`spring-boot-starter-test`工件:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
    <version>2.6.1</version>
</dependency>
```

我们可以检查 Maven Central 来验证我们使用的是最新版本的 [starter 库](https://web.archive.org/web/20220703152918/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22spring-boot-starter-test%22)。

## 3.如何使用`@TestPropertySource`

让我们假设我们正在使用一个属性的值，通过使用 [`@Value`](/web/20220703152918/https://www.baeldung.com/spring-value-annotation) Spring 注释来注入它:

```java
@Component
public class ClassUsingProperty {

    @Value("${baeldung.testpropertysource.one}")
    private String propertyOne;

    public String retrievePropertyOne() {
        return propertyOne;
    }
}
```

然后，我们将使用`@TestPropertySource `类级注释来定义一个新的配置源，并覆盖该属性的值:

```java
@RunWith(SpringRunner.class)
@ContextConfiguration(classes = ClassUsingProperty.class)
@TestPropertySource
public class DefaultTest {

    @Autowired
    ClassUsingProperty classUsingProperty;

    @Test
    public void givenDefaultTPS_whenVariableRetrieved_thenDefaultFileReturned() {
        String output = classUsingProperty.retrievePropertyOne();

        assertThat(output).isEqualTo("default-value");
    }
}
```

通常，每当我们使用这个测试注释时，我们还将包括`@ContextConfiguration `注释，以便为场景加载和配置`ApplicationContext`。

**默认情况下，`@TestPropertySource`注释试图加载一个相对于声明该注释的类的`properties`文件。**

在这种情况下，例如，如果我们的测试类在`com.baeldung.testpropertysource`包中，那么我们将需要文件`com/baeldung/testpropertysource/DefaultTest.properties `在我们的类路径中。

让我们将它添加到我们的资源文件夹中:

```java
# DefaultTest.properties
baeldung.testpropertysource.one=default-value
```

**此外，我们可以更改默认的配置文件位置，或者添加具有更高优先级的额外属性:**

```java
@TestPropertySource(locations = "/other-location.properties",
  properties = "baeldung.testpropertysource.one=other-property-value")
```

最后，我们可以指定是否要从超类继承`locations`和`properties` 值。因此，我们可以切换`inheritLocations` 和`inheritProperties `属性，默认情况下它们是`true`。

## 4.结论

通过这个简单的例子，我们已经了解了如何有效地使用`@TestPropertySource` Spring 注释。

我们可以在 Github 库中找到不同场景的例子。