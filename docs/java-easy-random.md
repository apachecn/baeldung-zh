# Java easy random 快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-easy-random>

## 1.概观

在本教程中，我们将展示如何用 [EasyRandom](https://web.archive.org/web/20220628235038/https://github.com/j-easy/easy-random) 库生成 Java 对象。

## 2.易随机

在某些情况下，我们需要一组用于测试目的的模型对象。或者，我们想用一些我们将要使用的数据填充我们的测试数据库。然后，也许我们会希望将虚拟 dto 集合发送回我们的客户端。

如果不复杂，设置一个、两个或几个这样的对象可能很容易。然而，可能会有这样的情况，我们会立即需要数百个，而不用手动设置。

这就是 easy random 介入的地方。EasyRandom 是一个**库，易于使用，几乎不需要设置，只需绕过类类型，它将为我们实例化整个对象图。**

让我们看看有多简单。

## 3.Maven 依赖性

首先，让我们将[`easy-random-core`Maven 依赖](https://web.archive.org/web/20220628235038/https://mvnrepository.com/artifact/org.jeasy/easy-random-core/4.0.0)添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.jeasy</groupId>
    <artifactId>easy-random-core</artifactId>
    <version>4.0.0</version>
</dependency>
```

## 4.对象生成

库中最重要的两个类是:

*   这将生成对象，并且
*   这使我们能够配置生成过程，并使其更可预测。

### 4.1.单一对象

我们的第一个例子生成了一个简单的随机`Person`对象，它没有嵌套对象，没有集合，只有一个`Integer`和两个`Strings`。

让我们用`nextObject(Class<T> t)`**生成我们对象的一个实例:**

```java
@Test
void givenDefaultConfiguration_thenGenerateSingleObject() {
    EasyRandom generator = new EasyRandom();
    Person person = generator.nextObject(Person.class);

    assertNotNull(person.getAge());
    assertNotNull(person.getFirstName());
    assertNotNull(person.getLastName());
}
```

这是对象在生成后的样子:

```java
Person[firstName='eOMtThyhVNLWUZNRcBaQKxI', lastName='yedUsFwdkelQbxeTeQOvaScfqIOOmaa', age=-1188957731]
```

正如我们所看到的，生成的字符串可能有点太长，并且 age 是负数。我们将在接下来的章节中展示如何对此进行调整。

### 4.2.对象的集合

现在，假设我们需要一组`Person`对象。另一种方法，`objects(Class<T> t, int size)`将允许我们这样做。

一件好事是，它返回对象流，所以最终，**我们可以添加中间操作**到它或我们想要的组中。

下面是我们如何让**生成五个`Person`** 的实例:

```java
@Test
void givenDefaultConfiguration_thenGenerateObjectsList() {
    EasyRandom generator = new EasyRandom();
    List<Person> persons = generator.objects(Person.class, 5)
        .collect(Collectors.toList());

    assertEquals(5, persons.size());
}
```

### 4.3.复杂对象生成

让我们来看看我们的`Employee`课:

```java
public class Employee {
    private long id;
    private String firstName;
    private String lastName;
    private Department department;
    private Collection<Employee> coworkers;
    private Map<YearQuarter, Grade> quarterGrades;
}
```

我们的类相对复杂，它有一个嵌套对象、一个集合和一个映射。

默认情况下，**集合的生成范围是从 1 到 100** ，所以我们的`Collection<Employee>` 大小将介于两者之间。

一件好事是，**对象将被缓存并重用**，因此不一定所有对象都是唯一的。尽管如此，我们可能不需要这么多。

我们很快就会看到如何调整集合的范围，但首先，让我们看看我们可能会遇到的另一个问题。

在我们的领域中，我们有一个代表一年中一个季度的`YearQuarter`类。

**将`endDate`设置为正好指向开始日期**后的 3 个月，这有点逻辑:

```java
public class YearQuarter {

    private LocalDate startDate;
    private LocalDate endDate;

    public YearQuarter(LocalDate startDate) {
        this.startDate = startDate;
        autoAdjustEndDate();
    }

    private void autoAdjustEndDate() {
        endDate = startDate.plusMonths(3L);
    }
}
```

我们必须注意到， **EasyRandom 使用反射来构造我们的对象**，所以通过库生成这个对象将导致数据，很可能，**对我们没有用，因为我们 3 个月的约束将不会被保留**。

让我们看看如何解决这个问题。

### 4.4.发电配置

在下面的配置中，我们通过 `**EasyRandomParameters**.`提供我们的**自定义配置**

首先，我们显式地声明我们想要的字符串长度和集合大小。接下来，我们从生成中排除一些字段，假设我们有理由只使用空值。

这里，我们使用了方便的`FieldPredicates`实用程序来链接排除谓词。

之后，我们通过另一个方便的`TypePredicates` 实用程序从`“not.existing.pkg”` Java 包中排除所有东西。

最后，**按照承诺，我们通过应用我们的自定义** `**YearQuarterRandomizer:**`来解决`YearQuarter`类的`startDate` 和 `endDate`代的问题

```java
@Test
void givenCustomConfiguration_thenGenerateSingleEmployee() {
    EasyRandomParameters parameters = new EasyRandomParameters();
    parameters.stringLengthRange(3, 3);
    parameters.collectionSizeRange(5, 5);
    parameters.excludeField(FieldPredicates.named("lastName").and(FieldPredicates.inClass(Employee.class)));
    parameters.excludeType(TypePredicates.inPackage("not.existing.pkg"));
    parameters.randomize(YearQuarter.class, new YearQuarterRandomizer());

    EasyRandom generator = new EasyRandom(parameters);
    Employee employee = generator.nextObject(Employee.class);

    assertEquals(3, employee.getFirstName().length());
    assertEquals(5, employee.getCoworkers().size());
    assertEquals(5, employee.getQuarterGrades().size());
    assertNotNull(employee.getDepartment());

    assertNull(employee.getLastName());

    for (YearQuarter key : employee.getQuarterGrades().keySet()) {
        assertEquals(key.getStartDate(), key.getEndDate().minusMonths(3L));
    }
}
```

## 5.结论

手动设置模型、DTO 或实体对象可能很麻烦，并导致代码可读性差和重复。EasyRandom 是一个很好的工具，可以节省时间和帮助它。

正如我们所看到的，这个库不会生成有意义的`String`对象，但是有另一个叫做 [Java Faker](https://web.archive.org/web/20220628235038/https://github.com/DiUS/java-faker) 的工具，我们可以用它为字段创建自定义的随机数发生器来进行排序。

此外，为了更深入地了解这个库，看看它还可以配置多少，我们可以看看它的 [Github Wiki 页面](https://web.archive.org/web/20220628235038/https://github.com/j-easy/easy-random/wiki)。

像往常一样，代码可以在 [GitHub](https://web.archive.org/web/20220628235038/https://github.com/eugenp/tutorials/tree/master/testing-modules/easy-random) 上找到。