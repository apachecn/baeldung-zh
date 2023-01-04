# Hibernate 类型库指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-types-library>

## 1.概观

在本教程中，我们将看看 Hibernate 类型。这个库为我们提供了一些核心 Hibernate ORM 中没有的类型。

## 2.属国

要启用 Hibernate 类型，我们只需添加[适当的`hibernate-types`依赖](https://web.archive.org/web/20220926185317/https://search.maven.org/artifact/com.vladmihalcea/hibernate-types-52):

```java
<dependency>
    <groupId>com.vladmihalcea</groupId>
    <artifactId>hibernate-types-52</artifactId>
    <version>2.9.7</version>
</dependency> 
```

这将适用于 Hibernate 版本`5.4, 5.3,`和`5.2.`

如果 Hibernate 的版本较旧，上面的`artifactId`值会有所不同。对于版本`5.1`和`5.0,`我们可以使用`hibernate-types-51.`，同样，版本`4.3`需要`hibernate-types-43,`，版本`4.2,`和 4.1 需要`hibernate-types-4.`

本教程中的示例需要一个数据库。使用 Docker，我们提供了一个数据库容器。因此，我们需要一份 [Docker](https://web.archive.org/web/20220926185317/https://www.docker.com/get-started) 的工作副本。

因此，要运行和创建我们的数据库，我们只需要执行:

```java
$ ./create-database.sh
```

## 3.支持的数据库

我们可以在 Oracle、SQL Server、PostgreSQL 和 MySQL 数据库中使用我们的类型。因此，Java 中的类型到数据库列类型的映射将根据我们使用的数据库而变化。在我们的例子中，我们将使用 MySQL 并将`JsonBinaryType`映射到 JSON 列类型。

关于支持的映射的文档可以在 Hibernate 类型库上找到[。](https://web.archive.org/web/20220926185317/https://github.com/vladmihalcea/hibernate-types)

## 4.数据模型

本教程的数据模型将允许我们存储关于专辑和歌曲的信息。专辑有封面和一首或多首歌曲。一首歌有艺术家和长度。封面艺术有两个图像 URL 和一个 UPC 代码。最后，艺术家有名字、国家和音乐流派。

过去，我们会创建表格来表示模型中的所有数据。但是，现在我们有了可用的类型，我们可以非常容易地将一些数据存储为 JSON。

对于本教程，我们将只为专辑和歌曲创建表格:

```java
public class Album extends BaseEntity {
    @Type(type = "json")
    @Column(columnDefinition = "json")
    private CoverArt coverArt;

    @OneToMany(fetch = FetchType.EAGER)
    private List<Song> songs;

   // other class members
}
```

```java
public class Song extends BaseEntity {

    private Long length = 0L;

    @Type(type = "json")
    @Column(columnDefinition = "json")
    private Artist artist;

    // other class members
}
```

**使用`JsonStringType` 我们将封面艺术和艺术家表示为这些表中的 JSON 列:**

```java
public class Artist implements Serializable {

    private String name;
    private String country;
    private String genre;

    // other class members
}
```

```java
public class CoverArt implements Serializable {

    private String frontCoverArtUrl;
    private String backCoverArtUrl;
    private String upcCode;

    // other class members
}
```

**需要注意的是`Artist`和`CoverArt`类是 POJOs 而不是实体。**此外，它们是我们数据库实体类的成员，用`@Type(type = “json”)`注释定义。

### 4.1.存储 JSON 类型

我们定义了专辑和歌曲模型来包含数据库将存储为 JSON 的成员。这是因为使用了提供的`json`类型。为了使该类型可供我们使用，我们必须使用类型定义来定义它:

```java
@TypeDefs({
  @TypeDef(name = "json", typeClass = JsonStringType.class),
  @TypeDef(name = "jsonb", typeClass = JsonBinaryType.class)
})
public class BaseEntity {
  // class members
}
```

`JsonStringType`和`JsonBinaryType`的`@Type`使`json`和`jsonb`类型可用。

最新的 MySQL 版本支持 JSON 作为列类型。因此， **JDBC 用这两种类型中的一种作为`String`来处理从列中读取的任何 JSON 或保存到列中的任何对象。**这意味着为了正确映射到列，我们必须在类型定义中使用`JsonStringType`。

### 4.2.冬眠

最终，我们的类型将使用 JDBC 和 Hibernate 自动转换成 SQL。所以，现在我们可以创建一些歌曲对象，一个专辑对象，并把它们保存到数据库中。随后，Hibernate 生成以下 SQL 语句:

```java
insert into song (name, artist, length, id) values ('A Happy Song', '{"name":"Superstar","country":"England","genre":"Pop"}', 240, 3);
insert into song (name, artist, length, id) values ('A Sad Song', '{"name":"Superstar","country":"England","genre":"Pop"}', 120, 4);
insert into song (name, artist, length, id) values ('A New Song', '{"name":"Newcomer","country":"Jamaica","genre":"Reggae"}', 300, 6)
insert into album (name, cover_art, id) values ('Album 0', '{"frontCoverArtUrl":"http://fakeurl-0","backCoverArtUrl":"http://fakeurl-1","upcCode":"b2b9b193-ee04-4cdc-be8f-3a276769ab5b"}', 7) 
```

正如所料，我们的`json`类型 Java 对象都由 Hibernate 翻译，并作为格式良好的 JSON 存储在我们的数据库中。

## 5.存储泛型类型

除了支持基于 JSON 的列，该库还从`java.time`包中添加了一些泛型类型:`YearMonth`、`Year,`和`Month`。

**现在，我们可以映射 Hibernate 或 JPA** 本身不支持的类型。此外，我们现在能够将它们存储为`Integer`、`String,`或`Date` 列。

例如，假设我们想要将一首歌曲的录制日期添加到我们的`song`模型中，并将其作为一个`Integer`存储在我们的数据库中。我们可以在我们的`Song`实体类定义中使用`YearMonthIntegerType`:

```java
@TypeDef(
  typeClass = YearMonthIntegerType.class,
  defaultForType = YearMonth.class
)
public class Song extends BaseEntity {
    @Column(
      name = "recorded_on",
      columnDefinition = "mediumint"
    )
    private YearMonth recordedOn = YearMonth.now();

    // other class members  
} 
```

我们的`recordedOn`属性值被转换成我们提供的`typeClass`。因此，**一个预定义的转换器将值作为`Integer`** 保存在我们的数据库中。

## 6.其他实用程序类别

Hibernate Types 有几个助手类，可以进一步改善开发人员使用 Hibernate 的体验。

`CamelCaseToSnakeCaseNamingStrategy`将 Java 类中的驼峰式属性映射到数据库中的蛇形列。

`ClassImportIntegrator`允许在 JPA 构造函数参数中使用简单的 Java DTO 类名值。

还有`ListResultTransformer`和 `MapResultTransformer` 类为 JPA 使用的结果对象提供了更简洁的实现。此外，它们支持使用 lambdas，并提供与旧 JPA 版本的向后兼容性。

## 7.结论

在本教程中，我们介绍了 Hibernate 类型 Java 库以及它添加到 Hibernate 和 JPA 中的新类型。我们还查看了该库提供的一些实用程序和泛型类型。

GitHub 上的[提供了示例和代码片段的实现。](https://web.archive.org/web/20220926185317/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-libraries)