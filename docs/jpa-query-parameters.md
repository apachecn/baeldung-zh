# JPA 查询参数用法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-query-parameters>

## 1.介绍

使用 JPA 构建查询并不困难；然而，我们有时会忘记一些简单的事情，而这些事情会产生巨大的影响。

其中之一就是 JPA 查询参数，这也是我们在本教程中要关注的。

## 2.什么是查询参数？

让我们从解释什么是查询参数开始。

查询参数是构建和执行参数化查询的一种方式。因此，与其说:

```
SELECT * FROM employees e WHERE e.emp_number = '123';
```

我们会做:

```
SELECT * FROM employees e WHERE e.emp_number = ?;
```

通过使用 JDBC 预处理语句，我们需要在执行查询之前设置参数:

```
pStatement.setString(1, 123);
```

## 3.为什么要使用查询参数？

我们可以不使用查询参数，而是使用文字，尽管我们不推荐这样做，正如我们现在看到的。

让我们重写前面的查询，使用 JPA API 通过`emp_number`获取雇员，但是我们将使用文字而不是参数，这样我们可以清楚地说明这种情况:

```
String empNumber = "A123";
TypedQuery<Employee> query = em.createQuery(
  "SELECT e FROM Employee e WHERE e.empNumber = '" + empNumber + "'", Employee.class);
Employee employee = query.getSingleResult();
```

这种方法有一些缺点:

