# Java 可选为返回类型

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-optional-return>

## 1.介绍

Java 8 中引入了`[Optional](/web/20221205153638/https://www.baeldung.com/java-optional)`类型。它提供了一种清晰明确的方式来传达可能没有值的消息，而不使用`null`。

当获得一个`Optional`返回类型时，我们可能会检查值是否丢失，从而导致应用程序中的`NullPointerException`减少。然而， `Optional`型并不适合所有地方。

尽管我们可以在任何我们认为合适的地方使用它，但在本教程中，我们将关注使用`Optional`作为返回类型的一些最佳实践。

## 2.`Optional`作为退货类型

一个`Optional`类型可以是大多数方法的返回类型，除了在本教程后面讨论的一些场景。

大多数时候，返回一个`Optional`就可以了:

```java
public static Optional<User> findUserByName(String name) {
    User user = usersByName.get(name);
    Optional<User> opt = Optional.ofNullable(user);
    return opt;
}
```

这很方便，因为我们可以在调用方法中使用`Optional` API:

```java
public static void changeUserName(String oldFirstName, String newFirstName) {
    findUserByFirstName(oldFirstName).ifPresent(user -> user.setFirstName(newFirstName));
}
```

静态方法或实用方法返回一个`Optional`值也是合适的。但是，有很多情况我们不应该返回`an Optional`类型。

## 3.何时不返回`Optional`

因为`Optional`是一个包装器和[基于值的](https://web.archive.org/web/20221205153638/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/doc-files/ValueBased.html)类，所以有些操作不能针对`Optional`对象进行。很多时候，返回实际类型比返回`Optional`类型更好。

一般来说，对于 POJOs 中的 getters，更适合返回实际类型，而不是一个`Optional`类型。特别是，实体 Beans、数据模型和 dto 拥有传统的 getters 非常重要。

我们将在下面研究一些重要的用例。

### 3.1.序列化

假设我们有一个简单的实体:

```java
public class Sock implements Serializable {
    Integer size;
    Optional<Sock> pair;

    // ... getters and setters
}
```

**这个其实根本行不通。**如果我们试图对此进行序列化，我们会得到一个`NotSerializableException`:

```java
new ObjectOutputStream(new ByteArrayOutputStream()).writeObject(new Sock()); 
```

实际上，虽然[序列化`Optional`可能与其他库](/web/20221205153638/https://www.baeldung.com/jackson-optional)，**一起工作，但它无疑增加了不必要的复杂性。**

让我们看看另一个同样的序列化不匹配的应用程序，这次是 JSON。

### 3.2.JSON

现代应用程序一直在将 Java 对象转换成 JSON。如果 getter 返回一个`Optional`类型，我们很可能会在最终的 JSON 中看到一些意外的数据结构。

假设我们有一个带有可选属性的 bean:

```java
private String firstName;

public Optional<String> getFirstName() {
    return Optional.ofNullable(firstName);
}

public void setFirstName(String firstName) {
    this.firstName = firstName;
}
```

因此，如果我们使用 Jackson 来序列化`Optional`的实例，我们将得到:

```java
{"firstName":{"present":true}} 
```

但是，我们真正想要的是:

```java
{"firstName":"Baeldung"}
```

因此，`Optional `对于序列化用例来说是一个难题。接下来，我们来看看序列化的表亲:**将数据写入数据库。**

### 3.3.作业的装配区（JobPackArea）

在 JPA 中，getter、setter 和 field 应该具有一致的名称和类型。例如，`String `类型的`firstName `字段应该与一个名为`getFirstName`的 getter 成对出现，该 getter 也返回一个`String.`

遵循这个约定会使一些事情变得更简单，包括 Hibernate 等库对反射的使用，为我们提供了很好的对象关系映射支持。

让我们看一下我们的同一个用例**POJO 中可选的名字。**

不过，这一次，它将是一个 JPA 实体:

```java
@Entity
public class UserOptionalField implements Serializable {
    @Id
    private long userId;

    private Optional<String> firstName;

    // ... getters and setters
}
```

让我们继续努力坚持下去:

```java
UserOptionalField user = new UserOptionalField();
user.setUserId(1l);
user.setFirstName(Optional.of("Baeldung"));
entityManager.persist(user);
```

遗憾的是，我们遇到了一个错误:

```java
Caused by: javax.persistence.PersistenceException: [PersistenceUnit: com.baeldung.optionalReturnType] Unable to build Hibernate SessionFactory
	at org.hibernate.jpa.boot.internal.EntityManagerFactoryBuilderImpl.persistenceException(EntityManagerFactoryBuilderImpl.java:1015)
	at org.hibernate.jpa.boot.internal.EntityManagerFactoryBuilderImpl.build(EntityManagerFactoryBuilderImpl.java:941)
	at org.hibernate.jpa.HibernatePersistenceProvider.createEntityManagerFactory(HibernatePersistenceProvider.java:56)
	at javax.persistence.Persistence.createEntityManagerFactory(Persistence.java:79)
	at javax.persistence.Persistence.createEntityManagerFactory(Persistence.java:54)
	at com.baeldung.optionalReturnType.PersistOptionalTypeExample.<clinit>(PersistOptionalTypeExample.java:11)
Caused by: org.hibernate.MappingException: Could not determine type for: java.util.Optional, at table: UserOptionalField, for columns: [org.hibernate.mapping.Column(firstName)]
```

我们可以尝试**偏离这个标准。**例如，我们可以保持属性为`String`，但是改变 getter:

```java
@Column(nullable = true) 
private String firstName; 

public Optional<String> getFirstName() { 
    return Optional.ofNullable(firstName); 
}
```

似乎我们可以有两种方式:getter 有一个`Optional`返回类型，持久字段有一个`firstName`。

然而，现在我们与 getter、setter 和 field 不一致了，利用 JPA 默认值和 IDE 源代码工具将变得更加困难。

在 JPA 拥有优雅的`Optional`类型支持之前，我们应该坚持传统的代码。它更简单也更好:

```java
private String firstName;

// ... traditional getter and setter
```

最后，让我们看看这对前端有什么影响——看看我们遇到的问题听起来是否熟悉。

### 3.4.表达式语言

为前端准备 DTO 存在类似的困难。

例如，假设我们使用 JSP 模板从请求中读取我们的`UserOptional` DTO 的`firstName`:

```java
<c:out value="${requestScope.user.firstName}" /> 
```

既然是`Optional`，我们就看不到`Baeldung`。相反，我们将看到`Optional`类型的`String`表示:

```java
Optional[Baeldung] 
```

这不仅仅是 JSP 的问题。任何模板语言，无论是 Velocity、Freemarker 还是其他什么，都需要增加对它的支持。在那之前，让我们继续保持我们的 dto 简单。

## 4.结论

在本教程中，我们学习了如何返回一个`Optional`对象，以及如何处理这种返回值。

另一方面，我们还了解到，在很多情况下，我们最好不要对 getter 使用`Optional`返回类型。虽然我们可以使用`Optional`类型暗示可能没有非空值，但是我们应该小心不要过度使用`Optional`返回类型，尤其是在实体 bean 或 DTO 的 getter 中。

本教程中例子的源代码可以在 [GitHub](https://web.archive.org/web/20221205153638/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-optional) 上找到。