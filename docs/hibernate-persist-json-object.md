# 使用 Hibernate 持久化 JSON 对象

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-persist-json-object>

## 1。概述

一些项目可能需要将 JSON 对象保存在关系数据库中。

在本教程中，我们将看到如何**获取一个 JSON 对象并将其持久化到关系数据库**中。

有几个框架可以提供这种功能，但是我们将只使用 [Hibernate](/web/20220628235228/https://www.baeldung.com/hibernate-5-spring) 和 [Jackson](/web/20220628235228/https://www.baeldung.com/jackson) 来看看几个简单的通用选项。

## 2。依赖性

在本教程中，我们将使用[基本 Hibernate 核心依赖关系](https://web.archive.org/web/20220628235228/https://search.maven.org/search?q=g:org.hibernate%20AND%20a:hibernate-core):

```java
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.4.0.Final</version>
</dependency>
```

我们还将使用 [Jackson](https://web.archive.org/web/20220628235228/https://search.maven.org/search?q=g:com.fasterxml.jackson.core%20AND%20a:jackson-databind) 作为我们的 JSON 库:

```java
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.13.0</version>
</dependency>
```

注意，这些技术并不局限于这两个库。我们可以替换我们喜欢的 JPA 提供者和 JSON 库。

## 3。序列化和反序列化方法

在关系数据库中持久化 JSON 对象的最基本方法是在持久化之前将对象转换成 T2。然后，当我们从数据库中检索它时，我们**将它转换回对象**。

我们可以用几种不同的方法来做这件事。

我们要看的第一个是使用**定制的序列化和反序列化方法。**

我们将从一个简单的`Customer`实体开始，它存储客户的名字和姓氏，以及关于该客户的一些属性。

一个标准的 JSON 对象将这些属性表示为一个`HashMap`，所以这就是我们在这里使用的:

```java
@Entity
@Table(name = "Customers")
public class Customer {

    @Id
    private int id;

    private String firstName;

    private String lastName;

    private String customerAttributeJSON;

    @Convert(converter = HashMapConverter.class)
    private Map<String, Object> customerAttributes;
}
```

我们不是将属性保存在一个单独的表中，而是将它们作为 JSON 存储在`Customers`表的一个列中。这有助于降低模式复杂性并提高查询性能。

首先，**我们将创建一个序列化方法，该方法将获取我们的`customerAttributes`并将其转换为一个 JSON 字符串**:

```java
public void serializeCustomerAttributes() throws JsonProcessingException {
    this.customerAttributeJSON = objectMapper.writeValueAsString(customerAttributes);
}
```

我们可以在持久化之前手动调用这个方法，或者从`setCustomerAttributes`方法调用它，这样每次更新属性时，JSON 字符串也会更新。

接下来，**我们将创建一个方法，当我们从数据库中检索`Customer`时，将 JSON 字符串反序列化回一个`HashMap`对象**:

```java
public void deserializeCustomerAttributes() throws IOException {
    this.customerAttributes = objectMapper.readValue(customerAttributeJSON, HashMap.class);
}
```

同样，有几个不同的地方我们可以调用这个方法，但是，在这个例子中，我们将手动调用它。

因此，持久化和检索我们的`Customer`对象看起来像这样:

```java
@Test
public void whenStoringAJsonColumn_thenDeserializedVersionMatches() {
    Customer customer = new Customer();
    customer.setFirstName("first name");
    customer.setLastName("last name");

    Map<String, Object> attributes = new HashMap<>();
    attributes.put("address", "123 Main Street");
    attributes.put("zipcode", 12345);

    customer.setCustomerAttributes(attributes);
    customer.serializeCustomerAttributes();

    String serialized = customer.getCustomerAttributeJSON();

    customer.setCustomerAttributeJSON(serialized);
    customer.deserializeCustomerAttributes();

    assertEquals(attributes, customer.getCustomerAttributes());
}
```

## 4。属性转换器

**如果我们使用 JPA 2.1 或更高版本的**，我们可以利用`AttributeConverters`来简化这个过程。

首先，我们将创建一个`AttributeConverter`的实现。我们将重用之前的代码:

```java
public class HashMapConverter implements AttributeConverter<Map<String, Object>, String> {

    @Override
    public String convertToDatabaseColumn(Map<String, Object> customerInfo) {

        String customerInfoJson = null;
        try {
            customerInfoJson = objectMapper.writeValueAsString(customerInfo);
        } catch (final JsonProcessingException e) {
            logger.error("JSON writing error", e);
        }

        return customerInfoJson;
    }

    @Override
    public Map<String, Object> convertToEntityAttribute(String customerInfoJSON) {

        Map<String, Object> customerInfo = null;
        try {
            customerInfo = objectMapper.readValue(customerInfoJSON, Map.class);
        } catch (final IOException e) {
            logger.error("JSON reading error", e);
        }

        return customerInfo;
    }

}
```

接下来，我们告诉 Hibernate 使用新的`AttributeConverter`作为`customerAttributes`字段，我们就完成了:

```java
@Convert(converter = HashMapConverter.class)
private Map<String, Object> customerAttributes;
```

使用这种方法，我们不再需要手动调用序列化和反序列化方法,因为 Hibernate 会为我们处理这些。我们可以简单地正常保存和检索`Customer`对象。

## 5。结论

在本文中，我们已经看到了几个使用 Hibernate 和 Jackson 持久化 JSON 对象的例子。

我们的第一个例子是一个简单、兼容的方法，使用定制的序列化和反序列化方法。其次，我们引入了`AttributeConverters`作为一种简化代码的强大方法。

和往常一样，请务必在 Github 上查看本教程[的源代码。](https://web.archive.org/web/20220628235228/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate5)