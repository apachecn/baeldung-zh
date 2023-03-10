# 在 Spring 应用程序中使用 Couchbase

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/couchbase-sdk-spring>

## 1。简介

在我们对 Couchbase 的[介绍的后续文章中，我们创建了一组 Spring 服务，它们可以一起用于创建一个 Spring 应用程序的基本持久层，而不需要使用 Spring 数据。](/web/20220627165207/https://www.baeldung.com/java-couchbase-sdk)

## 2。集群服务

为了满足 JVM 中只有一个`CouchbaseEnvironment`活动的约束，我们首先编写一个服务，连接到 Couchbase 集群并提供对数据桶的访问，而不直接暴露`Cluster`或`CouchbaseEnvironment`实例。

### 2.1。接口

这里是我们的`ClusterService`界面:

```java
public interface ClusterService {
    Bucket openBucket(String name, String password);
}
```

### 2.2。实施

我们的实现类实例化了一个`DefaultCouchbaseEnvironment`，并在 Spring 上下文初始化期间的`@PostConstruct`阶段连接到一个集群。

这确保了集群不为空，并且当该类被注入到其他服务类中时它是连接的，从而使它们能够打开一个或多个数据桶:

```java
@Service
public class ClusterServiceImpl implements ClusterService {
    private Cluster cluster;

    @PostConstruct
    private void init() {
        CouchbaseEnvironment env = DefaultCouchbaseEnvironment.create();
        cluster = CouchbaseCluster.create(env, "localhost");
    }
...
}
```

接下来，我们提供一个`ConcurrentHashMap`来包含打开的桶并实现`openBucket`方法:

```java
private Map<String, Bucket> buckets = new ConcurrentHashMap<>();

@Override
synchronized public Bucket openBucket(String name, String password) {
    if(!buckets.containsKey(name)) {
        Bucket bucket = cluster.openBucket(name, password);
        buckets.put(name, bucket);
    }
    return buckets.get(name);
}
```

## 3。铲斗服务

根据您构建应用程序的方式，您可能需要在多个 Spring 服务中提供对同一个数据桶的访问。

如果我们只是在应用程序启动期间尝试在两个或更多服务中打开同一个桶，第二个尝试这样做的服务可能会遇到`ConcurrentTimeoutException`。

为了避免这种情况，我们为每个 bucket 定义了一个`BucketService`接口和一个实现类。每个实现类充当`ClusterService`和需要直接访问特定`Bucket`的类之间的桥梁。

### 3.1。接口

这里是我们的`BucketService`界面:

```java
public interface BucketService {
    Bucket getBucket();
}
```

### 3.2。实施

下面的类提供了对“`baeldung-tutorial`”桶的访问:

```java
@Service
@Qualifier("TutorialBucketService")
public class TutorialBucketService implements BucketService {

    @Autowired
    private ClusterService couchbase;

    private Bucket bucket;

    @PostConstruct
    private void init() {
        bucket = couchbase.openBucket("baeldung-tutorial", "");
    }

    @Override
    public Bucket getBucket() {
        return bucket;
    }
}
```

通过在我们的`TutorialBucketService`实现类中注入`ClusterService`,并在用`@PostConstruct,`注释的方法中打开桶，我们已经确保当`TutorialBucketService`被注入其他服务时，桶将准备好使用。

## 4。持久层

现在我们已经有了一个获取`Bucket`实例的服务，我们将创建一个类似于存储库的持久层，它向其他服务提供实体类的 CRUD 操作，而不会向它们公开`Bucket`实例。

### 4.1。人实体

下面是我们希望保留的`Person`实体类:

```java
public class Person {

    private String id;
    private String type;
    private String name;
    private String homeTown;

    // standard getters and setters
}
```

### 4.2。在实体类和 JSON 之间转换

为了在实体类和 Couchbase 在持久化操作中使用的`JsonDocument`对象之间进行转换，我们定义了`JsonDocumentConverter`接口:

```java
public interface JsonDocumentConverter<T> {
    JsonDocument toDocument(T t);
    T fromDocument(JsonDocument doc);
}
```

### 4.3。实现 JSON 转换器

接下来，我们需要为`Person`实体实现一个`JsonConverter`。

```java
@Service
public class PersonDocumentConverter
  implements JsonDocumentConverter<Person> {
    ...
}
```

我们可以结合使用`Jackson`库和`JsonObject`类的`toJson`和`fromJson`方法来序列化和反序列化实体，但是这样做有额外的开销。

相反，对于`toDocument`方法，我们将使用`JsonObject`类的流畅方法创建并填充一个`JsonObject`，然后将其包装为`JsonDocument`:

```java
@Override
public JsonDocument toDocument(Person p) {
    JsonObject content = JsonObject.empty()
            .put("type", "Person")
            .put("name", p.getName())
            .put("homeTown", p.getHomeTown());
    return JsonDocument.create(p.getId(), content);
}
```

对于`fromDocument`方法，我们将使用`JsonObject`类的`getString`方法以及`Person`类的`fromDocument`方法中的设置器:

```java
@Override
public Person fromDocument(JsonDocument doc) {
    JsonObject content = doc.content();
    Person p = new Person();
    p.setId(doc.id());
    p.setType("Person");
    p.setName(content.getString("name"));
    p.setHomeTown(content.getString("homeTown"));
    return p;
}
```

### 4.4。CRUD 接口

我们现在创建一个通用的`CrudService`接口，为实体类定义持久化操作:

```java
public interface CrudService<T> {
    void create(T t);
    T read(String id);
    T readFromReplica(String id);
    void update(T t);
    void delete(String id);
    boolean exists(String id);
}
```

### 4.5。实施 CRUD 服务

有了实体和转换器类，我们现在为`Person`实体实现`CrudService`，注入上面显示的 bucket 服务和文档转换器，并在初始化期间检索 bucket:

```java
@Service
public class PersonCrudService implements CrudService<Person> {

    @Autowired
    private TutorialBucketService bucketService;

    @Autowired
    private PersonDocumentConverter converter;

    private Bucket bucket;

    @PostConstruct
    private void init() {
        bucket = bucketService.getBucket();
    }

    @Override
    public void create(Person person) {
        if(person.getId() == null) {
            person.setId(UUID.randomUUID().toString());
        }
        JsonDocument document = converter.toDocument(person);
        bucket.insert(document);
    }

    @Override
    public Person read(String id) {
        JsonDocument doc = bucket.get(id);
        return (doc != null ? converter.fromDocument(doc) : null);
    }

    @Override
    public Person readFromReplica(String id) {
        List<JsonDocument> docs = bucket.getFromReplica(id, ReplicaMode.FIRST);
        return (docs.isEmpty() ? null : converter.fromDocument(docs.get(0)));
    }

    @Override
    public void update(Person person) {
        JsonDocument document = converter.toDocument(person);
        bucket.upsert(document);
    }

    @Override
    public void delete(String id) {
        bucket.remove(id);
    }

    @Override
    public boolean exists(String id) {
        return bucket.exists(id);
    }
}
```

## 5。将所有这些放在一起

现在我们已经准备好了持久层的所有部分，下面是一个简单的注册服务示例，它使用`PersonCrudService`来持久化和检索注册者:

```java
@Service
public class RegistrationService {

    @Autowired
    private PersonCrudService crud;

    public void registerNewPerson(String name, String homeTown) {
        Person person = new Person();
        person.setName(name);
        person.setHomeTown(homeTown);
        crud.create(person);
    }

    public Person findRegistrant(String id) {
        try{
            return crud.read(id);
        }
        catch(CouchbaseException e) {
            return crud.readFromReplica(id);
        }
    }
}
```

## 6。结论

我们已经展示了，通过一些基本的 Spring 服务，将 Couchbase 合并到 Spring 应用程序中，并在不使用 Spring 数据的情况下实现一个基本的持久层是相当容易的。

本教程中显示的源代码可以在 [GitHub 项目](https://web.archive.org/web/20220627165207/https://github.com/eugenp/tutorials/tree/master/couchbase)中获得。

你可以在官方的 Couchbase 开发者文档网站上了解更多关于 Couchbase Java SDK 的信息。