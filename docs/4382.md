# Spock 框架中存根、模拟和间谍之间的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spock-stub-mock-spy>

## 1.概观

在本教程中，**我们将讨论在斯波克框架**中`Mock`、`Stub`和`Spy` 的区别。我们将说明该框架在基于交互的测试方面提供了什么。

`Spock`是一个针对`Java` 和`Groovy` 的测试框架，有助于自动化软件应用的手动测试过程。它引入了自己的 mocks、stubs 和 spies，并为通常需要额外库的测试提供了内置功能。

首先，我们将说明何时应该使用存根。然后，我们将经历嘲弄。最后，我们来描述一下最近推出的`Spy`。

## 2.Maven 依赖性

在我们开始之前，让我们添加我们的 [Maven 依赖项](https://web.archive.org/web/20220628054520/https://search.maven.org/classic/#search%7Cga%7C1%7C%20(g%3A%22org.spockframework%22%20AND%20a%3A%22spock-core%22)%20OR%20(g%3A%22org.codehaus.groovy%22%20AND%20a%3A%22groovy-all%22)):

```
<dependency>
    <groupId>org.spockframework</groupId>
    <artifactId>spock-core</artifactId>
    <version>1.3-RC1-groovy-2.5</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.codehaus.groovy</groupId>
    <artifactId>groovy-all</artifactId>
    <version>2.4.7</version>
    <scope>test</scope>
</dependency>
```

注意，我们需要 Spock 的 `1.3-RC1-groovy-2.5`版本。`Spy`将在 Spock 框架的下一个稳定版本中引入。**目前`Spy`在 1.3 版本的第一个候选版本中可用。**

要回顾 Spock 测试的基本结构，请查看我们关于用 Groovy 和 Spock 进行测试的介绍性文章。

## 3.基于交互的测试

基于交互的测试是一种帮助我们测试对象行为的技术——特别是它们如何相互交互。为此，我们可以使用称为 mocks 和 stubs 的虚拟实现。

当然，我们当然可以非常容易地编写我们自己的 mocks 和 stubs 实现。当我们的生产代码数量增加时，问题就出现了。手工编写和维护这些代码变得很困难。这就是为什么我们使用 mocking 框架，它提供了一种简洁的方式来简要描述预期的交互。Spock 内置了对嘲讽、存根和刺探的支持。

像大多数 Java 库一样，Spock 使用 [JDK 动态代理](/web/20220628054520/https://www.baeldung.com/java-dynamic-proxies)模仿接口，使用[字节伙伴](/web/20220628054520/https://www.baeldung.com/byte-buddy)或 [cglib](/web/20220628054520/https://www.baeldung.com/cglib) 代理模仿类。它在运行时创建模拟实现。

Java 已经有许多不同的、成熟的库来模仿类和接口。虽然这些都可以在 Spock 中使用，但是我们仍然有一个主要的原因为什么要使用 Spock 模仿，存根和间谍。通过将所有这些介绍给 Spock，**我们可以利用 Groovy 的所有功能**使我们的测试更具可读性，更容易编写，而且肯定更有趣！

## 4.存根方法调用

有时候，**在单元测试中，我们需要提供一个类**的虚拟行为。这可能是一个外部服务的客户端，或者是一个提供数据库访问的类。这种技术被称为存根。

在我们测试的代码中，存根是现有类依赖的可控替换。这对于进行以某种方式响应的方法调用很有用。当我们使用 stub 时，我们并不关心一个方法会被调用多少次。相反，我们只想说:用这个数据调用时返回这个值。

让我们转到具有业务逻辑的示例代码。

### 4.1.测试中的代码

让我们创建一个名为`Item`的模型类:

```
public class Item {
    private final String id;
    private final String name;

    // standard constructor, getters, equals
}
```

我们需要覆盖`equals(Object other)` 方法来使我们的断言工作。**当我们使用双等号(==):** 时，Spock 将在断言期间使用`equals`

```
new Item('1', 'name') == new Item('1', 'name')
```

现在，让我们用一个方法创建一个接口`ItemProvider`:

```
public interface ItemProvider {
    List<Item> getItems(List<String> itemIds);
}
```

我们还需要一个将被测试的类。我们将在`ItemService:`中添加一个`ItemProvider `作为依赖项

```
public class ItemService {
    private final ItemProvider itemProvider;

    public ItemService(ItemProvider itemProvider) {
        this.itemProvider = itemProvider;
    }

    List<Item> getAllItemsSortedByName(List<String> itemIds) {
        List<Item> items = itemProvider.getItems(itemIds);
        return items.stream()
          .sorted(Comparator.comparing(Item::getName))
          .collect(Collectors.toList());
    }

}
```

我们希望我们的代码依赖于一个抽象，而不是一个具体的实现。这就是我们使用接口的原因。这可以有许多不同的实现。例如，我们可以从文件中读取项目，创建外部服务的 HTTP 客户端，或者从数据库中读取数据。

在这段代码中，**我们需要清除外部依赖，因为我们只想测试包含在 `getAllItemsSortedByName`方法**中的逻辑。

### 4.2.在测试代码中使用存根对象

让我们使用`ItemProvider`依赖项的`Stub`来初始化`setup()`方法中的`ItemService`对象:

```
ItemProvider itemProvider
ItemService itemService

def setup() {
    itemProvider = Stub(ItemProvider)
    itemService = new ItemService(itemProvider)
}
```

现在，**让`itemProvider` 用特定的参数**在每次调用时返回一个条目列表:

```
itemProvider.getItems(['offer-id', 'offer-id-2']) >> 
  [new Item('offer-id-2', 'Zname'), new Item('offer-id', 'Aname')]
```

**我们使用> >操作数来存根方法。当用** `**[‘offer-id', ‘offer-id-2']**` **list 调用时，`getItems`方法将总是返回两个项目的列表。** `[] `是创建列表的`Groovy `快捷方式。

下面是整个测试方法:

```
def 'should return items sorted by name'() {
    given:
    def ids = ['offer-id', 'offer-id-2']
    itemProvider.getItems(ids) >> [new Item('offer-id-2', 'Zname'), new Item('offer-id', 'Aname')]

    when:
    List<Item> items = itemService.getAllItemsSortedByName(ids)

    then:
    items.collect { it.name } == ['Aname', 'Zname']
}
```

我们可以使用更多的存根功能，例如:使用参数匹配约束、在存根中使用值序列、在特定条件下定义不同的行为以及链接方法响应。

## 5.模仿类方法

现在，我们来谈谈 Spock 中的嘲讽类或接口。

有时，**我们想知道依赖对象的某个方法是否用指定的参数**调用。我们希望关注对象的行为，并通过查看方法调用来探索它们是如何交互的。嘲讽是对测试类中对象之间强制性交互的描述。

我们将在下面描述的示例代码中测试交互。

### 5.1.交互编码

举个简单的例子，我们将在数据库中保存条目。成功之后，我们希望在消息代理上发布一个关于系统中新项目的事件。

示例消息代理是 RabbitMQ 或 Kafka `, `所以一般来说，我们将只描述我们的契约:

```
public interface EventPublisher {
    void publish(String addedOfferId);
}
```

我们的测试方法将在数据库中保存非空项，然后发布事件。在我们的例子中，在数据库中保存条目是不相关的，所以我们只放一个注释:

```
void saveItems(List<String> itemIds) {
    List<String> notEmptyOfferIds = itemIds.stream()
      .filter(itemId -> !itemId.isEmpty())
      .collect(Collectors.toList());

    // save in database

    notEmptyOfferIds.forEach(eventPublisher::publish);
}
```

### 5.2.验证与模拟对象的交互

现在，让我们测试代码中的交互。

首先，**我们需要在我们的`setup() `方法**中模仿`EventPublisher `。所以基本上，我们创建一个新的实例字段，并使用`Mock(Class)`函数模拟它:

```
class ItemServiceTest extends Specification {

    ItemProvider itemProvider
    ItemService itemService
    EventPublisher eventPublisher

    def setup() {
        itemProvider = Stub(ItemProvider)
        eventPublisher = Mock(EventPublisher)
        itemService = new ItemService(itemProvider, eventPublisher)
}
```

现在，我们可以编写我们的测试方法了。我们将传递 3 个字符串: "、' a '、' b '，我们希望我们的`eventPublisher` 将发布 2 个带有' a '和' b '字符串的事件:

```
def 'should publish events about new non-empty saved offers'() {
    given:
    def offerIds = ['', 'a', 'b']

    when:
    itemService.saveItems(offerIds)

    then:
    1 * eventPublisher.publish('a')
    1 * eventPublisher.publish('b')
}
```

让我们仔细看看我们在最后`then` 部分的断言:

```
1 * eventPublisher.publish('a')
```

我们期望`itemService `将调用一个`eventPublisher.publish(String)` 并将‘a’作为参数。

在 stubbing 中，我们讨论了参数约束。同样的规则也适用于模拟。**我们可以验证 `eventPublisher.publish(String)` 被调用了两次，并且使用了任何非空的参数:**

```
2 * eventPublisher.publish({ it != null && !it.isEmpty() })
```

### 5.3.结合嘲笑和 Stubbing

在`Spock,` **中，一个`Mock`可能表现得和一个`Stub`一样。**所以我们可以对模仿对象说，对于给定的方法调用，它应该返回给定的数据。

让我们用`Mock(Class) `覆盖一个`ItemProvider` ，并创建一个新的`ItemService`:

```
given:
itemProvider = Mock(ItemProvider)
itemProvider.getItems(['item-id']) >> [new Item('item-id', 'name')]
itemService = new ItemService(itemProvider, eventPublisher)

when:
def items = itemService.getAllItemsSortedByName(['item-id'])

then:
items == [new Item('item-id', 'name')] 
```

我们可以从`given` 部分重写存根:

```
1 * itemProvider.getItems(['item-id']) >> [new Item('item-id', 'name')]
```

所以一般这一行说: **`itemProvider.getItems` 会用`[‘item-‘id']`实参调用一次，返回给定数组**。

我们已经知道模仿可以表现得和存根一样。所有关于参数约束、返回多个值和副作用的规则也适用于`Mock`。

## 6.斯波克的间谍课程

**间谍提供包装现有物品的能力。**这意味着我们可以监听调用者和真实对象之间的对话，但保留原始的对象行为。基本上， **`Spy `将方法调用委托给原始对象**。

与`Mock `和`Stub`相反，我们不能在接口上创建`Spy `。它包装了一个实际的对象，所以另外，我们需要为构造函数传递参数。否则，将调用该类型的默认构造函数。

### 6.1.测试中的代码

让我们为`EventPublisher. LoggingEventPublisher `创建一个简单的实现，它将在控制台中打印每个添加项目的 id。下面是接口方法的实现:

```
@Override
public void publish(String addedOfferId) {
    System.out.println("I've published: " + addedOfferId);
}
```

### 6.2.使用`Spy`进行测试

**我们用`Spy(Class)`的方法创造了类似于模仿和树桩的间谍。** `LoggingEventPublisher`没有任何其他的类依赖，所以我们不必传递构造函数参数:

```
eventPublisher = Spy(LoggingEventPublisher)
```

现在，让我们测试一下我们的间谍。我们需要一个新的`ItemService `实例，它包含我们的被监视对象:

```
given:
eventPublisher = Spy(LoggingEventPublisher)
itemService = new ItemService(itemProvider, eventPublisher)

when:
itemService.saveItems(['item-id'])

then:
1 * eventPublisher.publish('item-id')
```

我们验证了`eventPublisher.publish `方法只被调用了一次。**另外，方法调用被传递给了真实对象，所以我们将在控制台中看到`println` 的输出:**

```
I've published: item-id
```

注意，当我们在一个`Spy`的方法上使用 stub 时，那么它不会调用真正的对象方法。**一般要避免使用间谍。**如果我们不得不这样做，也许我们应该重新安排规范下的代码？

## 7.好的单元测试

让我们快速总结一下使用模拟对象是如何改进我们的测试的:

*   我们创建确定性测试套件
*   我们不会有任何副作用
*   我们的单元测试将会非常快
*   我们可以专注于单个 Java 类中包含的逻辑
*   我们的测试不受环境影响

## 8.结论

在这篇文章中，我们详细描述了 Groovy `.` **中的间谍、模仿和存根，这方面的知识将使我们的测试更快、更可靠、更易读。**

我们所有例子的实现都可以在 Github 项目中找到。