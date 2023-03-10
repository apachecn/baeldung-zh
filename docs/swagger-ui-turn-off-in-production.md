# 如何在生产中关闭 Swagger-ui

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/swagger-ui-turn-off-in-production>

## 1.概观

Swagger 用户界面允许我们查看关于 REST 服务的信息。这对于开发来说非常方便。然而，出于安全考虑，我们可能不希望在公共环境中允许这种行为。

在这个简短的教程中，我们将看看**如何让** **在制作**中扬眉吐气。

## 2.摇摆构型

为了用 Spring 设置 Swagger，我们在一个配置 bean 中定义它。

让我们创建一个`SwaggerConfig`类:

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig implements WebMvcConfigurer {

    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2).select()
                .apis(RequestHandlerSelectors.basePackage("com.baeldung"))
                .paths(PathSelectors.regex("/.*"))
                .build();
    }

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");
        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");
    }
}
```

默认情况下，这个配置 bean 总是被注入到我们的 Spring 上下文中。因此，Swagger 适用于所有环境。

为了在生产中禁用 Swagger，让我们切换是否注入这个配置 bean。

## 3.使用弹簧轮廓

在 Spring 中，我们可以[使用`@Profile`注释](/web/20220630142743/https://www.baeldung.com/spring-profiles)来启用或禁用 beans 的注入。

让我们尝试使用一个 [SpEL 表达式](/web/20220630142743/https://www.baeldung.com/spring-expression-language)来匹配`“swagger”`概要文件，而不是`“prod”`概要文件:

```java
@Profile({"!prod && swagger"})
```

这迫使我们明确我们想要激活 Swagger 的环境。它还有助于防止在生产中意外打开它。

我们可以将注释添加到配置中:

```java
@Configuration 
@Profile({"!prod && swagger"})
@EnableSwagger2 
public class SwaggerConfig implements WebMvcConfigurer {
    ...
}
```

现在，让我们通过使用不同的`spring.profiles.active`属性设置启动我们的应用程序来测试它是否工作:

```java
 -Dspring.profiles.active=prod // Swagger is disabled

  -Dspring.profiles.active=prod,anyOther // Swagger is disabled

  -Dspring.profiles.active=swagger // Swagger is enabled

  -Dspring.profiles.active=swagger,anyOtherNotProd // Swagger is enabled

  none // Swagger is disabled
```

## 4.使用条件句

对于特征切换来说，弹簧轮廓可能是太粗粒度的解决方案。这种方法会导致配置错误和冗长的、不可管理的概要文件列表。

作为替代，我们可以使用`@ConditionalOnExpression`，它允许为启用 bean 指定自定义属性:

```java
@Configuration
@ConditionalOnExpression(value = "${useSwagger:false}")
@EnableSwagger2
public class SwaggerConfig implements WebMvcConfigurer {
    ...
}
```

如果没有“`useSwagger`”属性，这里默认为`false`。

为了测试这一点，我们可以在`application.properties`(或`application.yaml`)文件中设置该属性，或者将其设置为一个 VM 选项:

```java
-DuseSwagger=true
```

```java
我们应该注意，这个例子没有包括任何保证我们的生产实例不会意外地将`useSwagger`设置为`true`的方法。
5.避免陷阱

如果启用 Swagger 是一个安全问题，那么我们需要选择一个防错但易于使用的策略。

当我们使用`@Profile`时，一些 SpEL 表达式可能会违背这些目标:

```
@Profile({"!prod"}) // Leaves Swagger enabled by default with no way to disable it in other profiles 
```java

```
@Profile({"swagger"}) // Allows activating Swagger in prod as well 
```java

```
@Profile({"!prod", "swagger"}) // Equivalent to {"!prod || swagger"} so it's worse than {"!prod"} as it provides a way to activate Swagger in prod too
```java

这就是为什么我们的`@Profile`示例使用了:

```
@Profile({"!prod && swagger"})
```java

这种解决方案可能是最严格的，因为它默认禁用 Swagger **，并保证在** `**“prod”**. `中不能启用 

 **## 6.结论

在本文中，我们研究了在生产中**禁用招摇** **的解决方案。**

我们看了如何通过`@Profile`和`@ConditionalOnExpression`注释切换打开 Swagger 的 bean。我们还考虑了如何防止错误配置和不希望的缺省。

和往常一样，本文的示例代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220630142743/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-http-2)** 
```