# 为 Spring 数据中的类配置 MongoDB 集合名称

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-mongodb-collection-name>

## 1.概观

在本教程中，我们将学习如何为我们的类配置 MongoDB 集合名称，以及一个实际的例子。我们将使用 [Spring 数据](/web/20220811173318/https://www.baeldung.com/spring-data-mongodb-tutorial)，这给了我们几个选项来用很少的配置实现这个。我们将通过构建一个简单的音乐商店来探索每个选项。这样，我们就能发现什么时候使用它们是有意义的。

## 2.使用案例和设置

我们的用例有四个简单的类:`MusicAlbum`、`Compilation`、`MusicTrack`和`Store`。**每个类都有不同的集合名称。另外，每个职业都有自己的`MongoRepository`。不需要自定义查询。此外，我们需要一个正确配置的 [MongoDB](/web/20220811173318/https://www.baeldung.com/spring-data-mongodb-tutorial) 数据库实例。**

### 2.1.按名称列出集合内容的服务

首先，让我们写一个[控制器](/web/20220811173318/https://www.baeldung.com/spring-controllers)来断言我们的配置正在工作。我们将通过按集合名称进行搜索来做到这一点。请注意，使用存储库时，集合名称配置是透明的:

```
@RestController
@RequestMapping("/collection")
public class CollectionController {
    @Autowired
    private MongoTemplate mongoDb;

    @GetMapping("/{name}")
    public List<DBObject> get(@PathVariable String name) {
        return mongoDb.findAll(DBObject.class, name);
    }
}
```

这个控制器基于`MongoTemplate` 并使用通用类型`DBObject`，它不依赖于类和存储库。同样，通过这种方式，我们将能够看到 MongoDB 的内部属性。最重要的是，Spring Data 用来解组 JSON 对象的“_class”属性保证了我们的配置是正确的。

### 2.2.API 服务

其次，我们将开始构建我们的服务，它只保存对象并检索它们的集合。**`Compilation`类将在我们的第一个配置示例**中创建:

```
@Service
public class MusicStoreService {
    @Autowired
    private CompilationRepository compilationRepository;

    public Compilation add(Compilation item) {
        return compilationRepository.save(item);
    }

    public List<Compilation> getCompilationList() {
        return compilationRepository.findAll();
    }

    // other service methods
} 
```

### 2.3.API 端点

最后，让我们编写一个控制器来与我们的应用程序接口。我们将为我们的服务方法公开端点:

```
@RestController
@RequestMapping("/music")
public class MusicStoreController {
    @Autowired
    private MusicStoreService service;

    @PostMapping("/compilation")
    public Compilation post(@RequestBody Compilation item) {
        return service.add(item);
    }

    @GetMapping("/compilation")
    public List<Compilation> getCompilationList() {
        return service.getCompilationList();
    }

    // other endpoint methods
}
```

之后，我们准备开始配置我们的类。

## 3.带有`@Document`注释的配置

从 Spring Data 版本 1.9 开始，`Document`注释可以满足我们的所有需求。它是 MongoDB 特有的，但类似于 [JPA 的`Entity`](/web/20220811173318/https://www.baeldung.com/jpa-entities) 注释。**在撰写本文时，还没有办法为集合名称定义一个[命名策略](/web/20220811173318/https://www.baeldung.com/hibernate-naming-strategy)，只能为字段名称定义。**因此，让我们探索一下有哪些可用的资源。

### 3.1.默认行为

默认行为认为集合名称与类名相同，但以小写字母开头。**简而言之，我们只需要添加`Document`注释，就可以了**:

```
@Document
public class Compilation {
    @Id
    private String id;

    // getters and setters
}
```

之后，来自我们的`Compilation`存储库的所有插入将进入 MongoDB 中一个名为“compilation”的集合:

```
$ curl -X POST http://localhost:8080/music/compilation -H 'Content-Type: application/json' -d '{
    "name": "Spring Hits"
}'

{ "id": "6272e26e04a673360d926ca1" } 
```

让我们列出“编译”集合的内容来验证我们的配置:

```
$ curl http://localhost:8080/collection/compilation

[
  {
    "name": "Spring Hits",
    "_class": "com.baeldung.boot.collection.name.data.Compilation"
  }
] 
```

这是配置集合名称最干净的方式，因为它基本上与我们的类名相同。一个缺点是，如果我们决定改变数据库命名约定，我们将需要重构所有的类。例如，如果我们决定使用 snake-case 作为集合名称，我们将无法利用默认行为。

### 3.2.覆盖`value`属性

`Document`注释让我们用`collection`属性覆盖默认行为。因为这个属性是`value`属性的别名，我们可以隐式地设置它:

```
@Document("albums")
public class MusicAlbum {
    @Id
    private String id;

    private String name;

    private String artist;

    // getters and setters
}
```

现在，这些文档不是放在名为“musicAlbum”的集合中，而是放在“albums”集合中。**这是用 Spring 数据**配置集合名称的最简单方式。要查看它的实际效果，让我们在收藏中添加一个相册:

```
$ curl -X POST 'http://localhost:8080/music/album' -H 'Content-Type: application/json' -d '{
  "name": "Album 1",
  "artist": "Artist A"
}'

{ "id": "62740de003d2452a61a75c35" } 
```

然后，我们可以获取我们的“相册”集合，确保我们的配置有效:

```
$ curl 'http://localhost:8080/collection/albums'

[
  {
    "name": "Album 1",
    "artist": "Artist A",
    "_class": "com.baeldung.boot.collection.name.data.MusicAlbum"
  }
] 
```

此外，这对于使我们的应用程序适应现有的数据库非常有用，因为现有数据库中的集合名称与我们的类不匹配。一个缺点是，如果我们需要添加一个默认前缀，我们需要为每个类都这样做。

### 3.3.对 SpEL 使用配置属性

**这种组合有助于完成单独使用`Document`注释**所做不到的事情。我们将从我们希望在类中重用的应用程序特定的属性开始。

**首先，让我们给我们的`[application.properties](/web/20220811173318/https://www.baeldung.com/properties-with-spring)`添加一个属性，我们将使用它作为集合名称**的后缀:

```
collection.suffix=db
```

现在，让我们通过`environment` bean 用 [SpEL](/web/20220811173318/https://www.baeldung.com/spring-expression-language) 引用它来创建我们的下一个类:

```
@Document("store-#{@environment.getProperty('collection.suffix')}")
public class Store {
    @Id
    private String id;

    private String name;

    // getters and setters
}
```

然后，让我们创建我们的第一个商店:

```
$ curl -X POST 'http://localhost:8080/music/store' -H 'Content-Type: application/json' -d '{
  "name": "Store A"
}'

{ "id": "62744c6267d3a034ec5e5719" }
```

因此，我们的类将被存储在一个名为“store-db”的集合中。同样，我们可以通过列出其内容来验证我们的配置:

```
$ curl 'http://localhost:8080/collection/store-db'

[
  {
    "name": "Store A",
    "_class": "com.baeldung.boot.collection.name.data.Store"
  }
] 
```

这样，如果我们改变我们的后缀，我们不必在每个类中手动改变它。相反，我们只是更新我们的属性文件。缺点是它更加样板化，无论如何我们还是要写类名。然而，**它还可以帮助实现[多租户](/web/20220811173318/https://www.baeldung.com/hibernate-5-multitenancy)支持**。例如，我们可以用租户 ID 来标记未共享的集合，而不是后缀。

### 3.4.对 SpEL 使用 Bean 方法

使用 SpEL 的另一个缺点是它需要额外的工作来以编程方式进行评估。但是，它提供了很多可能性，比如调用任何 bean 方法来确定我们的集合名称。因此，在我们的下一个例子中，**我们将创建一个 bean 来固定一个集合名称，以符合我们的命名规则**。

在我们的命名策略中，我们将使用 snake-case。首先，让我们借用 Spring Data 的`SnakeCaseFieldNamingStrategy`代码来创建我们的实用程序 bean:

```
public class Naming {
    public String fix(String name) {
        List<String> parts = ParsingUtils.splitCamelCaseToLower(name);
        List<String> result = new ArrayList<>();

        for (String part : parts) {
            if (StringUtils.hasText(part)) {
                result.add(part);
            }
        }

        return StringUtils.collectionToDelimitedString(result, "_");
    }
}
```

接下来，让我们将 bean 添加到我们的应用程序中:

```
@SpringBootApplication
public class SpringBootCollectionNameApplication {

    // main method

    @Bean
    public Naming naming() {
        return new Naming();
    }
} 
```

之后，我们将能够通过 SpEL 引用我们的 bean:

```
@Document("#{@naming.fix('MusicTrack')}")
public class MusicTrack {
    @Id
    private String id;

    private String name;

    private String artist;

    // getters and setters
}
```

让我们通过向我们的系列中添加一个项目来尝试一下:

```
$ curl -X POST 'http://localhost:8080/music/track' -H 'Content-Type: application/json' -d '{
  "name": "Track 1",
  "artist":"Artist A"
}'

{ "id": "62755987ae94c5278b9530cc" }
```

因此，我们的曲目将存储在名为“music_track”的收藏中:

```
$ curl 'http://localhost:8080/collection/music_track'

[
  {
    "name": "Track 1",
    "artist": "Artist A",
    "_class": "com.baeldung.boot.collection.name.data.MusicTrack"
  }
] 
```

**不幸的是，没有办法动态获得类名，这是一个主要的缺点**，但是它允许改变我们的数据库命名规则，而不必手动重命名我们所有的类。

## 4.结论

在本文中，我们探索了使用 Spring Data 提供的工具来配置集合名称的方法。此外，我们看到了每种方法的优点和缺点，因此我们可以决定哪种方法最适合特定的场景。在此期间，我们构建了一个简单的用例来展示不同的方法。

和往常一样，源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220811173318/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-mongodb-2)