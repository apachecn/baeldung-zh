# MicroStream 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/microstream-intro>

## 1.概观

[MicroStream](https://web.archive.org/web/20221120190808/https://github.com/microstream-one/microstream) 是为 JVM 打造的**对象图持久化引擎。我们可以用它来存储 Java 对象图，并在内存中恢复它们。使用定制的序列化概念，MicroStream 使我们能够存储任何 Java 类型，并加载整个对象图、部分子图或单个对象。**

在本教程中，我们将首先看看开发这样一个对象图持久化引擎的原因。然后，我们将把这种方法与传统的关系数据库和标准的 Java 序列化进行比较。我们将看到如何创建一个对象图存储，并使用它来保存、加载和删除数据。

最后，我们将使用本地系统内存和普通 Java APIs 查询数据。

## 2.对象关系不匹配

我们先来看看开发 MicroStream 的动机。在大多数 Java 项目中，我们需要某种数据库存储。

然而， **Java 和流行的关系或 NoSQL 数据库使用不同的数据结构**。因此，我们需要一种将 Java 对象映射到数据库结构的方法，反之亦然。这种映射需要编程工作和执行时间。例如，我们可以使用映射到表的实体和匹配关系数据库中字段的属性。

为了从数据库加载数据，我们经常需要执行复杂的多表 SQL 查询。尽管像 [Hibernate](/web/20221120190808/https://www.baeldung.com/tag/hibernate/) 这样的对象关系映射框架帮助开发人员弥合了这一鸿沟，但是在许多复杂的场景中，框架生成的查询并没有得到完全优化。

MicroStream 希望通过使用与持久化数据相同的内存操作结构来解决这种数据结构不匹配的问题。

## 3.使用 JVM 作为存储

MicroStream 使用 JVM 作为其存储，通过纯 Java 实现**快速的内存数据处理。它为我们提供了一个现代化的本地数据存储库，而不是使用与 JVM 分离的存储。**

### 3.1.数据库管理系统

**MicroStream 是持久性引擎，不是数据库管理系统** (DBMS)。一些标准的 DBMS 特性，如用户管理、连接管理和会话处理，在设计中被忽略了。

相反，MicroStream 致力于为我们提供一种存储和恢复应用程序数据的简单方法。

### 3.2.Java 序列化

MicroStream 使用定制的序列化概念**，旨在为传统 DBMS** 提供更高性能的替代方案。

由于几个限制，它不使用 Java 的内置序列化:

*   只能存储和恢复完整的对象图
*   在存储大小和性能方面效率低下
*   更改类结构时所需的人工工作

另一方面，自定义微流数据存储可以:

*   根据需要部分保存、加载或更新对象图
*   高效处理存储大小和性能
*   通过内部试探法或用户定义的映射策略映射数据，处理不断变化的类结构

## 4.对象图存储

MicroStream 试图通过仅使用一个数据结构和一个数据模型来简化软件开发。

对象实例存储为字节流，它们之间的引用用唯一的标识符映射。因此，对象图可以以简单和快速的方式存储。此外，它可以全部或部分加载。

### 4.1.属国

在我们开始使用 MicroStream 存储对象图之前，我们需要添加两个[依赖关系](https://web.archive.org/web/20221120190808/https://search.maven.org/search?q=microstream-storage-embedded):

```java
<dependency>
    <groupId>one.microstream</groupId>
    <artifactId>microstream-storage-embedded</artifactId>
    <version>07.00.00-MS-GA</version>
</dependency>
<dependency>
    <groupId>one.microstream</groupId>
    <artifactId>microstream-storage-embedded-configuration</artifactId>
    <version>07.00.00-MS-GA</version>
</dependency>
```

### 4.2.根实例

当使用对象图存储时，从根实例开始访问我们的**整个数据库。这个实例被称为对象图的根对象，它由 MicroStream 保持。**

对象图实例，包括根实例，可以是任何 Java 类型。因此，一个简单的 [`String`](/web/20221120190808/https://www.baeldung.com/java-string) 实例可以注册为实体图的根:

```java
EmbeddedStorageManager storageManager = EmbeddedStorage.start(directory);
storageManager.setRoot("baeldung-demo");
storageManager.storeRoot();
```

然而，由于这个根实例不包含子实例，所以我们的`String`实例包含了整个数据库。因此，我们通常需要**为我们的应用**定义一个定制的根类型:

```java
public class RootInstance {

    private final String name;
    private final List<Book> books;

    public RootInstance(String name) {
        this.name = name;
        books = new ArrayList<>();
    }

    // standard getters, hashcode and equals
}
```

我们可以通过调用`setRoot()`和`storeRoot()`方法，以类似的方式使用自定义类型注册一个根实例:

```java
EmbeddedStorageManager storageManager = EmbeddedStorage.start(directory);
storageManager.setRoot(new RootInstance("baeldung-demo"));
storageManager.storeRoot();
```

现在，我们的图书列表将是空的，但是使用我们的自定义根，我们将能够在以后存储图书实例:

```java
RootInstance rootInstance = (RootInstance) storageManager.root();
assertThat(rootInstance.getName()).isEqualTo("baeldung-demo");
assertThat(rootInstance.getBooks()).isEmpty()
storageManager.shutdown();
```

我们应该注意，一旦我们的应用程序完成了对存储的处理，为了安全起见，建议调用`shutdown()`方法。

## 5.操纵数据

让我们看看如何通过 MicroStream 持久化的对象图来执行标准的 CRUD 操作。

### 5.1.保管

当存储新实例时，我们需要确保在正确的对象上调用`store()`方法。正确的对象是新创建的实例的所有者——在我们的例子中，是一个列表`:`

```java
RootInstance rootInstance = (RootInstance) storageManager.root();
List<Book> books = rootInstance.getBooks();
books.addAll(booksToStore);
storageManager.store(books);
assertThat(books).hasSize(2);
```

存储一个新对象也会存储该对象引用的所有实例。另外，**执行`store()`方法保证数据已经被物理地写入底层存储层**，通常是一个文件系统。

### 5.2.急切装载

用 MicroStream 加载数据有两种方式，急切和懒惰。急切加载是从存储对象图中加载对象的默认方式。如果在启动时发现一个已经存在的数据库，那么**一个存储的对象图的所有对象被加载到内存**中。

启动一个`EmbeddedStorageManager`实例后，我们可以通过获取对象图的根实例来加载数据:

```java
EmbeddedStorageManager storageManager = EmbeddedStorage.start(directory);
if (storageManager.root() == null) {
    RootInstance rootInstance = new RootInstance("baeldung-demo");
    storageManager.setRoot(rootInstance);
    storageManager.storeRoot();
} else {
    RootInstance rootInstance = (RootInstance) storageManager.root();
    // Use existing root loaded from storage
}
```

根实例的`null`值表示底层存储中不存在数据库。

### 5.3.惰性装载

当我们处理大量数据时，一开始就将所有数据直接加载到内存中可能不是一个可行的选择。因此，MicroStream 还通过将一个实例包装到一个`Lazy`字段中来支持延迟加载。

`Lazy`是一个简单的包装类，类似于 JDK 的`[WeakReference](/web/20221120190808/https://www.baeldung.com/java-weak-reference).`,它的实例内部保存一个标识符和一个对实际实例的引用:

```java
private final Lazy<List<Book>> books;
```

包装在`Lazy` 中的新`ArrayList`可以使用`Reference()`方法实例化:

```java
books = Lazy.Reference(new ArrayList<>());
```

就像使用`WeakReference`一样，为了获得实际的实例，我们需要调用一个简单的`get()`方法:

```java
public List<Book> getBooks() {
    return Lazy.get(books);
}
```

`get()`方法调用将在需要时重新加载数据，开发人员无需处理任何底层数据库标识符。

### 5.4.删除

使用 MicroStream 删除数据不需要执行显式删除操作。相反，我们只需要**清除我们的对象图**中对该对象的任何引用，并存储这些更改:

```java
List<Book> books = rootInstance.getBooks();
books.remove(1);
storageManager.store(books);
```

我们应该注意，被删除的数据不会立即从存储器中擦除。相反，**后台内务处理进程运行预定的清理**。

## 6.查询系统

与标准 DBMS 不同，MicroStream 查询不直接在存储上操作，而是对本地系统内存中的数据运行。因此，没有必要学习任何特殊的查询语言，因为所有的操作都是用普通的 Java 完成的。

一种常见的方法可能是将 [`Streams`](/web/20221120190808/https://www.baeldung.com/java-streams) 与标准 Java [集合](/web/20221120190808/https://www.baeldung.com/java-collections)一起使用:

```java
List<Book> booksFrom1998 = rootInstance.getBooks().stream()
    .filter(book -> book.getYear() == 1998)
    .collect(Collectors.toList());
```

假设查询在内存中运行，内存消耗可能很高，但是查询可以快速运行。

数据存储和加载过程可以通过使用多线程来并行化。目前，水平缩放是不可能的，但 MicroStream 宣布他们目前正在开发一种对象图复制方法。这将支持未来在多个节点上进行集群和数据复制。

## 7.结论

在本文中，我们探索了用于 JVM 的对象图持久引擎 **MicroStream。我们了解了 MicroStream 如何通过将相同的结构应用于内存操作和数据持久性来解决对象关系数据结构不匹配的问题。**

我们探索了如何使用定制根实例创建对象图。此外，我们还看到了如何使用急切和惰性加载方法来存储、删除和加载数据。最后，我们看了 MicroStream 的基于普通 Java 内存操作的查询系统。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221120190808/https://github.com/eugenp/tutorials/tree/master/persistence-modules/core-java-persistence-2)