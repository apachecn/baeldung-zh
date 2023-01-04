# 带有 Spring 数据的 zoned datetime MongoDB

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-mongodb-zoneddatetime>

## 1.概观

在 Spring 项目中与 MongoDB 数据库交互时，Spring Data MongoDB 模块提高了可读性和可用性。

**在本教程中，我们将关注如何在读写 MongoDB 数据库时处理`ZonedDateTime` Java 对象。**

## 2.设置

要使用 Spring Data MongoDB 模块，我们需要添加以下依赖项:

```
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-mongodb</artifactId>
    <version>3.0.3.RELEASE</version>
</dependency>
```

这个库的最新版本可以在[这里](https://web.archive.org/web/20220822101438/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.springframework.data%22%20AND%20a%3A%22spring-data-mongodb%22)找到。

让我们定义一个名为`Action`(带有一个`ZonedDateTime`属性)的模型类:

```
@Document
public class Action {
    @Id
    private String id;

    private String description;
    private ZonedDateTime time;

    // constructor, getters and setters 
}
```

为了与 MongoDB 交互，我们还将创建一个扩展了`MongoRepository`的接口:

```
public interface ActionRepository extends MongoRepository<Action, String> { }
```

现在我们将定义一个测试，将一个`Action`对象插入到 MongoDB 中，并断言它是以正确的时间存储的。在 assert 评估中，我们删除了纳秒信息，因为 MongoDB `Date`类型的精度是毫秒:

```
@Test
public void givenSavedAction_TimeIsRetrievedCorrectly() {
    String id = "testId";
    ZonedDateTime now = ZonedDateTime.now(ZoneOffset.UTC);

    actionRepository.save(new Action(id, "click-action", now));
    Action savedAction = actionRepository.findById(id).get();

    Assert.assertEquals(now.withNano(0), savedAction.getTime().withNano(0)); 
}
```

开箱即用，当运行我们的测试时，我们将得到以下错误:

```
org.bson.codecs.configuration.CodecConfigurationException:
  Can't find a codec for class java.time.ZonedDateTime
```

**Spring Data MongoDB 没有定义`ZonedDateTime`转换器。**让我们看看如何配置它们。

## 3.MongoDB 转换器

我们可以通过定义一个从 MongoDB 读取的转换器和一个写入 MongoDB 的转换器来处理`ZonedDateTime`对象(跨所有模型)。

对于读取，我们将从一个`Date`对象转换成一个`ZonedDateTime`对象。在下一个例子中，我们使用`ZoneOffset.UTC`,因为`Date`对象不存储区域信息:

```
public class ZonedDateTimeReadConverter implements Converter<Date, ZonedDateTime> {
    @Override
    public ZonedDateTime convert(Date date) {
        return date.toInstant().atZone(ZoneOffset.UTC);
    }
}
```

然后，我们将从一个`ZonedDateTime`对象转换成一个`Date`对象。如果需要，我们可以将区域信息添加到另一个字段:

```
public class ZonedDateTimeWriteConverter implements Converter<ZonedDateTime, Date> {
    @Override
    public Date convert(ZonedDateTime zonedDateTime) {
        return Date.from(zonedDateTime.toInstant());
    }
}
```

由于`Date`对象不存储区域偏移，我们在示例中使用`UTC`。随着`ZonedDateTimeReadConverter `和`ZonedDateTimeWriteConverter`被添加到`MongoCustomConversions`，我们的测试现在将通过。

存储对象的简单打印如下所示:

```
Action{id='testId', description='click', time=2018-11-08T08:03:11.257Z}
```

想了解更多关于如何注册 MongoDB 转换器的知识，可以参考[本教程](/web/20220822101438/https://www.baeldung.com/spring-data-mongodb-index-annotations-converter)。

## 4.结论

在这篇简短的文章中，我们看到了如何创建 MongoDB 转换器来处理 Java `ZonedDateTime`对象。

所有这些片段的实现都可以在 GitHub 上找到[。](https://web.archive.org/web/20220822101438/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-mongodb)