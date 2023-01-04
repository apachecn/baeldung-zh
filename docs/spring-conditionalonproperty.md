# Spring @ ConditionalOnProperty 批注

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-conditionalonproperty>

## 1.概观

在这个简短的教程中，我们将检查`@ConditionalOnProperty`注释的主要**目的。**

首先，我们先来了解一下`@ConditionalOnProperty`是什么的背景。然后我们将看一些实际的例子来帮助理解它是如何工作的以及它带来了什么特性。

## 2.`@ConditionalOnProperty`的目的

通常，当开发基于 Spring 的应用程序时，**我们需要[根据配置属性](/web/20221031142400/https://www.baeldung.com/spring-bean)**的存在和值有条件地创建一些 bean。

例如，我们可能希望注册一个`DataSource` bean 来指向生产或测试数据库，这取决于我们是否将属性值设置为“prod”或“test”

幸运的是，实现这一点并不像乍看上去那么困难。Spring 框架提供了 [`@ConditionalOnProperty`注释](/web/20221031142400/https://www.baeldung.com/spring-boot-custom-auto-configuration#3-property-conditions)正是为了这个目的。

简而言之，`@ConditionalOnProperty` 仅在环境属性存在并具有特定值时才启用 bean 注册。默认情况下，指定的属性必须被定义并且不等于`false`。

现在我们已经熟悉了`@ConditionalOnProperty`注释的目的，让我们更深入地了解它是如何工作的。

## 3.`@ConditionalOnProperty`实践中的注解

为了举例说明`@ConditionalOnProperty,` 的用法，我们将开发一个基本的通知系统。现在为了简单起见，让我们假设我们想要发送电子邮件通知。

首先，我们需要创建一个简单的服务来发送通知消息。例如，考虑一下`NotificationSender`接口:

```java
public interface NotificationSender {
    String send(String message);
}
```

接下来，让我们提供一个用于发送电子邮件的`NotificationSender`接口的实现:

```java
public class EmailNotification implements NotificationSender {
    @Override
    public String send(String message) {
        return "Email Notification: " + message;
    }
}
```

现在让我们看看如何利用`@ConditionalOnProperty`注释。**让我们这样配置`NotificationSender` bean，只有当属性`notification.service`被定义为**时，它才会被加载:

```java
@Bean(name = "emailNotification")
@ConditionalOnProperty(prefix = "notification", name = "service")
public NotificationSender notificationSender() {
    return new EmailNotification();
}
```

我们可以看到， **`prefix`和`name`属性用来表示应该检查的配置属性**。

最后，我们需要添加最后一块拼图。让我们在 [`application.properties`文件](/web/20221031142400/https://www.baeldung.com/properties-with-spring#1-applicationproperties---the-default-property-file)中定义我们的自定义属性:

```java
notification.service=email
```

## 4.高级配置

正如我们已经知道的，`@ConditionalOnProperty`注释允许我们根据配置属性的存在有条件地注册 beans。

然而，我们可以用这个注释`.`做更多的事情，所以让我们来探索一下吧！

假设我们想要添加另一个通知服务，比如允许我们发送 SMS 通知的服务。

为此，我们需要创建另一个`NotificationSender` 实现:

```java
public class SmsNotification implements NotificationSender {
    @Override
    public String send(String message) {
        return "SMS Notification: " + message;
    }
}
```

因为我们有两个实现，所以让我们看看如何使用`@ConditionalOnProperty` 有条件地加载正确的`NotificationSender` bean。

为此，注释提供了`havingValue` 属性。非常有趣的是， **it** **定义了一个属性必须具有的值，以便将特定的 bean 添加到 [Spring 容器](/web/20221031142400/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring#the-spring-ioc-container)** 。

现在让我们指定在什么条件下我们想要在上下文中注册`SmsNotification`实现:

```java
@Bean(name = "smsNotification")
@ConditionalOnProperty(prefix = "notification", name = "service", havingValue = "sms")
public NotificationSender notificationSender2() {
    return new SmsNotification();
}
```

在`havingValue`属性的帮助下，我们清楚地表明，只有当`notification.service `被设置为`sms`时，我们才希望加载`SmsNotification`。

值得一提的是，`@ConditionalOnProperty` 还有一个属性叫做 [`matchIfMissing`](https://web.archive.org/web/20221031142400/https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/condition/ConditionalOnProperty.html#matchIfMissing--) 。**该属性指定在属性不可用的情况下，条件是否应该匹配**。

现在让我们把所有的部分放在一起，并编写一个简单的测试用例来确认一切按预期工作:

```java
@Test
public void whenValueSetToEmail_thenCreateEmailNotification() {
    this.contextRunner.withPropertyValues("notification.service=email")
        .withUserConfiguration(NotificationConfig.class)
        .run(context -> {
            assertThat(context).hasBean("emailNotification");
            NotificationSender notificationSender = context.getBean(EmailNotification.class);
            assertThat(notificationSender.send("Hello From Baeldung!")).isEqualTo("Email Notification: Hello From Baeldung!");
            assertThat(context).doesNotHaveBean("smsNotification");
        });
}
```

## 5.结论

在这篇简短的文章中，我们强调了使用`@ConditionalOnProperty` 注释`.` 的目的，然后我们通过一个实例学习了如何使用它来有条件地加载 Spring beans。

和往常一样，本文的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221031142400/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-autoconfiguration)