# DDD 有界上下文和 Java 模块

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-modules-ddd-bounded-contexts>

## 1.概观

**领域驱动设计(DDD)是一套原则和工具，帮助我们设计有效的软件架构来交付更高的商业价值**。有界上下文是通过将整个应用程序域分离成多个语义一致的部分来将架构从泥淖中拯救出来的核心和基本模式之一。

同时，利用 [Java 9 模块系统](/web/20220628162506/https://www.baeldung.com/java-9-modularity)，我们可以创建强封装的模块。

在本教程中，我们将创建一个简单的商店应用程序，并了解如何在为有界上下文定义显式边界的同时利用 Java 9 模块。

## 2.DDD 有界背景

现在的软件系统已经不是简单的 [CRUD 应用](/web/20220628162506/https://www.baeldung.com/spring-boot-crud-thymeleaf)。实际上，典型的单体企业系统由一些遗留代码库和新添加的特性组成。然而，随着每次改变，维护这样的系统变得越来越困难。最终，它可能变得完全不可维护。

### 2.1.有界语境与无处不在的语言

为了解决这个问题，DDD 提出了有界语境的概念。**有界上下文是特定术语和规则一致适用的领域的逻辑边界**。在这个界限内，**所有的术语、定义和概念形成了无处不在的语言。**

特别是，无处不在的语言的主要好处是将特定业务领域不同领域的项目成员组织在一起。

此外，多个上下文可能处理同一件事情。然而，它在每种上下文中可能有不同的含义。

[![](img/078def9ff408346fd9cb016033a1792f.png)](/web/20220628162506/https://www.baeldung.com/wp-content/uploads/2020/03/General-Bounded-Context.svg)

### 2.2.订单上下文

让我们通过定义订单上下文来开始实现我们的应用程序。这个上下文包含两个实体:`OrderItem`和`CustomerOrder`。

[![](img/808256e4d6529bc9816c6db064ea7229.png)](/web/20220628162506/https://www.baeldung.com/wp-content/uploads/2020/03/OrderContext.svg)
`CustomerOrder`实体是一个[聚合根](/web/20220628162506/https://www.baeldung.com/spring-persisting-ddd-aggregates):

```
public class CustomerOrder {
    private int orderId;
    private String paymentMethod;
    private String address;
    private List<OrderItem> orderItems;

    public float calculateTotalPrice() {
        return orderItems.stream().map(OrderItem::getTotalPrice)
          .reduce(0F, Float::sum);
    }
}
```

正如我们所看到的，这个类包含了`calculateTotalPrice`业务方法。但是，在现实世界的项目中，它可能要复杂得多——例如，在最终价格中包括折扣和税收。

接下来，让我们创建`OrderItem`类:

```
public class OrderItem {
    private int productId;
    private int quantity;
    private float unitPrice;
    private float unitWeight;
}
```

我们已经定义了实体，但是我们还需要向应用程序的其他部分公开一些 API。让我们创建`CustomerOrderService`类:

```
public class CustomerOrderService implements OrderService {
    public static final String EVENT_ORDER_READY_FOR_SHIPMENT = "OrderReadyForShipmentEvent";

    private CustomerOrderRepository orderRepository;
    private EventBus eventBus;

    @Override
    public void placeOrder(CustomerOrder order) {
        this.orderRepository.saveCustomerOrder(order);
        Map<String, String> payload = new HashMap<>();
        payload.put("order_id", String.valueOf(order.getOrderId()));
        ApplicationEvent event = new ApplicationEvent(payload) {
            @Override
            public String getType() {
                return EVENT_ORDER_READY_FOR_SHIPMENT;
            }
        };
        this.eventBus.publish(event);
    }
}
```

这里，我们要强调一些要点。`placeOrder`方法负责处理客户订单。**订单处理后，事件发布到`EventBus`** 。我们将在下一章讨论事件驱动的通信。该服务提供了`OrderService`接口的默认实现:

```
public interface OrderService extends ApplicationService {
    void placeOrder(CustomerOrder order);

    void setOrderRepository(CustomerOrderRepository orderRepository);
}
```

此外，该服务要求`CustomerOrderRepository`保存订单:

```
public interface CustomerOrderRepository {
    void saveCustomerOrder(CustomerOrder order);
}
```

重要的是**这个接口不是在这个上下文中实现的，而是由基础设施模块**提供的，我们将在后面看到。

### 2.3.运输环境

现在，让我们定义装运上下文。它也很简单，包含三个实体:`Parcel`、`PackageItem`和`ShippableOrder`。

[![](img/3ad640780fee230db75a801650a86ec5.png)](/web/20220628162506/https://www.baeldung.com/wp-content/uploads/2020/03/ShippingContext.svg)

让我们从`ShippableOrder`实体开始:

```
public class ShippableOrder {
    private int orderId;
    private String address;
    private List<PackageItem> packageItems;
}
```

在这种情况下，实体不包含`paymentMethod`字段。这是因为，在我们的运输环境中，我们不关心使用哪种付款方式。发货上下文只负责处理订单的发货。

另外，`Parcel`实体特定于运输上下文:

```
public class Parcel {
    private int orderId;
    private String address;
    private String trackingId;
    private List<PackageItem> packageItems;

    public float calculateTotalWeight() {
        return packageItems.stream().map(PackageItem::getWeight)
          .reduce(0F, Float::sum);
    }

    public boolean isTaxable() {
        return calculateEstimatedValue() > 100;
    }

    public float calculateEstimatedValue() {
        return packageItems.stream().map(PackageItem::getWeight)
          .reduce(0F, Float::sum);
    }
}
```

正如我们所看到的，它还包含特定的业务方法，并充当聚合根。

最后，我们来定义一下`ParcelShippingService`:

```
public class ParcelShippingService implements ShippingService {
    public static final String EVENT_ORDER_READY_FOR_SHIPMENT = "OrderReadyForShipmentEvent";
    private ShippingOrderRepository orderRepository;
    private EventBus eventBus;
    private Map<Integer, Parcel> shippedParcels = new HashMap<>();

    @Override
    public void shipOrder(int orderId) {
        Optional<ShippableOrder> order = this.orderRepository.findShippableOrder(orderId);
        order.ifPresent(completedOrder -> {
            Parcel parcel = new Parcel(completedOrder.getOrderId(), completedOrder.getAddress(), 
              completedOrder.getPackageItems());
            if (parcel.isTaxable()) {
                // Calculate additional taxes
            }
            // Ship parcel
            this.shippedParcels.put(completedOrder.getOrderId(), parcel);
        });
    }

    @Override
    public void listenToOrderEvents() {
        this.eventBus.subscribe(EVENT_ORDER_READY_FOR_SHIPMENT, new EventSubscriber() {
            @Override
            public <E extends ApplicationEvent> void onEvent(E event) {
                shipOrder(Integer.parseInt(event.getPayloadValue("order_id")));
            }
        });
    }

    @Override
    public Optional<Parcel> getParcelByOrderId(int orderId) {
        return Optional.ofNullable(this.shippedParcels.get(orderId));
    }
}
```

这个服务类似地使用`ShippingOrderRepository`通过 id 获取订单。**更重要的是，它订阅了由另一个上下文发布的`OrderReadyForShipmentEvent`事件。**当这个事件发生时，服务应用一些规则并发送订单。为了简单起见，我们将订单储存在 [`HashMap`](/web/20220628162506/https://www.baeldung.com/java-hashmap) 中。

## 3.上下文地图

到目前为止，我们定义了两个上下文。但是，我们没有在它们之间设置任何显式的关系。为此，DDD 提出了语境映射的概念。**上下文图是系统**不同上下文之间关系的可视化描述。这张地图显示了不同的部分是如何共存形成这个领域的。

有界上下文之间有五种主要类型的关系:

*   `Partnership`–两个环境之间的关系，这两个环境合作将两个团队与相关的目标联系起来
*   `Shared Kernel`–一种关系，将几个上下文的公共部分提取到另一个上下文/模块中，以减少代码重复
*   `Customer-supplier`–两个上下文之间的连接，其中一个上下文(上游)产生数据，另一个上下文(下游)消费数据。在这种关系中，双方都希望建立尽可能好的沟通
*   `Conformist`–这种关系也有上游和下游，但是，下游总是符合上游的 API
*   这种类型的关系广泛用于遗留系统，使它们适应新的架构，并逐渐从遗留代码库迁移出来。反腐败层充当一个[适配器](/web/20220628162506/https://www.baeldung.com/hexagonal-architecture-ddd-spring)来转换来自上游的数据，并防止不希望的变化

在我们的特定例子中，我们将使用共享内核关系。我们不会定义它的纯形式，但它将主要作为系统中事件的调解人。

因此，SharedKernel 模块不包含任何具体的实现，只包含接口。

让我们从`EventBus`界面开始:

```
public interface EventBus {
    <E extends ApplicationEvent> void publish(E event);

    <E extends ApplicationEvent> void subscribe(String eventType, EventSubscriber subscriber);

    <E extends ApplicationEvent> void unsubscribe(String eventType, EventSubscriber subscriber);
}
```

该接口将在稍后的基础设施模块中实现。

接下来，我们用默认方法创建一个基本服务接口来支持事件驱动的通信:

```
public interface ApplicationService {

    default <E extends ApplicationEvent> void publishEvent(E event) {
        EventBus eventBus = getEventBus();
        if (eventBus != null) {
            eventBus.publish(event);
        }
    }

    default <E extends ApplicationEvent> void subscribe(String eventType, EventSubscriber subscriber) {
        EventBus eventBus = getEventBus();
        if (eventBus != null) {
            eventBus.subscribe(eventType, subscriber);
        }
    }

    default <E extends ApplicationEvent> void unsubscribe(String eventType, EventSubscriber subscriber) {
        EventBus eventBus = getEventBus();
        if (eventBus != null) {
            eventBus.unsubscribe(eventType, subscriber);
        }
    }

    EventBus getEventBus();

    void setEventBus(EventBus eventBus);
}
```

因此，有界上下文中的服务接口扩展了这个接口，使其具有公共的事件相关功能。

## 4.Java 9 模块化

现在，是时候探索 Java 9 模块系统如何支持定义的应用程序结构了。

Java 平台模块系统(JPMS)鼓励构建更可靠、封装更强的模块。因此，这些特征有助于隔离我们的上下文并建立清晰的界限。

让我们看看最终的模块图:

[![](img/227d66f6670fb4b12a43b611bbcb7657.png)](/web/20220628162506/https://www.baeldung.com/wp-content/uploads/2020/03/ModulesDiagram.svg)

### 4.1.共享内核模块

让我们从 SharedKernel 模块开始，它对其他模块没有任何依赖性。所以，`module-info.java`看起来像是:

```
module com.baeldung.dddmodules.sharedkernel {
    exports com.baeldung.dddmodules.sharedkernel.events;
    exports com.baeldung.dddmodules.sharedkernel.service;
}
```

我们导出模块接口，因此它们可用于其他模块。

### 4.2.`OrderContext`模块

接下来，让我们将焦点转移到 OrderContext 模块。它只需要 SharedKernel 模块中定义的接口:

```
module com.baeldung.dddmodules.ordercontext {
    requires com.baeldung.dddmodules.sharedkernel;
    exports com.baeldung.dddmodules.ordercontext.service;
    exports com.baeldung.dddmodules.ordercontext.model;
    exports com.baeldung.dddmodules.ordercontext.repository;
    provides com.baeldung.dddmodules.ordercontext.service.OrderService
      with com.baeldung.dddmodules.ordercontext.service.CustomerOrderService;
}
```

同样，我们可以看到这个模块导出了`OrderService`接口的默认实现。

### 4.3.`ShippingContext`模块

与前面的模块类似，让我们创建 ShippingContext 模块定义文件:

```
module com.baeldung.dddmodules.shippingcontext {
    requires com.baeldung.dddmodules.sharedkernel;
    exports com.baeldung.dddmodules.shippingcontext.service;
    exports com.baeldung.dddmodules.shippingcontext.model;
    exports com.baeldung.dddmodules.shippingcontext.repository;
    provides com.baeldung.dddmodules.shippingcontext.service.ShippingService
      with com.baeldung.dddmodules.shippingcontext.service.ParcelShippingService;
}
```

同样，我们导出了`ShippingService`接口的默认实现。

### 4.4.基础设施模块

现在是描述基础设施模块的时候了。该模块包含已定义接口的实现细节。我们将从为`EventBus`接口创建一个简单的实现开始:

```
public class SimpleEventBus implements EventBus {
    private final Map<String, Set<EventSubscriber>> subscribers = new ConcurrentHashMap<>();

    @Override
    public <E extends ApplicationEvent> void publish(E event) {
        if (subscribers.containsKey(event.getType())) {
            subscribers.get(event.getType())
              .forEach(subscriber -> subscriber.onEvent(event));
        }
    }

    @Override
    public <E extends ApplicationEvent> void subscribe(String eventType, EventSubscriber subscriber) {
        Set<EventSubscriber> eventSubscribers = subscribers.get(eventType);
        if (eventSubscribers == null) {
            eventSubscribers = new CopyOnWriteArraySet<>();
            subscribers.put(eventType, eventSubscribers);
        }
        eventSubscribers.add(subscriber);
    }

    @Override
    public <E extends ApplicationEvent> void unsubscribe(String eventType, EventSubscriber subscriber) {
        if (subscribers.containsKey(eventType)) {
            subscribers.get(eventType).remove(subscriber);
        }
    }
}
```

接下来，我们需要实现`CustomerOrderRepository`和`ShippingOrderRepository`接口。**在大多数情况下，`Order`实体将被存储在同一个表中，但在有限的上下文中用作不同的实体模型。**

常见的情况是，单个实体包含来自业务领域不同领域或低级数据库映射的混合代码。对于我们的实现，我们已经根据有界上下文分割了我们的实体:`CustomerOrder`和`ShippableOrder`。

首先，让我们创建一个代表整个持久模型的类:

```
public static class PersistenceOrder {
    public int orderId;
    public String paymentMethod;
    public String address;
    public List<OrderItem> orderItems;

    public static class OrderItem {
        public int productId;
        public float unitPrice;
        public float itemWeight;
        public int quantity;
    }
}
```

我们可以看到这个类包含了来自`CustomerOrder`和`ShippableOrder`实体的所有字段。

为了简单起见，让我们模拟一个内存数据库:

```
public class InMemoryOrderStore implements CustomerOrderRepository, ShippingOrderRepository {
    private Map<Integer, PersistenceOrder> ordersDb = new HashMap<>();

    @Override
    public void saveCustomerOrder(CustomerOrder order) {
        this.ordersDb.put(order.getOrderId(), new PersistenceOrder(order.getOrderId(),
          order.getPaymentMethod(),
          order.getAddress(),
          order
            .getOrderItems()
            .stream()
            .map(orderItem ->
              new PersistenceOrder.OrderItem(orderItem.getProductId(),
                orderItem.getQuantity(),
                orderItem.getUnitWeight(),
                orderItem.getUnitPrice()))
            .collect(Collectors.toList())
        ));
    }

    @Override
    public Optional<ShippableOrder> findShippableOrder(int orderId) {
        if (!this.ordersDb.containsKey(orderId)) return Optional.empty();
        PersistenceOrder orderRecord = this.ordersDb.get(orderId);
        return Optional.of(
          new ShippableOrder(orderRecord.orderId, orderRecord.orderItems
            .stream().map(orderItem -> new PackageItem(orderItem.productId,
              orderItem.itemWeight,
              orderItem.quantity * orderItem.unitPrice)
            ).collect(Collectors.toList())));
    }
}
```

这里，我们通过将持久模型转换为适当的类型或从适当的类型转换来持久化和检索不同类型的实体。

最后，让我们创建模块定义:

```
module com.baeldung.dddmodules.infrastructure {
    requires transitive com.baeldung.dddmodules.sharedkernel;
    requires transitive com.baeldung.dddmodules.ordercontext;
    requires transitive com.baeldung.dddmodules.shippingcontext;
    provides com.baeldung.dddmodules.sharedkernel.events.EventBus
      with com.baeldung.dddmodules.infrastructure.events.SimpleEventBus;
    provides com.baeldung.dddmodules.ordercontext.repository.CustomerOrderRepository
      with com.baeldung.dddmodules.infrastructure.db.InMemoryOrderStore;
    provides com.baeldung.dddmodules.shippingcontext.repository.ShippingOrderRepository
      with com.baeldung.dddmodules.infrastructure.db.InMemoryOrderStore;
}
```

使用`provides with`子句，我们提供了在其他模块中定义的一些接口的实现。

此外，这个模块充当依赖项的聚合器，所以我们使用了`requires transitive`关键字。**因此，需要基础设施模块的模块将会获得所有这些依赖项。**

### 4.5.主模块

最后，让我们定义一个模块作为应用程序的入口点:

```
module com.baeldung.dddmodules.mainapp {
    uses com.baeldung.dddmodules.sharedkernel.events.EventBus;
    uses com.baeldung.dddmodules.ordercontext.service.OrderService;
    uses com.baeldung.dddmodules.ordercontext.repository.CustomerOrderRepository;
    uses com.baeldung.dddmodules.shippingcontext.repository.ShippingOrderRepository;
    uses com.baeldung.dddmodules.shippingcontext.service.ShippingService;
    requires transitive com.baeldung.dddmodules.infrastructure;
}
```

由于我们刚刚在基础结构模块上设置了可传递的依赖关系，所以我们不需要在这里显式地要求它们。

另一方面，我们用关键字`uses`列出了这些依赖关系。`uses`子句指示`ServiceLoader`，我们将在下一章中发现，这个模块想要使用这些接口。然而，**它不要求实现在编译时可用。**

## 5.运行应用程序

最后，我们几乎准备好构建我们的应用程序了。我们将利用 [Maven](/web/20220628162506/https://www.baeldung.com/maven) 来构建我们的项目。这使得使用模块更加容易。

### 5.1.项目结构

我们的项目包含[五个模块和父模块](/web/20220628162506/https://www.baeldung.com/maven-multi-module-project-java-jpms)。让我们看看我们的项目结构:

```
ddd-modules (the root directory)
pom.xml
|-- infrastructure
    |-- src
        |-- main
            | -- java
            module-info.java
            |-- com.baeldung.dddmodules.infrastructure
    pom.xml
|-- mainapp
    |-- src
        |-- main
            | -- java
            module-info.java
            |-- com.baeldung.dddmodules.mainapp
    pom.xml
|-- ordercontext
    |-- src
        |-- main
            | -- java
            module-info.java
            |--com.baeldung.dddmodules.ordercontext
    pom.xml
|-- sharedkernel
    |-- src
        |-- main
            | -- java
            module-info.java
            |-- com.baeldung.dddmodules.sharedkernel
    pom.xml
|-- shippingcontext
    |-- src
        |-- main
            | -- java
            module-info.java
            |-- com.baeldung.dddmodules.shippingcontext
    pom.xml
```

### 5.2.主要应用

现在，除了主应用程序之外，我们已经拥有了一切，所以让我们定义我们的`main`方法:

```
public static void main(String args[]) {
    Map<Class<?>, Object> container = createContainer();
    OrderService orderService = (OrderService) container.get(OrderService.class);
    ShippingService shippingService = (ShippingService) container.get(ShippingService.class);
    shippingService.listenToOrderEvents();

    CustomerOrder customerOrder = new CustomerOrder();
    int orderId = 1;
    customerOrder.setOrderId(orderId);
    List<OrderItem> orderItems = new ArrayList<OrderItem>();
    orderItems.add(new OrderItem(1, 2, 3, 1));
    orderItems.add(new OrderItem(2, 1, 1, 1));
    orderItems.add(new OrderItem(3, 4, 11, 21));
    customerOrder.setOrderItems(orderItems);
    customerOrder.setPaymentMethod("PayPal");
    customerOrder.setAddress("Full address here");
    orderService.placeOrder(customerOrder);

    if (orderId == shippingService.getParcelByOrderId(orderId).get().getOrderId()) {
        System.out.println("Order has been processed and shipped successfully");
    }
}
```

下面简单讨论一下我们的主要方法。在这个方法中，我们通过使用先前定义的服务来模拟一个简单的客户订单流。首先，我们创建了包含三个项目的订单，并提供了必要的运输和支付信息。接下来，我们提交了订单，并最终检查它是否已发货并处理成功。

但是我们如何获得所有的依赖关系，为什么`createContainer`方法返回`Map<Class<?>,`对象>？让我们仔细看看这个方法。

### 5.3.使用 ServiceLoader 进行依赖注入

在这个项目中，我们没有任何 [Spring IoC](/web/20220628162506/https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring) 依赖项，所以我们将使用 [`ServiceLoader` API](/web/20220628162506/https://www.baeldung.com/java-spi#4-serviceloader) 来发现服务的实现。这不是一个新特性——`ServiceLoader`API 本身从 Java 6 开始就已经存在了。

我们可以通过调用`ServiceLoader`类的一个静态`load`方法来获得一个加载器实例。**`load`方法返回`Iterable`类型，这样我们可以迭代发现的实现。**

现在，让我们应用加载程序来解决我们的依赖性:

```
public static Map<Class<?>, Object> createContainer() {
    EventBus eventBus = ServiceLoader.load(EventBus.class).findFirst().get();

    CustomerOrderRepository customerOrderRepository = ServiceLoader.load(CustomerOrderRepository.class)
      .findFirst().get();
    ShippingOrderRepository shippingOrderRepository = ServiceLoader.load(ShippingOrderRepository.class)
      .findFirst().get();

    ShippingService shippingService = ServiceLoader.load(ShippingService.class).findFirst().get();
    shippingService.setEventBus(eventBus);
    shippingService.setOrderRepository(shippingOrderRepository);
    OrderService orderService = ServiceLoader.load(OrderService.class).findFirst().get();
    orderService.setEventBus(eventBus);
    orderService.setOrderRepository(customerOrderRepository);

    HashMap<Class<?>, Object> container = new HashMap<>();
    container.put(OrderService.class, orderService);
    container.put(ShippingService.class, shippingService);

    return container;
}
```

这里，**我们为我们需要的每个接口调用静态的`load`方法，每次都创建一个新的加载器实例。**因此，它不会缓存已经解决的依赖关系——相反，它每次都会创建新的实例。

通常，服务实例可以通过两种方式之一创建。服务实现类必须有一个公共的无参数构造函数，或者它必须使用一个静态的`provider`方法。

因此，我们的大多数服务都没有无参数的构造函数和依赖关系的 setter 方法。但是，正如我们已经看到的，`InMemoryOrderStore`类实现了两个接口:`CustomerOrderRepository`和`ShippingOrderRepository`。

然而，如果我们使用`load`方法请求这些接口中的每一个，我们将得到`InMemoryOrderStore`的不同实例。这不是我们想要的行为，所以让我们使用`provider`方法技术来缓存实例:

```
public class InMemoryOrderStore implements CustomerOrderRepository, ShippingOrderRepository {
    private volatile static InMemoryOrderStore instance = new InMemoryOrderStore();

    public static InMemoryOrderStore provider() {
        return instance;
    }
}
```

我们已经应用了[单例模式](/web/20220628162506/https://www.baeldung.com/java-singleton)来缓存`InMemoryOrderStore`类的单个实例，并从`provider`方法返回它。

如果服务提供者声明了一个`provider`方法，那么`ServiceLoader`调用这个方法来获得一个服务的实例。否则，它将尝试通过[反射](/web/20220628162506/https://www.baeldung.com/java-reflection)使用无参数构造函数创建一个实例。因此，我们可以在不影响我们的`createContainer`方法的情况下改变服务提供者机制。

最后，我们通过 setters 向服务提供已解析的依赖关系，并返回已配置的服务。

最后，我们可以运行应用程序。

## 6.结论

在本文中，我们讨论了一些关键的 DDD 概念:有界上下文、无处不在的语言和上下文映射。虽然将一个系统划分为有界的上下文有很多好处，但同时，没有必要在所有地方都应用这种方法。

接下来，我们看到了如何使用 Java 9 模块系统和有界上下文来创建强封装的模块。

此外，我们已经介绍了发现依赖关系的默认`ServiceLoader`机制。

该项目的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220628162506/https://github.com/eugenp/tutorials/tree/master/ddd-contexts)