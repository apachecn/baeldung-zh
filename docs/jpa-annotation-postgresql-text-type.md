# PostgreSQL 文本类型的 JPA 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-annotation-postgresql-text-type>

## 1.介绍

在这个快速教程中，**我们将解释如何使用由 [JPA](/web/20221126105445/https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa) 规范**定义的注释来管理 PostgreSQL 文本类型。

## 2.PostgreSQL 中的文本类型

使用 PostgresSQL 时，我们可能需要定期存储任意长度的字符串。

为此，PostgreSQL 提供了三种字符类型:

*   字符
*   可变字符
*   文本

不幸的是，文本类型不属于 SQL 标准管理的类型。这意味着如果我们想在我们的持久化实体中使用 JPA 注释，我们可能会遇到问题。

这是因为 JPA 规范利用了 SQL 标准。因此，它**没有定义一种简单的方法来处理这种类型的对象，例如使用一个`@Text`注释。**

幸运的是，我们有几种管理 PostgreSQL 数据库的文本数据类型的可能性:

*   我们可以使用`@Lob`注释
*   或者，我们也可以结合使用`@Column`注释和`columnDefinition` 属性

现在让我们看看从`@Lob`注释开始的两个解决方案。

## 3.`@Lob`

顾名思义，lob 是一个大的 T2 项目。在数据库术语中， **lob 列用于存储很长的文本或二进制文件**。

我们可以从两种 lob 中选择:

*   CLOB–用于存储文本的字符 LOB
*   BLOB–可用于存储二进制数据的二进制 LOB

**我们可以使用 JPA `@Lob`注释将大型字段映射到大型数据库对象类型。**

当我们在一个`String`类型属性上使用`@Lob`记录时，JPA 规范规定持久性提供者应该使用一个大字符类型对象来存储属性值。因此，PostgreSQL 可以将字符 lob 转换为文本类型。

假设我们有一个简单的`Exam`实体对象，带有一个`description`字段，可以有任意长度:

```java
@Entity
public class Exam {

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;

    @Lob
    private String description;
} 
```

使用描述字段上的`@Lob`注释，**我们指示 Hibernate 使用 PostgreSQL 文本类型来管理这个字段。**

## 4.`@Column`

**管理文本类型的另一个选项是使用`@Column`注释和`columnDefinition`属性。**

让我们再次使用同一个`Exam`实体对象，但这次我们将添加一个文本字段，它可以是任意长度:

```java
@Entity
public class Exam {

    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;

    @Lob
    private String description;

    @Column(columnDefinition="TEXT")
    private String text;

}
```

在这个例子中，我们使用注释`@Column(columnDefinition=”TEXT”)`。使用`columnDefinition`属性允许我们指定在构造这种类型的数据列时将使用的 SQL 片段。

## 5.将这一切结合在一起

在本节中，我们将编写一个简单的单元测试来验证我们的解决方案是否有效:

```java
@Test
public void givenExam_whenSaveExam_thenReturnExpectedExam() {
    Exam exam = new Exam();
    exam.setDescription("This is a description. Sometimes the description can be very very long! ");
    exam.setText("This is a text. Sometimes the text can be very very long!");

    exam = examRepository.save(exam);

    assertEquals(examRepository.find(exam.getId()), exam);
}
```

在本例中，我们首先创建一个新的`Exam`对象，并将其保存到数据库中。然后，我们从数据库中检索`Exam`对象，并将结果与我们创建的原始考试进行比较。

为了证明这一点，如果我们快速修改我们的`Exam`实体上的描述字段:

```java
@Column(length = 20)
private String description; 
```

当我们再次运行测试时，我们会看到一个错误:

```java
ERROR o.h.e.jdbc.spi.SqlExceptionHelper - Value too long for column "TEXT VARCHAR(20)"
```

## 6.结论

在本教程中，我们介绍了在 PostgreSQL 文本类型中使用 JPA 注释的两种方法。

我们首先解释了文本类型的用途，然后我们看到了如何使用 JPA 注释`@Lob`和`@Column` **来保存使用 PostgreSQL 定义的文本类型的`String`对象。**

和往常一样，这篇文章的完整源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221126105445/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa-2)