# JPA 与 JDBC 的比较

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-vs-jdbc>

## 1.概观

在本教程中，我们将看看 [Java 数据库连接(JDBC) API](/web/20221018114335/https://www.baeldung.com/java-jdbc) 和 [Java 持久性 API (JPA)](/web/20221018114335/https://www.baeldung.com/learn-jpa-hibernate) 之间的区别。

## 2.什么是 JDBC

JDBC 是 Java 应用程序与数据库通信的编程级接口。应用程序使用这个 API 与 JDBC 管理器进行通信。它是我们的应用程序代码用来与数据库通信的通用 API。除了 API 之外，还有供应商提供的符合 JDBC 标准的驱动程序，用于我们正在使用的数据库。

## 3.什么是 JPA

JPA 是一个 Java 标准，它允许我们将 Java 对象绑定到关系数据库中的记录。 **[这是一种可能的方法](https://web.archive.org/web/20221018114335/https://en.wikipedia.org/wiki/List_of_object%E2%80%93relational_mapping_software#Java)到对象关系映射(ORM)** ，允许开发者使用 Java 对象在关系数据库中检索、存储、更新和删除数据。JPA 规范有几种实现。

## 4.JPA vs JDBC

当决定如何与后端数据库系统通信时，软件架构师面临着一个重大的技术挑战。JPA 和 JDBC 之间的争论通常是决定性的因素，因为这两种数据库技术采用非常不同的方法来处理持久数据。我们来分析一下两者的关键区别。

### 4.1.数据库交互

JDBC 允许我们编写 SQL 命令来从关系数据库中读取数据和更新数据。 **JPA 不像 JDBC，允许开发者利用面向对象的语义构建数据库驱动的 Java 程序** 。JPA 注释 描述了给定的 Java 类及其变量如何映射到数据库 中的给定表及其列。

让我们看看如何将一个`Employee`类映射到一个`employee`数据库表:

```
@Entity
@Table(name = "employee")
public class Employee implements Serializable {
    @Column(name = "employee_name")
    private String employeeName;
}
```

然后 JPA 框架处理所有耗时、易错的编码，这些编码需要在面向对象的 Java 代码和后端数据库 之间进行转换。

### 4.2.管理关联

当将查询中的数据库表与 JDBC 相关联时，我们需要写出完整的 SQL 查询，而对于 JPA，我们只需使用注释来创建一对一、一对多、多对一和多对多的关联。

假设我们的`employee`表与`communication`表有一对多的关系:

```
@Entity
@Table(name = "employee")
public class Employee implements Serializable {

    @OneToMany(mappedBy = "employee", fetch = FetchType.EAGER)
    @OrderBy("firstName asc")
    private Set communications;
}
```

这个关系的所有者是`Communication`，所以我们使用`Employee`中的`mappedBy`属性使它成为一个双向关系。

### 4.3.数据库依赖性

JDBC 是数据库相关的，这意味着不同的脚本必须针对不同的数据库 编写。 另一方面， **JPA 是数据库不可知的，这意味着相同的代码可以****在各种数据库中使用，只需很少(或不需要)修改** 。

### 4.4.异常处理

因为 JDBC 抛出检查过的异常，比如`SQLException,` ，我们必须把它写在`try-catch`块中。**另一方面，JPA 框架只使用[未检查的异常，像 Hibernate](/web/20221018114335/https://www.baeldung.com/hibernate-exceptions)T5。因此，我们不需要在每个使用它们的地方都捕捉或声明它们。**

### 4.5.表演

JPA 和 JDBC 的本质区别在于谁来编码:JPA 框架还是本地开发者。无论哪种方式，我们都必须处理[对象-关系阻抗不匹配](https://web.archive.org/web/20221018114335/https://en.wikipedia.org/wiki/Object%E2%80%93relational_impedance_mismatch)。

公平地说，当我们错误地编写 SQL 查询时，JDBC 的性能会非常缓慢。在选择这两种技术时，性能不应该成为争论的焦点。专业开发人员完全有能力开发出无论使用何种技术都能运行良好的 Java 应用程序。

### 4.6.JDBC 属地

基于 JPA 的应用程序仍然使用 JDBC。因此，当我们利用 JPA 时，我们的代码实际上使用 JDBC API 进行所有的数据库交互。换句话说， **JPA 充当了一个抽象层，它对开发人员隐藏了底层的 JDBC 调用，使得数据库编程变得相当容易**。

### 4.7.事务管理

在 JDBC 中，事务管理是通过使用提交和回滚显式处理的。另一方面，**事务管理是在 JPA** 中隐式提供的。

## 5.利弊

**JDBC 相对于 JPA 最明显的好处就是更容易理解**。 另一方面，如果一个开发人员不掌握 JPA 框架的内部运作或者数据库设计，他们将无法写出好的代码 。

还有， JPA 被很多开发者认为更适合更复杂的应用 。但是， JDBC 被认为是 更好的选择，如果一个应用程序将使用一个简单的数据库，并且我们不打算将它迁移到不同的数据库供应商。

对于开发人员来说，JPA 相对于 JDBC 的主要优势在于，他们可以使用面向对象的原则和最佳实践来编写 Java 应用程序，而不必担心数据库语义。因此，**开发可以更快地完成，尤其是当软件开发人员缺乏对 SQL 和关系数据库**的扎实理解时。

此外，因为一个经过良好测试的健壮框架正在处理数据库和 Java 应用程序之间的交互，所以当使用 JPA 时，我们应该会看到来自数据库映射层的错误减少了。

## 6.结论

在这个快速教程中，我们探索了 JPA 和 JDBC 之间的主要区别。

虽然 JPA 带来了许多优势，但是如果 JPA 不能最好地满足我们当前的应用程序需求，我们还有许多其他高质量的替代方案可以使用。