# 用 JPA 实现简单的标记

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-tagging>

[This article is part of a series:](javascript:void(0);)[• A Simple Tagging Implementation with Elasticsearch](/web/20220628150935/https://www.baeldung.com/elasticsearch-tagging)
• A Simple Tagging Implementation with JPA (current article)[• An Advanced Tagging Implementation with JPA](/web/20220628150935/https://www.baeldung.com/jpa-tagging-advanced)
[• A Simple Tagging Implementation with MongoDB](/web/20220628150935/https://www.baeldung.com/mongodb-tagging)

## 1。概述

标记是一种标准的设计模式，它允许我们对数据模型中的项目进行分类和过滤。

在本文中，我们将使用 Spring 和 JPA 实现标记。我们将使用 Spring 数据来完成这项任务。此外，如果您想使用 Hibernate，这个实现将非常有用。

这是实现标记系列的第二篇文章。要了解如何用 Elasticsearch 实现它，请点击[这里](/web/20220628150935/https://www.baeldung.com/elasticsearch-tagging)。

## 2。添加标签

首先，我们将探索最简单的标记实现:字符串列表。我们可以通过向实体添加一个新字段来实现标记，如下所示:

```java
@Entity
public class Student {
    // ...

    @ElementCollection
    private List<String> tags = new ArrayList<>();

    // ...
}
```

注意在我们的新字段中使用了`ElementCollection`注释。因为我们在数据存储前面运行，所以我们需要告诉它如何存储我们的标签。

如果我们不添加注释，它们将被存储在一个单独的 blob 中，这将更难处理。这个注释创建了另一个名为`STUDENT_TAGS`(即`<entity>_<field>`)的表，这将使我们的查询更加健壮。

这在我们的实体和标签之间建立了一对多的关系！我们在这里实现了最简单的标记版本。因此，我们可能会有很多重复的标签(每个实体一个)。这个概念我们以后再讲。

## 3。建筑查询

标签允许我们对数据执行一些有趣的查询。我们可以搜索带有特定标签的实体，过滤表扫描，甚至限制特定查询返回的结果。让我们来看看这些案例中每一个。

### 3.1。搜索标签

我们添加到数据模型中的`tag`字段可以像模型中的其他字段一样进行搜索。在构建查询时，我们将标签保存在单独的表中。

下面是我们如何搜索包含特定标签的实体:

```java
@Query("SELECT s FROM Student s JOIN s.tags t WHERE t = LOWER(:tag)")
List<Student> retrieveByTag(@Param("tag") String tag);
```

因为标签存储在另一个表中，所以我们需要在查询中连接它们——这将返回所有带有匹配标签的`Student`实体。

首先，让我们设置一些测试数据:

```java
Student student = new Student(0, "Larry");
student.setTags(Arrays.asList("full time", "computer science"));
studentRepository.save(student);

Student student2 = new Student(1, "Curly");
student2.setTags(Arrays.asList("part time", "rocket science"));
studentRepository.save(student2);

Student student3 = new Student(2, "Moe");
student3.setTags(Arrays.asList("full time", "philosophy"));
studentRepository.save(student3);

Student student4 = new Student(3, "Shemp");
student4.setTags(Arrays.asList("part time", "mathematics"));
studentRepository.save(student4);
```

接下来，让我们测试一下，确保它能正常工作:

```java
// Grab only the first result
Student student2 = studentRepository.retrieveByTag("full time").get(0);
assertEquals("name incorrect", "Larry", student2.getName());
```

我们将取回存储库中第一个带有`full time`标签的学生。这正是我们想要的。

此外，我们可以扩展这个例子来展示如何过滤更大的数据集。下面是一个例子:

```java
List<Student> students = studentRepository.retrieveByTag("full time");
assertEquals("size incorrect", 2, students.size());
```

通过一点点重构，我们可以修改存储库以接受多个标签作为过滤器，这样我们就可以进一步细化我们的结果。

### 3.2。过滤查询

简单标记的另一个有用的应用是对特定的查询应用过滤器。虽然前面的例子也允许我们进行过滤，但是它们处理的是表中的所有数据。

因为我们还需要过滤其他搜索，所以让我们看一个例子:

```java
@Query("SELECT s FROM Student s JOIN s.tags t WHERE s.name = LOWER(:name) AND t = LOWER(:tag)")
List<Student> retrieveByNameFilterByTag(@Param("name") String name, @Param("tag") String tag);
```

我们可以看到，这个查询与上面的查询几乎相同。一个`tag`只不过是我们查询中使用的另一个约束。

我们的使用示例看起来也很熟悉:

```java
Student student2 = studentRepository.retrieveByNameFilterByTag(
  "Moe", "full time").get(0);
assertEquals("name incorrect", "moe", student2.getName());
```

因此，我们可以将标签`filter`应用于这个实体上的任何查询。这给了用户在界面中找到他们需要的准确数据的很大权力。

## 4。高级标记

我们简单的标记实现是一个很好的起点。但是，由于一对多的关系，我们可能会遇到一些问题。

首先，我们将得到一个充满重复标签的表格。这对于小型项目来说不是问题，但是大型系统可能会有数百万(甚至数十亿)的重复条目。

此外，我们的`Tag`模型不是很健壮。如果我们想跟踪标签最初是什么时候创建的呢？在我们当前的实现中，我们没有办法做到这一点。

最后，我们不能跨多个实体类型共享我们的`tags`。这会导致更多的重复，从而影响我们的系统性能。

**多对多关系将解决我们的大多数问题。**要了解如何使用`@manytomany`注释，请查看[这篇文章](/web/20220628150935/https://www.baeldung.com/hibernate-many-to-many)(因为这超出了本文的范围)。

## 5。结论

标记是一种能够查询数据的简单明了的方法，结合 Java 持久性 API，我们得到了一个易于实现的强大过滤功能。

尽管简单的实现可能并不总是最合适的，但我们已经强调了帮助解决这种情况的途径。

和往常一样，本文中使用的代码可以在 GitHub 上的[中找到。](https://web.archive.org/web/20220628150935/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-filtering)

Next **»**[An Advanced Tagging Implementation with JPA](/web/20220628150935/https://www.baeldung.com/jpa-tagging-advanced)**«** Previous[A Simple Tagging Implementation with Elasticsearch](/web/20220628150935/https://www.baeldung.com/elasticsearch-tagging)