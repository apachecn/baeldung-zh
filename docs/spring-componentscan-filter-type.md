# spring @ components can–过滤器类型

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-componentscan-filter-type>

## 1.概观

在之前的教程中，我们学习了 Spring 组件扫描的[基础。](/web/20221001115718/https://www.baeldung.com/spring-component-scanning)

在这篇文章中，我们将看到`[@ComponentScan](https://web.archive.org/web/20221001115718/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/annotation/ComponentScan.html)` 注释`.`中可用的不同类型的过滤器选项

## 2.@ `ComponentScan` 过滤器

默认情况下，用`@Component, @Repository, @Service, @Controller`标注的类被注册为[Spring bean](/web/20221001115718/https://www.baeldung.com/spring-bean)。同样的情况也适用于使用定制注释进行注释的类，定制注释是用`@Component`进行注释的。我们可以通过使用`@ComponentScan`注释的`includeFilters and`参数来扩展这个行为。

**[`ComponentScan.Filter`](https://web.archive.org/web/20221001115718/https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/context/annotation/ComponentScan.Filter.html):**共有五种滤镜可供选择

*   `ANNOTATION`
*   `ASSIGNABLE_TYPE`
*   `ASPECTJ`
*   `REGEX`
*   `CUSTOM`

我们将在下一节中详细讨论这些。

我们应该注意，所有这些过滤器都可以包含或排除扫描的类。为了简单起见，在我们的例子中，我们将只包括类。

## 3. `FilterType.ANNOTATION`

**`ANNOTATION`过滤器类型包括或排除组件扫描中用给定注释标记的类别。**

比方说，我们有一个`@Anima` l 注释:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Animal { }
```

现在，让我们定义一个使用`@Animal`的`Elephant`类:

```java
@Animal
public class Elephant { }
```

最后，让我们使用`FilterType.ANNOTATION`来告诉 Spring 扫描`@Animal`注释的类:

```java
@Configuration
@ComponentScan(includeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION,
        classes = Animal.class))
public class ComponentScanAnnotationFilterApp { }
```

正如我们所见，扫描仪很好地接收了我们的`Elephant`:

```java
@Test
public void whenAnnotationFilterIsUsed_thenComponentScanShouldRegisterBeanAnnotatedWithAnimalAnootation() {
    ApplicationContext applicationContext =
            new AnnotationConfigApplicationContext(ComponentScanAnnotationFilterApp.class);
    List<String> beans = Arrays.stream(applicationContext.getBeanDefinitionNames())
            .filter(bean -> !bean.contains("org.springframework")
                    && !bean.contains("componentScanAnnotationFilterApp"))
            .collect(Collectors.toList());
    assertThat(beans.size(), equalTo(1));
    assertThat(beans.get(0), equalTo("elephant"));
}
```

## 4. `FilterType.` `ASSIGNABLE_TYPE`

**`ASSIGNABLE_TYPE`在组件扫描期间过滤所有扩展类或实现指定类型接口的类。**

首先，让我们声明`Animal`接口:

```java
public interface Animal { }
```

同样，让我们声明我们的`Elephant`类，这次实现`Animal` 接口`:`

```java
public class Elephant implements Animal { }
```

让我们声明我们的`Cat`类，它也实现了`Animal:`

```java
public class Cat implements Animal { }
```

现在，让我们使用`ASSIGNABLE_TYPE` 来引导 Spring 扫描`Animal`-实现类:

```java
@Configuration
@ComponentScan(includeFilters = @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE,
        classes = Animal.class))
public class ComponentScanAssignableTypeFilterApp { }
```

我们将看到`Cat`和`Elephant`都被扫描:

```java
@Test
public void whenAssignableTypeFilterIsUsed_thenComponentScanShouldRegisterBean() {
    ApplicationContext applicationContext =
      new AnnotationConfigApplicationContext(ComponentScanAssignableTypeFilterApp.class);
    List<String> beans = Arrays.stream(applicationContext.getBeanDefinitionNames())
      .filter(bean -> !bean.contains("org.springframework")
        && !bean.contains("componentScanAssignableTypeFilterApp"))
      .collect(Collectors.toList());
    assertThat(beans.size(), equalTo(2));
    assertThat(beans.contains("cat"), equalTo(true));
    assertThat(beans.contains("elephant"), equalTo(true));
}
```

## 5. `FilterType.REGEX`

**`REGEX`过滤器检查类名是否匹配给定的正则表达式模式。`FilterType.REGEX`检查简单类名和完全限定类名。**

再次声明我们的`Elephant`类。这次没有实现任何接口或者用任何注释进行注释`:`

```java
public class Elephant { }
```

让我们再声明一个类`Cat`:

```java
public class Cat { }
```

现在，让我们声明`Loin`类:

```java
public class Loin { }
```

还是用`FilterType`吧。`REGEX`指示 Spring 扫描匹配正则表达式`.*[nt].` 的类，我们的正则表达式计算所有包含`nt:`的内容

```java
@Configuration
@ComponentScan(includeFilters = @ComponentScan.Filter(type = FilterType.REGEX,
        pattern = ".*[nt]"))
public class ComponentScanRegexFilterApp { }
```

这次在我们的测试中，我们将看到 Spring 扫描了`Elephant`，但是没有扫描`Lion` `:`

```java
@Test
public void whenRegexFilterIsUsed_thenComponentScanShouldRegisterBeanMatchingRegex() {
    ApplicationContext applicationContext =
      new AnnotationConfigApplicationContext(ComponentScanRegexFilterApp.class);
    List<String> beans = Arrays.stream(applicationContext.getBeanDefinitionNames())
      .filter(bean -> !bean.contains("org.springframework")
        && !bean.contains("componentScanRegexFilterApp"))
      .collect(Collectors.toList());
    assertThat(beans.size(), equalTo(1));
    assertThat(beans.contains("elephant"), equalTo(true));
}
```

## 6. `FilterType.ASPECTJ`

**当我们想用表达式挑出一个复杂的类子集时，我们需要使用`FilterType` `ASPECTJ`。**

对于这个用例，我们可以重用与上一节相同的三个类。

让我们使用`FilterType.ASPECTJ`命令 Spring 扫描匹配我们的`AspectJ`表达式的类:

```java
@Configuration
@ComponentScan(includeFilters = @ComponentScan.Filter(type = FilterType.ASPECTJ,
  pattern = "com.baeldung.componentscan.filter.aspectj.* "
  + "&& !(com.baeldung.componentscan.filter.aspectj.L* "
  + "|| com.baeldung.componentscan.filter.aspectj.C*)"))
public class ComponentScanAspectJFilterApp { }
```

虽然有点复杂，但是我们这里的逻辑希望 beans 的类名既不是以“L”也不是以“C”开头，所以这又给我们留下了`Elephant` s:

```java
@Test
public void whenAspectJFilterIsUsed_thenComponentScanShouldRegisterBeanMatchingAspectJCreteria() {
    ApplicationContext applicationContext =
      new AnnotationConfigApplicationContext(ComponentScanAspectJFilterApp.class);
    List<String> beans = Arrays.stream(applicationContext.getBeanDefinitionNames())
      .filter(bean -> !bean.contains("org.springframework")
        && !bean.contains("componentScanAspectJFilterApp"))
      .collect(Collectors.toList());
    assertThat(beans.size(), equalTo(1));
    assertThat(beans.get(0), equalTo("elephant"));
}
```

## 7.`FilterType.CUSTOM`

**如果以上过滤器类型都不符合我们的要求，那么** **我们还可以创建一个自定义过滤器类型。**例如，假设我们只想扫描名称不超过五个字符的类。

要创建自定义过滤器，我们需要实现`org.springframework.core.type.filter.TypeFilter`:

```java
public class ComponentScanCustomFilter implements TypeFilter {

    @Override
    public boolean match(MetadataReader metadataReader,
      MetadataReaderFactory metadataReaderFactory) throws IOException {
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        String fullyQualifiedName = classMetadata.getClassName();
        String className = fullyQualifiedName.substring(fullyQualifiedName.lastIndexOf(".") + 1);
        return className.length() > 5 ? true : false;
    }
}
```

还是用`FilterType`吧。`CUSTOM`它使用我们的自定义过滤器`ComponentScanCustomFilter:`传送 Spring 来扫描类

```java
@Configuration
@ComponentScan(includeFilters = @ComponentScan.Filter(type = FilterType.CUSTOM,
  classes = ComponentScanCustomFilter.class))
public class ComponentScanCustomFilterApp { }
```

现在是时候看看我们定制过滤器的测试用例了`ComponentScanCustomFilter:`

```java
@Test
public void whenCustomFilterIsUsed_thenComponentScanShouldRegisterBeanMatchingCustomFilter() {
    ApplicationContext applicationContext =
      new AnnotationConfigApplicationContext(ComponentScanCustomFilterApp.class);
    List<String> beans = Arrays.stream(applicationContext.getBeanDefinitionNames())
      .filter(bean -> !bean.contains("org.springframework")
        && !bean.contains("componentScanCustomFilterApp")
        && !bean.contains("componentScanCustomFilter"))
      .collect(Collectors.toList());
    assertThat(beans.size(), equalTo(1));
    assertThat(beans.get(0), equalTo("elephant"));
}
```

## 8.摘要

在本教程中，我们介绍了与`@ComponentScan.`相关的过滤器

像往常一样，完整的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221001115718/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-di)