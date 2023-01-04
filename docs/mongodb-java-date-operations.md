# 在 MongoDB 的 CRUD 操作中使用日期

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mongodb-java-date-operations>

## 1.概观

在本教程中，我们将使用 [MongoDB](/web/20220821031626/https://www.baeldung.com/java-mongodb) Java 驱动程序来执行与日期相关的 CRUD 操作，比如创建和更新带有日期字段的文档，以及查询、更新和删除日期字段在给定范围内的文档。

## 2.设置

在开始实现之前，让我们设置一下我们的工作环境。

### 2.1.Maven 依赖性

首先，**你应该已经安装了 MongoDB。**如果你没有，你可以按照官方的 MongoDB 安装[指南](https://web.archive.org/web/20220821031626/https://www.mongodb.com/docs/manual/administration/install-community/)来做。

接下来，让我们将 [MongoDB Java 驱动程序](https://web.archive.org/web/20220821031626/https://mvnrepository.com/artifact/org.mongodb/mongodb-driver-sync)作为依赖项添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongodb-driver-sync</artifactId>
    <version>4.6.0</version>
</dependency>
```

### 2.2.POJO 数据模型

让我们定义一个 [POJO](/web/20220821031626/https://www.baeldung.com/java-pojo-class) 来表示数据库中包含的文档:

```java
public class Event {
    private String title;
    private String location;
    private LocalDateTime dateTime;

    public Event() {}
    public Event(String title, String location, LocalDateTime dateTime) {
        this.title = title;
        this.location = location;
        this.dateTime = dateTime;
    }

    // standard setters and getters
}
```

注意，我们已经声明了两个构造函数。 ****MongoDB 通过** [**默认**](https://web.archive.org/web/20220821031626/https://www.mongodb.com/docs/drivers/java/sync/current/fundamentals/data-formats/pojo-customization/#pojos-without-no-argument-constructors) 使用无参构造函数。在本教程中，另一个构造函数供我们自己使用。**

我们还要注意，虽然`dateTime` 可能是一个`String`变量，**最佳实践是对日期字段**使用特定于日期/时间的 JDK 类。使用`String` 字段来表示日期需要额外的努力来确保值的格式正确。

我们现在准备将客户机连接到我们的数据库。

### 2.3.MongoDB 客户端

为了让 MongoDB 序列化/反序列化我们的`Event` POJO，我们需要向 MongoDB 的`CodecRegistry`注册`PojoCodecProvider` :

```java
CodecProvider codecProvider = PojoCodecProvider.builder().automatic(true).build();
CodecRegistry codecRegistry = fromRegistries(getDefaultCodecRegistry(), fromProviders(codecProvider));
```

让我们创建一个数据库、集合和客户机，它将使用我们注册的`PojoCodecProvider` :

```java
MongoClient mongoClient = MongoClients.create(uri);
MongoDatabase db = mongoClient.getDatabase("calendar").withCodecRegistry(codecRegistry);
MongoCollection<Event> collection = db.getCollection("my_events", Event.class);
```

我们现在准备创建文档并执行与日期相关的 CRUD 操作。

## 3.创建带有日期字段的文档

在我们的 POJO 中，我们使用了 [`LocalDateTime`](/web/20220821031626/https://www.baeldung.com/java-8-date-time-intro) 而不是`String`，以便更容易处理日期值。现在让我们通过使用`LocalDateTime`的便利 API 构造`Event`对象来利用这一点:

```java
Event pianoLessonsEvent = new Event("Piano lessons", "Foo Blvd",
  LocalDateTime.of(2022, 6, 4, 11, 0, 0));
Event soccerGameEvent = new Event("Soccer game", "Bar Avenue",
  LocalDateTime.of(2022, 6, 10, 17, 0, 0));
```

我们可以将新的`Event`插入我们的数据库，如下所示:

```java
InsertOneResult pianoLessonsInsertResult = collection.insertOne(pianoLessonsEvent);
InsertOneResult soccerGameInsertResult = collection.insertOne(soccerGameEvent);
```

让我们通过检查插入文档的 id 来验证插入是否成功:

```java
assertNotNull(pianoLessonsInsertResult.getInsertedId());
assertNotNull(soccerGameInsertResult.getInsertedId());
```

## 4.查询符合日期条件的单据

现在我们的数据库中已经有了`Event` s，让我们根据它们的日期字段来检索它们。

**我们可以使用等式过滤器(`eq`)来检索匹配特定日期和时间的文档:**

```java
LocalDateTime dateTime = LocalDateTime.of(2022, 6, 10, 17, 0, 0);
Event event = collection.find(eq("dateTime", dateTime)).first();
```

让我们检查结果`Event`的各个字段:

```java
assertEquals("Soccer game", event.title);
assertEquals("Bar Avenue", event.location);
assertEquals(dateTime, event.dateTime);
```

**我们还可以使用 MongoDB `BasicDBObject` 类以及`gte` 和`lte` 操作符来构建使用日期范围**的 **更复杂的查询:**

```java
LocalDateTime from = LocalDateTime.of(2022, 06, 04, 12, 0, 0);
LocalDateTime to = LocalDateTime.of(2022, 06, 10, 17, 0, 0);
BasicDBObject object = new BasicDBObject();
object.put("dateTime", BasicDBObjectBuilder.start("$gte", from).add("$lte", to).get());
List list = new ArrayList(collection.find(object).into(new ArrayList()));
```

由于足球比赛是我们查询的日期范围内唯一的`Event`,我们应该在`list`中只看到一个`Event` 对象，不包括钢琴课:

```java
assertEquals(1, events.size());
assertEquals("Soccer game", events.get(0).title);
assertEquals("Bar Avenue", events.get(0).location);
assertEquals(dateTime, events.get(0).dateTime);
```

## 5.更新文档

让我们探索基于日期字段更新文档的两个用例。首先，我们将更新单个文档的日期字段，然后我们将更新匹配一个日期范围的多个文档。

### 5.1.**更新文档的日期字段**

**要更新一个 MongoDB 文档，我们可以使用`updateOne()` 方法**。让我们也使用`currentDate()`方法来设置钢琴课事件的`dateTime`字段:

```java
Document document = new Document().append("title", "Piano lessons");
Bson update = Updates.currentDate("dateTime");
UpdateOptions options = new UpdateOptions().upsert(false);
UpdateResult result = collection.updateOne(document, update, options);
```

注意，`updateOne()`的第一个参数是一个`Document`对象，MongoDB 将使用它来匹配数据库中的单个条目。如果多个文档匹配，MongoDB 将只更新它遇到的第一个文档。我们还要注意，我们将`false`传递给了`upsert()` 方法。如果我们传入了`true`，那么如果现有的文档都不匹配，MongoDB 就会插入一个新文档。

我们可以通过检查修改了多少文档来确认操作是否成功:

```java
assertEquals(1, result.getModifiedCount());
```

### 5.2.更新符合日期标准的文档

**为了更新多个文档，MongoDB 提供了`updateMany` 方法。**在这个例子中，我们将更新多个符合查询日期范围的事件。

与`updateOne()`不同，`updateMany()` 方法期望第二个`Bson`对象封装查询标准，该标准将定义我们想要更新的文档。在这种情况下，我们将通过引入`lt` 字段操作符来指定涵盖 2022 年所有事件的日期范围:

```java
LocalDate updateManyFrom = LocalDate.of(2022, 1, 1);
LocalDate updateManyTo = LocalDate.of(2023, 1, 1);
Bson query = and(gte("dateTime", from), lt("dateTime", to));
Bson updates = Updates.currentDate("dateTime");
UpdateResult result = collection.updateMany(query, updates);
```

就像使用`updateOne()`，**一样，我们可以通过检查我们的`result` 对象:**的更新计数来确认这个操作更新了多个事件

```java
assertEquals(2, result.getModifiedCount());
```

## 6.删除符合日期标准的文档

与更新一样，我们可以一次从数据库中删除一个或多个文档。假设我们需要删除 2022 年的所有事件。让我们使用一个`Bson`日期范围查询和`deleteMany()`方法来完成:

```java
LocalDate deleteFrom = LocalDate.of(2022, 1, 1);
LocalDate deleteTo = LocalDate.of(2023, 1, 1);
Bson query = and(gte("dateTime", deleteFrom), lt("dateTime", deleteTo));
DeleteResult result = collection.deleteMany(query); 
```

由于我们在本教程中创建的所有事件都有一个 2022 `dateTime`字段值，`deleteMany()` 将它们全部从我们的集合中删除。我们可以通过检查删除计数来确认这一点:

```java
assertEquals(2, result.getDeletedCount());
```

## 7.使用时区

**MongoDB 用 UTC 存储日期，这是不可更改的。**因此，如果我们希望我们的日期字段特定于一个时区，我们可以将时区偏移量存储在一个单独的字段中，并自己进行转换。让我们添加那个字段作为`String`:

```java
public String timeZoneOffset;
```

我们需要调整构造函数，以便在创建事件时设置新字段:

```java
public Event(String title, String location, LocalDateTime dateTime, String timeZoneOffset) {
    this.title = title;
    this.location = location;
    this.dateTime = dateTime;
    this.timeZoneOffset = timeZoneOffset;
}
```

我们现在可以创建特定时区的事件并将其插入到数据库中。让我们使用`ZoneOffset` 类来避免手动格式化时区偏移量`String`:

```java
LocalDateTime utcDateTime = LocalDateTime.of(2022, 6, 20, 11, 0, 0);
Event pianoLessonsTZ = new Event("Piano lessons", "Baz Bvld", utcDateTime, ZoneOffset.ofHours(2).toString());
InsertOneResult pianoLessonsTZInsertResult = collection.insertOne(pianoLessonsTZ);
assertNotNull(pianoLessonsTZInsertResult.getInsertedId());
```

注意，**因为偏移量是相对于 UTC 的，所以`dateTime `成员变量必须代表 UTC 时间，这样我们就可以在后面正确地转换它**。一旦我们从集合中检索到文档，我们就可以使用偏移字段和 [`OffsetDateTime`](/web/20220821031626/https://www.baeldung.com/java-8-date-time-intro) 类进行转换:

```java
OffsetDateTime dateTimeWithOffset = OffsetDateTime.of(pianoLessonsTZ.dateTime, ZoneOffset.of(pianoLessonsTZ.timeZoneOffset));
```

## 8.结论

在本文中，我们学习了如何使用 Java 和 MongoDB 数据库执行与日期相关的 CRUD 操作。

我们使用日期值来创建、检索、更新或删除数据库中的文档。在整个示例中，我们讨论了各种助手类，并介绍了在处理日期时很有帮助的 MongoDB 操作符。最后，为了解决 MongoDB 如何只用 UTC 存储日期的问题，我们学习了如何处理需要特定于时区的日期/时间值。

和往常一样，本教程中使用的示例代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220821031626/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb-queries)