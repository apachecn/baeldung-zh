# 春季帽子介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-hateoas-tutorial>

## 1。概述

本文解释了使用 Spring HATEOAS 项目创建超媒体驱动的 REST web 服务的过程。

## 2。弹簧帽

Spring HATEOAS 项目是一个 API 库，我们可以使用它来轻松地创建遵循 HATEOAS(超文本作为应用程序状态的引擎)原则的 REST 表示。

一般来说，这个原则意味着 API 应该通过返回关于下一个潜在步骤的相关信息以及每个响应来指导客户完成应用程序。

在本文中，我们将使用 Spring HATEOAS 构建一个示例，目标是分离客户端和服务器，理论上允许 API 在不破坏客户端的情况下更改其 URI 方案。

## 3。准备工作

首先，让我们添加 Spring HATEOAS 依赖项:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hateoas</artifactId>
    <version>2.6.4</version>
</dependency>
```

如果我们不使用 Spring Boot，我们可以在项目中添加以下库:

```
<dependency>
    <groupId>org.springframework.hateoas</groupId>
    <artifactId>spring-hateoas</artifactId>
    <version>1.4.1</version>
</dependency>
<dependency>
    <groupId>org.springframework.plugin</groupId>
    <artifactId>spring-plugin-core</artifactId>
    <version>1.2.0.RELEASE</version>
</dependency>
```

像往常一样，我们可以在 Maven Central 中搜索到最新版本的 [starter HATEOAS](https://web.archive.org/web/20220815025352/https://search.maven.org/search?q=a:spring-boot-starter-hateoas) 、[spring HATEOAS](https://web.archive.org/web/20220815025352/https://search.maven.org/search?q=a:spring-hateoas)和 [spring-plugin-core](https://web.archive.org/web/20220815025352/https://search.maven.org/search?q=a:spring-plugin-core) 依赖项。

接下来，我们有没有 Spring HATEOAS 支持的`Customer`资源:

```
public class Customer {

    private String customerId;
    private String customerName;
    private String companyName;

    // standard getters and setters
} 
```

我们有一个不支持 Spring HATEOAS 的控制器类:

```
@RestController
@RequestMapping(value = "/customers")
public class CustomerController {
    @Autowired
    private CustomerService customerService;

    @GetMapping("/{customerId}")
    public Customer getCustomerById(@PathVariable String customerId) {
        return customerService.getCustomerDetail(customerId);
    }
} 
```

最后，`Customer`资源表示:

```
{
    "customerId": "10A",
    "customerName": "Jane",
    "customerCompany": "ABC Company"
} 
```

## 4。添加 HATEOAS 支持

在 Spring HATEOAS 项目中，我们既不需要查找 Servlet 上下文，也不需要将 path 变量连接到基本 URI。

相反， **Spring HATEOAS 为创建 URI 提供了三个抽象——`RepresentationModel, Link, and WebMvcLinkBuilder`**。我们可以使用这些来创建元数据，并将其与资源表示相关联。

### 4.1。给资源添加超媒体支持

该项目提供了一个名为`RepresentationModel`的基类，用于在创建资源表示时继承:

```
public class Customer extends RepresentationModel<Customer> {
    private String customerId;
    private String customerName;
    private String companyName;

