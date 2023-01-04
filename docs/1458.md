# spring boot 中的 beandefinnotionverrideexception

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-bean-definition-override-exception>

## 1.介绍

Spring Boot 2.1 的升级让人们惊讶于`BeanDefinitionOverrideException`的意外发生。这可能会让开发人员感到困惑，让他们想知道 Spring 中的 bean 覆盖行为发生了什么。

在本教程中，我们将解开这个问题，并学习如何最好地解决它。

## 2.Maven 依赖性

对于我们的示例 Maven 项目，我们需要添加 [Spring Boot 启动器](https://web.archive.org/web/20221206082907/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.boot%22%20AND%20a%3A%22spring-boot-starter%22)依赖项:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>2.3.3.RELEASE</version>
</dependency>
```

## 3.Bean 覆盖

Spring beans 通过它们在一个`ApplicationContext`中的名字来识别。

因此，当我们在与另一个 bean 同名的`ApplicationContext`中定义一个 bean 时， **bean 覆盖是一种默认行为。它的工作原理是在名称冲突的情况下简单地替换以前的 bean。**

**从 Spring 5.1 开始，引入了 [`BeanDefinitionOverrideException`](https://web.archive.org/web/20221206082907/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/support/BeanDefinitionOverrideException.html) ，允许开发者自动抛出异常，以防止任何意外的 bean 覆盖**。默认情况下，原始行为仍然可用，这允许 bean 覆盖。

## 4.Spring Boot 2.1 的配置更改

[Spring Boot 2.1 默认禁用 bean 覆盖](https://web.archive.org/web/20221206082907/https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.1-Release-Notes#bean-overriding)作为防御手段。主要目的是**提前注意重复的 bean 名称，以防止意外覆盖 bean**。

因此，如果我们的 Spring Boot 应用程序依赖 bean 覆盖，那么在我们将 Spring Boot 版本升级到 2.1 及更高版本后，很可能会遇到`BeanDefinitionOverrideException`。

在接下来的部分中，我们将看一个发生`BeanDefinitionOverrideException`的例子，然后我们将讨论一些解决方案。

## 5.识别冲突中的 Beans

让我们创建两个不同的弹簧配置，每个配置都有一个`testBean()`方法，以产生`BeanDefinitionOverrideException:`

```
@Configuration
public class TestConfiguration1 {

    class TestBean1 {
        private String name;

        // standard getters and setters

    }

    @Bean
    public TestBean1 testBean(){
        return new TestBean1();
    }
} 
```

```
@Configuration
public class TestConfiguration2 {

    class TestBean2 {
        private String name;

        // standard getters and setters

    }

    @Bean
    public TestBean2 testBean(){
        return new TestBean2();
    }
} 
```

接下来，我们将创建我们的 Spring Boot 测试类:

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = {TestConfiguration1.class, TestConfiguration2.class})
public class SpringBootBeanDefinitionOverrideExceptionIntegrationTest {

    @Test
    public void whenBeanOverridingAllowed_thenTestBean2OverridesTestBean1() {
        Object testBean = applicationContext.getBean("testBean");

        assertThat(testBean.getClass()).isEqualTo(TestConfiguration2.TestBean2.class);
    }
} 
```

运行测试会产生一个`BeanDefinitionOverrideException`。但是，该异常为我们提供了一些有用的信息:

```
Invalid bean definition with name 'testBean' defined in ... 
... com.baeldung.beandefinitionoverrideexception.TestConfiguration2 ...
Cannot register bean definition [ ... defined in ... 
... com.baeldung.beandefinitionoverrideexception.TestConfiguration2] for bean 'testBean' ...
There is already [ ... defined in ...
... com.baeldung.beandefinitionoverrideexception.TestConfiguration1] bound. 
```

请注意，该异常揭示了两条重要的信息。

第一个是冲突的 bean 名称，`testBean`:

```
Invalid bean definition with name 'testBean' ... 
```

第二个向我们展示了受影响的配置的完整路径:

```
... com.baeldung.beandefinitionoverrideexception.TestConfiguration2 ...
... com.baeldung.beandefinitionoverrideexception.TestConfiguration1 ... 
```

结果，我们可以看到两个不同的 beans 被标识为引起冲突的`testBean,` 。此外，这些 beans 包含在配置类`TestConfiguration1`和`TestConfiguration2`中。

## 6.可能的解决方案

根据我们的配置，Spring Beans 有默认名称，除非我们显式地设置它们。

因此，第一个可能的解决方案是重命名我们的 beans。Spring 中设置 bean 名称有一些常见的方法。

### 6.1.更改方法名称

默认情况下， **Spring 将带注释方法的名称作为 bean 名称**。

因此，如果我们在一个配置类中定义了 beans，就像我们的例子一样，那么简单地改变方法名将会阻止`BeanDefinitionOverrideException`:

```
@Bean
public TestBean1 testBean1() {
    return new TestBean1();
} 
```

```
@Bean
public TestBean2 testBean2() {
    return new TestBean2();
} 
```

### 6.2.`@Bean`注释

Spring 的`@Bean`注释是定义 bean 的一种非常常见的方式。

所以另一个选择是设置`@Bean`注释的`name`属性:

```
@Bean("testBean1")
public TestBean1 testBean() {
    return new TestBean1();
} 
```

```
@Bean("testBean2")
public TestBean1 testBean() {
    return new TestBean2();
} 
```

### 6.3.原型注释

定义 bean 的另一种方式是使用[原型注释](/web/20221206082907/https://www.baeldung.com/spring-bean-annotations)。启用 Spring 的`@ComponentScan`特性后，我们可以使用`@Component`注释在类级别定义 bean 名称:

```
@Component("testBean1")
class TestBean1 {

    private String name;

    // standard getters and setters

} 
```

```
@Component("testBean2")
class TestBean2 {

    private String name;

    // standard getters and setters

} 
```

### 6.4.来自第三方库的 Beans

在某些情况下，**可能会遇到由来自第三方 spring 支持的库**的 beans 引起的名称冲突。

当这种情况发生时，我们应该尝试识别哪个冲突的 bean 属于我们的应用程序，以确定我们是否可以使用上述任何解决方案。

然而，如果我们不能改变任何 bean 定义，那么配置 Spring Boot 来允许 bean 覆盖可能是一个变通办法。

为了启用 bean 覆盖，我们将在我们的`application.properties`文件中将`spring.main.allow-bean-definition-overriding`属性设置为`true`:

```
spring.main.allow-bean-definition-overriding=true 
```

通过这样做，我们告诉 Spring Boot 允许 bean 覆盖，而不对 bean 定义做任何更改。

最后要注意的是，我们应该知道**很难猜测哪个 bean 具有优先权，因为 bean 的创建顺序是由依赖关系决定的，而依赖关系在运行时**中最受影响。因此，允许 bean 覆盖会产生意外的行为，除非我们足够了解 bean 的依赖层次。

## 7.结论

在本文中，我们解释了`BeanDefinitionOverrideException`在 Spring 中的含义，为什么它会突然出现，以及在 Spring Boot 2.1 升级后如何解决它。

和往常一样，本文的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221206082907/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-exceptions)