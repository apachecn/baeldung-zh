# 使用 Java 将数据从 JSON 文件导入 MongoDB

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-import-json-mongodb>

## 1.介绍

在本教程中，我们将学习如何从文件中读取 JSON 数据，并使用 Spring Boot 将它们导入 [MongoDB。**这在很多情况下都很有用:恢复数据、批量插入新数据或插入默认值。** MongoDB 在内部使用 JSON 来构建它的文档，所以自然地，我们将使用它来存储可导入的文件。作为纯文本，这种策略还有一个优点是](/web/20220524050538/https://www.baeldung.com/spring-data-mongodb-tutorial)[容易压缩](/web/20220524050538/https://www.baeldung.com/json-reduce-data-size)。

此外，我们将学习如何在必要时根据我们的自定义类型验证我们的输入文件。最后，我们将公开一个 API，这样我们就可以在运行时在我们的 web 应用程序中使用它。

## 2.属国

让我们将这些 Spring Boot 依赖项添加到我们的`pom.xml`:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

我们还需要一个 MongoDB 的运行实例，这需要一个正确配置的`application.properties`文件。

## 3.导入 JSON 字符串

**将 JSON 导入 MongoDB 最简单的方法是先将其转换成一个“`org.bson.Document`”对象。**这个类代表一个没有特定类型的通用 MongoDB 文档。因此，我们不必担心为我们可能导入的各种对象创建存储库。

我们的策略获取 JSON(来自文件、[资源](/web/20220524050538/https://www.baeldung.com/spring-classpath-file-access)或字符串)，将其转换为`Document` s，并使用`MongoTemplate`保存它们。**批处理操作通常执行得更好，因为与单独插入每个对象相比，往返次数减少了。**

最重要的是，我们将认为我们的输入每个换行符只有一个 JSON 对象。这样，我们可以很容易地给对象定界。我们将把这些功能封装到我们将要创建的两个类中:`ImportUtils`和`ImportJsonService`。让我们从服务类别开始:

```
@Service
public class ImportJsonService {

    @Autowired
    private MongoTemplate mongo;
}
```

接下来，让我们添加一个将 JSON 行解析成文档的方法:

```
private List<Document> generateMongoDocs(List<String> lines) {
    List<Document> docs = new ArrayList<>();
    for (String json : lines) {
        docs.add(Document.parse(json));
    }
    return docs;
}
```

然后我们添加一个方法，将一列`Document`对象插入到所需的`collection`中。此外，批处理操作也可能部分失败。在这种情况下，我们可以通过检查`exception`的`cause`返回插入文件的数量:

```
private int insertInto(String collection, List<Document> mongoDocs) {
    try {
        Collection<Document> inserts = mongo.insert(mongoDocs, collection);
        return inserts.size();
    } catch (DataIntegrityViolationException e) {
        if (e.getCause() instanceof MongoBulkWriteException) {
            return ((MongoBulkWriteException) e.getCause())
              .getWriteResult()
              .getInsertedCount();
        }
        return 0;
    }
}
```

最后，让我们把那些方法结合起来。这个函数接受输入并返回一个字符串，显示读取了多少行和成功插入了多少行:

```
public String importTo(String collection, List<String> jsonLines) {
    List<Document> mongoDocs = generateMongoDocs(jsonLines);
    int inserts = insertInto(collection, mongoDocs);
    return inserts + "/" + jsonLines.size();
}
```

## 4.用例

现在我们已经准备好处理输入，我们可以构建一些用例。让我们创建`ImportUtils`类来帮助我们。**这个类将负责把输入转换成 JSON 的行。**它将只包含静态方法。让我们从阅读一个简单的`String`开始:

```
public static List<String> lines(String json) {
    String[] split = json.split("[\\r\\n]+");
    return Arrays.asList(split);
} 
```

由于我们使用换行符作为分隔符， [regex](/web/20220524050538/https://www.baeldung.com/java-split-string) 可以很好地将字符串分成多行。这个正则表达式处理 Unix 和 Windows 的行尾。接下来，介绍一种将[文件](/web/20220524050538/https://www.baeldung.com/reading-file-in-java)转换成字符串列表的方法:

```
public static List<String> lines(File file) {
    return Files.readAllLines(file.toPath());
}
```

类似地，我们以一个将[类路径资源](/web/20220524050538/https://www.baeldung.com/spring-classpath-file-access)转换成列表的方法结束:

```
public static List<String> linesFromResource(String resource) {
    Resource input = new ClassPathResource(resource);
    Path path = input.getFile().toPath();
    return Files.readAllLines(path);
}
```

### 4.1.使用 CLI 在启动过程中导入文件

在我们的第一个用例中，我们将实现通过应用程序参数导入文件的功能。我们将利用 Spring Boot [`ApplicationRunner`](/web/20220524050538/https://www.baeldung.com/running-setup-logic-on-startup-in-spring) 接口在引导时完成这项工作。**例如，我们可以读取命令行参数来定义要导入的文件:**

```
@SpringBootApplication
public class SpringBootJsonConvertFileApplication implements ApplicationRunner {
    private static final String RESOURCE_PREFIX = "classpath:";

    @Autowired
    private ImportJsonService importService;

    public static void main(String ... args) {
        SpringApplication.run(SpringBootPersistenceApplication.class, args);
    }

    @Override
    public void run(ApplicationArguments args) {
        if (args.containsOption("import")) {
            String collection = args.getOptionValues("collection")
              .get(0);

            List<String> sources = args.getOptionValues("import");
            for (String source : sources) {
                List<String> jsonLines = new ArrayList<>();
                if (source.startsWith(RESOURCE_PREFIX)) {
                    String resource = source.substring(RESOURCE_PREFIX.length());
                    jsonLines = ImportUtils.linesFromResource(resource);
                } else {
                    jsonLines = ImportUtils.lines(new File(source));
                }

                String result = importService.importTo(collection, jsonLines);
                log.info(source + " - result: " + result);
            }
        }
    }
} 
```

使用`getOptionValues()`我们可以处理一个或多个文件。**这些文件可以来自我们的类路径，也可以来自我们的文件系统。**我们用`RESOURCE_PREFIX`来区分它们。每个以“`classpath:`”开头的参数都将从我们的 resources 文件夹中读取，而不是从文件系统中读取。之后会全部导入到想要的`collection`中。

让我们通过在`src/main/resources/data.json.log`下创建一个文件来开始使用我们的应用程序:

```
{"name":"Book A", "genre": "Comedy"}
{"name":"Book B", "genre": "Thriller"}
{"name":"Book C", "genre": "Drama"}
```

在[构建了](/web/20220524050538/https://www.baeldung.com/maven)之后，我们可以使用下面的例子来运行它(为了可读性，添加了换行符)。在我们的示例中，将导入两个文件，一个来自类路径，另一个来自文件系统:

```
java -cp target/spring-boot-persistence-mongodb/WEB-INF/lib/*:target/spring-boot-persistence-mongodb/WEB-INF/classes \
  -Djdk.tls.client.protocols=TLSv1.2 \
  com.baeldung.SpringBootPersistenceApplication \
  --import=classpath:data.json.log \
  --import=/tmp/data.json \
  --collection=books
```

### 4.2.HTTP POST 上传的 JSON 文件

此外，如果我们创建一个 [REST 控制器](/web/20220524050538/https://www.baeldung.com/spring-controller-vs-restcontroller)，我们将有一个端点来上传和导入 JSON 文件。为此，我们需要一个`[MultipartFile](/web/20220524050538/https://www.baeldung.com/spring-rest-template-multipart-upload)`参数:

```
@RestController
@RequestMapping("/import-json")
public class ImportJsonController {
    @Autowired
    private ImportJsonService service;

    @PostMapping("/file/{collection}")
    public String postJsonFile(@RequestPart("parts") MultipartFile jsonStringsFile, @PathVariable String collection)  {
        List<String> jsonLines = ImportUtils.lines(jsonStringsFile);
        return service.importTo(collection, jsonLines);
    }
}
```

现在我们可以像这样导入带有 [POST](/web/20220524050538/https://www.baeldung.com/curl-rest) 的文件，这里的`/tmp/data.json`指的是一个已存在的文件:

```
curl -X POST http://localhost:8082/import-json/file/books -F "[[email protected]](/web/20220524050538/https://www.baeldung.com/cdn-cgi/l/email-protection)/tmp/books.json"
```

### 4.3.将 JSON 映射到特定的 Java 类型

我们一直只使用 JSON，没有绑定到任何类型，这是使用 MongoDB 的优势之一。**现在我们要验证我们的输入。**在这种情况下，让我们添加一个`[ObjectMapper](/web/20220524050538/https://www.baeldung.com/jackson-object-mapper-tutorial)`，对我们的服务进行如下更改:

```
private <T> List<Document> generateMongoDocs(List<String> lines, Class<T> type) {
    ObjectMapper mapper = new ObjectMapper();

    List<Document> docs = new ArrayList<>();
    for (String json : lines) {
        if (type != null) {
            mapper.readValue(json, type);
        }
        docs.add(Document.parse(json));
    }
    return docs;
}
```

这样，如果指定了`type`参数，我们的`mapper`将尝试将我们的 JSON 字符串解析为该类型。**在默认配置下，如果出现任何未知属性，将抛出异常。**下面是我们使用 MongoDB [仓库](/web/20220524050538/https://www.baeldung.com/spring-data-crud-repository-save)的简单 bean 定义:

```
@Document("books")
public class Book {
    @Id
    private String id;
    private String name;
    private String genre;
    // getters and setters
}
```

现在，为了使用我们的文档生成器的改进版本，让我们也改变这个方法:

```
public String importTo(Class<?> type, List<String> jsonLines) {
    List<Document> mongoDocs = generateMongoDocs(jsonLines, type);
    String collection = type.getAnnotation(org.springframework.data.mongodb.core.mapping.Document.class)
      .value();
    int inserts = insertInto(collection, mongoDocs);
    return inserts + "/" + jsonLines.size();
}
```

现在，我们不是传递集合的名称，而是传递一个`Class`。我们假设它有我们在`Book`中使用的`Document`注释，所以它可以检索集合名称。然而，由于注释和`Document`类具有相同的名称，我们必须指定整个包。

## 5.结论

在本文中，我们从文件、资源或简单字符串中分离 JSON 输入，并将其导入 MongoDB。我们将这一功能集中在一个服务类和一个实用程序类中，这样我们就可以在任何地方重用它。我们的用例包括一个 CLI 和一个 REST 选项，以及如何使用它的示例命令。

和往常一样，源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220524050538/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence-mongodb)