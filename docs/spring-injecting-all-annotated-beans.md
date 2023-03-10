# 查找带有自定义注释的所有 Beans

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-injecting-all-annotated-beans>

## 1.概观

在本教程中，我们将解释如何在 Spring 中找到用自定义注释注释的所有 beans。我们将根据我们使用的 Spring 版本展示不同的方法。

## 2.使用 Spring Boot 2.2 或更高版本

从 Spring Boot 2.2 开始，**我们可以用`getBeansWithAnnotation` 的方法**。

让我们建立一个例子。首先，我们将定义我们的自定义注释。让我们用`[@Retention](/web/20220603091232/https://www.baeldung.com/java-default-annotations)(RetentionPolicy.RUNTIME)` 对其进行注释，以确保注释可以在运行时被程序访问:

```java
@Retention( RetentionPolicy.RUNTIME )
public @interface MyCustomAnnotation {

}
```

现在，让我们定义第一个用我们的注释注释的 bean。我们也用 [`@Component`](/web/20220603091232/https://www.baeldung.com/spring-component-annotation) 来注释:

```java
@Component
@MyCustomAnnotation
public class MyComponent {

}
```

然后，让我们用我们的注释定义另一个 bean。然而，这次我们将借助一个`@Configuration`文件中的 [`@Bean`](/web/20220603091232/https://www.baeldung.com/spring-bean-annotations) 注释方法来创建它:

```java
public class MyService {

}

@Configuration
public class MyConfigurationBean {

    @Bean
    @MyCustomAnnotation
    MyService myService() {
        return new MyService();
    }
}
```

现在，让我们编写一个测试来检查`getBeansWithAnnotation` 方法是否可以检测到我们的两个 beans:

```java
@Test
void whenApplicationContextStarted_ThenShouldDetectAllAnnotatedBeans() {
    try (AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext( MyComponent.class, MyConfigurationBean.class )) {
        Map<String,Object> beans = applicationContext.getBeansWithAnnotation(MyCustomAnnotation.class);
        assertEquals(2, beans.size());
        assertTrue(beans.keySet().containsAll(List.of("myComponent", "myService")));
    }
}
```

## 3.对于较旧的 Spring 版本

### 3.1.历史关联

在 5.2 之前的 Spring Framework 版本中，`getBeansWithAnnotation` 方法只能检测在类或接口级别注释的 bean，而不能检测在工厂方法级别注释的 bean。

在 Spring Boot 2.2 中，Spring 框架依赖已经升级到了 5.2，所以这就是为什么对于 Spring 的旧版本，我们刚刚编写的测试会失败:

*   `MyComponent` bean 被正确地检测到，因为注释是在类级别
*   没有检测到`MyService` bean，因为它是通过工厂方法创建的

让我们看看如何避免这种行为。

### 3.2.用@Qualifier 修饰我们的自定义注释

有一个相当简单的解决方法:我们可以简单地用`[@Qualifier](/web/20220603091232/https://www.baeldung.com/spring-qualifier-annotation)` 来修饰我们的注释。

我们的注释将看起来像这样:

```java
@Retention( RetentionPolicy.RUNTIME )
@Qualifier
public @interface MyCustomAnnotation {

}
```

现在，我们能够自动连接两个带注释的 beans。让我们通过测试来验证这一点:

```java
@Autowired
@MyCustomAnnotation
private List<Object> annotatedBeans;

@Test
void whenAutowiring_ThenShouldDetectAllAnnotatedBeans() {
    assertEquals(2, annotatedBeans.size());
    List<String> classNames = annotatedBeans.stream()
        .map(Object::getClass)
        .map(Class::getName)
        .map(s -> s.substring(s.lastIndexOf(".") + 1))
        .collect(Collectors.toList());
    assertTrue(classNames.containsAll(List.of("MyComponent", "MyService")));
}
```

这个解决方法是最简单的，但是，**它可能不符合我们的需要，例如，如果我们没有注释**。

我们还要注意，用 [`@Qualifier`](/web/20220603091232/https://www.baeldung.com/spring-qualifier-annotation) 来修饰我们的自定义注释会将它变成一个 Spring 限定符。

### 3.3.列出通过工厂方法创建的 Beans

既然我们已经理解了问题主要出现在通过工厂方法创建的 beans 上，那么让我们关注如何只列出那些。我们将提出一个在所有情况下都有效的解决方案，而不需要对我们的自定义注释进行任何修改。我们将使用反射来访问 beans 的注释。

假设我们可以访问 Spring `[ApplicationContext](/web/20220603091232/https://www.baeldung.com/spring-application-context)`，我们将遵循一系列步骤:

*   访问`BeanFactory`
*   查找与每个 bean 相关联的`BeanDefinition`
*   检查`BeanDefinition` 的源是否是一个`AnnotatedTypeMetadata`，这意味着我们将能够访问 bean 的注释
*   如果 bean 有注释，检查期望的注释是否在其中

让我们创建自己的`BeanUtils`实用程序类，并在方法中实现这个逻辑:

```java
public class BeanUtils {

    public static List<String> getBeansWithAnnotation(GenericApplicationContext applicationContext, Class<?> annotationClass) {
        List<String> result = new ArrayList<String>();
        ConfigurableListableBeanFactory factory = applicationContext.getBeanFactory();
        for(String name : factory.getBeanDefinitionNames()) {
            BeanDefinition bd = factory.getBeanDefinition(name);
            if(bd.getSource() instanceof AnnotatedTypeMetadata) {
                AnnotatedTypeMetadata metadata = (AnnotatedTypeMetadata) bd.getSource();
                if (metadata.getAnnotationAttributes(annotationClass.getName()) != null) {
                    result.add(name);
                }
            }
        }
        return result;
    }
}
```

或者，我们也可以使用`[Streams](/web/20220603091232/https://www.baeldung.com/java-streams)`编写相同的函数:

```java
public static List<String> getBeansWithAnnotation(GenericApplicationContext applicationContext, Class<?> annotationClass) {
    ConfigurableListableBeanFactory factory = applicationContext.getBeanFactory();
    return Arrays.stream(factory.getBeanDefinitionNames())
        .filter(name -> isAnnotated(factory, name, annotationClass))
        .collect(Collectors.toList());
}

private static boolean isAnnotated(ConfigurableListableBeanFactory factory, String beanName, Class<?> annotationClass) {
    BeanDefinition beanDefinition = factory.getBeanDefinition(beanName);
    if(beanDefinition.getSource() instanceof AnnotatedTypeMetadata) {
        AnnotatedTypeMetadata metadata = (AnnotatedTypeMetadata) beanDefinition.getSource();
        return metadata.getAnnotationAttributes(annotationClass.getName()) != null;
    }
    return false;
}
```

在这些方法中，我们使用了一个`GenericApplicationContext`，它是 Spring `[ApplicationContext](/web/20220603091232/https://www.baeldung.com/spring-application-context)`的一个实现，没有采用特定的 bean 定义格式。

例如，为了访问`GenericApplicationContext`，我们可以将它注入到一个 Spring 组件中:

```java
@Component
public class AnnotatedBeansComponent {

    @Autowired
    GenericApplicationContext applicationContext;

    public List<String> getBeansWithAnnotation(Class<?> annotationClass) {
        return BeanUtils.getBeansWithAnnotation(applicationContext, annotationClass);
    }
}
```

## 4.结论

在本文中，我们已经讨论了如何列出用给定注释注释的 beans。我们已经看到，从 Spring Boot 2.2 开始，这是通过`getBeansWithAnnotation`方法自然完成的。

另一方面，我们展示了一些替代方法来克服该方法先前行为的限制:要么只在我们的注释上添加 [`@Qualifier`](/web/20220603091232/https://www.baeldung.com/spring-qualifier-annotation) ，要么查找 beans，使用反射来检查它们是否有注释。

和往常一样，完整的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220603091232/https://github.com/eugenp/tutorials/tree/master/spring-di-3)