    // standard getters and setters
} 
```

**`Customer`资源从`RepresentationModel`类扩展来继承`add()`方法**。因此，一旦我们创建了一个链接，我们就可以轻松地将该值设置为资源表示，而无需向其添加任何新的字段。

### 4.2。创建链接

**Spring HATEOAS 提供了一个`Link`对象来存储元数据(资源的位置或 URI)。**

首先，我们将手动创建一个简单的链接:

```
Link link = new Link("http://localhost:8080/spring-security-rest/api/customers/10A"); 
```

`Link`对象遵循`[Atom](https://web.archive.org/web/20220815025352/https://en.wikipedia.org/wiki/Atom_(standard))`链接语法，由标识与资源关系的`rel`和实际链接本身的`href`属性组成。

下面是`Customer`资源包含新链接后的样子:

```
{
    "customerId": "10A",
    "customerName": "Jane",
    "customerCompany": "ABC Company",
    "_links":{
        "self":{
            "href":"http://localhost:8080/spring-security-rest/api/customers/10A"
         }
    }
} 
```

与响应相关联的 URI 被限定为`self`链接。`self` 关系的语义很清楚——它只是可以访问资源的规范位置。

### 4.3。创建更好的链接

该库提供的另一个非常重要的抽象是**`WebMvcLinkBuilder`——它通过避免硬编码链接简化了 URIs** 的构建。

下面的代码片段展示了如何使用`WebMvcLinkBuilder`类构建客户自我链接:

```
linkTo(CustomerController.class).slash(customer.getCustomerId()).withSelfRel(); 
```

让我们来看看:

*   `linkTo()`方法检查控制器类并获得它的根映射
*   `slash()`方法添加`customerId`值作为链接的路径变量
*   最后，`withSelfMethod()`将关系限定为自链接

## 5。关系

在上一节中，我们已经展示了自引用关系。然而，更复杂的系统也可能涉及其他关系。

例如，一个`customer` 可以与订单有关系。让我们也将`Order` 类建模为资源:

```
public class Order extends RepresentationModel<Order> {
    private String orderId;
    private double price;
    private int quantity;

    // standard getters and setters
} 
```

此时，我们可以用返回特定客户的所有订单的方法来扩展`CustomerController`:

```
@GetMapping(value = "/{customerId}/orders", produces = { "application/hal+json" })
public CollectionModel<Order> getOrdersForCustomer(@PathVariable final String customerId) {
    List<Order> orders = orderService.getAllOrdersForCustomer(customerId);
    for (final Order order : orders) {
        Link selfLink = linkTo(methodOn(CustomerController.class)
          .getOrderById(customerId, order.getOrderId())).withSelfRel();
        order.add(selfLink);
    }

    Link link = linkTo(methodOn(CustomerController.class)
      .getOrdersForCustomer(customerId)).withSelfRel();
    CollectionModel<Order> result = CollectionModel.of(orders, link);
    return result;
} 
```

我们的方法返回一个符合 HAL 返回类型的`CollectionModel` 对象，以及每个订单和完整列表的一个“`_self”`链接。

这里需要注意的一点是，客户订单的超链接依赖于`getOrdersForCustomer()`方法的映射。我们将这些类型的链接称为方法链接，并展示`WebMvcLinkBuilder`如何帮助它们的创建。

## 6。控制器方法的链接

`WebMvcLinkBuilder`为 Spring MVC 控制器提供了丰富的支持。下面的例子展示了如何基于`CustomerController`类的`getOrdersForCustomer()`方法构建 HATEOAS 超链接:

```
Link ordersLink = linkTo(methodOn(CustomerController.class)
  .getOrdersForCustomer(customerId)).withRel("allOrders"); 
```

**`methodOn()`通过在代理控制器上虚拟调用目标方法**来获得方法映射，并将`customerId`设置为 URI 的路径变量。

## 7。弹簧帽子在动

**让我们把自我链接和方法链接的创建放在一个`getAllCustomers()`方法中:**

```
@GetMapping(produces = { "application/hal+json" })
public CollectionModel<Customer> getAllCustomers() {
    List<Customer> allCustomers = customerService.allCustomers();

    for (Customer customer : allCustomers) {
        String customerId = customer.getCustomerId();
        Link selfLink = linkTo(CustomerController.class).slash(customerId).withSelfRel();
        customer.add(selfLink);
        if (orderService.getAllOrdersForCustomer(customerId).size() > 0) {
            Link ordersLink = linkTo(methodOn(CustomerController.class)
              .getOrdersForCustomer(customerId)).withRel("allOrders");
            customer.add(ordersLink);
        }
    }

    Link link = linkTo(CustomerController.class).withSelfRel();
    CollectionModel<Customer> result = CollectionModel.of(allCustomers, link);
    return result;
}
```

接下来，让我们调用`getAllCustomers()`方法:

```
curl http://localhost:8080/spring-security-rest/api/customers 
```

并检查结果:

```
{
  "_embedded": {
    "customerList": [{
        "customerId": "10A",
        "customerName": "Jane",
        "companyName": "ABC Company",
        "_links": {
          "self": {
            "href": "http://localhost:8080/spring-security-rest/api/customers/10A"
          },
          "allOrders": {
            "href": "http://localhost:8080/spring-security-rest/api/customers/10A/orders"
          }
        }
      },{
        "customerId": "20B",
        "customerName": "Bob",
        "companyName": "XYZ Company",
        "_links": {
          "self": {
            "href": "http://localhost:8080/spring-security-rest/api/customers/20B"
          },
          "allOrders": {
            "href": "http://localhost:8080/spring-security-rest/api/customers/20B/orders"
          }
        }
      },{
        "customerId": "30C",
        "customerName": "Tim",
        "companyName": "CKV Company",
        "_links": {
          "self": {
            "href": "http://localhost:8080/spring-security-rest/api/customers/30C"
          }
        }
      }]
  },
  "_links": {
    "self": {
      "href": "http://localhost:8080/spring-security-rest/api/customers"
    }
  }
}
```

在每个资源表示中，有一个`self`链接和一个`allOrders`链接来提取客户的所有订单。如果客户没有订单，则订单链接不会出现。

这个例子展示了 Spring HATEOAS 如何在 rest web 服务中促进 API 的可发现性。**如果链接存在，客户端可以跟随它并获得客户的所有订单:**

```
curl http://localhost:8080/spring-security-rest/api/customers/10A/orders 
```

```
{
  "_embedded": {
    "orderList": [{
        "orderId": "001A",
        "price": 150,
        "quantity": 25,
        "_links": {
          "self": {
            "href": "http://localhost:8080/spring-security-rest/api/customers/10A/001A"
          }
        }
      },{
        "orderId": "002A",
        "price": 250,
        "quantity": 15,
        "_links": {
          "self": {
            "href": "http://localhost:8080/spring-security-rest/api/customers/10A/002A"
          }
        }
      }]
  },
  "_links": {
    "self": {
      "href": "http://localhost:8080/spring-security-rest/api/customers/10A/orders"
    }
  }
}
```

## 8。结论

在本教程中，我们已经讨论了如何使用 Spring HATEOAS 项目构建一个超媒体驱动的 Spring REST web 服务。

在示例中，我们看到客户端可以有一个应用程序的单一入口点，并且可以基于响应表示中的元数据采取进一步的操作。

这允许服务器在不中断客户端的情况下更改其 URI 方案。此外，应用程序可以通过在表示中添加新链接或 URIs 来宣传新功能。

最后，本文的完整实现可以在 GitHub 项目中找到。