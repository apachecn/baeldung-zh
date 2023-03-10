# 在 Java 中使用 AssertJ 提取值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-extract-values-assertj>

## 1.概观

AssertJ 是一个 Java 断言库，它允许我们流畅地编写断言，并使它们更具可读性。

在本教程中，我们将探索 AssertJ 的提取方法，以便在不破坏测试断言流的情况下流畅地进行检查。

## 2.履行

让我们从一个`Person`示例类开始:

```java
class Person {
    private String firstName;
    private String lastName;
    private Address address;

    Person(String firstName, String lastName, Address address) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.address = address;
    }

    // getters and setter omitted
}
```

每个`Person`将与一些`Address:`相关联

```java
class Address {
    private String street;
    private String city;
    private ZipCode zipCode;

    Address(String street, String city, ZipCode zipCode) {
        this.street = street;
        this.city = city;
        this.zipCode = zipCode;
    }

    // getters and setter omitted
}
```

并且每个`Address`将有一个`ZipCode`被包括作为一个类:

```java
class ZipCode {
    private long zipcode;

    ZipCode(long zipcode) {
        this.zipcode = zipcode;
    }

    // getters and setter omitted
}
```

现在假设在创建了一个`Person`对象后，我们需要测试以下情况:

*   `Address`不是`null`
*   `Address`不在受限地址列表中
*   `ZipCode`对象不是`null`
*   `ZipCode`值介于 1000 和 100000 之间

## 3.使用 AssertJ 的常见断言

给定以下`Person`对象:

```java
Person person = new Person("aName", "aLastName", new Address("aStreet", "aCity", new ZipCode(90210)));
```

我们可以提取`Address`对象:

```java
Address address = person.getAddress();
```

那么我们可以断言`Adress`不为空:

```java
assertThat(address).isNotNull();
```

我们还可以检查`Address`是否不在受限地址列表中:

```java
assertThat(address).isNotIn(RESTRICTED_ADDRESSES);
```

下一步是检查`ZipCode`:

```java
ZipCode zipCode = address.getZipCode();
```

并断言它不为空:

```java
assertThat(zipCode).isNotNull();
```

最后，我们可以提取`ZipCode`值并断言它在 1000 到 100000 之间:

```java
assertThat(zipCode.getZipcode()).isBetween(1000L, 100_000L);
```

上面的代码很简单，但我们需要帮助来流利地阅读它，因为它需要多行来处理。我们还需要分配变量，以便能够在以后断言它们，这不是一个干净的代码体验。

## 4.使用 AssertJ 的提取方法

现在让我们看看提取方法如何帮助我们:

```java
assertThat(person)
  .extracting(Person::getAddress)
  .isNotNull()
  .isNotIn(RESTRICTED_ADDRESSES)
  .extracting(Address::getZipCode)
  .isNotNull()
  .extracting(ZipCode::getZipcode, as(InstanceOfAssertFactories.LONG))
  .isBetween(1_000L, 100_000L);
```

正如我们所看到的，代码没有很大的不同，但它很流畅，更容易阅读。

## 5.结论

在本文中，我们可以看到提取要断言的对象值的两种方法:

*   提取到稍后要断言的变量中
*   使用 AssertJ 的提取方法流畅地提取

本文中使用的例子可以在 GitHub 上找到[。](https://web.archive.org/web/20221218212940/https://github.com/eugenp/tutorials/tree/master/testing-modules/assertion-libraries)