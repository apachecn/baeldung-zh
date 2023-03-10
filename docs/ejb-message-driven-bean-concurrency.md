# 使用 MDB 的并发策略

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/ejb-message-driven-bean-concurrency>

## 1。简介

消息驱动 Beans，也称为“MDB”，在异步上下文中处理消息。我们可以在[这篇文章](/web/20221129021743/https://www.baeldung.com/ejb-message-driven-beans)中学习 MDB 的基础知识。

本教程将讨论使用消息驱动 Beans 实现并发的一些策略和最佳实践。

如果您想了解更多关于使用 Java 的并发性的基础知识，您可以从这里开始。

为了更好地使用 MDB 和并发，需要考虑一些问题。请记住，这些考虑应该由业务规则和我们应用程序的需求来驱动，这一点很重要。

## 2。调优线程池

调优线程池可能是主要的关注点。为了充分利用并发性，我们必须**调整可用于消费消息的 MDB 实例的数量**。当一个实例忙于处理一条消息时，其他实例能够继续处理下一条消息。

`MessageListener`线程负责执行 MDB 的`onMessage`方法。这个线程是`MessageListener`线程池的一部分，这意味着它被一次又一次地共享和重用。**这个池还有一个配置，允许我们设置线程数量，这可能会影响性能:**

*   设置较小的池大小将导致消息消耗缓慢(“MDB 节流”)
*   设置非常大的池大小可能会降低性能，甚至根本不起作用。

在`Wildfly`上，我们可以通过访问管理控制台来设置该值。默认独立配置文件上未启用 JMS 功能；我们需要使用`full profile`来启动服务器。

通常在本地安装上，我们通过 http://127 . 0 . 0 . 1:9990/console/index . html 访问，之后需要进入配置/子系统/消息/服务器，选择我们的服务器，点击“查看”。

选择“属性”选项卡，单击“编辑”并更改“线程池最大大小”的值。默认值为 30。

## 3。调整最大会话数

另一个需要注意的可配置属性是`Maximum Sessions`。这定义了特定侦听器端口的并发性。通常，这个值默认为 1，但是增加这个值可以为 MDB 应用程序提供更多的可伸缩性和可用性。

我们可以通过注释或`.xml`描述符来配置它。通过注释，我们使用了`@ActivationConfigProperty`:

```java
@MessageDriven(activationConfig = {
    @ActivationConfigProperty(
        propertyName=”maxSession”, propertyValue=”50”
    )
})
```

如果选择的配置方法是`.xml`描述符，我们可以这样配置`maxSession`:

```java
<activation-config>
    <activation-config-property>
        <activation-config-property-name>maxSession</activation-config-property-name>
        <activation-config-property-value>50</activation-config-property-value>
    </activation-config-property>
</activation-config>
```

## 4。部署环境

当我们需要高可用性时，我们应该考虑在应用服务器集群上部署 MDB。因此，它可以在集群中的任何服务器上执行，并且许多应用服务器可以并发地调用它，这也提高了可伸缩性。

对于这种特殊情况，我们需要做出一个重要的选择:

*   使**集群中的所有服务器都有资格接收消息**，这允许使用其所有处理能力，或者
*   通过一次只允许一台服务器接收 **消息**，确保消息处理有序

如果我们使用企业总线，一个好的实践是将 MDB 部署到与总线成员相同的服务器或集群，以优化消息传递性能。

## 5。消息模型和消息类型

虽然这不像只是为池设置另一个值那么清楚，但是消息模型和消息类型可能会影响使用并发的一个最佳优势:性能。

例如，当选择 XML 作为消息类型时，**消息的大小会影响处理它所花费的时间**。这是一个重要的考虑因素，尤其是在应用程序处理大量消息的情况下。

关于消息模型，**如果应用程序需要向许多消费者发送相同的消息，`publish-subscribe`模型可能是正确的选择**。这将减少处理消息的开销，提供更好的性能。

为了在发布-订阅模型上从`Topic`消费，我们可以使用注释:

```java
@ActivationConfigProperty(
  propertyName = "destinationType", 
  propertyValue = "javax.jms.Topic")
```

同样，我们也可以在`.xml`部署描述符中配置这些值:

```java
<activation-config>
    <activation-config-property>
        <activation-config-property-name>destinationType</activation-config-property-name>
        <activation-config-property-value>javax.jms.Topic</activation-config-property-value>
    </activation-config-property>
</activation-config>
```

如果不需要向许多消费者发送完全相同的消息，那么常规的 PTP(点对点)模型就足够了。

为了从队列中消费，我们将注释设置为:

```java
@ActivationConfigProperty(
  propertyName = "destinationType", 
  propertyValue = "javax.jms.Queue")
```

如果我们使用的是`.xml`部署描述符，我们可以设置它:

```java
<activation-config>
    <activation-config-property>
        <activation-config-property-name>destinationType</activation-config-property-name>
        <activation-config-property-value>javax.jms.Queue</activation-config-property-value>
    </activation-config-property>
</activation-config>
```

## 6。结论

正如许多计算机科学家和 IT 作家已经指出的，我们不再有处理器的速度快速增长。为了让我们的程序工作得更快，我们需要使用更多的处理器和内核。

本文讨论了使用 MDB 充分利用并发性的一些最佳实践。