*   **嵌入参数引入了安全风险，使我们容易受到 [JPQL 注入攻击](/web/20221206104402/https://www.baeldung.com/sql-injection)。攻击者可能会注入任何意外的、可能有危险的 JPQL 表达式，而不是预期的值。**
*   根据我们使用的 JPA 实现和应用程序的启发，查询缓存可能会耗尽。每次我们使用新的值/参数时，可能会构建、编译和缓存一个新的查询。最起码不会有效率，还可能导致意想不到的`OutOfMemoryError.`

## 4.JPA 查询参数

与 JDBC 准备的语句参数类似，JPA 指定了两种不同的方式来编写参数化查询，即使用:

*   位置参数
*   命名参数

我们可以使用位置参数或命名参数，但不能在同一个查询中混合使用。

### 4.1.位置参数

使用位置参数是避免前面提到的问题的一种方法。

让我们看看如何在位置参数的帮助下编写这样的查询:

```
TypedQuery<Employee> query = em.createQuery(
  "SELECT e FROM Employee e WHERE e.empNumber = ?1", Employee.class);
String empNumber = "A123";
Employee employee = query.setParameter(1, empNumber).getSingleResult();
```

正如我们在前面的例子中看到的，**我们通过键入一个问号，后跟一个正整数**，在查询中声明这些参数。我们将从`1`开始并继续向前，每次递增 1。

我们可能在同一个查询中多次使用同一个参数，这使得这些参数更类似于命名参数。

参数编号是一个非常有用的特性，因为它提高了可用性、可读性和可维护性。

值得一提的是，**原生 SQL 查询也支持位置参数绑定**。

### 4.2.集合值位置参数

如前所述，我们也可以使用集值参数:

```
TypedQuery<Employee> query = entityManager.createQuery(
  "SELECT e FROM Employee e WHERE e.empNumber IN (?1)" , Employee.class);
List<String> empNumbers = Arrays.asList("A123", "A124");
List<Employee> employees = query.setParameter(1, empNumbers).getResultList();
```

### 4.3.命名参数

命名参数与位置参数非常相似；但是，通过使用它们，我们使参数更加明确，查询变得更加易读:

```
TypedQuery<Employee> query = em.createQuery(
  "SELECT e FROM Employee e WHERE e.empNumber = :number" , Employee.class);
String empNumber = "A123";
Employee employee = query.setParameter("number", empNumber).getSingleResult();
```

前面的示例查询与第一个相同，但是我们使用了命名参数`:number`，而不是`?1`。

我们可以看到，我们用冒号声明了参数，后跟一个字符串标识符(JPQL 标识符)，这是我们将在运行时设置的实际值的占位符。在执行查询之前，我们必须通过发出`setParameter`方法来设置一个或多个参数。

值得注意的一件有趣的事情是， **`TypedQuery`支持方法链接，**在需要设置多个参数时非常有用。

让我们继续使用两个命名参数来创建前面查询的变体，以说明方法链接:

```
TypedQuery<Employee> query = em.createQuery(
  "SELECT e FROM Employee e WHERE e.name = :name AND e.age = :empAge" , Employee.class);
String empName = "John Doe";
int empAge = 55;
List<Employee> employees = query
  .setParameter("name", empName)
  .setParameter("empAge", empAge)
  .getResultList();
```

这里我们检索具有给定姓名和年龄的所有雇员。正如我们清楚地看到的，人们可能会期望，我们可以用**多个参数构建查询，并根据需要尽可能多地使用它们。**

如果出于某种原因，我们确实需要在同一个查询中多次使用同一个参数，我们只需要通过发出“`setParameter`”方法来设置它一次。在运行时，指定的值将替换参数的每次出现。

最后，值得一提的是**Java 持久性 API 规范并没有强制要求本地查询支持命名参数**。即使某些实现，比如 Hibernate，支持它，我们也需要考虑到，如果我们使用它，查询将不会是可移植的。

### 4.4.集合值命名参数

为了清楚起见，让我们也演示一下这是如何与集合值参数一起工作的:

```
TypedQuery<Employee> query = entityManager.createQuery(
  "SELECT e FROM Employee e WHERE e.empNumber IN (:numbers)" , Employee.class);
List<String> empNumbers = Arrays.asList("A123", "A124");
List<Employee> employees = query.setParameter("numbers", empNumbers).getResultList();
```

正如我们所见，它的工作方式与位置参数相似。

## 5.标准查询参数

JPA 查询可以通过使用[JPA Criteria API](/web/20221206104402/https://www.baeldung.com/hibernate-criteria-queries-metamodel)来构建， [Hibernate 的官方文档](https://web.archive.org/web/20221206104402/https://docs.jboss.org/hibernate/orm/5.2/topical/html_single/metamodelgen/MetamodelGenerator.html)对此有非常详细的解释。

在这种类型的查询中，我们通过使用对象而不是名称或索引来表示参数。

让我们再次构建相同的查询，但是这次使用 Criteria API 来演示在处理`CriteriaQuery`时如何处理查询参数:

```
CriteriaBuilder cb = em.getCriteriaBuilder();

CriteriaQuery<Employee> cQuery = cb.createQuery(Employee.class);
Root<Employee> c = cQuery.from(Employee.class);
ParameterExpression<String> paramEmpNumber = cb.parameter(String.class);
cQuery.select(c).where(cb.equal(c.get(Employee_.empNumber), paramEmpNumber));

TypedQuery<Employee> query = em.createQuery(cQuery);
String empNumber = "A123";
query.setParameter(paramEmpNumber, empNumber);
Employee employee = query.getResultList();
```

对于这种类型的查询，参数的机制有一点不同，因为我们使用了参数对象，但本质上没有区别。

在前面的例子中，我们可以看到`Employee_`类的用法。我们用 Hibernate 元模型生成器生成了这个类。这些组件是静态 JPA 元模型的一部分，它允许以强类型方式构建条件查询。

## 6.结论

在本文中，我们重点关注了使用 JPA 查询参数或输入参数构建查询的机制。

我们了解到我们有两种类型的查询参数，位置的和命名的，由我们决定哪一种最适合我们的目标。

同样值得注意的是，所有查询参数必须是单值的，除了`in`表达式。对于`in`表达式，我们可以使用集合值输入参数，比如数组或`List`,如前面的例子所示。

和往常一样，这篇文章的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221206104402/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa-2)