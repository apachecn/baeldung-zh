# 覆盖 Spring 测试中的属性

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-tests-override-properties>

## 1.概观

在本教程中，我们将看看在 Spring 测试中覆盖属性的各种方法。

Spring 实际上为此提供了许多解决方案，所以我们在这里有很多要探索的。

## 2.属国

当然，为了使用 Spring 测试，我们需要添加一个测试依赖项:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <version>2.6.1</version>
    <scope>test</scope>
</dependency>
```

这种依赖性也包括我们的 JUnit 5。

## 3.设置

首先，我们将在应用程序中创建一个使用我们的属性的类:

```
@Component
public class PropertySourceResolver {

    @Value("${example.firstProperty}") private String firstProperty;
    @Value("${example.secondProperty}") private String secondProperty;

    public String getFirstProperty() {
        return firstProperty;
    }

    public String getSecondProperty() {
        return secondProperty;
    }
}
```

接下来，我们将为它们赋值。我们可以通过在`src/main/resources:`中创建`application.properties`来做到这一点

```
example.firstProperty=defaultFirst
example.secondProperty=defaultSecond
```

## 4.覆盖属性文件

现在，我们将通过将属性文件放入测试资源来覆盖属性。**该文件必须与默认文件** **在同一个类路径**中。

此外，它应该**包含默认文件中指定的所有属性键**。因此，我们将把`application.properties` 文件添加到`src/test/resources`中:

```
example.firstProperty=file
example.secondProperty=file
```

让我们添加将利用我们的解决方案的测试:

```
@SpringBootTest
public class TestResourcePropertySourceResolverIntegrationTest {

    @Autowired private PropertySourceResolver propertySourceResolver;

    @Test
    public void shouldTestResourceFile_overridePropertyValues() {
        String firstProperty = propertySourceResolver.getFirstProperty();
        String secondProperty = propertySourceResolver.getSecondProperty();

        assertEquals("file", firstProperty);
        assertEquals("file", secondProperty);
    }
}
```

当我们想要覆盖文件中的多个属性时，这种方法非常有效。

如果我们不把`example.secondProperty `放在文件中，应用程序上下文就不会发现这个属性。

## 5.弹簧轮廓

在这一节中，我们将学习如何使用 Spring Profiles 来处理我们的问题。**与前面的** **方法不同，这种方法合并了默认文件和概要文件**的属性。

首先，让我们在`src/test/resources:`中创建一个`application**–**test.properties` 文件

```
example.firstProperty=profile
```

然后我们将创建一个使用`test`概要文件的测试:

```
@SpringBootTest
@ActiveProfiles("test")
public class ProfilePropertySourceResolverIntegrationTest {

    @Autowired private PropertySourceResolver propertySourceResolver;

    @Test
    public void shouldProfiledProperty_overridePropertyValues() {
        String firstProperty = propertySourceResolver.getFirstProperty();
        String secondProperty = propertySourceResolver.getSecondProperty();

        assertEquals("profile", firstProperty);
        assertEquals("defaultSecond", secondProperty);
    }
}
```

这种方法允许我们使用默认值和测试值。因此，当我们需要用**覆盖一个文件的多个属性时，这是一个很好的方法，但是我们仍然希望使用默认的** **属性。**

我们可以在我们的 [`Spring Profiles`](/web/20220626075642/https://www.baeldung.com/spring-profiles) 文章中了解更多关于春天的简介。

## 6.`@SpringBootTest`

覆盖属性值的另一种方法是使用 `@SpringBootTest` 注释:

```
@SpringBootTest(properties = { "example.firstProperty=annotation" })
public class SpringBootPropertySourceResolverIntegrationTest {

    @Autowired private PropertySourceResolver propertySourceResolver;

    @Test
    public void shouldSpringBootTestAnnotation_overridePropertyValues() {
        String firstProperty = propertySourceResolver.getFirstProperty();
        String secondProperty = propertySourceResolver.getSecondProperty();

        Assert.assertEquals("annotation", firstProperty);
        Assert.assertEquals("defaultSecond", secondProperty);
    }
}
```

**我们可以看到，****`example.firstProperty`已经被覆盖，而`example.secondProperty`还没有**。因此，当我们只需要覆盖测试的特定属性时，这是一个很好的解决方案。这是唯一需要使用 Spring Boot 的方法。

## 7.`TestPropertySourceUtils`

在这一节中，我们将学习如何通过使用`ApplicationContextInitializer. `中的`TestPropertySourceUtils`类来覆盖属性

`TestPropertySourceUtils`提供了两种方法，我们可以用它们来定义不同的属性值。

让我们创建一个将在测试中使用的初始化器类:

```
public class PropertyOverrideContextInitializer
  implements ApplicationContextInitializer<ConfigurableApplicationContext> {

    static final String PROPERTY_FIRST_VALUE = "contextClass";

    @Override
    public void initialize(ConfigurableApplicationContext configurableApplicationContext) {
        TestPropertySourceUtils.addInlinedPropertiesToEnvironment(
          configurableApplicationContext, "example.firstProperty=" + PROPERTY_FIRST_VALUE);

        TestPropertySourceUtils.addPropertiesFilesToEnvironment(
          configurableApplicationContext, "context-override-application.properties");
    }
}
```

接下来，我们将把`context-override-application.properties`文件添加到`src/test/resources:`中

```
example.secondProperty=contextFile
```

最后，我们应该创建一个将使用我们的初始化器的测试类:

```
@SpringBootTest
@ContextConfiguration(
  initializers = PropertyOverrideContextInitializer.class,
  classes = Application.class)
public class ContextPropertySourceResolverIntegrationTest {

    @Autowired private PropertySourceResolver propertySourceResolver;

    @Test
    public void shouldContext_overridePropertyValues() {
        final String firstProperty = propertySourceResolver.getFirstProperty();
        final String secondProperty = propertySourceResolver.getSecondProperty();

        assertEquals(PropertyOverrideContextInitializer.PROPERTY_FIRST_VALUE, firstProperty);
        assertEquals("contextFile", secondProperty);
    }
}
```

`example.firstProperty`已被内联方法覆盖。

在第二个方法中,`example.secondProperty` 已被特定文件覆盖。这种方法允许我们在初始化上下文时定义不同的属性值。

## 8.结论

在本文中，我们关注了在测试中覆盖属性的多种方法。我们还讨论了何时使用每种解决方案，或者在某些情况下，何时混合使用它们。

当然，我们还有[`@TestPropertySource`注释](/web/20220626075642/https://www.baeldung.com/spring-test-property-source)供我们使用。

与往常一样，本文中示例的代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220626075642/https://github.com/eugenp/tutorials/tree/master/testing-modules/spring-testing)