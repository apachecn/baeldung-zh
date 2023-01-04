# JPA 中的持久枚举

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-persisting-enums-in-jpa>

## 1.概观

在 JPA 2.0 和更低版本中，没有方便的方法将[枚举](/web/20221205120635/https://www.baeldung.com/a-guide-to-java-enums)值映射到数据库列。每种选择都有其局限性和缺点。使用 JPA 2.1 特性可以避免这些问题。

在本教程中，我们将看看使用 JPA 在数据库中持久化枚举的不同可能性。我们还将描述它们的优缺点，并提供简单的代码示例。

## 2.使用`@Enumerated`注释

**在 2.1 之前的 JPA 中，将枚举值映射到其数据库表示的最常见的选项是使用`@Enumerated`注释。**这样，我们可以指示 JPA 提供者将一个枚举转换成它的序号或`String`值。

我们将在这一部分探讨这两种选择。

但是让我们首先创建一个简单的`@Entity`，我们将在整个教程中使用它:

```java
@Entity
public class Article {
    @Id
    private int id;

    private String title;

    // standard constructors, getters and setters
}
```

### 2.1.映射序数值

**如果我们将`@Enumerated(EnumType.ORDINAL)`注释放在 enum 字段上，JPA 将在持久化数据库中的给定实体时使用`[Enum.ordinal()](https://web.archive.org/web/20221205120635/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Enum.html#ordinal())`值。**

让我们介绍第一个枚举:

```java
public enum Status {
    OPEN, REVIEW, APPROVED, REJECTED;
}
```

接下来，让我们将其添加到`Article`类中，并用`@Enumerated(EnumType.ORDINAL)`对其进行注释:

```java
@Entity
public class Article {
    @Id
    private int id;

    private String title;

    @Enumerated(EnumType.ORDINAL)
    private Status status;
}
```

现在，当持久化一个`Article`实体时:

```java
Article article = new Article();
article.setId(1);
article.setTitle("ordinal title");
article.setStatus(Status.OPEN); 
```

JPA 将触发以下 SQL 语句:

```java
insert 
into
    Article
    (status, title, id) 
values
    (?, ?, ?)
binding parameter [1] as [INTEGER] - [0]
binding parameter [2] as [VARCHAR] - [ordinal title]
binding parameter [3] as [INTEGER] - [1]
```

当我们需要修改我们的枚举时，这种映射会产生一个问题。如果我们在中间添加一个新值或者重新排列枚举的顺序，我们将打破现有的数据模型。

这样的问题可能很难发现，也很难解决，因为我们必须更新所有的数据库记录。

### 2.2.映射字符串值

**类似地，如果我们用`@Enumerated(EnumType.STRING)`注释枚举字段，JPA 将在存储实体时使用`[Enum.name()](https://web.archive.org/web/20221205120635/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Enum.html#name())`值。**

让我们创建第二个枚举:

```java
public enum Type {
    INTERNAL, EXTERNAL;
}
```

让我们将它添加到我们的`Article`类中，并用`@Enumerated(EnumType.STRING)`对其进行注释:

```java
@Entity
public class Article {
    @Id
    private int id;

    private String title;

    @Enumerated(EnumType.ORDINAL)
    private Status status;

    @Enumerated(EnumType.STRING)
    private Type type;
}
```

现在，当持久化一个`Article`实体时:

```java
Article article = new Article();
article.setId(2);
article.setTitle("string title");
article.setType(Type.EXTERNAL);
```

JPA 将执行以下 SQL 语句:

```java
insert 
into
    Article
    (status, title, type, id) 
values
    (?, ?, ?, ?)
binding parameter [1] as [INTEGER] - [null]
binding parameter [2] as [VARCHAR] - [string title]
binding parameter [3] as [VARCHAR] - [EXTERNAL]
binding parameter [4] as [INTEGER] - [2]
```

使用`@Enumerated(EnumType.STRING)`，我们可以安全地添加新的枚举值或改变我们的枚举顺序。**然而，重命名枚举值仍然会破坏数据库数据。**

此外，尽管这种数据表示比`@Enumerated(EnumType.ORDINAL)`选项更具可读性，但它也消耗了更多不必要的空间。当我们需要处理大量数据时，这可能会成为一个重大问题。

## 3.使用`@PostLoad`和`@PrePersist`注释

我们处理数据库中持久枚举的另一个选择是使用标准的 JPA 回调方法。**我们可以在`@PostLoad`和`@PrePersist`事件中来回映射我们的枚举。**

这个想法是在一个实体中有两个属性。第一个映射到数据库值，第二个是保存实际枚举值的`@Transient`字段。然后，业务逻辑代码使用瞬态属性。

为了更好地理解这个概念，让我们创建一个新的 enum，并在映射逻辑中使用它的`int`值:

```java
public enum Priority {
    LOW(100), MEDIUM(200), HIGH(300);

    private int priority;

    private Priority(int priority) {
        this.priority = priority;
    }

    public int getPriority() {
        return priority;
    }

    public static Priority of(int priority) {
        return Stream.of(Priority.values())
          .filter(p -> p.getPriority() == priority)
          .findFirst()
          .orElseThrow(IllegalArgumentException::new);
    }
}
```

我们还添加了`Priority.of()`方法，以便根据`int`值轻松获得`Priority`实例。

现在，为了在我们的`Article`类中使用它，我们需要添加两个属性并实现回调方法:

```java
@Entity
public class Article {

    @Id
    private int id;

    private String title;

    @Enumerated(EnumType.ORDINAL)
    private Status status;

    @Enumerated(EnumType.STRING)
    private Type type;

    @Basic
    private int priorityValue;

    @Transient
    private Priority priority;

    @PostLoad
    void fillTransient() {
        if (priorityValue > 0) {
            this.priority = Priority.of(priorityValue);
        }
    }

    @PrePersist
    void fillPersistent() {
        if (priority != null) {
            this.priorityValue = priority.getPriority();
        }
    }
}
```

现在，当持久化一个`Article`实体时:

```java
Article article = new Article();
article.setId(3);
article.setTitle("callback title");
article.setPriority(Priority.HIGH);
```

JPA 将触发以下 SQL 查询:

```java
insert 
into
    Article
    (priorityValue, status, title, type, id) 
values
    (?, ?, ?, ?, ?)
binding parameter [1] as [INTEGER] - [300]
binding parameter [2] as [INTEGER] - [null]
binding parameter [3] as [VARCHAR] - [callback title]
binding parameter [4] as [VARCHAR] - [null]
binding parameter [5] as [INTEGER] - [3]
```

尽管与前面描述的解决方案相比，这个选项为我们选择数据库值的表示提供了更多的灵活性，但它并不理想。在实体中用两个属性表示一个枚举感觉不太好。另外，如果我们使用这种类型的映射，我们就不能在 JPQL 查询中使用 enum 的值。

## 4.使用 JPA 2.1 `@Converter`注释

**为了克服上述解决方案的局限性，JPA 2.1 版本引入了一个新的标准化 API，可用于将实体属性转换为数据库值，反之亦然。**我们需要做的就是创建一个实现`javax.persistence.AttributeConverter`的新类，并用`@Converter`对其进行注释。

我们来看一个实际的例子。

首先，我们将创建一个新的枚举:

```java
public enum Category {
    SPORT("S"), MUSIC("M"), TECHNOLOGY("T");

    private String code;

    private Category(String code) {
        this.code = code;
    }

    public String getCode() {
        return code;
    }
}
```

我们还需要将它添加到`Article`类中:

```java
@Entity
public class Article {

    @Id
    private int id;

    private String title;

    @Enumerated(EnumType.ORDINAL)
    private Status status;

    @Enumerated(EnumType.STRING)
    private Type type;

    @Basic
    private int priorityValue;

    @Transient
    private Priority priority;

    private Category category;
}
```

现在让我们创建一个新的`CategoryConverter`:

```java
@Converter(autoApply = true)
public class CategoryConverter implements AttributeConverter<Category, String> {

    @Override
    public String convertToDatabaseColumn(Category category) {
        if (category == null) {
            return null;
        }
        return category.getCode();
    }

    @Override
    public Category convertToEntityAttribute(String code) {
        if (code == null) {
            return null;
        }

        return Stream.of(Category.values())
          .filter(c -> c.getCode().equals(code))
          .findFirst()
          .orElseThrow(IllegalArgumentException::new);
    }
}
```

我们已经将`@Converter`的值`autoApply`设置为`true`，这样 JPA 将自动将转换逻辑应用到所有映射的`Category`类型的属性。否则，我们必须将`@Converter`注释直接放在实体的字段上。

现在让我们持久化一个`Article`实体:

```java
Article article = new Article();
article.setId(4);
article.setTitle("converted title");
article.setCategory(Category.MUSIC);
```

然后，JPA 将执行以下 SQL 语句:

```java
insert 
into
    Article
    (category, priorityValue, status, title, type, id) 
values
    (?, ?, ?, ?, ?, ?)
Converted value on binding : MUSIC -> M
binding parameter [1] as [VARCHAR] - [M]
binding parameter [2] as [INTEGER] - [0]
binding parameter [3] as [INTEGER] - [null]
binding parameter [4] as [VARCHAR] - [converted title]
binding parameter [5] as [VARCHAR] - [null]
binding parameter [6] as [INTEGER] - [4]
```

正如我们所看到的，如果我们使用`AttributeConverter`接口，我们可以简单地设置我们自己的将枚举转换成相应数据库值的规则。**此外，我们可以安全地添加新的枚举值或更改现有的值，而不会破坏已经持久化的数据。**

整个解决方案易于实现，并且解决了前面几节中介绍的选项的所有缺点。

## 5.在 JPQL 中使用枚举

现在让我们看看在 JPQL 查询中使用枚举是多么容易。

为了找到所有类别为`Category.SPORT`的`Article`实体，我们需要执行以下语句:

```java
String jpql = "select a from Article a where a.category = com.baeldung.jpa.enums.Category.SPORT";

List<Article> articles = em.createQuery(jpql, Article.class).getResultList();
```

需要注意的是，在这种情况下，我们需要使用完全限定的枚举名。

当然，我们并不局限于静态查询。

使用命名参数是完全合法的:

```java
String jpql = "select a from Article a where a.category = :category";

TypedQuery<Article> query = em.createQuery(jpql, Article.class);
query.setParameter("category", Category.TECHNOLOGY);

List<Article> articles = query.getResultList();
```

这个例子提供了一种非常方便的方式来形成动态查询。

另外，我们不需要使用完全限定名。

## 6.结论

在本文中，我们讨论了在数据库中保存枚举值的各种方法。我们介绍了在 2.0 及以下版本中使用 JPA 时的选项，以及 JPA 2.1 及以上版本中可用的新 API。

值得注意的是，这些并不是在 JPA 中处理枚举的唯一可能性。有些数据库，如 PostgreSQL，提供了专用的列类型来存储枚举值。然而，这些解决方案超出了本文的范围。

**根据经验，如果我们使用 JPA 2.1 或更高版本，我们应该总是使用`AttributeConverter`接口和`@Converter`注释。**

像往常一样，所有代码示例都可以在我们的 [GitHub 库](https://web.archive.org/web/20221205120635/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa)上找到。