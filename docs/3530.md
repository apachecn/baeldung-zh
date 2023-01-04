# SqlResultSetMapping 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-sql-resultset-mapping>

 ![](img/1467ba2fc663f26d1a6310152a062bfe.png)

JPA can behave very differently depending on the exact circumstances under which it is used. Code that works in our local environment or in staging performs very poorly (or even flat out fails) when thrown against real-scale databases in production environments.

Debugging these JPA issues in production is pretty difficult - existing APMs don’t provide enough granular insights at the code level, and tracking every single place someone queried entities one by one instead of in bulk can be a grueling, time-consuming task.

**Lightrun** is a new approach to debugging in production. Using Lightrun’s Logs and Snapshots, you can now get debugger-level granularity in production without opening inbound ports, redeploying, restarting, or even stropping the running application.

In addition, instrumenting Lightrun Metrics at runtime allows you to track down persistence issues securely and in real-time. Want to see it in action? **Check out our 2-minute tutorial** for debugging JPA performance issues in production using Lightrun:

[>> Debugging Spring Persistence and JPA Issues Using Lightrun](/web/20220523135020/https://www.baeldung.com/lightrun-n-jpa)

## 1.介绍

在本指南中，我们将看看 Java 持久性 API (JPA)中的`SqlResultSetMapping`。

这里的核心功能包括将数据库 SQL 语句的结果集映射到 Java 对象。

## 2.设置

在我们看它的用法之前，让我们做一些设置。

### 2.1.Maven 依赖性

我们需要的 Maven 依赖项是 Hibernate 和 H2 数据库。 [Hibernate](https://web.archive.org/web/20220523135020/http://hibernate.org/) 给了我们 JPA 规范的实现。我们使用 [H2 数据库](https://web.archive.org/web/20220523135020/http://www.h2database.com/html/main.html)作为内存数据库。

### 2.2.数据库ˌ资料库

接下来，我们将创建两个表，如下所示:

```
CREATE TABLE EMPLOYEE
(id BIGINT,
 name VARCHAR(10));
```

`EMPLOYEE `表存储一个结果`Entity`对象。`SCHEDULE_DAYS `包含通过`employeeId:`列链接到`EMPLOYEE`表的记录

```
CREATE TABLE SCHEDULE_DAYS
(id IDENTITY,
 employeeId BIGINT,
 dayOfWeek  VARCHAR(10));
```

数据创建的脚本可在本指南的代码中找到。

### 2.3.实体对象

我们的`Entity `对象应该看起来相似:

```
@Entity
public class Employee {
    @Id
    private Long id;
    private String name;
}
```

对象的命名可能不同于数据库表。我们可以用@ `Table `来注释这个类，以显式地映射它们:

```
@Entity
@Table(name = "SCHEDULE_DAYS")
public class ScheduledDay {

    @Id
    @GeneratedValue
    private Long id;
    private Long employeeId;
    private String dayOfWeek;
}
```

## 3.标量映射

现在我们有了数据，可以开始映射查询结果了。

### 3.1.`ColumnResult`

虽然`SqlResultSetMapping`和`Query` 注释也适用于`Repository` 类，但是在这个例子中，我们使用的是`Entity `类的注释。

每个`SqlResultSetMapping`注释只需要一个属性，`name. `,但是，如果没有一个成员类型，什么都不会被映射。成员类型有`ColumnResult`、`ConstructorResult`和`EntityResult`。

在这种情况下，` ColumnResult `将任何列映射到标量结果类型:

```
@SqlResultSetMapping(
  name="FridayEmployeeResult",
  columns={@ColumnResult(name="employeeId")})
```

`ColumnResult `属性`name`标识查询中的列:

```
@NamedNativeQuery(
  name = "FridayEmployees",
  query = "SELECT employeeId FROM schedule_days WHERE dayOfWeek = 'FRIDAY'",
  resultSetMapping = "FridayEmployeeResult") 
```

注意，`NamedNativeQuery`注释**中`resultSetMapping`** 的值**很重要，因为它与我们`ResultSetMapping` 声明中的`name`属性相匹配。**

因此，`NamedNativeQuery`结果集按照预期被映射。同样，`StoredProcedure` API 也需要这种关联。

### 3.2.`ColumnResult`测试

我们需要一些特定于 Hibernate 的对象来运行我们的代码:

```
@BeforeAll
public static void setup() {
    emFactory = Persistence.createEntityManagerFactory("java-jpa-scheduled-day");
    em = emFactory.createEntityManager();
}
```

最后，我们调用命名查询来运行我们的测试:

```
@Test
public void whenNamedQuery_thenColumnResult() {
    List<Long> employeeIds = em.createNamedQuery("FridayEmployees").getResultList();
    assertEquals(2, employeeIds.size());
}
```

## 4.构造函数映射

让我们看看什么时候需要将结果集映射到整个对象。

### 4.1.`ConstructorResult`

类似于我们的`ColumnResult`示例，我们将在我们的`Entity`类`ScheduledDay`上添加`SqlResultMapping`注释。但是，为了使用构造函数进行映射，我们需要创建一个:

```
public ScheduledDay (
  Long id, Long employeeId, 
  Integer hourIn, Integer hourOut, 
  String dayofWeek) {
    this.id = id;
    this.employeeId = employeeId;
    this.dayOfWeek = dayofWeek;
}
```

此外，映射指定了目标类和列(两者都是必需的):

```
@SqlResultSetMapping(
    name="ScheduleResult",
    classes={
      @ConstructorResult(
        targetClass=com.baeldung.sqlresultsetmapping.ScheduledDay.class,
        columns={
          @ColumnResult(name="id", type=Long.class),
          @ColumnResult(name="employeeId", type=Long.class),
          @ColumnResult(name="dayOfWeek")})})
```

**`ColumnResults`的顺序非常重要。**如果列顺序错误，构造函数将无法被识别。在我们的示例中，排序与表列相匹配，因此实际上并不需要。

```
@NamedNativeQuery(name = "Schedules",
  query = "SELECT * FROM schedule_days WHERE employeeId = 8",
  resultSetMapping = "ScheduleResult")
```

`ConstructorResult `的另一个独特的区别是，结果对象实例化为“新的”或“分离的”。当匹配的主键存在于`EntityManager` 中时，映射的`Entity`将处于分离状态，否则它将是新的。

有时，由于 SQL 数据类型与 Java 数据类型不匹配，我们可能会遇到运行时错误。所以我们可以用`type.`显式声明

### 4.2.`ConstructorResult`测试

让我们在单元测试中测试`ConstructorResult`:

```
@Test
public void whenNamedQuery_thenConstructorResult() {
  List<ScheduledDay> scheduleDays
    = Collections.checkedList(
      em.createNamedQuery("Schedules", ScheduledDay.class).getResultList(), ScheduledDay.class);
    assertEquals(3, scheduleDays.size());
    assertTrue(scheduleDays.stream().allMatch(c -> c.getEmployeeId().longValue() == 3));
}
```

## 5.实体映射

最后，对于一个代码较少的简单实体映射，我们来看看`EntityResult`。

### 5.1.单一实体

`EntityResult` 要求我们指定实体类，`Employee`。我们使用可选的`fields `属性进行更多的控制。结合`FieldResult,` ，我们可以映射不匹配的别名和字段:

```
@SqlResultSetMapping(
  name="EmployeeResult",
  entities={
    @EntityResult(
      entityClass = com.baeldung.sqlresultsetmapping.Employee.class,
        fields={
          @FieldResult(name="id",column="employeeNumber"),
          @FieldResult(name="name", column="name")})})
```

现在，我们的查询应该包括别名列:

```
@NamedNativeQuery(
  name="Employees",
  query="SELECT id as employeeNumber, name FROM EMPLOYEE",
  resultSetMapping = "EmployeeResult")
```

与`ConstructorResult`类似，`EntityResult`需要一个构造函数。然而，这里有一个默认的。

### 5.2.多个实体

一旦我们映射了一个实体，映射多个实体就非常简单了:

```
@SqlResultSetMapping(
  name = "EmployeeScheduleResults",
  entities = {
    @EntityResult(entityClass = com.baeldung.sqlresultsetmapping.Employee.class),
    @EntityResult(entityClass = com.baeldung.sqlresultsetmapping.ScheduledDay.class)
```

### 5.3.`EntityResult`测试

让我们来看看`EntityResult`的行动:

```
@Test
public void whenNamedQuery_thenSingleEntityResult() {
    List<Employee> employees = Collections.checkedList(
      em.createNamedQuery("Employees").getResultList(), Employee.class);
    assertEquals(3, employees.size());
    assertTrue(employees.stream().allMatch(c -> c.getClass() == Employee.class));
}
```

因为多个实体结果连接两个实体，所以只在其中一个类上的查询注释是令人困惑的。

因此，我们在测试中定义了查询:

```
@Test
public void whenNamedQuery_thenMultipleEntityResult() {
    Query query = em.createNativeQuery(
      "SELECT e.id, e.name, d.id, d.employeeId, d.dayOfWeek "
        + " FROM employee e, schedule_days d "
        + " WHERE e.id = d.employeeId", "EmployeeScheduleResults");

    List<Object[]> results = query.getResultList();
    assertEquals(4, results.size());
    assertTrue(results.get(0).length == 2);

    Employee emp = (Employee) results.get(1)[0];
    ScheduledDay day = (ScheduledDay) results.get(1)[1];

    assertTrue(day.getEmployeeId() == emp.getId());
}
```

## 6.结论

在本指南中，我们看了使用`SqlResultSetMapping` 注释`. ` `SqlResultSetMapping `的不同选项，它是 Java 持久性 API 的关键部分。

代码片段可以在 GitHub 上找到[。](https://web.archive.org/web/20220523135020/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa)