# 带有 Spring 数据的 MongoDB 组合键

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-mongodb-composite-key>

## 1.介绍

在本教程中，我们将在 Spring Data MongoDB 应用程序中创建组合键。我们将了解不同的策略以及如何配置它们。

## 2.什么是组合键以及何时使用它

组合键是文档中唯一标识它的属性的组合。使用复合主键并不比使用单个自动生成的属性更好或者更差。我们甚至可以将这些方法与独特的指数结合起来。

通常，没有单一的属性能够唯一地标识文档。在这种情况下，我们可以将其留空， [MongoDB 将为其“`_id`”属性生成一个唯一值](/web/20221128040535/https://www.baeldung.com/spring-boot-mongodb-auto-generated-field)。或者，我们可以选择多个属性，当它们组合在一起时，服务于那个目的。**在这种情况下，我们必须为 ID 属性创建一个自定义类来保存所有这些属性。**让我们看看这是如何工作的。

## 3.使用`@Id`注释创建一个组合键

`@Id`注释可以用来注释定制类型的属性，完全控制它的生成。对我们的 ID 类的唯一要求是我们覆盖了 [`equals()`和`hashCode()`](/web/20221128040535/https://www.baeldung.com/java-equals-hashcode-contracts) ，并且有一个默认的[无参数构造函数](/web/20221128040535/https://www.baeldung.com/java-constructors#noargs)。

在我们的第一个例子中，我们将为活动门票创建一个文档。它的 ID 将是`venue`和`date`属性的组合。让我们从我们的 ID 类开始:

```
public class TicketId {
    private String venue;
    private String date;

    // getters and setters

    // override hashCode() and equals()
}
```

由于无参数构造函数是隐式的，我们不需要其他构造函数，所以我们不需要编写它。此外，我们将使用`String`日期来简化例子。接下来，让我们创建我们的`Ticket`类，并用`@Id`注释我们的`TicketId`属性:

```
@Document
public class Ticket {
    @Id
    private TicketId id;

    private String event;

    // getters and setters
}
```

**对于我们的`MongoRepository`，我们可以指定我们的`TicketId`作为 ID 类型，这就是所有需要的设置:**

```
public interface TicketRepository extends MongoRepository<Ticket, TicketId> {
} 
```

### 3.1.测试我们的模型

**因此，尝试两次插入具有相同 ID 的票，将会抛出一个`DuplicateKeyException`。**我们可以通过一个测试来验证这一点:

```
@Test
public void givenCompositeId_whenDupeInsert_thenExceptionIsThrown() {
    TicketId ticketId = new TicketId();
    ticketId.setDate("2020-01-01");
    ticketId.setVenue("V");

    Ticket ticket = new Ticket(ticketId, "Event C");
    service.insert(ticket);

    assertThrows(DuplicateKeyException.class, () -> {        
        service.insert(ticket);
    });
}
```

这确保了我们的钥匙是有效的。

### 3.2.按 ID 查找

因为我们将`TicketId`定义为存储库中的 ID 类，所以我们仍然可以使用默认的`findById()`方法。让我们编写一个测试来看看它的运行情况:

```
@Test
public void givenCompositeId_whenSearchingByIdObject_thenFound() {
    TicketId ticketId = new TicketId();
    ticketId.setDate("2020-01-01");
    ticketId.setVenue("Venue B");

    service.insert(new Ticket(ticketId, "Event B"));

    Optional<Ticket> optionalTicket = ticketRepository.findById(ticketId);

    assertThat(optionalTicket.isPresent());
    Ticket savedTicket = optionalTicket.get();

    assertEquals(savedTicket.getId(), ticketId);
}
```

当我们想要绝对控制我们的 ID 属性时，我们应该使用这种方法。类似地，这将确保我们的 ID 对象中的属性不能被修改。一个缺点是我们失去了 MongoDB 生成的 ID，可读性更差。但是，更容易在链接中使用，例如。

## 4.警告

当使用嵌套对象作为 ID 时，属性的顺序很重要。当使用我们的存储库时，这通常不是问题，因为 Java 对象总是以相同的顺序构造的。**但是，如果我们改变`TicketId`类中字段的顺序，我们可以插入另一个具有相同值的文档。**例如，这些物体被认为是不同的:

```
{
  "id": {
    "venue":"Venue A",
    "date": "2023-05-27"
  },
  "event": "Event 1"
}
```

之后，如果我们改变`TicketId`中的字段顺序，我们将能够插入相同的值。**不会抛出异常:**

```
{
  "id": {
    "date": "2023-05-27",
    "venue":"Venue A"
  },
  "event": "Event 1"
}
```

如果我们在我们的`Ticket`类的属性上使用唯一索引，而不是 ID 类，这就不会发生。换句话说，它只发生在嵌套的对象上。

## 5.结论

在本文中，我们看到了为 MongoDB 文档创建组合键的利弊。以及用一个简单的用例实现它们所需的配置。但是，我们也学到了一个重要的注意事项。

和往常一样，源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221128040535/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-mongodb-2)