# 用 Spring Boot 创建一个定制的故障分析器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-failure-analyzer>

## 1。概述

Spring Boot 中的一个`FailureAnalyzer`提供了一种方法来**拦截在应用程序启动期间发生的异常**导致应用程序启动失败。

`FailureAnalyzer`用一个由`FailureAnalysis`对象表示的可读性更强的消息替换异常的堆栈跟踪，该消息包含错误描述和建议的操作过程。

Boot 包含一系列常见启动异常的分析器，如`PortInUseException`、`NoUniqueBeanDefinitionException`和、`UnsatisfiedDependencyException`。这些可以在`org.springframework.boot.diagnostics` 包中找到。

在这个快速教程中，我们将看一看**如何将我们自己的自定义`FailureAnalyzer`** 添加到现有的中。

## 2。`FailureAnalyzer`创建自定义

为了创建一个自定义的`FailureAnalyzer`，我们将简单地扩展抽象类`AbstractFailureAnalyzer`——它拦截一个指定的异常类型并实现`analyze()` API。

该框架提供了一个`BeanNotOfRequiredTypeFailureAnalyzer`实现，仅当被注入的 bean 属于动态代理类时，该实现才处理异常`BeanNotOfRequiredTypeException`。

让我们创建一个自定义的`FailureAnalyzer`来处理所有类型为`BeanNotOfRequiredTypeException.` 的异常。我们的类截获异常并创建一个`FailureAnalysis`对象，其中包含有用的描述和动作消息:

```java
public class MyBeanNotOfRequiredTypeFailureAnalyzer 
  extends AbstractFailureAnalyzer<BeanNotOfRequiredTypeException> {

    @Override
    protected FailureAnalysis analyze(Throwable rootFailure, 
      BeanNotOfRequiredTypeException cause) {
        return new FailureAnalysis(getDescription(cause), getAction(cause), cause);
    }

    private String getDescription(BeanNotOfRequiredTypeException ex) {
        return String.format("The bean %s could not be injected as %s "
          + "because it is of type %s",
          ex.getBeanName(),
          ex.getRequiredType().getName(),
          ex.getActualType().getName());
    }

    private String getAction(BeanNotOfRequiredTypeException ex) {
        return String.format("Consider creating a bean with name %s of type %s",
          ex.getBeanName(),
          ex.getRequiredType().getName());
    }
}
```

## 3。`FailureAnalyzer`登记自定义

对于 Spring Boot 要考虑的自定义`FailureAnalyzer`,必须将其注册到一个标准的`resources/META-INF/spring.factories`文件中，该文件包含带有我们的类的全名值的`org.springframework.boot.diagnostics.FailureAnalyzer`键:

```java
org.springframework.boot.diagnostics.FailureAnalyzer=\
  com.baeldung.failureanalyzer.MyBeanNotOfRequiredTypeFailureAnalyzer
```

## 4。习俗`FailureAnalyzer`在行动

让我们创建一个非常简单的例子，在这个例子中，我们尝试注入一个不正确类型的 bean，以查看我们的自定义`FailureAnalyzer`的行为。

让我们创建两个类`MyDAO`和`MySecondDAO`，并将第二个类标注为一个名为`myDAO`的 bean:

```java
public class MyDAO { }
```

```java
@Repository("myDAO")
public class MySecondDAO { }
```

接下来，在一个`MyService`类中，我们将尝试将`MySecondDAO`类型的`myDAO` bean 注入到`MyDAO`类型的变量中:

```java
@Service
public class MyService {

    @Resource(name = "myDAO")
    private MyDAO myDAO;
}
```

运行 Spring Boot 应用程序时，启动将失败，控制台输出如下:

```java
***************************
APPLICATION FAILED TO START
***************************

Description:

The bean myDAO could not be injected as com.baeldung.failureanalyzer.MyDAO 
  because it is of type com.baeldung.failureanalyzer.MySecondDAO$EnhancerBySpringCGLIB$d902559e

Action:

Consider creating a bean with name myDAO of type com.baeldung.failureanalyzer.MyDAO
```

## 5。结论

在这个快速教程中，我们重点关注了如何实现一个定制的 Spring Boot `FailureAnalyzer`。

和往常一样，这个例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220629013021/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-basic-customization)