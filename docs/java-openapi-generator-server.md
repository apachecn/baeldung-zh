# 使用 Open API 生成器实现 OpenAPI 服务器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-openapi-generator-server>

## 1.概观

顾名思义， [OpenAPI 生成器](https://web.archive.org/web/20221022125227/https://github.com/OpenAPITools/openapi-generator)根据 [OpenAPI](/web/20221022125227/https://www.baeldung.com/spring-rest-openapi-documentation) 规范生成代码。它可以为客户端库、服务器存根、文档和配置创建代码。

它支持各种语言和框架。值得注意的是，它支持 C++、C#、Java、PHP、Python、Ruby、Scala——几乎所有广泛使用的语言。

在本教程中，我们将学习如何通过 Maven 插件使用 OpenAPI Generator 实现一个基于 Spring 的服务器存根。

使用生成器的其他方式是通过其 [CLI](https://web.archive.org/web/20221022125227/https://openapi-generator.tech/docs/installation/) 或[在线工具](https://web.archive.org/web/20221022125227/http://api.openapi-generator.tech/index.html)。

## 2.YAML 文件

首先，我们需要一个指定 API 的 YAML 文件。我们将把它作为我们的生成器的输入来生成一个服务器存根。

这是我们`petstore.yml`的一个片段:

```java
openapi: "3.0.0"
paths:
  /pets:
    get:
      summary: List all pets
      operationId: listPets
      tags:
        - pets
      parameters:
        - name: limit
          in: query
          ...
      responses:
        ...
    post:
      summary: Create a pet
      operationId: createPets
      ...
  /pets/{petId}:
    get:
      summary: Info for a specific pet
      operationId: showPetById
      ...
components:
  schemas:
    Pet:
      type: object
      required:
        - id
        - name
      properties:
        id:
          type: integer
          format: int64
        name:
          type: string
        tag:
          type: string
    Error:
      type: object
      required:
        - code
        - message
      properties:
        code:
          type: integer
          format: int32
        message:
          type: string
```

## 3.Maven 依赖性

### 3.1.OpenAPI 生成器插件

接下来，让我们为生成器插件添加 [Maven 依赖关系](https://web.archive.org/web/20221022125227/https://search.maven.org/search?q=a:openapi-generator-maven-plugin%20AND%20g:%20org.openapitools):

```java
<plugin>
    <groupId>org.openapitools</groupId>
    <artifactId>openapi-generator-maven-plugin</artifactId>
    <version>5.1.0</version>
    <executions>
        <execution>
            <goals>
                <goal>generate</goal>
            </goals>
            <configuration>
                <inputSpec>
                    ${project.basedir}/src/main/resources/petstore.yml
                </inputSpec>
                <generatorName>spring</generatorName>
                <apiPackage>com.baeldung.openapi.api</apiPackage>
                <modelPackage>com.baeldung.openapi.model</modelPackage>
                <supportingFilesToGenerate>
                    ApiUtil.java
                </supportingFilesToGenerate>
                <configOptions>
                    <delegatePattern>true</delegatePattern>
                </configOptions>
            </configuration>
        </execution>
    </executions>
</plugin>
```

如我们所见，我们将 YAML 文件作为`inputSpec`传入。之后，由于我们需要一个基于 Spring 的服务器，我们使用了`generatorName`作为`spring`。

然后`apiPackage`指定 API 将被生成到的包名。

接下来，我们有生成器放置数据模型的`modelPackage` 。

通过将`delegatePattern`设置为`true`，我们要求创建一个接口，该接口可以作为定制的 [`@Service`](/web/20221022125227/https://www.baeldung.com/spring-bean-annotations#service) 类来实现。

重要的是，无论我们使用 CLI、Maven/Gradle 插件还是在线生成选项，OpenAPI 生成器的 **[选项都是相同的。](https://web.archive.org/web/20221022125227/https://openapi-generator.tech/docs/generators/spring/)**

### 3.2.Maven 依赖性

由于我们将生成一个 Spring 服务器，**我们还需要它的依赖项( [Spring Boot 启动网站](https://web.archive.org/web/20221022125227/https://search.maven.org/search?q=a:spring-boot-starter-web%20AND%20g:org.springframework.boot)和 [Spring 数据 JPA](https://web.archive.org/web/20221022125227/https://search.maven.org/search?q=a:spring-data-jpa%20AND%20g:org.springframework.data) )，这样生成的代码就能按预期编译和运行**:

```java
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>2.4.4</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-jpa</artifactId>
        <version>2.4.6</version>
    </dependency>
</dependencies>
```

除了上面的 Spring 依赖项，我们还需要 [`jackson-databind`](https://web.archive.org/web/20221022125227/https://search.maven.org/search?q=a:jackson-databind-nullable) 和`[swagger2](https://web.archive.org/web/20221022125227/https://search.maven.org/search?q=a:springfox-swagger2)`依赖项，这样我们生成的代码才能成功编译:

```java
<dependency>
    <groupId>org.openapitools</groupId>
    <artifactId>jackson-databind-nullable</artifactId>
    <version>0.2.1</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
```

## 4.代码生成

要生成服务器存根，我们只需运行以下命令:

```java
mvn clean install
```

结果，我们得到了以下结果:

[![](img/b14d626bac783a720141d00eb32f542b.png)](/web/20221022125227/https://www.baeldung.com/wp-content/uploads/2021/03/OpenAPI-generatedCode.png)

现在我们来看看代码，从`apiPackage`的内容开始。

首先，**我们得到一个名为`PetsApi`** 的 API 接口，它包含了 YAML 规范中定义的所有请求映射。

以下是片段:

```java
@javax.annotation.Generated(value = "org.openapitools.codegen.languages.SpringCodegen", 
  date = "2021-03-22T23:26:32.308871+05:30[Asia/Kolkata]")
@Validated
@Api(value = "pets", description = "the pets API")
public interface PetsApi {
    /**
     * GET /pets : List all pets
     *
     * @param limit How many items to return at one time (max 100) (optional)
     * @return A paged array of pets (status code 200)
     *         or unexpected error (status code 200)
     */
    @ApiOperation(value = "List all pets", nickname = "listPets", notes = "", 
      response = Pet.class, responseContainer = "List", tags={ "pets", })
    @ApiResponses(value = { @ApiResponse(code = 200, message = "A paged array of pets", 
      response = Pet.class, responseContainer = "List"),
      @ApiResponse(code = 200, message = "unexpected error", response = Error.class) })
    @GetMapping(value = "/pets", produces = { "application/json" })
    default ResponseEntity<List> listPets(@ApiParam(
      value = "How many items to return at one time (max 100)") 
      @Valid @RequestParam(value = "limit", required = false) Integer limit) {
        return getDelegate().listPets(limit);
    }

    // other generated methods
} 
```

其次，由于我们使用了委托模式，OpenAPI 还为我们生成了一个名为`PetsApiDelegate`的委托者接口。

特别是，在这个接口中声明的**方法返回默认情况下没有实现的 HTTP 状态 501**:

```java
@javax.annotation.Generated(value = "org.openapitools.codegen.languages.SpringCodegen", 
  date = "2021-03-22T23:26:32.308871+05:30[Asia/Kolkata]")
public interface PetsApiDelegate {
    /**
     * GET /pets : List all pets
     *
     * @param limit How many items to return at one time (max 100) (optional)
     * @return A paged array of pets (status code 200)
     *         or unexpected error (status code 200)
     * @see PetsApi#listPets
     */
    default ResponseEntity<List<Pet>> listPets(Integer limit) {
        getRequest().ifPresent(request -> {
            for (MediaType mediaType: MediaType.parseMediaTypes(request.getHeader("Accept"))) {
                if (mediaType.isCompatibleWith(MediaType.valueOf("application/json"))) {
                    String exampleString = "{ \"name\" : \"name\", \"id\" : 0, \"tag\" : \"tag\" }";
                    ApiUtil.setExampleResponse(request, "application/json", exampleString);
                    break;
                }
            }
        });
        return new ResponseEntity<>(HttpStatus.NOT_IMPLEMENTED);
    }

    // other generated method declarations
} 
```

在那之后，我们看到**有一个`PetsApiController`类简单地连接了委托人**:

```java
@javax.annotation.Generated(value = "org.openapitools.codegen.languages.SpringCodegen", 
  date = "2021-03-22T23:26:32.308871+05:30[Asia/Kolkata]")
@Controller
@RequestMapping("${openapi.swaggerPetstore.base-path:}")
public class PetsApiController implements PetsApi {

    private final PetsApiDelegate delegate;

    public PetsApiController(
      @org.springframework.beans.factory.annotation.Autowired(required = false) PetsApiDelegate delegate) {
        this.delegate = Optional.ofNullable(delegate).orElse(new PetsApiDelegate() {});
    }

    @Override
    public PetsApiDelegate getDelegate() {
        return delegate;
    }
} 
```

在`modelPackage`中，基于我们的 YAML 输入中定义的`schemas`，生成了一对被称为`Error`和`Pet`的**数据模型 POJOs。**

让我们来看看其中的一个——`Pet`:

```java
@javax.annotation.Generated(value = "org.openapitools.codegen.languages.SpringCodegen", 
  date = "2021-03-22T23:26:32.308871+05:30[Asia/Kolkata]")
public class Pet {
  @JsonProperty("id")
  private Long id;

  @JsonProperty("name")
  private String name;

  @JsonProperty("tag")
  private String tag;

  // constructor

  @ApiModelProperty(required = true, value = "")
  @NotNull
  public Long getId() {
    return id;
  }

  // other getters and setters

  // equals, hashcode, and toString methods
} 
```

## 5.测试服务器

现在，要使服务器存根发挥服务器的功能，只需添加一个委托者接口的实现。

为了简单起见，我们不在这里这样做，而是只测试存根。

此外，在此之前，我们需要一个弹簧:

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### 5.1.使用`curl`进行测试

启动应用程序后，我们只需运行命令:

```java
curl -I http://localhost:8080/pets/
```

这是预期的结果:

```java
HTTP/1.1 501 
Content-Length: 0
Date: Fri, 26 Mar 2021 17:29:25 GMT
Connection: close
```

### 5.2.集成测试

或者，我们可以编写一个简单的[集成测试](/web/20221022125227/https://www.baeldung.com/spring-boot-testing#integration-testing-with-springboottest)来实现相同的功能:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class OpenApiPetsIntegrationTest {
    private static final String PETS_PATH = "/pets/";

    @Autowired
    private MockMvc mockMvc;

    @Test
    public void whenReadAll_thenStatusIsNotImplemented() throws Exception {
        this.mockMvc.perform(get(PETS_PATH)).andExpect(status().isNotImplemented());
    }

    @Test
    public void whenReadOne_thenStatusIsNotImplemented() throws Exception {
        this.mockMvc.perform(get(PETS_PATH + 1)).andExpect(status().isNotImplemented());
    }
}
```

## 6.结论

在本文中，**我们看到了如何使用 OpenAPI 生成器的 Maven 插件从 YAML 规范生成基于 Spring 的服务器存根。**

下一步，我们还可以用它来[生成一个客户端](/web/20221022125227/https://www.baeldung.com/spring-boot-rest-client-swagger-codegen#generate-rest-client-with-openapi-generator)。

和往常一样，源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221022125227/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-libraries-2)