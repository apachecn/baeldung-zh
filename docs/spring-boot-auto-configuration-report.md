# 在 Spring Boot 显示自动配置报告

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-auto-configuration-report>

## 1。概述

Spring Boot 中的自动配置机制试图根据应用程序的依赖关系自动配置应用程序。

在这个快速教程中，**我们将看到 Spring Boot 如何在启动时记录它的自动配置报告。**

## 2。示例应用程序

让我们编写一个简单的 Spring Boot 应用程序，我们将在我们的例子中使用:

```java
@SpringBootApplication
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}
```

## 3。应用属性方法

在启动这个应用程序时，我们没有得到很多关于 Spring Boot 如何或为什么决定组成我们的应用程序的配置的信息。

但是，**我们可以让 Spring Boot 简单地通过在我们的`application.properties`文件中启用调试模式**来创建一个报告:

```java
debug=true 
```

或者我们的`application.yml`文件:

```java
debug: true
```

## 4。命令行方式

或者，如果我们不想使用属性文件方法，我们可以通过**用`–debug`开关**启动应用程序来触发自动配置报告:

```java
$ java -jar myproject-0.0.1-SNAPSHOT.jar --debug 
```

## 5。报告输出

自动配置报告包含有关 Spring Boot 在类路径上找到并自动配置的类的信息。它还显示了 Spring Boot 已知但在类路径中找不到的类的信息。

而且，因为我们已经设置了`debug=true`，所以我们可以在输出中看到它:

```java
============================
CONDITIONS EVALUATION REPORT
============================

Positive matches:
-----------------

   AopAutoConfiguration matched:
      - @ConditionalOnClass found required classes 'org.springframework.context.annotation.EnableAspectJAutoProxy', 
        'org.aspectj.lang.annotation.Aspect', 'org.aspectj.lang.reflect.Advice', 'org.aspectj.weaver.AnnotatedElement'; 
        @ConditionalOnMissingClass did not find unwanted class (OnClassCondition)
      - @ConditionalOnProperty (spring.aop.auto=true) matched (OnPropertyCondition)

   AopAutoConfiguration.CglibAutoProxyConfiguration matched:
      - @ConditionalOnProperty (spring.aop.proxy-target-class=true) matched (OnPropertyCondition)

   AuditAutoConfiguration#auditListener matched:
      - @ConditionalOnMissingBean (types: org.springframework.boot.actuate.audit.listener.AbstractAuditListener; 
        SearchStrategy: all) did not find any beans (OnBeanCondition)

   AuditAutoConfiguration.AuditEventRepositoryConfiguration matched:
      - @ConditionalOnMissingBean (types: org.springframework.boot.actuate.audit.AuditEventRepository; 
        SearchStrategy: all) did not find any beans (OnBeanCondition)

   AuditEventsEndpointAutoConfiguration#auditEventsEndpoint matched:
      - @ConditionalOnBean (types: org.springframework.boot.actuate.audit.AuditEventRepository; 
        SearchStrategy: all) found bean 'auditEventRepository'; 
        @ConditionalOnMissingBean (types: org.springframework.boot.actuate.audit.AuditEventsEndpoint; 
        SearchStrategy: all) did not find any beans (OnBeanCondition)
      - @ConditionalOnEnabledEndpoint no property management.endpoint.auditevents.enabled found 
        so using endpoint default (OnEnabledEndpointCondition)

Negative matches:
-----------------

   ActiveMQAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'javax.jms.ConnectionFactory', 
           'org.apache.activemq.ActiveMQConnectionFactory' (OnClassCondition)

   AopAutoConfiguration.JdkDynamicAutoProxyConfiguration:
      Did not match:
         - @ConditionalOnProperty (spring.aop.proxy-target-class=false) did not find property 
           'proxy-target-class' (OnPropertyCondition)

   ArtemisAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'javax.jms.ConnectionFactory', 
           'org.apache.activemq.artemis.jms.client.ActiveMQConnectionFactory' (OnClassCondition)

   AtlasMetricsExportAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required class 'io.micrometer.atlas.AtlasMeterRegistry' 
           (OnClassCondition)

   AtomikosJtaConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required class 'com.atomikos.icatch.jta.UserTransactionManager' 
           (OnClassCondition) 
```

## 6。结论

在这个快速教程中，我们看到了如何显示和阅读 Spring Boot 自动配置报告。

和往常一样，上面例子的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220728232403/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-autoconfiguration)