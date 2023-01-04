# UUID 作为 MongoDB 中的实体 ID

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-mongodb-uuid>

## 1.概观

默认情况下，[MongoDB Java 驱动程序生成类型为`ObjectId`的 id](/web/20220814185514/https://www.baeldung.com/java-mongodb-last-inserted-id "Get Last Inserted Document ID in MongoDB With Java Driver")。有时，我们可能想要使用另一种类型的数据作为对象的唯一标识符，比如一个 [UUID](/web/20220814185514/https://www.baeldung.com/java-uuid) 。但是**MongoDB Java 驱动不能自动生成 UUIDs】。**

在本教程中，我们将看看用 MongoDB Java 驱动程序和 [Spring 数据 MongoDB](/web/20220814185514/https://www.baeldung.com/spring-data-mongodb-guide) 生成 UUIDs 的三种方法。

## 2.共同点

一个应用程序很少只管理一种类型的数据。为了简化 MongoDB 数据库中 ID 的管理，实现一个抽象类来定义所有 [`Document`](https://web.archive.org/web/20220814185514/https://www.mongodb.com/docs/manual/core/document/) 类的 ID 更容易。

```java
public abstract class UuidIdentifiedEntity {

    @Id   
    protected UUID id;    

    public void setId(UUID id) {

        if (this.id != null) {
            throw new UnsupportedOperationException("ID is already defined");
        }

        this.id = id;
    }

    // Getter
}
```

对于本教程中的例子，我们将假设所有的类都保存在 MongoDB 数据库中，从这个类继承而来。

## 3.配置 UUID 支持

为了允许在 MongoDB 中存储 UUIDs，我们必须配置驱动程序。这个配置非常简单，只告诉驱动程序如何在数据库中存储 UUIDs。如果几个应用程序使用同一个数据库，我们必须小心处理这个问题。

**我们所要做的就是在启动时指定 MongoDB 客户端中的`uuidRepresentation`参数**:

```java
@Bean
public MongoClient mongo() throws Exception {
    ConnectionString connectionString = new ConnectionString("mongodb://localhost:27017/test");
    MongoClientSettings mongoClientSettings = MongoClientSettings.builder()
      .uuidRepresentation(UuidRepresentation.STANDARD)
      .applyConnectionString(connectionString).build();
    return MongoClients.create(mongoClientSettings);
} 
```

如果我们使用 Spring Boot，我们可以在 application.properties 文件中指定该参数:

```java
spring.data.mongodb.uuid-representation=standard
```

## 4.使用生命周期事件

处理 UUID 生成的第一种方法是使用 Spring 的生命周期事件。对于 MongoDB 实体，我们不能使用 JPA 注释`@PrePersist`等等。因此，我们必须实现注册在 [`ApplicationContext`](/web/20220814185514/https://www.baeldung.com/spring-application-context) 中的事件监听器类。**为此，我们的类必须扩展 Spring 的 [`AbstractMongoEventListener`类](https://web.archive.org/web/20220814185514/https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/mapping/event/AbstractMongoEventListener.html)**:

```java
public class UuidIdentifiedEntityEventListener extends AbstractMongoEventListener<UuidIdentifiedEntity> {

    @Override
    public void onBeforeConvert(BeforeConvertEvent<UuidIdentifiedEntity> event) {

        super.onBeforeConvert(event);
        UuidIdentifiedEntity entity = event.getSource();

        if (entity.getId() == null) {
            entity.setId(UUID.randomUUID());
        } 
    }    
}
```

在本例中，我们使用的是`OnBeforeConvert`事件，该事件在 Spring 将 Java 对象转换为`Document`对象并将其发送给 MongoDB 驱动程序之前被触发。

键入我们的事件来捕获`UuidIdentifiedEntity`类允许处理这个抽象超类型的所有子类。一转换使用 UUID 作为 ID 的对象，Spring 就会调用我们的代码。

我们必须意识到，Spring 将事件处理委托给一个 [`TaskExecutor`](https://web.archive.org/web/20220814185514/https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/task/TaskExecutor.html) ，这可能是异步的。 **Spring 不保证在对象被有效转换之前事件被处理。**如果您的`TaskExecutor`是异步的，则不建议使用此方法，因为 ID 可能会在对象转换后生成，从而导致异常:

`InvalidDataAccessApiUsageException: Cannot autogenerate id of type java.util.UUID for entity`

我们可以在`ApplicationContext`中注册事件监听器，方法是用 [`@Component`](/web/20220814185514/https://www.baeldung.com/spring-component-annotation) 对其进行注释，或者在`@Configuration`类中生成它:

```java
@Bean
public UuidIdentifiedEntityEventListener uuidIdentifiedEntityEventListener() {
    return new UuidIdentifiedEntityEventListener();
}
```

## 5.使用实体回调

Spring infrastructure 提供了钩子来在实体生命周期的某些点执行定制代码。这些被称为`EntityCallbacks,`，在我们的例子中，我们可以在对象被保存到数据库之前使用它们来生成一个 UUID。

与之前看到的事件监听器方法不同，**回调保证它们的执行是同步的**，代码将在对象生命周期中的预期点运行。

Spring Data MongoDB 提供了一组我们可以在应用程序中使用的回调。在我们的例子中，我们将使用与前面相同的事件。**回调可以直接在`@Configuration`类**中提供:

```java
@Bean
public BeforeConvertCallback<UuidIdentifiedEntity> beforeSaveCallback() {

    return (entity, collection) -> {

        if (entity.getId() == null) {
            entity.setId(UUID.randomUUID());
        }
        return entity;
    };
} 
```

我们也可以使用一个实现了 [`BeforeConvertCallback`](https://web.archive.org/web/20220814185514/https://docs.spring.io/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/mapping/event/BeforeConvertCallback.html) 接口的`Component`。

## 6.使用自定义存储库

Spring Data MongoDB 提供了第三种方法来实现我们的目标:使用定制的存储库实现。通常，我们只需要声明一个继承自`MongoRepository,` 的接口，然后 Spring 处理与存储库相关的代码。

如果我们想改变 Spring 数据处理对象的方式，我们可以定义 Spring 将在存储库级别执行的定制代码。为此，**我们必须首先定义一个扩展`MongoRepository`** 的接口:

```java
@NoRepositoryBean
public interface CustomMongoRepository<T extends UuidIdentifiedEntity> extends MongoRepository<T, UUID> { } 
```

[`@NoRepositoryBean`](/web/20220814185514/https://www.baeldung.com/spring-data-annotations) 注释阻止 Spring 生成与`MongoRepository`相关的普通代码。该接口强制使用 UUID 作为对象中的 ID 类型。

然后，我们必须创建一个存储库类，它将定义处理 UUIDs 所需的行为:

```java
public class CustomMongoRepositoryImpl<T extends UuidIdentifiedEntity> 
  extends SimpleMongoRepository<T, UUID> implements CustomMongoRepository<T>
```

在这个库中，**我们必须通过覆盖`SimpleMongoRepository`的相关方法**来捕获所有需要生成 ID 的方法调用。在我们的例子中，这些方法是`save()`和`insert()`:

```java
@Override
public <S extends T> S save(S entity) {
    generateId(entity);
    return super.save(entity);
} 
```

最后，我们需要告诉 Spring 使用我们的自定义类作为存储库的实现，而不是默认实现。我们在`@Configuration`类中这样做:

```java
@EnableMongoRepositories(basePackages = "com.baeldung.repository", repositoryBaseClass = CustomMongoRepositoryImpl.class) 
```

然后，我们可以像往常一样声明我们的存储库，不做任何更改:

```java
public interface BookRepository extends MongoRepository<Book, UUID> { }
```

## 7.结论

在本文中，我们看到了用 Spring 数据 MongoDB 实现 UUIDs 作为 MongoDB 对象 ID 的三种方法。

和往常一样，本文中使用的代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220814185514/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-mongodb)