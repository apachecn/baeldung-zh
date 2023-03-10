# JPA 实体平等

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-entity-equality>

## 1.概观

在本教程中，我们将看看如何处理 JPA 实体对象的等式。

## 2.考虑

一般来说，相等仅仅意味着两个对象是相同的。然而，在 Java 中，我们可以通过重写 [`Object.equals()`和`Object.hashCode()`](/web/20221127130432/https://www.baeldung.com/java-equals-hashcode-contracts) 方法来改变等式的定义。**最终，Java 允许我们定义平等意味着什么。但是首先，我们需要考虑几件事。**

### 2.1.收集

Java 集合将对象组合在一起。分组逻辑使用称为哈希代码的特殊值来确定对象的组。

如果由`hashCode()`方法返回的值对于所有实体都是相同的，这可能会导致不期望的行为。假设我们的实体对象有一个定义为`id`的主键，但是我们将`hashCode()`方法定义为:

```java
@Override
public int hashCode() {
    return 12345;
} 
```

集合在比较不同的对象时将无法区分它们，因为它们将共享相同的哈希代码。幸运的是，解决这个问题就像在生成哈希代码时使用惟一的键一样简单。例如，我们可以使用我们的`id`来定义`hashCode()`方法:

```java
@Override
public int hashCode() {
    return id * 12345;
} 
```

在这种情况下，我们使用实体的`id`来定义哈希代码。现在，集合可以比较、排序和存储我们的实体。

### 2.2.瞬态实体

**新创建的与[持久上下文](/web/20221127130432/https://www.baeldung.com/jpa-hibernate-persistence-context)没有关联的 JPA 实体对象被认为处于瞬态**。这些对象通常没有填充它们的`@Id`成员。因此，如果`equals()`或`hashCode()`在他们的计算中使用`id`，这意味着所有的瞬态对象将是相等的，因为它们的`id`都将是`null`。这种情况并不多见。

### 2.3.子类

在定义等式时，子类也是一个关注点。在`equals()`方法中比较类是很常见的。因此，在比较对象是否相等时，包含`getClass()`方法将有助于过滤掉子类。

让我们定义一个`equals()`方法，该方法只有在对象属于同一类并且具有相同的`id`时才有效:

```java
@Override
public boolean equals(Object o) {
    if (o == null || this.getClass() != o.getClass()) {
        return false;
    }
    return o.id.equals(this.id);
} 
```

## 3.定义平等

考虑到这些因素，我们在处理等式时有几个选择。因此，我们采取的方法取决于我们计划如何使用对象的细节。让我们看看我们的选择。

### 3.1.没有覆盖

默认情况下，Java 通过从`Object`类派生的所有对象来提供`equals()`和`hashCode()`方法。所以，我们能做的最简单的事就是什么都不做。不幸的是，这意味着当比较对象时，为了被认为是相等的，它们必须是相同的实例，而不是代表相同对象的两个单独的实例。

### 3.2.使用数据库密钥

在大多数情况下，我们处理的是存储在数据库中的 JPA 实体。**通常，这些实体有一个唯一值的主键。因此，具有相同主键值的此实体的任何实例都是相等的。**因此，我们可以像上面对子类所做的那样覆盖`equals()`,也可以只使用主键覆盖`hashCode() `。

### 3.3.使用业务密钥

或者，我们可以使用业务关键字来比较 JPA 实体。在这种情况下，对象的键由实体的成员组成，而不是主键。这个键应该使 JPA 实体是唯一的。**在比较实体时，使用业务键给我们带来了相同的期望结果**,而不需要主键或数据库生成的键。

假设我们知道一个电子邮件地址总是唯一的，即使它不是`@Id`字段。我们可以在`hashCode()`和`equals()`方法中包含电子邮件字段:

```java
public class EqualByBusinessKey {

    private String email;

    @Override
    public int hashCode() {
        return java.util.Objects.hashCode(email);
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (obj == null) {
            return false;
        }
        if (obj instanceof EqualByBusinessKey) {
            if (((EqualByBusinessKey) obj).getEmail().equals(getEmail())) {
                return true;
            }
        }

        return false;
    }
} 
```

## 4.结论

在本教程中，我们讨论了在编写 JPA 实体对象时处理等式的各种方法。我们还描述了在选择方法时应该考虑的因素。和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221127130432/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa-3)