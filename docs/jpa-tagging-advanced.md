# 用 JPA 实现高级标记

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-tagging-advanced>

[This article is part of a series:](javascript:void(0);)[• A Simple Tagging Implementation with Elasticsearch](/web/20220627174425/https://www.baeldung.com/elasticsearch-tagging)
[• A Simple Tagging Implementation with JPA](/web/20220627174425/https://www.baeldung.com/jpa-tagging)
• An Advanced Tagging Implementation with JPA (current article)[• A Simple Tagging Implementation with MongoDB](/web/20220627174425/https://www.baeldung.com/mongodb-tagging)

## 1。概述

标记是一种设计模式，允许我们对数据执行高级过滤和排序。本文是用 JPA 实现简单标记的[的延续。](/web/20220627174425/https://www.baeldung.com/jpa-tagging)

因此，我们将从那篇文章停止的地方开始，讨论标记的高级用例。

## 2。背书标签

可能最著名的高级标记实现是背书标记。我们可以在 Linkedin 等网站上看到这种模式。

本质上，标签是一个字符串名称和一个数值的组合。然后，我们可以用数字来表示标签被投票或“背书”的次数。

下面是如何创建这种标记的示例:

```java
@Embeddable
public class SkillTag {
    private String name;
    private int value;

    // constructors, getters, setters
}
```

要使用这个标签，我们只需将其中的一个`List`添加到我们的数据对象中:

```java
@ElementCollection
private List<SkillTag> skillTags = new ArrayList<>();
```

我们在上一篇文章中提到过,`@ElementCollection`注释会自动为我们创建一个一对多的映射。

这是这种关系的一个模型用例。因为每个标签都有与存储它的实体相关联的个性化数据，所以我们不能用多对多存储机制来节省空间。

在本文的后面，我们将介绍一个多对多有意义的例子。

因为我们已经将技能标签嵌入到原始实体中，所以我们可以像查询任何其他属性一样查询它。

下面是一个示例查询，用于查找任何拥有超过一定数量背书的学生:

```java
@Query(
  "SELECT s FROM Student s JOIN s.skillTags t WHERE t.name = LOWER(:tagName) AND t.value > :tagValue")
List<Student> retrieveByNameFilterByMinimumSkillTag(
  @Param("tagName") String tagName, @Param("tagValue") int tagValue);
```

接下来，让我们看一个如何使用它的例子:

```java
Student student = new Student(1, "Will");
SkillTag skill1 = new SkillTag("java", 5);
student.setSkillTags(Arrays.asList(skill1));
studentRepository.save(student);

Student student2 = new Student(2, "Joe");
SkillTag skill2 = new SkillTag("java", 1);
student2.setSkillTags(Arrays.asList(skill2));
studentRepository.save(student2);

List<Student> students = 
  studentRepository.retrieveByNameFilterByMinimumSkillTag("java", 3);
assertEquals("size incorrect", 1, students.size());
```

现在，我们可以搜索该标签的存在或者该标签是否有一定数量的背书。

因此，我们可以将它与其他查询参数结合起来创建各种复杂的查询。

## 3。位置标签

另一个流行的标记实现是位置标记。我们可以通过两种主要方式使用位置标签。

首先，它可以用来标记一个地球物理位置。

此外，它还可用于标记照片或视频等媒体中的位置。在所有这些情况下，模型的实现几乎是相同的。

以下是给照片添加标签的示例:

```java
@Embeddable
public class LocationTag {
    private String name;
    private int xPos;
    private int yPos;

    // constructors, getters, setters
}
```

位置标签最值得注意的一点是，仅使用一个数据库来执行地理定位过滤是多么困难。如果我们需要在地理范围内进行搜索，一个更好的方法是将模型加载到一个搜索引擎中(比如 Elasticsearch ),该引擎内置了对地理位置的支持。

因此，我们应该专注于通过标签名称过滤这些位置标签。

该查询看起来类似于上一篇文章中的简单标记实现:

```java
@Query("SELECT s FROM Student s JOIN s.locationTags t WHERE t.name = LOWER(:tag)")
List<Student> retrieveByLocationTag(@Param("tag") String tag);
```

使用位置标签的例子看起来也很熟悉:

```java
Student student = new Student(0, "Steve");
student.setLocationTags(Arrays.asList(new LocationTag("here", 0, 0));
studentRepository.save(student);

Student student2 = studentRepository.retrieveByLocationTag("here").get(0);
assertEquals("name incorrect", "Steve", student2.getName());
```

如果弹性搜索是不可能的，我们仍然需要在地理边界上搜索，使用简单的几何形状将使查询标准更具可读性。

我们将把寻找一个点是在圆内还是在矩形内留给读者作为一个简单的练习。

## 4。键值标签

有时，我们需要存储稍微复杂一些的标签。我们可能希望用键标记的一个小子集来标记一个实体，但是它可以包含各种各样的值。

例如，我们可以用一个`department`标签来标记一个学生，并将其值设置为`Computer Science`。每个学生都有`department`键，但是他们可能都有不同的相关值。

实现看起来类似于上面的标记:

```java
@Embeddable
public class KVTag {
    private String key;
    private String value;

    // constructors, getters and setters
}
```

我们可以像这样将它添加到我们的模型中:

```java
@ElementCollection
private List<KVTag> kvTags = new ArrayList<>();
```

现在我们可以向我们的存储库添加一个新的查询:

```java
@Query("SELECT s FROM Student s JOIN s.kvTags t WHERE t.key = LOWER(:key)")
List<Student> retrieveByKeyTag(@Param("key") String key);
```

我们还可以快速添加一个查询，通过值或者键和值进行搜索。这让我们在搜索数据时更加灵活。

让我们对此进行测试，并验证它是否都工作正常:

```java
@Test
public void givenStudentWithKVTags_whenSave_thenGetByTagOk(){
    Student student = new Student(0, "John");
    student.setKVTags(Arrays.asList(new KVTag("department", "computer science")));
    studentRepository.save(student);

    Student student2 = new Student(1, "James");
    student2.setKVTags(Arrays.asList(new KVTag("department", "humanities")));
    studentRepository.save(student2);

    List<Student> students = studentRepository.retrieveByKeyTag("department");

    assertEquals("size incorrect", 2, students.size());
}
```

遵循这种模式，我们可以设计更复杂的嵌套对象，并在需要时用它们来标记数据。

大多数用例可以用我们今天讨论的高级实现来满足，但是选项是根据需要变得复杂。

## 5。重新实现标记

最后，我们将探索标记的最后一个领域。到目前为止，我们已经看到了如何使用`@ElementCollection`注释使向模型添加标签变得容易。虽然使用起来很简单，但它有一个相当重要的权衡。幕后的一对多实现会导致我们的数据存储中出现大量重复数据。

为了节省空间，我们需要创建另一个表，将我们的`Student`实体连接到我们的`Tag`实体。幸运的是，Spring JPA 将为我们完成大部分繁重的工作。

我们将重新实现我们的`Student`和`Tag`实体，看看这是如何完成的。

### 5.1。定义实体

首先，我们需要重建我们的模型。我们将从一个`ManyStudent`模型开始:

```java
@Entity
public class ManyStudent {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private int id;
    private String name;

    @ManyToMany(cascade = CascadeType.ALL)
    @JoinTable(name = "manystudent_manytags",
      joinColumns = @JoinColumn(name = "manystudent_id", 
      referencedColumnName = "id"),
      inverseJoinColumns = @JoinColumn(name = "manytag_id", 
      referencedColumnName = "id"))
    private Set<ManyTag> manyTags = new HashSet<>();

    // constructors, getters and setters
}
```

这里有几件事需要注意。

首先，我们正在生成我们的 ID，所以表链接更容易在内部管理。

接下来，我们使用`@ManyToMany`注释告诉 Spring 我们想要两个类之间的链接。

最后，我们使用`@JoinTable`注释来建立实际的连接表。

现在我们可以继续我们的新标签模型，我们称之为`ManyTag`:

```java
@Entity
public class ManyTag {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private int id;
    private String name;

    @ManyToMany(mappedBy = "manyTags")
    private Set<ManyStudent> students = new HashSet<>();

    // constructors, getters, setters
}
```

因为我们已经在学生模型中设置了连接表，所以我们所要担心的就是在这个模型中设置引用。

我们使用`mappedBy`属性告诉 JPA 我们想要这个到我们之前创建的连接表的链接。

### 5.2。定义存储库

除了模型之外，我们还需要建立两个存储库:每个实体一个。我们将让 Spring 数据来完成所有繁重的工作:

```java
public interface ManyTagRepository extends JpaRepository<ManyTag, Long> {
}
```

由于我们目前不需要只搜索标签，我们可以将 repository 类留空。

我们的学生资源库只是稍微复杂一点:

```java
public interface ManyStudentRepository extends JpaRepository<ManyStudent, Long> {
    List<ManyStudent> findByManyTags_Name(String name);
}
```

同样，我们让 Spring 数据为我们自动生成查询。

### 5.3。测试

最后，让我们看看这一切在测试中是什么样子的:

```java
@Test
public void givenStudentWithManyTags_whenSave_theyGetByTagOk() {
    ManyTag tag = new ManyTag("full time");
    manyTagRepository.save(tag);

    ManyStudent student = new ManyStudent("John");
    student.setManyTags(Collections.singleton(tag));
    manyStudentRepository.save(student);

    List<ManyStudent> students = manyStudentRepository
      .findByManyTags_Name("full time");

    assertEquals("size incorrect", 1, students.size());
}
```

将标签存储在一个单独的可搜索表中所增加的灵活性远远超过了代码中增加的少量复杂性。

这也允许我们通过移除重复的标签来减少存储在系统中的标签总数。

然而，多对多并不适用于我们希望将特定于实体的状态信息与标签一起存储的情况。

## 6。结论

这篇文章是从上一篇文章停止的地方开始的。

首先，我们介绍了几个在设计标记实现时有用的高级模型。

最后，我们在多对多映射的上下文中重新检查了上一篇文章中标记的实现。

要查看我们今天讨论的工作示例，请查看 GitHub 上的[代码。](https://web.archive.org/web/20220627174425/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-jpa-filtering)

Next **»**[A Simple Tagging Implementation with MongoDB](/web/20220627174425/https://www.baeldung.com/mongodb-tagging)**«** Previous[A Simple Tagging Implementation with JPA](/web/20220627174425/https://www.baeldung.com/jpa-tagging)