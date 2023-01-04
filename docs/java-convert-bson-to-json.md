# Java 中 BSON 到 JSON 文档的转换

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-bson-to-json>

## 1.概观

在上一篇文章中，我们已经看到了如何从 MongoDB 中以 Java 对象的形式检索 BSON 文档。

这是开发 REST API 的一种非常常见的方式，因为我们可能想在将这些对象转换成 JSON 之前修改它们(例如使用 [Jackson](/web/20221225164845/https://www.baeldung.com/jackson) )。

然而，我们可能不想对我们的文档做任何更改。为了省去编写冗长的 Java 对象映射代码的麻烦，我们可以**使用直接的 BSON 到 JSON 文档转换**。

让我们看看 [MongoDB BSON API](https://web.archive.org/web/20221225164845/https://search.maven.org/artifact/org.mongodb/bson) 如何为这个用例工作。

## 2.用 Morphia 在 MongoDB 中创建 BSON 文档

首先，让我们使用 Morphia 设置我们的依赖关系，如本文中描述的[。](/web/20221225164845/https://www.baeldung.com/mongodb-morphia)

下面是我们的示例实体，它包括各种属性类型:

```java
@Entity("Books")
public class Book {
    @Id
    private String isbn;

    @Embedded
    private Publisher publisher;

    @Property("price")
    private double cost;

    @Property
    private LocalDateTime publishDate;

    // Getters and setters ...
}
```

然后，让我们为我们的测试创建一个新的 BSON 实体，并将其保存到 MongoDB:

```java
public class BsonToJsonIntegrationTest {

    private static final String DB_NAME = "library";
    private static Datastore datastore;

    @BeforeClass
    public static void setUp() {
        Morphia morphia = new Morphia();
        morphia.mapPackage("com.baeldung.morphia");
        datastore = morphia.createDatastore(new MongoClient(), DB_NAME);
        datastore.ensureIndexes();

        datastore.save(new Book()
          .setIsbn("isbn")
          .setCost(3.95)
          .setPublisher(new Publisher(new ObjectId("fffffffffffffffffffffffa"),"publisher"))
          .setPublishDate(LocalDateTime.parse("2020-01-01T18:13:32Z", DateTimeFormatter.ISO_DATE_TIME)));
    }
}
```

## 3.默认 BSON 到 JSON 文档转换

现在让我们测试一下默认的转换，这非常简单:简单地从 BSON `Document`类中调用 **`toJson`方法:**

```java
@Test
public void givenBsonDocument_whenUsingStandardJsonTransformation_thenJsonDateIsObjectEpochTime() {
     String json = null;
     try (MongoClient mongoClient = new MongoClient()) {
         MongoDatabase mongoDatabase = mongoClient.getDatabase(DB_NAME);
         Document bson = mongoDatabase.getCollection("Books").find().first();
         assertEquals(expectedJson, bson.toJson());
     }
}
```

`expectedJson`值为:

```java
{
    "_id": "isbn",
    "className": "com.baeldung.morphia.domain.Book",
    "publisher": {
        "_id": {
            "$oid": "fffffffffffffffffffffffa"
        },
        "name": "publisher"
    },
    "price": 3.95,
    "publishDate": {
        "$date": 1577898812000
    }
}
```

这似乎对应于标准的 JSON 映射。

然而，我们可以看到**日期被默认转换为一个带有`$date`字段的对象，格式为[纪元时间](https://web.archive.org/web/20221225164845/https://en.wikipedia.org/wiki/Unix_time)。**现在让我们看看如何改变这个日期格式。

## 4.宽松的 BSON 到 JSON 日期转换

例如，如果我们想要一个更经典的 ISO 日期表示(比如对于一个 JavaScript 客户端)，我们可以将 **`relaxed` JSON 模式传递给`toJson`方法，使用`JsonWriterSettings.builder`** :

```java
bson.toJson(JsonWriterSettings
  .builder()
  .outputMode(JsonMode.RELAXED)
  .build());
```

因此，我们可以看到`publishDate`字段的“宽松”转换:

```java
{
    ...
    "publishDate": {
        "$date": "2020-01-01T17:13:32Z"
    }
    ...
}
```

这种格式看起来是正确的，但是我们仍然有`$date`字段——让我们看看如何使用自定义转换器去掉它。

## 5.自定义 BSON 到 JSON 日期转换

首先，我们必须为类型`Long`实现 **BSON `Converter`接口**，因为日期值从纪元时间开始以毫秒表示。我们使用`DateTimeFormatter.ISO_INSTANT`来获得预期的输出格式:

```java
public class JsonDateTimeConverter implements Converter<Long> {

    private static final Logger LOGGER = LoggerFactory.getLogger(JsonDateTimeConverter.class);
    static final DateTimeFormatter DATE_TIME_FORMATTER = DateTimeFormatter.ISO_INSTANT
        .withZone(ZoneId.of("UTC"));

    @Override
    public void convert(Long value, StrictJsonWriter writer) {
        try {
            Instant instant = new Date(value).toInstant();
            String s = DATE_TIME_FORMATTER.format(instant);
            writer.writeString(s);
        } catch (Exception e) {
            LOGGER.error(String.format("Fail to convert offset %d to JSON date", value), e);
        }
    }
}
```

然后，我们可以将这个类**的一个实例作为日期时间转换器传递给`JsonWriterSettings`构建器**:

```java
bson.toJson(JsonWriterSettings
  .builder()
  .dateTimeConverter(new JsonDateTimeConverter())
  .build());
```

最后，我们得到一个普通的 JSON ISO 日期格式:

```java
{
    ...
    "publishDate": "2020-01-01T17:13:32Z"
    ...
}
```

## 6.结论

在本文中，我们已经看到了 BSON 到 JSON 文档转换的默认行为。

我们强调了如何使用 BSON `Converter` 定制日期格式，这是一个常见问题。

当然，**我们可以用同样的方法来转换其他数据类型**:例如，数字、布尔值、空值或对象 id。

和往常一样，代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221225164845/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb)