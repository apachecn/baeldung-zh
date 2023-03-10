# DDD 聚合和@域事件

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-ddd>

## 1。概述

在本教程中，我们将解释如何使用`@DomainEvents`注释和`AbstractAggregateRoot`类来方便地发布和处理由 aggregate 产生的领域事件——领域驱动设计中的关键战术设计模式之一。

**聚合接受业务命令，这通常会导致产生一个与业务领域相关的事件——领域事件**。

如果你想了解更多关于 DDD 和聚合的知识，最好从 Eric Evans 的[原著](https://web.archive.org/web/20220712173816/https://www.amazon.com/exec/obidos/ASIN/0321125215/domainlanguag-20)开始。还有一个由沃恩·弗农写的关于有效总体设计的伟大的[系列](https://web.archive.org/web/20220712173816/http://dddcommunity.org/wp-content/uploads/files/pdf_articles/Vernon_2011_1.pdf)。绝对值得一读。

手动处理域事件可能很麻烦。令人欣慰的是， **Spring Framework 允许我们在使用数据存储库处理聚合根**时轻松发布和处理域事件。

## 2。Maven 依赖关系

Ingalls 发布系列中引入的春季数据`@DomainEvents`。它适用于任何类型的存储库。

本文提供的代码示例使用 Spring Data JPA。**将 Spring 域事件与我们的项目集成的最简单的方法是使用 [Spring Boot 数据 JPA 启动器](https://web.archive.org/web/20220712173816/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-data-jpa) :**

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

## 3。手动发布事件

首先，让我们尝试手动发布域事件。我们将在下一节解释`@DomainEvents`的用法。

根据本文的需要，我们将为域事件使用一个空的标记类——`DomainEvent`。

我们将使用标准的`ApplicationEventPublisher`接口。

**我们可以在两个好地方发布事件:服务层或直接在聚合内部**。

### 3.1.服务层

在服务方法中调用存储库`save`方法后，我们可以简单地发布事件。

如果服务方法是事务的一部分，并且我们在用`@TransactionalEventListener`注释的监听器中处理事件，那么只有在事务成功提交后事件才会被处理。

因此，当事务回滚且聚合没有更新时，没有处理“假”事件的风险:

```java
@Service
public class DomainService {

    // ...
    @Transactional
    public void serviceDomainOperation(long entityId) {
        repository.findById(entityId)
            .ifPresent(entity -> {
                entity.domainOperation();
                repository.save(entity);
                eventPublisher.publishEvent(new DomainEvent());
            });
    }
}
```

这里有一个测试，证明事件确实是由服务`DomainOperation`发布的:

```java
@DisplayName("given existing aggregate,"
    + " when do domain operation on service,"
    + " then domain event is published")
@Test
void serviceEventsTest() {
    Aggregate existingDomainEntity = new Aggregate(1, eventPublisher);
    repository.save(existingDomainEntity);

    // when
    domainService.serviceDomainOperation(existingDomainEntity.getId());

    // then
    verify(eventHandler, times(1)).handleEvent(any(DomainEvent.class));
}
```

### 3.2.总计

**我们也可以直接从聚合**中发布事件。

通过这种方式，我们可以在类内部管理域事件的创建，这样感觉更自然:

```java
@Entity
class Aggregate {
    // ...
    void domainOperation() {
        // some business logic
        if (eventPublisher != null) {
            eventPublisher.publishEvent(new DomainEvent());
        }
    }
}
```

不幸的是，由于 Spring Data 从存储库中初始化实体的方式，这可能不会像预期的那样工作。

下面是显示真实行为的相应测试:

```java
@DisplayName("given existing aggregate,"
    + " when do domain operation directly on aggregate,"
    + " then domain event is NOT published")
@Test
void aggregateEventsTest() {
    Aggregate existingDomainEntity = new Aggregate(0, eventPublisher);
    repository.save(existingDomainEntity);

    // when
    repository.findById(existingDomainEntity.getId())
      .get()
      .domainOperation();

    // then
    verifyNoInteractions(eventHandler);
}
```

如我们所见，该事件根本没有发布。在聚合中包含依赖关系可能不是一个好主意。在本例中，`ApplicationEventPublisher`不是由 Spring 数据自动初始化的。

聚合是通过调用默认构造函数来构造的。为了让它像我们预期的那样运行，我们需要手动重新创建实体(例如，使用定制工厂或方面编程)。

此外，我们应该避免在 aggregate 方法完成后立即发布事件。至少，除非我们 100%确定这个方法是交易的一部分。否则，我们可能会在变更尚未持续时发布“虚假”事件。这可能会导致系统不一致。

如果我们想避免这种情况，我们必须记住总是在事务内部调用聚合方法。不幸的是，这种方式将我们的设计与持久性技术严重耦合。我们需要记住，我们并不总是使用事务性系统。

因此，让我们的聚合简单地管理一组域事件，并在它们即将被持久化时返回它们，通常是一个更好的主意。

在下一节中，我们将解释如何通过使用@DomainEvents 和`@AfterDomainEvents`注释使域事件的发布更易于管理。

## 4.使用`@DomainEvents`发布事件

**由于 Spring Data Ingalls 发布系列，我们可以使用`@DomainEvents`注释来自动发布领域事件**。

每当使用正确的存储库保存实体时，Spring Data 都会自动调用用`@DomainEvents`标注的方法。

然后，使用`ApplicationEventPublisher`接口发布该方法返回的事件:

```java
@Entity
public class Aggregate2 {

    @Transient
    private final Collection<DomainEvent> domainEvents;
    // ...
    public void domainOperation() {
        // some domain operation
        domainEvents.add(new DomainEvent());
    }

    @DomainEvents
    public Collection<DomainEvent> events() {
        return domainEvents;
    }
}
```

以下是解释这种行为的示例:

```java
@DisplayName("given aggregate with @DomainEvents,"
    + " when do domain operation and save,"
    + " then event is published")
@Test
void domainEvents() {

    // given
    Aggregate2 aggregate = new Aggregate2();

    // when
    aggregate.domainOperation();
    repository.save(aggregate);

    // then
    verify(eventHandler, times(1)).handleEvent(any(DomainEvent.class));
}
```

发布域事件后，调用用`@AfterDomainEventsPublication`标注的方法。

此方法的目的通常是清除所有事件的列表，以便它们在将来不会再次发布:

```java
@AfterDomainEventPublication
public void clearEvents() {
    domainEvents.clear();
}
```

让我们将这个方法添加到`Aggregate2`类中，看看它是如何工作的:

```java
@DisplayName("given aggregate with @AfterDomainEventPublication,"
    + " when do domain operation and save twice,"
    + " then an event is published only for the first time")
@Test
void afterDomainEvents() {

    // given
    Aggregate2 aggregate = new Aggregate2();

    // when
    aggregate.domainOperation();
    repository.save(aggregate);
    repository.save(aggregate);

    // then
    verify(eventHandler, times(1)).handleEvent(any(DomainEvent.class));
}
```

我们清楚地看到，这个事件只是第一次发表。**如果我们从`clearEvents`方法中移除了`@AfterDomainEventPublication`注释，那么同样的事件将会被第二次发布**。

然而，实际会发生什么取决于实现者。Spring 只保证调用这个方法——仅此而已。

## 5.使用 AbstractAggregateRoot 模板

**由于有了`AbstractAggregateRoot`模板类**，可以进一步简化领域事件的发布。当我们想要将新的域事件添加到事件集合中时，我们所要做的就是调用`register`方法:

```java
@Entity
public class Aggregate3 extends AbstractAggregateRoot<Aggregate3> {
    // ...
    public void domainOperation() {
        // some domain operation
        registerEvent(new DomainEvent());
    }
}
```

这与上一节中显示的示例相对应。

为了确保一切按预期运行，以下是测试:

```java
@DisplayName("given aggregate extending AbstractAggregateRoot,"
    + " when do domain operation and save twice,"
    + " then an event is published only for the first time")
@Test
void afterDomainEvents() {

    // given
    Aggregate3 aggregate = new Aggregate3();

    // when
    aggregate.domainOperation();
    repository.save(aggregate);
    repository.save(aggregate);

    // then
    verify(eventHandler, times(1)).handleEvent(any(DomainEvent.class));
}

@DisplayName("given aggregate extending AbstractAggregateRoot,"
    + " when do domain operation and save,"
    + " then an event is published")
@Test
void domainEvents() {
    // given
    Aggregate3 aggregate = new Aggregate3();

    // when
    aggregate.domainOperation();
    repository.save(aggregate);

    // then
    verify(eventHandler, times(1)).handleEvent(any(DomainEvent.class));
}
```

正如我们所看到的，我们可以产生更少的代码，并达到完全相同的效果。

## 6。实施注意事项

虽然一开始使用`@DomainEvents`功能看起来是个好主意，但是我们需要注意一些陷阱。

### 6.1.未发布的事件

当使用 JPA 时，当我们想要持久保存更改时，我们不一定要调用 save 方法。

如果我们的代码是事务的一部分(例如，用`@Transactional`注释)并对现有实体进行了更改，那么我们通常只是让事务提交，而不显式调用存储库的`save`方法。因此，即使我们的聚合产生了新的领域事件，它们也永远不会被发布。

**我们还需要记住的是`@DomainEvents`特性只有在使用 Spring 数据仓库**时才有效。这可能是一个重要的设计因素。

### 6.2.丢失的事件

**如果在事件发布期间发生异常，侦听器将永远不会得到通知**。

即使我们能以某种方式保证事件监听器的通知，目前也没有反压力让发布者知道出了问题。如果事件侦听器被异常中断，事件将保持未使用状态，并且永远不会再次发布。

Spring dev 团队知道这个设计缺陷。一位首席开发人员甚至提出了这个问题的可能解决方案。

### 6.3.当地环境

使用简单的`ApplicationEventPublisher`接口发布领域事件。

**默认情况下，当使用`ApplicationEventPublisher`时，事件在同一个线程**中发布和消费。一切都发生在同一个容器里。

通常，我们希望通过某种消息代理发送事件，这样其他分布式客户端/系统就会得到通知。在这种情况下，我们需要手动将事件转发给消息代理。

也可以使用 [Spring Integration](https://web.archive.org/web/20220712173816/https://spring.io/projects/spring-integration) 或者第三方解决方案，比如 [Apache Camel](https://web.archive.org/web/20220712173816/https://camel.apache.org/spring-event.html) 。

## 7。结论

在本文中，我们学习了如何使用`@DomainEvents`注释管理聚合域事件。

**这种方法可以极大地简化事件基础设施，因此我们可以只关注领域逻辑**。我们只需要知道没有灵丹妙药，Spring 处理域事件的方式也不例外。

GitHub 上的[提供了所有示例的完整源代码。](https://web.archive.org/web/20220712173816/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-annotations)