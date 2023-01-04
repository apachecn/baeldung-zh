# StreamEx 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/streamex>

## 1。概述

Java 8 最令人兴奋的特性之一是 [`Stream` API](/web/20221129013614/https://www.baeldung.com/java-8-streams) ，简单来说，它是处理元素序列的强大工具。

`[StreamEx](https://web.archive.org/web/20221129013614/https://amaembo.github.io/streamex/javadoc/)`是一个为标准流 API 提供额外功能以及性能改进的库。

以下是一些核心特性:

*   完成日常任务的更短更方便的方法
*   与原装 JDK 100%兼容`Streams`
*   并行处理的友好性:任何新特性都尽可能利用并行流的优势
*   性能和最小的开销。如果`StreamEx`允许使用比标准`Stream,`更少的代码来解决任务，它应该不会比通常的方式慢很多(有时甚至更快)

在本教程中，我们将展示`StreamEx` API 的一些特性。

## 2。设置示例

要使用`StreamEx`，我们需要向`pom.xml`添加以下依赖项:

```java
<dependency>
    <groupId>one.util</groupId>
    <artifactId>streamex</artifactId>
    <version>0.6.5</version>
</dependency>
```

该库的最新版本可以在 [Maven Central](https://web.archive.org/web/20221129013614/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22one.util%22%20AND%20a%3A%22streamex%22) 上找到。

在本教程中，我们将使用一个简单的`User`类:

```java
public class User {
    int id;
    String name;
    Role role = new Role();

    // standard getters, setters, and constructors
}
```

和一个简单的`Role`类:

```java
public class Role {
}
```

## 3。收集器快捷方式

`Streams`最流行的终端操作之一是`collect`操作；这允许重新包装`Stream`元素到我们选择的集合中。

问题是对于简单的场景，代码会变得不必要地冗长:

```java
users.stream()
  .map(User::getName)
  .collect(Collectors.toList());
```

### 3.1.收集到一个集合

现在，有了 StreamEx，我们不需要提供一个`Collector`来指定我们需要一个`List`、 `Set, Map, InmutableList,` 等等。：

```java
List<String> userNames = StreamEx.of(users)
  .map(User::getName)
  .toList();
```

**如果我们想执行比从`Stream`中提取元素并将其放入集合中更复杂的事情，那么`collect`操作在 API 中仍然可用。**

### 3.2.高级收集器

另一个简写是`groupingBy`:

```java
Map<Role, List<User>> role2users = StreamEx.of(users)
  .groupingBy(User::getRole);
```

这将产生一个带有方法引用中指定的键类型的`Map`,类似于 SQL 中的 group by 操作。

使用普通的`Stream` API，我们需要编写:

```java
Map<Role, List<User>> role2users = users.stream()
  .collect(Collectors.groupingBy(User::getRole));
```

类似的简写形式可以在`Collectors.joining():`中找到

```java
StreamEx.of(1, 2, 3)
  .joining("; "); // "1; 2; 3"
```

它接受`Stream` a 中的所有元素，产生一个`String`将它们连接起来。

## 4。添加、删除和选择元素

在某些场景中，**我们得到了一个不同类型的对象列表，我们需要根据类型对它们进行过滤:**

```java
List usersAndRoles = Arrays.asList(new User(), new Role());
List<Role> roles = StreamEx.of(usersAndRoles)
  .select(Role.class)
  .toList();
```

**我们可以用这个简便的操作将元素添加到我们的** `**Stream**,`的开头或结尾:

```java
List<String> appendedUsers = StreamEx.of(users)
  .map(User::getName)
  .prepend("(none)")
  .append("LAST")
  .toList();
```

**我们可以使用`nonNull()`** 删除不需要的空元素，并将`Stream`用作`Iterable`:

```java
for (String line : StreamEx.of(users).map(User::getName).nonNull()) {
    System.out.println(line);
}
```

## 5。数学运算和原始类型支持

`StreamEx`增加了对基本类型的支持，正如我们在这个不言自明的例子中看到的:

```java
short[] src = {1,2,3};
char[] output = IntStreamEx.of(src)
  .map(x -> x * 5)
  .toCharArray();
```

现在让我们以无序的方式获取一个由`double`元素组成的数组。我们希望创建一个数组，其中包含每对之间的差异。

我们可以使用`pairMap`方法来执行这个操作:

```java
public double[] getDiffBetweenPairs(double... numbers) {
    return DoubleStreamEx.of(numbers)
      .pairMap((a, b) -> b - a)
      .toArray();
}
```

## 6.地图操作

### 6.1.按键过滤

另一个有用的特性是能够从一个`Map`创建一个`Stream`，并通过使用元素所指向的值来过滤元素。

在这种情况下，我们采用所有非空值:

```java
Map<String, Role> nameToRole = new HashMap<>();
nameToRole.put("first", new Role());
nameToRole.put("second", null);
Set<String> nonNullRoles = StreamEx.ofKeys(nameToRole, Objects::nonNull)
  .toSet();
```

### 6.2.对键值对进行操作

我们还可以通过创建一个`EntryStream`实例来操作键值对:

```java
public Map<User, List<Role>> transformMap( 
    Map<Role, List<User>> role2users) {
    Map<User, List<Role>> users2roles = EntryStream.of(role2users)
     .flatMapValues(List::stream)
     .invert()
     .grouping();
    return users2roles;
}
```

特殊操作`EntryStream.of`获取一个`Map`,并将其转换为键值对象的`Stream`。然后，我们使用`flatMapValues`操作将我们的角色列表转换为单值的`Stream`。

接下来，我们可以`invert`键-值对，使`User` 类成为键，`Role`类成为值。

最后，我们可以使用`grouping`操作将我们的地图转换为接收到的地图的反转，所有操作只需要四次。

### 6.3.键值映射

我们还可以独立地映射键和值:

```java
Map<String, String> mapToString = EntryStream.of(users2roles)
  .mapKeys(String::valueOf)
  .mapValues(String::valueOf)
  .toMap();
```

这样，我们可以快速地将我们的键或值转换成另一种需要的类型。

## 7。文件操作

使用`StreamEx`，我们可以高效地读取文件，也就是说，不需要一次加载全部文件。处理大文件时非常方便:

```java
StreamEx.ofLines(reader)
  .remove(String::isEmpty)
  .forEach(System.out::println);
```

注意，我们使用了`remove()`方法来过滤掉空行。

这里要注意的一点是`StreamEx`不会自动关闭文件。因此，我们必须记住在文件读取和写入时手动执行关闭操作，以避免不必要内存开销。

## 8。结论

在本教程中，我们学习了`StreamEx`，它是不同的实用程序。还有更多的事情要做——他们这里有一张方便的小抄。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221129013614/https://github.com/eugenp/tutorials/tree/master/libraries-5)