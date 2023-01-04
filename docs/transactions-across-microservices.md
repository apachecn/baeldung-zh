# 微服务交易指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/transactions-across-microservices>

## 1。简介

在本文中，我们将讨论跨微服务实现事务的选项。

我们还将了解分布式微服务场景中事务的一些替代方案。

## 2。避免跨微服务的事务

分布式事务是一个非常复杂的过程，有许多可能失败的活动部分。此外，如果这些部分运行在不同的机器上，甚至运行在不同的数据中心，提交事务的过程可能会变得非常漫长和不可靠。

这可能会严重影响用户体验和整体系统带宽。所以解决分布式事务的最好方法之一就是完全避免它们。

### 2.1。需要交易的架构示例

通常，微服务被设计成独立且有用的。它应该能够解决一些原子业务任务。

如果我们可以将我们的系统拆分成这样的微服务，我们很有可能根本不需要在它们之间实现事务。

例如，让我们考虑一个在用户之间广播消息的系统。

`user`微服务将关注用户档案(创建新用户、编辑档案数据等。)和下面的基础域类:

```
@Entity
public class User implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    @Basic
    private String name;

    @Basic
    private String surname;

    @Basic
    private Instant lastMessageTime;
}
```

微服务将与广播有关。它封装了实体`Message`及其周围的一切:

```
@Entity
public class Message implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    @Basic
    private long userId;

    @Basic
    private String contents;

    @Basic
    private Instant messageTimestamp;

}
```

每个微服务都有自己的数据库。注意，我们没有从实体`Message`中引用实体`User`，因为用户类不能从`message`微服务中访问。我们仅通过 id 引用用户。

现在，`User`实体包含了`lastMessageTime`字段，因为我们想在她的配置文件中显示关于最后一次用户活动时间的信息。

然而，要向用户添加新消息并更新她的`lastMessageTime`，我们现在必须跨微服务实现一个事务。

### 2.2。无交易的替代方法

我们可以改变我们的微服务架构，将字段`lastMessageTime`从`User`实体中移除。

然后，我们可以通过向 messages 微服务发出一个单独的请求，并为该用户的所有消息找到最大的`messageTimestamp`值，从而在用户配置文件中显示这个时间。

很可能，如果`message`微服务处于高负载甚至宕机状态，我们将无法在她的个人资料中显示用户最后一条消息的时间。

但是这可能比仅仅因为用户微服务没有及时响应而无法提交分布式事务来保存消息更容易接受。

