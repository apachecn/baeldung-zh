# 对 MapStruct 使用多个源对象

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mapstruct-multiple-source-objects>

## 1.概观

在本教程中，我们将看到如何使用多个源对象与 [MapStruct](/web/20221224152125/https://www.baeldung.com/mapstruct) 。

## 2.单源对象

MapStruct 最常见的用例是将一个对象映射到另一个对象。假设我们有一个`Customer`类:

```java
class Customer {

    private String firstName;
    private String lastName;

    // getters and setters

}
```

让我们进一步假设有一个对应的`CustomerDto`:

```java
class CustomerDto {

    private String forename;
    private String surname;

    // getters and setters

}
```

我们现在可以定义一个映射器，将一个`Customer`对象映射到一个`CustomerDto`对象:

```java
@Mapper
public interface CustomerDtoMapper {

    @Mapping(source = "firstName", target = "forename")
    @Mapping(source = "lastName", target = "surname")
    CustomerDto from(Customer customer);

}
```

## 3.多个源对象

有时我们希望目标对象拥有来自多个源对象的属性。假设我们编写一个购物应用程序。

我们需要构建一个送货地址来运送我们的货物:

```java
class DeliveryAddress {

    private String forename;
    private String surname;
    private String street;
    private String postalcode;
    private String county;

    // getters and setters

}
```

每个客户可以有多个地址。一个可以是家庭住址。另一个可以是工作地址:

```java
class Address {

    private String street;
    private String postalcode;
    private String county;

    // getters and setters

}
```

我们现在需要一个映射器，它从一个客户及其一个地址中创建交货地址。MapStruct 通过拥有多个源对象来支持这一点:

```java
@Mapper
interface DeliveryAddressMapper {

    @Mapping(source = "customer.firstName", target = "forename")
    @Mapping(source = "customer.lastName", target = "surname")
    @Mapping(source = "address.street", target = "street")
    @Mapping(source = "address.postalcode", target = "postalcode")
    @Mapping(source = "address.county", target = "county")
    DeliveryAddress from(Customer customer, Address address);

}
```

让我们通过编写一个小测试来看看这一点:

```java
// given a customer
Customer customer = new Customer().setFirstName("Max")
  .setLastName("Powers");

// and some address
Address homeAddress = new Address().setStreet("123 Some Street")
  .setCounty("Nevada")
  .setPostalcode("89123");

// when calling DeliveryAddressMapper::from
DeliveryAddress deliveryAddress = deliveryAddressMapper.from(customer, homeAddress);

// then a new DeliveryAddress is created, based on the given customer and his home address
assertEquals(deliveryAddress.getForename(), customer.getFirstName());
assertEquals(deliveryAddress.getSurname(), customer.getLastName());
assertEquals(deliveryAddress.getStreet(), homeAddress.getStreet());
assertEquals(deliveryAddress.getCounty(), homeAddress.getCounty());
assertEquals(deliveryAddress.getPostalcode(), homeAddress.getPostalcode());
```

**当我们有不止一个参数时，我们可以在`@Mapping`注释中用点符号来表示它们。**例如，为了处理名为`customer`的参数的属性`firstName`，我们简单地写下“`customer.firstName`”。

然而，我们不限于两个源对象。任何数字都可以。

## 4.用`@MappingTarget`更新现有对象

到目前为止，我们已经有了创建目标类新实例的映射器。对于多个源对象，我们现在还可以提供一个要更新的实例。

例如，假设我们想要更新一个交货地址的与客户相关的属性。我们所需要的就是让其中一个参数与方法返回的类型相同，并用`@MappingTarget`对其进行注释:

```java
@Mapper
interface DeliveryAddressMapper {

    @Mapping(source = "address.postalcode", target = "postalcode")
    @Mapping(source = "address.county", target = "county")
    DeliveryAddress updateAddress(@MappingTarget DeliveryAddress deliveryAddress, Address address);

}
```

因此，让我们继续用一个`DeliveryAddress`实例做一个快速测试:

```java
// given a delivery address
DeliveryAddress deliveryAddress = new DeliveryAddress().setForename("Max")
  .setSurname("Powers")
  .setStreet("123 Some Street")
  .setCounty("Nevada")
  .setPostalcode("89123");

// and some new address
Address newAddress = new Address().setStreet("456 Some other street")
  .setCounty("Arizona")
  .setPostalcode("12345");

// when calling DeliveryAddressMapper::updateAddress
DeliveryAddress updatedDeliveryAddress = deliveryAddressMapper.updateAddress(deliveryAddress, newAddress);

// then the *existing* delivery address is updated
assertSame(deliveryAddress, updatedDeliveryAddress);

assertEquals(deliveryAddress.getStreet(), newAddress.getStreet());
assertEquals(deliveryAddress.getCounty(), newAddress.getCounty());
assertEquals(deliveryAddress.getPostalcode(), newAddress.getPostalcode());
```

## 5.结论

MapStruct 允许我们向映射方法传递多个源参数。例如，当我们想要将多个实体合并成一个时，这就很方便了。

另一个用例是让目标对象本身成为源参数之一。使用`@MappingTarget`注释，可以就地更新给定的对象。

请务必在 GitHub 上查看所有这些样本[。](https://web.archive.org/web/20221224152125/https://github.com/eugenp/tutorials/tree/master/mapstruct)