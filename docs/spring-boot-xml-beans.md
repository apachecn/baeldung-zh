# Spring Boot 的 XML 定义的 Beans

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-xml-beans>

## 1.介绍

在 Spring 3.0 之前，XML 是定义和配置 beans 的唯一方式。Spring 3.0 引入了 [`JavaConfig`](https://web.archive.org/web/20220628090426/https://docs.spring.io/spring-javaconfig/docs/1.0.0.M4/reference/html/) ，允许我们使用 Java 类配置 beans。然而，XML 配置文件今天仍在使用。

在本教程中，我们将讨论如何将 XML 配置集成到 Spring Boot 中。

## 2.`@ImportResource` 注解

`@ImportResource`注释允许我们导入一个或多个包含 bean 定义的资源。

假设我们有一个包含 bean 定义的`beans.xml`文件:

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <bean class="com.baeldung.springbootxml.Pojo">
        <property name="field" value="sample-value"></property>
    </bean>
</beans>
```

为了在 Spring Boot 应用程序中使用它，我们可以**使用`@ImportResource` 注释**，告诉它在哪里可以找到配置文件:

```java
@Configuration
@ImportResource("classpath:beans.xml")
public class SpringBootXmlApplication implements CommandLineRunner {

    @Autowired 
    private Pojo pojo;

    public static void main(String[] args) {
        SpringApplication.run(SpringBootXmlApplication.class, args);
    }
}
```

在这种情况下，`Pojo`实例将被注入在`beans.xml`中定义的 bean。

## 3.访问 XML 配置中的属性

如何在 XML 配置文件中使用属性？假设我们想要使用在我们的`application.properties`文件中声明的一个属性:

```java
sample=string loaded from properties!
```

让我们更新`beans.xml`中的`Pojo`定义，以包含`sample`属性:

```java
<bean class="com.baeldung.springbootxml.Pojo">
    <property name="field" value="${sample}"></property>
</bean>
```

接下来，让我们验证是否正确包含了该属性:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = SpringBootXmlApplication.class)
public class SpringBootXmlApplicationIntegrationTest {

    @Autowired 
    private Pojo pojo;
    @Value("${sample}") 
    private String sample;

    @Test
    public void whenCallingGetter_thenPrintingProperty() {
        assertThat(pojo.getField())
                .isNotBlank()
                .isEqualTo(sample);
    }
}
```

不幸的是，这个测试会失败，因为默认情况下，**XML 配置文件不能解析占位符**。但是，我们可以通过包括 [`@EnableAutoConfiguration`](https://web.archive.org/web/20220628090426/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/EnableAutoConfiguration.html) 的注释来解决这个问题:

```java
@Configuration
@EnableAutoConfiguration
@ImportResource("classpath:beans.xml")
public class SpringBootXmlApplication implements CommandLineRunner {
    // ...
}
```

该注释支持自动配置，并尝试配置 beans。

## 4.推荐方法

我们可以继续使用 XML 配置文件。但是出于几个原因，我们也可以考虑将所有配置转移到`JavaConfig`。首先，**在 Java 中配置 beans 是类型安全的**，所以我们将在编译时捕捉类型错误。另外， **XML 配置会变得相当大**，使得维护变得困难。

## 5.结论

在本文中，我们看到了如何使用 XML 配置文件在 Spring Boot 应用程序中定义 beans。和往常一样，我们使用的例子的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220628090426/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-basic-customization-2)