当我们必须跨多个微服务实现业务流程[时，当然会有更复杂的场景，我们不希望这些微服务之间存在不一致。](/web/20220727101414/https://www.baeldung.com/cs/microservices-cross-cutting-concerns)

## 3。两阶段提交协议

[两阶段提交协议](https://web.archive.org/web/20220727101414/https://en.wikipedia.org/wiki/Two-phase_commit_protocol)(或 2PC)是一种跨不同软件组件(多个数据库、消息队列等)实现事务的机制。)

### 3.1。2PC 的架构

分布式事务中的一个重要参与者是事务协调器。分布式事务由两个步骤组成:

*   准备阶段—在此阶段，事务的所有参与者都为提交做准备，并通知协调者他们已准备好完成事务
*   提交或回滚阶段—在此阶段，事务协调器向所有参与者发出提交或回滚命令

2PC 的问题是，与单个微服务的运行时间相比，它非常慢。

**协调微服务之间的事务，即使它们在同一个网络上，也会降低系统速度**，因此这种方法通常不用于高负载场景。

### 3.2。XA 标准

XA 标准是在支持资源上进行 2PC 分布式事务的规范。任何符合 JTA 标准的应用服务器(JBoss、GlassFish 等)。)支持开箱即用。

例如，参与分布式事务的资源可以是两个不同微服务的两个数据库。

然而，为了利用这种机制，资源必须被部署到单个 JTA 平台上。这对于微服务架构来说并不总是可行的。

### 3.3。静止标准吃水

另一个提议的标准是 [REST-AT](https://web.archive.org/web/20220727101414/https://github.com/jbosstm/documentation/tree/master/rts/docs) ，它已经由 RedHat 进行了一些开发，但是仍然没有脱离草案阶段。然而，WildFly 应用服务器支持开箱即用。

该标准允许使用应用服务器作为事务协调器，使用特定的 REST API 来创建和加入分布式事务。

希望参与两阶段事务的 RESTful web 服务也必须支持特定的 REST API。

不幸的是，要将分布式事务连接到微服务的本地资源，我们仍然必须将这些资源部署到单个 JTA 平台上，或者自己解决一个重要的任务来编写这个桥。

## 4。最终一致性和补偿

到目前为止，跨微服务处理一致性的最可行的模型之一是[最终一致性](https://web.archive.org/web/20220727101414/https://en.wikipedia.org/wiki/Eventual_consistency)。

该模型不强制跨微服务的分布式 ACID 事务。相反，它建议使用一些机制来确保系统在将来的某个时刻最终保持一致。

### 4.1。最终一致性案例

例如，假设我们需要解决以下任务:

*   注册用户配置文件
*   做一些自动的背景检查，确保用户可以真正访问系统

第二个任务是确保，比如说，这个用户不会因为某种原因被禁止进入我们的服务器。

但这可能需要时间，我们希望将其提取到单独的微服务中。让用户等这么久才知道她注册成功是不合理的。

解决这个问题的一个方法是使用消息驱动的方法，包括补偿。让我们考虑以下架构:

*   负责注册用户档案的微服务
*   负责背景调查的微服务
*   支持持久队列的消息传递平台

消息传递平台可以确保微服务发送的消息是持久的。那么如果接收者当前不可用，它们将在稍后的时间被递送

### 4.2。快乐场景

在这种架构中，一个令人满意的场景是:

*   微服务注册了一个用户，将她的信息保存在本地数据库中
*   `user`微服务用一个标志来标记这个用户。这可能意味着该用户尚未通过验证，不能访问全部系统功能
*   向用户发送注册确认，并警告用户并非系统的所有功能都可以立即使用
*   `user`微服务向`validation`微服务发送消息，对用户进行背景调查
*   `validation`微服务运行后台检查，并向`user`微服务发送包含检查结果的消息
    *   如果结果是肯定的，`user`微服务解除对用户的阻塞
    *   如果结果是否定的，`user`微服务删除用户账户

在我们完成所有这些步骤后，系统应该处于一致的状态。然而，在一段时间内，用户实体似乎处于不完整的状态。

**最后一步，用户微服务移除无效账号，是一个补偿阶段**。

### 4.3。故障场景

现在让我们考虑一些故障场景:

*   如果`validation`微服务不可访问，那么具有持久队列功能的消息平台确保`validation`微服务将在稍后的某个时间接收到该消息
*   假设消息传递平台失败，那么`user`微服务会在稍后的某个时间尝试再次发送消息，例如，通过对所有尚未验证的用户进行预定的批处理
*   如果`validation`微服务接收到消息，验证用户，但由于消息平台故障而无法发送回答案，`validation`微服务也会在稍后重试发送消息
*   如果其中一条消息丢失，或者发生了其他故障，`user`微服务会通过预定的批处理找到所有未验证的用户，并再次发送验证请求

即使一些消息被多次发布，这也不会影响微服务数据库中数据的一致性。

通过仔细考虑所有可能的故障场景，我们可以确保我们的系统满足最终一致性的条件。同时，我们不需要处理代价高昂的分布式事务。

但我们必须意识到，确保最终的一致性是一项复杂的任务。它没有一个适用于所有情况的单一解决方案。

## 5。结论

在本文中，我们讨论了一些跨微服务实现事务的机制。

首先，我们也探索了一些替代这种交易方式的方法。