# 弹簧条件注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-conditional-annotations>

## 1.介绍

在本教程中，我们来看看 [`@Conditional`](https://web.archive.org/web/20220827110142/https://docs.spring.io/spring-framework/docs/5.3.7/javadoc-api/org/springframework/context/annotation/Conditional.html) 的注释。它用于指示给定组件是否符合基于定义条件的注册条件。

我们将学习如何使用预定义的条件注释，将它们与不同的条件相结合，以及创建我们自定义的、基于条件的注释。

## 2.**申报条件**

在我们开始实现之前，让我们先看看在哪些情况下我们可以使用条件注释。

最常见的用法是**包含或排除整个配置类**:

```
@Configuration
@Conditional(IsDevEnvCondition.class)
class DevEnvLoggingConfiguration {
    // ...
} 
```

或者只是一颗豆子:

```
@Configuration
class DevEnvLoggingConfiguration {

    @Bean
    @Conditional(IsDevEnvCondition.class)
    LoggingService loggingService() {
        return new LoggingService();
    }
}
```

通过这样做，我们可以将应用程序的行为基于给定的条件。例如，环境类型或我们客户的特定需求。在上面的例子中，我们只为开发环境初始化额外的日志服务。

使组件有条件的另一种方法是将条件直接放在组件类上:

```
@Service
@Conditional(IsDevEnvCondition.class)
class LoggingService {
    // ...
}
```

我们可以将上面的例子应用于任何用`@Component`、`@Service`、`@Repository,`或`@Controller`注释声明的 bean。

## 3.预定义的条件注释

Spring 附带了一组预定义的条件注释。让我们来看看一些最受欢迎的。

首先，让我们看看如何将一个组件基于一个配置属性值 :

```
@Service
@ConditionalOnProperty(
  value="logging.enabled", 
  havingValue = "true", 
  matchIfMissing = true)
class LoggingService {
    // ...
}
```

第一个属性`value,`告诉我们将查看什么配置属性。第二个，`havingValue,`定义了这个条件所需的值。最后，`matchIfMissing` 属性告诉 Spring，如果缺少参数，条件是否应该匹配。

类似地，我们可以**将条件基于表达式**:

```
@Service
@ConditionalOnExpression(
  "${logging.enabled:true} and '${logging.level}'.equals('DEBUG')"
)
class LoggingService {
    // ...
}
```

现在，只有当`logging.enabled`配置属性被设置为`true,`并且`logging.level`被设置为`DEBUG.`时，Spring 才会创建`LoggingService`

我们可以应用的另一个条件是检查给定的 bean 是否被创建:

```
@Service
@ConditionalOnBean(CustomLoggingConfiguration.class)
class LoggingService {
    // ...
}
```

或者给定的类存在于类路径中:

```
@Service
@ConditionalOnClass(CustomLogger.class)
class LoggingService {
    // ...
}
```

我们可以通过应用`@ConditionalOnMissingBean`或`@ConditionalOnMissingClass`注释来实现相反的行为。

此外，我们可以**让我们的组件依赖于给定的 Java 版本**:

```
@Service
@ConditionalOnJava(JavaVersion.EIGHT)
class LoggingService {
    // ...
}
```

在上面的例子中，`LoggingService`只有在运行时环境是 Java 8 时才会被创建。

最后，我们可以使用 [`@ConditionalOnWarDeployment`](https://web.archive.org/web/20220827110142/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/condition/ConditionalOnWarDeployment.html) 注释来启用 bean only in war 打包:

```
@Configuration
@ConditionalOnWarDeployment
class AdditionalWebConfiguration {
    // ...
}
```

请注意，对于具有嵌入式服务器的应用程序，此条件将返回 false。

## 4.**定义自定义条件**

Spring 允许我们通过**创建我们的定制条件模板**来定制`@Conditional`注释的行为。要创建一个，我们只需实现 [`Condition`](https://web.archive.org/web/20220827110142/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Condition.html) 接口:

```
class Java8Condition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        return JavaVersion.getJavaVersion().equals(JavaVersion.EIGHT);
    }
}
```

`matches`方法告诉`Spring`条件是否已经通过。它有两个参数，分别为我们提供关于 bean 初始化的上下文和所使用的`@Conditional`注释的元数据的信息。

正如我们所看到的，在我们的例子中，我们只是检查 Java 版本是否是 8。

之后，我们应该把我们的新条件作为属性放在`@Conditional`注释中:

```
@Service
@Conditional(Java8Condition.class)
public class Java8DependedService {
    // ...
}
```

这样，`Java8DependentService`将仅在来自`Java8Condition`类的条件匹配时被创建。

## 5.**组合条件**

对于更复杂的解决方案，我们可以**用 or 或 AND 逻辑操作符**对条件注释进行分组。

要应用 OR 运算符，我们需要创建一个自定义条件来扩展 [`AnyNestedCondition`](https://web.archive.org/web/20220827110142/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/condition/AnyNestedCondition.html) 类。在其中，我们需要为每个条件创建一个空的`static`类，并用适当的`@Conditional`实现对其进行注释。

例如，让我们创建一个需要 Java 8 或 Java 9 的条件:

```
class Java8OrJava9 extends AnyNestedCondition {

    Java8OrJava9() {
        super(ConfigurationPhase.REGISTER_BEAN);
    }

    @Conditional(Java8Condition.class)
    static class Java8 { }

    @Conditional(Java9Condition.class)
    static class Java9 { }

}
```

另一方面，AND 运算符要简单得多。我们可以简单地对条件进行分组:

```
@Service
@Conditional({IsWindowsCondition.class, Java8Condition.class})
@ConditionalOnJava(JavaVersion.EIGHT)
public class LoggingService {
    // ...
}
```

在上面的例子中，只有当`IsWindowsCondition`和`Java8Condition`都匹配时，才会创建`LoggingService` 。

## 6.摘要

在本文中，我们学习了如何使用和创建条件注释。和往常一样，GitHub 上的所有源代码[都是可用的。](https://web.archive.org/web/20220827110142/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-annotations-2)