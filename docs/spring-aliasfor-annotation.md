# 春天注释的别名

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-aliasfor-annotation>

## 1.概观

在本教程中，**我们将学习关于****@`AliasFor`春天**的注解。

首先，我们将看到使用它的框架中的例子。接下来，我们将看几个定制的例子。

## 2.注解

[`@AliasFor`](https://web.archive.org/web/20220827110142/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/annotation/AliasFor.html) 是 4.2 版以来框架的一部分。几个核心的 [Spring 注释](/web/20220827110142/https://www.baeldung.com/spring-core-annotations)已经被更新，现在包含了这个注释。

我们可以用它来修饰单个注释或由元注释组成的注释中的属性。也就是说，元注释是可以应用于另一个的注释。

在同一个注释中，**我们使用`@AliasFor`来声明属性的别名，这样我们就可以互换地应用它们**。或者，我们可以在组合注释中使用它来覆盖元注释中的属性。换句话说，当我们用`@AliasFor`修饰组合注释中的属性**时，它会覆盖元注释**中的指定属性。

有趣的是， [`@Bean`](/web/20220827110142/https://www.baeldung.com/spring-core-annotations#bean) 、 [`@ComponentScan`](/web/20220827110142/https://www.baeldung.com/spring-component-scanning) 、 [`@Scope`](/web/20220827110142/https://www.baeldung.com/spring-bean-scopes) 、 [`@RequestMapping`](/web/20220827110142/https://www.baeldung.com/spring-requestmapping) 、 [`@RestController`](/web/20220827110142/https://www.baeldung.com/spring-controller-vs-restcontroller) 等众核 Spring 注释现在都使用`@AliasFor`来配置它们的内部属性别名。

下面是注释的定义:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Documented
public @interface AliasFor {
    @AliasFor("attribute")
    String value() default "";

    @AliasFor("value")
    String attribute() default "";

    Class<? extends Annotation> annotation() default Annotation.class;
}
```

重要的是，**我们可以隐式和显式地使用这个注释**。隐式用法仅限于注释中的别名。相比之下，元注释中的属性也可以显式使用。

我们将在接下来的章节中通过示例详细了解这一点。

## 3.批注中的显式别名

让我们考虑一个核心 Spring 注释， [`@ComponentScan`](/web/20220827110142/https://www.baeldung.com/spring-component-scanning) ，来理解单个注释内的显式别名:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Repeatable(ComponentScans.class)
public @interface ComponentScan {

    @AliasFor("basePackages")
    String[] value() default {};

    @AliasFor("value")
    String[] basePackages() default {};
...
}
```

正如我们所见，`value`在这里被明确定义为`basePackages`的别名，反之亦然。这意味着**我们可以互换使用**。

实际上，这两种用法是相似的:

```java
@ComponentScan(basePackages = "com.baeldung.aliasfor")

@ComponentScan(value = "com.baeldung.aliasfor")
```

此外，由于这两个属性也被标记为`default`，让我们写得更简洁一些:

```java
@ComponentScan("com.baeldung.aliasfor")
```

此外，Spring 对这种场景有一些实现要求。首先，别名属性应该声明相同的默认值。此外，它们应该具有相同的返回类型。如果我们违反了这些约束中的任何一个，框架就会抛出一个`AnnotationConfigurationException`。

## 4.元注释中属性的显式别名

接下来，让我们看一个元注释的例子，并从它创建一个复合注释。然后，**我们将看到自定义别名**中别名的明确用法。

首先，让我们考虑将框架注释`RequestMapping` 作为我们的元注释:

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Mapping
public @interface RequestMapping {
    String name() default "";

    @AliasFor("path")
    String[] value() default {};

    @AliasFor("value")
    String[] path() default {};

    RequestMethod[] method() default {};
    ...
}
```

接下来，我们将从它创建一个组合注释`MyMapping`:

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@RequestMapping
public @interface MyMapping {
    @AliasFor(annotation = RequestMapping.class, attribute = "method")
    RequestMethod[] action() default {};
} 
```

我们可以看到，在`@MyMapping`中，`action`是`@RequestMapping`中属性`method`的显式别名。也就是说，我们的组合注释中的 **`action`覆盖了元注释中的`method`**。

类似于注释中的别名，元注释属性别名也必须具有相同的返回类型。比如我们这里的`RequestMethod[]`。此外，属性`annotation`应该引用元注释，就像我们使用`annotation = RequestMapping.class`一样。

为了演示，让我们添加一个名为`MyMappingController`的控制器类。我们将用我们的自定义注释来修饰它的方法。

具体来说，这里我们将只给`@MyMapping`、`route,`和`action`添加两个属性:

```java
@Controller
public class MyMappingController {

    @MyMapping(action = RequestMethod.PATCH, route = "/test")
    public void mappingMethod() {}

}
```

最后，为了了解显式别名的行为，让我们添加一个简单的测试:

```java
@Test
public void givenComposedAnnotation_whenExplicitAlias_thenMetaAnnotationAttributeOverridden() {
    for (Method method : controllerClass.getMethods()) {
        if (method.isAnnotationPresent(MyMapping.class)) {
            MyMapping annotation = AnnotationUtils.findAnnotation(method, MyMapping.class);
            RequestMapping metaAnnotation = 
              AnnotationUtils.findAnnotation(method, RequestMapping.class);

            assertEquals(RequestMethod.PATCH, annotation.action()[0]);

            assertEquals(0, metaAnnotation.method().length);
        }
    }
}
```

正如我们所看到的，我们的定制注释的属性`action`已经覆盖了元注释`@RequestMapping`的属性`method`。

## 5.批注中的隐式别名

为了理解这一点，**让我们在`@MyMapping`** 中再添加几个别名:

```java
@AliasFor(annotation = RequestMapping.class, attribute = "path")
String[] value() default {};

@AliasFor(annotation = RequestMapping.class, attribute = "path")
String[] mapping() default {};

@AliasFor(annotation = RequestMapping.class, attribute = "path")
String[] route() default {};
```

在这种情况下，`value`、`mapping`和`route`是`@RequestMapping`中`path`的显式元注释覆盖。所以也是彼此的隐性别名。换句话说，**对于`@MyMapping`，我们可以互换使用这三个属性。**

为了演示这一点，我们将使用与上一节相同的控制器。这是另一个测试:

```java
@Test
public void givenComposedAnnotation_whenImplictAlias_thenAttributesEqual() {
    for (Method method : controllerClass.getMethods()) {
        if (method.isAnnotationPresent(MyMapping.class)) {
            MyMapping annotationOnBean = 
              AnnotationUtils.findAnnotation(method, MyMapping.class);

            assertEquals(annotationOnBean.mapping()[0], annotationOnBean.route()[0]);
            assertEquals(annotationOnBean.value()[0], annotationOnBean.route()[0]);
        }
    }
}
```

值得注意的是，我们没有在控制器方法的注释中定义属性`value`和`mapping`。然而，**仍然隐含着与`route`** 相同的价值。

## 6.结论

在本教程中，**我们学习了 Spring 框架**中的`@AliasFor`注释。在我们的例子中，我们看了显式和隐式的使用场景。

和往常一样，源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220827110142/https://github.com/eugenp/tutorials/tree/master/spring-core-5)