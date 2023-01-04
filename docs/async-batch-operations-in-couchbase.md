# Couchbase 中的异步批处理操作

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/async-batch-operations-in-couchbase>

## 1。简介

在这篇关于在 Spring 应用程序中使用 Couchbase 的教程[的后续文章中，我们将探索 Couchbase SDK 的异步特性，以及如何使用它来批量执行持久化操作，从而使我们的应用程序实现对 Couchbase 资源的最佳利用。](/web/20220926200754/https://www.baeldung.com/couchbase-sdk-spring)

### 1.1。`CrudService`界面

首先，我们扩充了通用的`CrudService`接口，以包含批处理操作:

```
public interface CrudService<T> {
    ...

    List<T> readBulk(Iterable<String> ids);

    void createBulk(Iterable<T> items);

    void updateBulk(Iterable<T> items);

    void deleteBulk(Iterable<String> ids);

    boolean exists(String id);
}
```

### 1.2。`CouchbaseEntity`界面

我们为想要持久化的实体定义了一个接口:

```
public interface CouchbaseEntity {

    String getId();

    void setId(String id);

}
```

### 1.3。`AbstractCrudService`阶级

然后我们将在一个通用的抽象类中实现这些方法。这个类是从我们在之前的教程的[中使用的`PersonCrudService`类派生而来的，开始如下:](/web/20220926200754/https://www.baeldung.com/couchbase-sdk-spring)

```
public abstract class AbstractCrudService<T extends CouchbaseEntity> implements CrudService<T> {
    private BucketService bucketService;
    private Bucket bucket;
    private JsonDocumentConverter<T> converter;

    public AbstractCrudService(BucketService bucketService, JsonDocumentConverter<T> converter) {
        this.bucketService = bucketService;
        this.converter = converter;
    }

    protected void loadBucket() {
        bucket = bucketService.getBucket();
    }

    ...
}
```

## 2。异步桶接口

Couchbase SDK 提供了用于执行异步操作的`AsyncBucket`接口。给定一个`Bucket`实例，您可以通过`async()`方法获得它的异步版本:

```
AsyncBucket asyncBucket = bucket.async();
```

## 3。批量操作

为了使用`AsyncBucket`接口执行批处理操作，我们使用了`[RxJava](https://web.archive.org/web/20220926200754/https://github.com/ReactiveX/RxJava/wiki)`库。

### 3.1。批量读取

这里我们实现了`readBulk`方法。首先，我们使用 RxJava 中的`AsyncBucket`和`flatMap`机制将文档异步检索到`Observable<JsonDocument>`中，然后我们使用`RxJava`中的`toBlocking`机制将这些文档转换成实体列表:

```
@Override
public List<T> readBulk(Iterable<String> ids) {
    AsyncBucket asyncBucket = bucket.async();
    Observable<JsonDocument> asyncOperation = Observable
      .from(ids)
      .flatMap(new Func1<String, Observable<JsonDocument>>() {
          public Observable<JsonDocument> call(String key) {
              return asyncBucket.get(key);
          }
    });

    List<T> items = new ArrayList<T>();
    try {
        asyncOperation.toBlocking()
          .forEach(new Action1<JsonDocument>() {
              public void call(JsonDocument doc) {
                  T item = converter.fromDocument(doc);
                  items.add(item);
              }
        });
    } catch (Exception e) {
        logger.error("Error during bulk get", e);
    }

    return items;
}
```

### 3.2。批量插入

我们再次使用`RxJava's flatMap`构造来简化`createBulk`方法。

由于批量突变请求产生的速度比它们的响应产生的速度快，有时会导致过载情况，因此每当遇到`BackpressureException`时，我们都会以指数延迟进行重试:

```
@Override
public void createBulk(Iterable<T> items) {
    AsyncBucket asyncBucket = bucket.async();
    Observable
      .from(items)
      .flatMap(new Func1<T, Observable<JsonDocument>>() {
          @SuppressWarnings("unchecked")
          @Override
          public Observable<JsonDocument> call(final T t) {
              if(t.getId() == null) {
                  t.setId(UUID.randomUUID().toString());
              }
              JsonDocument doc = converter.toDocument(t);
              return asyncBucket.insert(doc)
                .retryWhen(RetryBuilder
                  .anyOf(BackpressureException.class)
                  .delay(Delay.exponential(TimeUnit.MILLISECONDS, 100))
                  .max(10)
                  .build());
          }
      })
      .last()
      .toBlocking()
      .single();
}
```

### 3.3。批量更新

我们在`updateBulk`方法中使用类似的机制:

```
@Override
public void updateBulk(Iterable<T> items) {
    AsyncBucket asyncBucket = bucket.async();
    Observable
      .from(items)
      .flatMap(new Func1<T, Observable<JsonDocument>>() {
          @SuppressWarnings("unchecked")
          @Override
          public Observable<JsonDocument> call(final T t) {
              JsonDocument doc = converter.toDocument(t);
              return asyncBucket.upsert(doc)
                .retryWhen(RetryBuilder
                  .anyOf(BackpressureException.class)
                  .delay(Delay.exponential(TimeUnit.MILLISECONDS, 100))
                  .max(10)
                  .build());
          }
      })
      .last()
      .toBlocking()
      .single();
}
```

### 3.4。批量删除

我们将`deleteBulk`方法编写如下:

```
@Override
public void deleteBulk(Iterable<String> ids) {
    AsyncBucket asyncBucket = bucket.async();
    Observable
      .from(ids)
      .flatMap(new Func1<String, Observable<JsonDocument>>() {
          @SuppressWarnings("unchecked")
          @Override
          public Observable<JsonDocument> call(String key) {
              return asyncBucket.remove(key)
                .retryWhen(RetryBuilder
                  .anyOf(BackpressureException.class)
                  .delay(Delay.exponential(TimeUnit.MILLISECONDS, 100))
                  .max(10)
                  .build());
          }
      })
      .last()
      .toBlocking()
      .single();
}
```

## 4。`PersonCrudService`

最后，我们编写了一个 Spring 服务`PersonCrudService`，它为`Person`实体扩展了我们的`AbstractCrudService`。

因为所有的 Couchbase 交互都是在抽象类中实现的，所以实体类的实现是琐碎的，因为我们只需要确保注入所有的依赖项并加载我们的 bucket:

```
@Service
public class PersonCrudService extends AbstractCrudService<Person> {

    @Autowired
    public PersonCrudService(
      @Qualifier("TutorialBucketService") BucketService bucketService,
      PersonDocumentConverter converter) {
        super(bucketService, converter);
    }

    @PostConstruct
    private void init() {
        loadBucket();
    }
}
```

## 5。结论

本教程中显示的源代码可以在 [github 项目](https://web.archive.org/web/20220926200754/https://github.com/eugenp/tutorials/tree/master/couchbase)中获得。

你可以在官方 [Couchbase 开发者文档网站](https://web.archive.org/web/20220926200754/https://docs.couchbase.com/java-sdk/2.6/start-using-sdk.html)了解更多关于 Couchbase Java SDK 的信息。