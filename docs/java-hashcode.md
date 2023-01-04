# Java 中的 hashCode()指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-hashcode>

## 1。概述

哈希是计算机科学的一个基本概念。

在 Java 中，高效的散列算法支持一些最流行的集合，例如 *[散列表](https://web.archive.org/web/20220926202041/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/HashMap.html)* (查看这篇深入的[文章](/web/20220926202041/https://www.baeldung.com/java-hashmap))和 *[散列表](https://web.archive.org/web/20220926202041/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/HashSet.html)* 。

在本教程中，我们将关注`hashCode()`如何工作，它如何在集合中发挥作用，以及如何正确地实现它。

## 延伸阅读:

## [Java equals()和 hashCode()契约](/web/20220926202041/https://www.baeldung.com/java-equals-hashcode-contracts)

Learn about the contracts that equals() and hasCode() need to fulfill and the relationship between the two methods[Read more](/web/20220926202041/https://www.baeldung.com/java-equals-hashcode-contracts) →

## [用 Eclipse](/web/20220926202041/https://www.baeldung.com/java-eclipse-equals-and-hashcode) 生成 equals()和 hashCode()

A quick and practical guide to generating equals() and hashcode() with the Eclipse IDE[Read more](/web/20220926202041/https://www.baeldung.com/java-eclipse-equals-and-hashcode) →

## [龙目岛项目简介](/web/20220926202041/https://www.baeldung.com/intro-to-project-lombok)

A comprehensive and very practical introduction to many useful usecases of Project Lombok on standard Java code.[Read more](/web/20220926202041/https://www.baeldung.com/intro-to-project-lombok) →

## 2。在数据结构中使用 *hashCode()*

在某些情况下，集合上最简单的操作可能是低效的。

举例来说，这触发了线性搜索，这对于巨大的列表是非常无效的:

```java
List<String> words = Arrays.asList("Welcome", "to", "Baeldung");
if (words.contains("Baeldung")) {
    System.out.println("Baeldung is in the list");
}
```

Java 提供了许多数据结构来专门处理这个问题。比如几个 *Map* 接口实现就是[哈希表](/web/20220926202041/https://www.baeldung.com/cs/hash-tables)。

当使用哈希表时，**这些集合使用 *hashCode()* 方法计算给定键的哈希值。**然后，它们在内部使用这个值来存储数据，这样访问操作会更加高效。

## 3。理解 *hashCode()* 如何工作

简单来说， *hashCode()* 返回一个整数值，由哈希算法生成。

相等的对象(根据它们的`equals()`)必须返回相同的散列码。**不同的对象不需要返回不同的哈希码。**

*hashCode()* 的总承包合同规定:

*   只要在 Java 应用程序的执行过程中对同一对象多次调用， *hashCode()* 必须始终返回相同的值，前提是在对象的 equals 比较中使用的信息没有被修改。从一个应用程序的一次执行到同一应用程序的另一次执行，这个值不需要保持一致。

*   根据`equals(Object)`方法，如果两个对象相等，那么在这两个对象上调用 *hashCode()* 方法必须产生相同的值。

*   如果根据 [`equals(java.lang.Object)`](https://web.archive.org/web/20220926202041/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html#equals(java.lang.Object)) 方法，两个对象不相等，那么在这两个对象上调用 *hashCode* 方法不需要产生不同的整数结果。但是，开发人员应该知道，为不相等的对象生成不同的整数结果可以提高哈希表的性能。

> “实际上，由类`Object`定义的 *hashCode()* 方法确实为不同的对象返回不同的整数。(这通常是通过将对象的内部地址转换成整数来实现的，但是 JavaTM 编程语言并不要求这种实现技术。)"

## 4。一个幼稚的`hashCode()`实现

完全遵守上述契约的简单的 hashCode() 实现实际上非常简单。

为了演示这一点，我们将定义一个示例 *User* 类，它覆盖了该方法的默认实现:

```java
public class User {

    private long id;
    private String name;
    private String email;

    // standard getters/setters/constructors

    @Override
    public int hashCode() {
        return 1;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null) return false;
        if (this.getClass() != o.getClass()) return false;
        User user = (User) o;
        return id == user.id 
          && (name.equals(user.name) 
          && email.equals(user.email));
    }

    // getters and setters here
}
```

*用户*类为完全遵守各自契约的`equals()`和`hashCode()`提供定制实现。更重要的是，让 *hashCode()* 返回任何固定值也没有什么不合理的。

然而，这种实现将哈希表的功能降低到基本为零，因为每个对象都存储在同一个桶中。

在这种情况下，散列表查找是线性执行的，不会给我们带来任何真正的好处。我们将在第 7 节详细讨论这一点。

## 5。改善`hashCode()`实施

让我们通过包含*用户*类的所有字段来改进当前的 *hashCode()* 实现，以便它可以为不相等的对象产生不同的结果:

```java
@Override
public int hashCode() {
    return (int) id * name.hashCode() * email.hashCode();
}
```

这种基本的散列算法肯定比前一种算法好得多。这是因为它通过将*名称*和*电子邮件*字段和 *id* 的散列码相乘来计算对象的散列码。

一般来说，我们可以说这是一个合理的 *hashCode()* 实现，只要我们保持 *equals()* 实现与之一致。

## 6。标准 *hashCode()* 实现

我们用来计算哈希代码的哈希算法越好，哈希表的性能就越好。

让我们来看一个“标准”实现，它使用两个素数来为计算出的散列码增加更多的唯一性:

```java
@Override
public int hashCode() {
    int hash = 7;
    hash = 31 * hash + (int) id;
    hash = 31 * hash + (name == null ? 0 : name.hashCode());
    hash = 31 * hash + (email == null ? 0 : email.hashCode());
    return hash;
}
```

虽然我们需要理解 *hashCode()* 和 *equals()* 方法扮演的角色，但我们不必每次都从头实现它们。这是因为大多数 ide 可以生成定制的 *hashCode()* 和 *equals()* 实现。从 Java 7 开始，我们有了一个`Objects.hash()`实用方法来轻松散列:

```java
Objects.hash(name, email)
```

[IntelliJ IDEA](https://web.archive.org/web/20220926202041/https://www.jetbrains.com/idea/) 生成以下实现:

```java
@Override
public int hashCode() {
    int result = (int) (id ^ (id >>> 32));
    result = 31 * result + name.hashCode();
    result = 31 * result + email.hashCode();
    return result;
}
```

而[日蚀](https://web.archive.org/web/20220926202041/https://www.eclipse.org/downloads/)产生了这个:

```java
@Override
public int hashCode() {
    final int prime = 31;
    int result = 1;
    result = prime * result + ((email == null) ? 0 : email.hashCode());
    result = prime * result + (int) (id ^ (id >>> 32));
    result = prime * result + ((name == null) ? 0 : name.hashCode());
    return result;
}
```

除了上述基于 IDE 的 *hashCode()* 实现，还可以自动生成一个高效的实现，例如使用 [Lombok](https://web.archive.org/web/20220926202041/https://projectlombok.org/features/EqualsAndHashCode) 。

在这种情况下，我们需要将 [lombok-maven](https://web.archive.org/web/20220926202041/https://search.maven.org/classic/#search%7Cga%7C1%7Clombok) 依赖项添加到`pom.xml`:

```java
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok-maven</artifactId>
    <version>1.16.18.0</version>
    <type>pom</type>
</dependency>
```

现在用 *@EqualsAndHashCode* 来注释*用户*类就足够了:

```java
@EqualsAndHashCode 
public class User {
    // fields and methods here
}
```

类似地，如果我们希望 [Apache Commons Lang 的 *HashCodeBuilder* 类](https://web.archive.org/web/20220926202041/https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/builder/HashCodeBuilder.html)为我们生成一个 *hashCode()* 实现，我们在 pom 文件中包含 [commons-lang](https://web.archive.org/web/20220926202041/https://search.maven.org/classic/#search%7Cga%7C1%7Capache-commons-lang) Maven 依赖项:

```java
<dependency>
    <groupId>commons-lang</groupId>
    <artifactId>commons-lang</artifactId>
    <version>2.6</version>
</dependency>
```

而 *hashCode()* 可以这样实现:

```java
public class User {
    public int hashCode() {
        return new HashCodeBuilder(17, 37).
        append(id).
        append(name).
        append(email).
        toHashCode();
    }
}
```

一般来说，在实现 *hashCode()* 时，没有通用的方法。我们强烈推荐阅读[约书亚·布洛赫的《有效的 Java](https://web.archive.org/web/20220926202041/https://www.amazon.com/Effective-Java-3rd-Joshua-Bloch/dp/0134685997) 。它为实现有效的散列算法提供了一系列[全面的指导方针](https://web.archive.org/web/20220926202041/https://es.slideshare.net/MukkamalaKamal/joshua-bloch-effect-java-chapter-3)。

请注意，所有这些实现都以某种形式使用了数字 31。这是因为 31 有一个很好的属性。它的乘法可以用比标准乘法更快的按位移位来代替:

```java
31 * i == (i << 5) - i
```

## 7 .**。处理哈希冲突**

哈希表的内在行为带来了这些数据结构的一个相关方面:即使使用有效的哈希算法，两个或更多的对象也可能具有相同的哈希代码，即使它们不相等。因此，即使它们有不同的散列表键，它们的散列码也会指向同一个桶。

这种情况通常被称为哈希冲突，存在各种方法来处理它，每种方法都有其优点和缺点。Java 的`HashMap`使用[独立链接方法](https://web.archive.org/web/20220926202041/https://en.wikipedia.org/wiki/Hash_table#Separate_chaining_with_linked_lists)来处理冲突:

当两个或更多的对象指向同一个桶时，它们被简单地存储在一个链表中。在这种情况下，哈希表是一个链表数组，具有相同哈希的每个对象都被附加到数组中桶索引处的链表中。

在最坏的情况下，几个桶会绑定一个链表，对链表中对象的检索会线性执行。"

哈希冲突方法简单地说明了为什么高效地实现`hashCode()`*如此重要。*

Java 8 为`HashMap`实现带来了一个有趣的[增强。如果一个桶的大小超过了某个阈值，一个树映射将替换链表。这允许实现`O(`登录`)`查找，而不是悲观`O(n)`。](https://web.archive.org/web/20220926202041/https://openjdk.java.net/jeps/180)

## 8。创建一个简单的应用程序

现在我们将测试一个标准的 *hashCode()* 实现的功能。

让我们创建一个简单的 Java 应用程序，它向一个*散列表*添加一些*用户*对象，并在每次调用方法时使用 [SLF4J](/web/20220926202041/https://www.baeldung.com/slf4j-with-log4j2-logback) 向控制台记录一条消息。

下面是示例应用程序的入口点:

```java
public class Application {

    public static void main(String[] args) {
        Map<User, User> users = new HashMap<>();
        User user1 = new User(1L, "John", "[[email protected]](/web/20220926202041/https://www.baeldung.com/cdn-cgi/l/email-protection)");
        User user2 = new User(2L, "Jennifer", "[[email protected]](/web/20220926202041/https://www.baeldung.com/cdn-cgi/l/email-protection)");
        User user3 = new User(3L, "Mary", "[[email protected]](/web/20220926202041/https://www.baeldung.com/cdn-cgi/l/email-protection)");

        users.put(user1, user1);
        users.put(user2, user2);
        users.put(user3, user3);
        if (users.containsKey(user1)) {
            System.out.print("User found in the collection");
        }
    }
} 
```

这是 *hashCode()* 的实现:

```java
public class User {

    // ...

    public int hashCode() {
        int hash = 7;
        hash = 31 * hash + (int) id;
        hash = 31 * hash + (name == null ? 0 : name.hashCode());
        hash = 31 * hash + (email == null ? 0 : email.hashCode());
        logger.info("hashCode() called - Computed hash: " + hash);
        return hash;
    }
}
```

这里需要注意的是，每次对象被存储在哈希映射中并用 *containsKey()* 方法检查时， *hashCode()* 被调用，计算出的哈希代码被输出到控制台:

```java
[main] INFO com.baeldung.entities.User - hashCode() called - Computed hash: 1255477819
[main] INFO com.baeldung.entities.User - hashCode() called - Computed hash: -282948472
[main] INFO com.baeldung.entities.User - hashCode() called - Computed hash: -1540702691
[main] INFO com.baeldung.entities.User - hashCode() called - Computed hash: 1255477819
User found in the collection
```

## 9。结论

很明显，产生高效的 *hashCode()* 实现通常需要混合一些数学概念(即素数和任意数)、逻辑和基本数学运算。

不管怎样，我们可以有效地实现 *hashCode()* 而完全不需要借助这些技术。我们只需要确保散列算法为不同的对象产生不同的散列码，并且它与 *equals()* 的实现一致。

和往常一样，本文中展示的所有代码示例都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220926202041/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-methods)