# Hibernate 的“对象引用了未保存的瞬态实例”错误

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-unsaved-transient-instance-error>

## 1.概观

在本教程中，我们将看到如何解决一个常见的[休眠](/web/20220523133522/https://www.baeldung.com/tag/hibernate/)错误—`“org.hibernate.TransientObjectException: object references an unsaved transient instance”`。当我们试图持久化一个[托管实体](/web/20220523133522/https://www.baeldung.com/hibernate-entity-lifecycle#managed-entity)时，我们从`Hibernate` [会话](https://web.archive.org/web/20220523133522/https://docs.jboss.org/hibernate/orm/5.2/javadocs/org/hibernate/Session.html)中得到这个错误，并且那个实体引用了一个未保存的[瞬态](/web/20220523133522/https://www.baeldung.com/hibernate-entity-lifecycle#transient)实例。

## 2。描述问题

[`TransientObjectException`](https://web.archive.org/web/20220523133522/https://docs.jboss.org/hibernate/orm/5.2/javadocs/org/hibernate/TransientObjectException.html) 是“当用户将瞬态实例传递给需要持久实例的会话方法时抛出的”。

现在，避免这个异常的最直接的解决方案是**通过持久化一个新的实例或者从数据库中获取一个实例来获得所需实体**的持久化实例，然后**在持久化它**之前将它关联到依赖实例中。然而，这样做只涵盖了这个特定的场景，并没有迎合其他用例。

为了涵盖所有场景，我们需要一个解决方案来**级联我们的保存/更新/删除操作，用于依赖于另一个实体**的存在的实体关系。我们可以通过在实体关联中使用适当的`CascadeType`来实现这一点。

在接下来的部分中，我们将创建一些 Hibernate 实体及其关联。然后，我们将尝试持久化这些实体，看看为什么`session`会抛出异常。最后，我们将通过使用适当的`CascadeType`来解决这些异常。

## 3。`@OneToOne`协会

在这一节中，我们将看到如何解决`@OneToOne`关联中的`TransientObjectException`。

### 3.1。实体

首先，让我们创建一个`User`实体:

```java
@Entity
@Table(name = "user")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private int id;

    @Column(name = "first_name")
    private String firstName;

    @Column(name = "last_name")
    private String lastName;

    @OneToOne
    @JoinColumn(name = "address_id", referencedColumnName = "id")
    private Address address;

    // standard getters and setters
}
```

让我们创建相关的`Address`实体:

```java
@Entity
@Table(name = "address")
public class Address {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private Long id;

    @Column(name = "city")
    private String city;

    @Column(name = "street")
    private String street;

    @OneToOne(mappedBy = "address")
    private User user;

    // standard getters and setters
}
```

### 3.2。产生错误

接下来，我们将添加一个单元测试来在数据库中保存一个`User `:

```java
@Test
public void whenSaveEntitiesWithOneToOneAssociation_thenSuccess() {
    User user = new User("Bob", "Smith");
    Address address = new Address("London", "221b Baker Street");
    user.setAddress(address);
    Session session = sessionFactory.openSession();
    session.beginTransaction();
    session.save(user);
    session.getTransaction().commit();
    session.close();
}
```

现在，当我们运行上面的测试时，我们得到一个异常:

```java
java.lang.IllegalStateException: org.hibernate.TransientObjectException: object references an unsaved transient instance - save the transient instance before flushing: com.baeldung.hibernate.exception.transientobject.entity.Address
```

这里，在这个例子中，我们**将一个新的/瞬态`Address`实例与一个新的/瞬态`User`实例**相关联。然后，当我们试图持久化`User `实例时，我们得到了 **`TransientObjectException `，因为 Hibernate **会话期望`Address`实体成为持久化实例**。换句话说，在持久化`User`时，`Address`应该已经保存在数据库中/可用。**

### 3.3。解决错误

最后，让我们更新`User `实体，并为`User-Address`关联使用一个合适的`CascadeType `:

```java
@Entity
@Table(name = "user")
public class User {
    ...
    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "address_id", referencedColumnName = "id")
    private Address address;
    ...
}
```

现在，每当我们保存/删除一个`User`，Hibernate 会话也会保存/删除相关的`Address`，并且会话不会抛出`TransientObjectException.`

## 4。`@OneToMany`和@ `ManyToOne`关联

在这一节中，我们将看到如何解决`@OneToMany`和`@ManyToOne`关联中的`TransientObjectException`。

### 4.1。实体

首先，让我们创建一个`Employee` 实体:

```java
@Entity
@Table(name = "employee")
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private int id;

    @Column(name = "name")
    private String name;

    @ManyToOne
    @JoinColumn(name = "department_id")
    private Department department;

    // standard getters and setters
}
```

以及相关联的`Department`实体:

```java
@Entity
@Table(name = "department")
public class Department {

    @Id
    @Column(name = "id")
    private String id;

    @Column(name = "name")
    private String name;

    @OneToMany(mappedBy = "department")
    private Set<Employee> employees = new HashSet<>();

    public void addEmployee(Employee employee) {
        employees.add(employee);
    }

    // standard getters and setters
}
```

### 4.2。产生错误

接下来，我们将添加一个单元测试来将一个`Employee`持久化到数据库中:

```java
@Test
public void whenPersistEntitiesWithOneToManyAssociation_thenSuccess() {
    Department department = new Department();
    department.setName("IT Support");
    Employee employee = new Employee("John Doe");
    employee.setDepartment(department);

    Session session = sessionFactory.openSession();
    session.beginTransaction();
    session.persist(employee);
    session.getTransaction().commit();
    session.close();
}
```

现在，当我们运行上面的测试时，我们得到一个异常:

```java
java.lang.IllegalStateException: org.hibernate.TransientObjectException: object references an unsaved transient instance - save the transient instance before flushing: com.baeldung.hibernate.exception.transientobject.entity.Department
```

这里，在这个例子中，我们**将一个新的/瞬态`Employee`实例与一个新的/瞬态`Department`实例**相关联。然后，当我们试图持久化`Employee `实例时，我们得到了 **`TransientObjectException `，因为 Hibernate **会话期望`Department`实体成为持久化实例**。换句话说，在持久化`Employee`时，`Department`应该已经保存在数据库中/可用。**

### 4.3。解决错误

最后，让我们更新`Employee`实体，并为`Employee-Department`关联使用一个合适的`CascadeType `:

```java
@Entity
@Table(name = "employee")
public class Employee {
    ...
    @ManyToOne
    @Cascade(CascadeType.SAVE_UPDATE)
    @JoinColumn(name = "department_id")
    private Department department;
    ...
}
```

让我们更新`Department`实体，为`Department-Employees`关联使用一个合适的`CascadeType `:

```java
@Entity
@Table(name = "department")
public class Department {
    ...
    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL, orphanRemoval = true)
    private Set<Employee> employees = new HashSet<>();
    ...
}
```

现在，**通过在`Employee-Department`关联上使用`@Cascade(CascadeType.SAVE_UPDATE)`** ，**每当我们将新的`Department`实例与新的`Employee`实例关联并保存`Employee`** 时，Hibernate **会话也会保存关联的`Department`实例**。

类似地，**通过在`Department-Employees`关联上使用** `**cascade = CascadeType.ALL** `，休眠**会话将从`Department`级联所有操作到关联的`Employee` (s)** 。例如，删除一个`Department`将会删除与该`Department`相关的所有`Employee`。

## 5。`@ManyToMany`协会

在这一节中，我们将看到如何解决`@ManyToMany`关联中的 `TransientObjectException`。

### 5.1。实体

让我们创建一个`Book`实体:

```java
@Entity
@Table(name = "book")
public class Book {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private int id;

    @Column(name = "title")
    private String title;

    @ManyToMany
    @JoinColumn(name = "author_id")
    private Set<Author> authors = new HashSet<>();

    public void addAuthor(Author author) {
        authors.add(author);
    }

    // standard getters and setters
}
```

让我们创建相关的`Author`实体:

```java
@Entity
@Table(name = "author")
public class Author {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
    private int id;

    @Column(name = "name")
    private String name;

    @ManyToMany
    @JoinColumn(name = "book_id")
    private Set<Book> books = new HashSet<>();

    public void addBook(Book book) {
        books.add(book);
    }

    // standard getters and setters
}
```

### 5.2。产生问题

接下来，让我们添加一些单元测试，分别在数据库中保存一个有多个作者的`Book`和一个有多个书籍的`Author`:

```java
@Test
public void whenSaveEntitiesWithManyToManyAssociation_thenSuccess_1() {
    Book book = new Book("Design Patterns: Elements of Reusable Object-Oriented Software");
    book.addAuthor(new Author("Erich Gamma"));
    book.addAuthor(new Author("John Vlissides"));
    book.addAuthor(new Author("Richard Helm"));
    book.addAuthor(new Author("Ralph Johnson"));

    Session session = sessionFactory.openSession();
    session.beginTransaction();
    session.save(book);
    session.getTransaction().commit();
    session.close();
}

@Test
public void whenSaveEntitiesWithManyToManyAssociation_thenSuccess_2() {
    Author author = new Author("Erich Gamma");
    author.addBook(new Book("Design Patterns: Elements of Reusable Object-Oriented Software"));
    author.addBook(new Book("Introduction to Object Orient Design in C"));

    Session session = sessionFactory.openSession();
    session.beginTransaction();
    session.save(author);
    session.getTransaction().commit();
    session.close();
}
```

现在，当我们运行上述测试时，我们分别得到以下异常:

```java
java.lang.IllegalStateException: org.hibernate.TransientObjectException: object references an unsaved transient instance - save the transient instance before flushing: com.baeldung.hibernate.exception.transientobject.entity.Author

java.lang.IllegalStateException: org.hibernate.TransientObjectException: object references an unsaved transient instance - save the transient instance before flushing: com.baeldung.hibernate.exception.transientobject.entity.Book
```

类似地，在这些例子中，当我们将新的/瞬态实例与一个实例相关联并试图持久化该实例时，我们得到了`TransientObjectException `。

### 5.3。解决问题

最后，让我们更新`Author`实体，并为`Author`的`-Book`关联使用适当的`CascadeType`:

```java
@Entity
@Table(name = "author")
public class Author {
    ...
    @ManyToMany
    @Cascade({ CascadeType.SAVE_UPDATE, CascadeType.MERGE, CascadeType.PERSIST})
    @JoinColumn(name = "book_id")
    private Set<Book> books = new HashSet<>();
    ...
}
```

类似地，让我们更新`Book`实体，并为`Book` s- `Author` s 关联使用适当的`CascadeType`:

```java
@Entity
@Table(name = "book")
public class Book {
    ...
    @ManyToMany
    @Cascade({ CascadeType.SAVE_UPDATE, CascadeType.MERGE, CascadeType.PERSIST})
    @JoinColumn(name = "author_id")
    private Set<Author> authors = new HashSet<>();
    ...
}
```

注意，**我们不能在`@ManyToMany`** 关联中使用`CascadeType.ALL`，因为**如果删除了`Author`，我们不想删除`Book`，反之亦然**。

## 6.结论

总而言之，这篇文章展示了**如何定义一个合适的`CascadeType`来解决`“o` `rg.hibernate.TransientObjectException: object references an unsaved transient instance”`** 错误。

和往常一样，你可以在 GitHub 上找到这个例子[的代码。](https://web.archive.org/web/20220523133522/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-exceptions)