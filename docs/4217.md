# 放心地获取和验证响应数据

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-assured-response>

## 1.概观

在本教程中，我们将讨论如何使用 REST-assured 测试 REST 服务，重点是**从我们的 REST API**捕获和验证响应数据。

## 2.测试类的设置

在之前的教程中，我们已经探索了一般情况下的[放心](/web/20220901001343/https://www.baeldung.com/rest-assured-tutorial)，我们已经展示了如何操作请求[头、cookies 和参数](/web/20220901001343/https://www.baeldung.com/rest-assured-header-cookie-parameter)。

在这个现有设置的基础上，我们添加了一个简单的 REST 控制器`AppController`，它在内部调用服务`AppService`。我们将在测试示例中使用这些类。

为了创建我们的测试类，我们需要做更多的设置。因为我们的类路径中有`spring-boot-starter-test`,所以我们可以很容易地利用 Spring 测试工具。

首先，让我们创建我们的`AppControllerIntegrationTest`类的框架:

```
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class AppControllerIntegrationTest {

    @LocalServerPort
    private int port;

    private String uri;

    @PostConstruct
    public void init() {
        uri = "http://localhost:" + port;
    }

    @MockBean
    AppService appService;

     //test cases
}
```

在这个 JUnit 测试中，我们用几个特定于 Spring 的注释对我们的类进行了注释，这些注释在随机可用的端口中本地启动应用程序。在`@PostConstruct`中，我们捕获了完整的 URI，我们将在其上进行 REST 调用。

我们还在`AppService`上使用了`@MockBean`，因为我们需要模拟这个类上的方法调用。

## 3.验证 JSON 响应

JSON 是 REST APIs 中交换数据最常用的格式。响应可以由单个 JSON 对象或一组 JSON 对象组成。我们将在这一节中讨论这两个问题。

### 3.1.单个 JSON 对象

假设我们需要测试`/movie/{id}`端点，如果找到了`id`，它将返回一个`Movie` JSON 对象。

我们将使用 [Mockito](/web/20220901001343/https://www.baeldung.com/mockito-series) 框架模拟`AppService`调用以返回一些模拟数据:

```
@Test
public void givenMovieId_whenMakingGetRequestToMovieEndpoint_thenReturnMovie() {

    Movie testMovie = new Movie(1, "movie1", "summary1");
    when(appService.findMovie(1)).thenReturn(testMovie);

    get(uri + "/movie/" + testMovie.getId()).then()
      .assertThat()
      .statusCode(HttpStatus.OK.value())
      .body("id", equalTo(testMovie.getId()))
      .body("name", equalTo(testMovie.getName()))
      .body("synopsis", notNullValue());
}
```

上面，我们首先嘲笑了返回一个对象的`appService.findMovie(1)`调用。然后，我们在 REST-assured 提供的用于发出 GET 请求的`get()`方法中构造了 REST URL。最后，我们做了四个断言。

首先，**我们检查响应状态代码，然后检查`body`元素**。我们使用 [Hamcrest](/web/20220901001343/https://www.baeldung.com/java-junit-hamcrest-guide) 来断言期望值。

还要注意，如果响应 JSON 是嵌套的，我们可以通过使用类似于`“key1.key2.key3”`的`dot`操作符来测试嵌套的键。

### 3.2.验证后提取 JSON 响应

在某些情况下，我们可能需要在验证后提取响应，以对其执行额外的操作。

**我们可以提取一个类的 JSON 响应，使用`extract()`方法:**

```
Movie result = get(uri + "/movie/" + testMovie.getId()).then()
  .assertThat()
  .statusCode(HttpStatus.OK.value())
  .extract()
  .as(Movie.class);
assertThat(result).isEqualTo(testMovie);
```

在这个例子中，我们指示 REST-assured 提取对一个`Movie`对象的 JSON 响应，然后在提取的对象上断言。

**我们还可以使用`extract().asString()` API:** 提取对`String,` 的完整响应

```
String responseString = get(uri + "/movie/" + testMovie.getId()).then()
  .assertThat()
  .statusCode(HttpStatus.OK.value())
  .extract()
  .asString();
assertThat(responseString).isNotEmpty();
```

最后，**我们也可以从响应 JSON 中提取特定的字段**。

让我们来看一个 POST API 的测试，它需要一个`Movie` JSON 主体，如果成功插入，将返回相同的内容:

```
@Test
public void givenMovie_whenMakingPostRequestToMovieEndpoint_thenCorrect() {
    Map<String, String> request = new HashMap<>();
    request.put("id", "11");
    request.put("name", "movie1");
    request.put("synopsis", "summary1");

    int movieId = given().contentType("application/json")
      .body(request)
      .when()
      .post(uri + "/movie")
      .then()
      .assertThat()
      .statusCode(HttpStatus.CREATED.value())
      .extract()
      .path("id");
    assertThat(movieId).isEqualTo(11);
}
```

上面，我们首先制作了需要发布的请求对象。**然后，我们使用`path()`方法从返回的 JSON 响应中提取出`id`字段。**

### 3.3.JSON 数组

如果是 JSON 数组，我们也可以验证响应:

```
@Test
public void whenCallingMoviesEndpoint_thenReturnAllMovies() {

Set<Movie> movieSet = new HashSet<>();
movieSet.add(new Movie(1, "movie1", "summary1"));
movieSet.add(new Movie(2, "movie2", "summary2"));
when(appService.getAll()).thenReturn(movieSet);

get(uri + "/movies").then()
    .statusCode(HttpStatus.OK.value())
    .assertThat()
    .body("size()", is(2));
}
```

我们再次首先用一些数据模拟了`appService.getAll()`,并向我们的端点发出请求。**然后我们断言我们的响应数组的`statusCode`和`size`。**

这也可以通过提取来实现:

```
Movie[] movies = get(uri + "/movies").then()
  .statusCode(200)
  .extract()
  .as(Movie[].class);
assertThat(movies.length).isEqualTo(2);
```

## 4.验证标题和 Cookies

我们可以使用具有相同名称的方法来验证响应的标头或 cookie:

```
@Test
public void whenCallingWelcomeEndpoint_thenCorrect() {
    get(uri + "/welcome").then()
        .assertThat()
        .header("sessionId", notNullValue())
        .cookie("token", notNullValue());
}
```

我们还可以分别提取标题和 cookies:

```
Response response = get(uri + "/welcome");

String headerName = response.getHeader("sessionId");
String cookieValue = response.getCookie("token");
assertThat(headerName).isNotBlank();
assertThat(cookieValue).isNotBlank();
```

## 5.验证文件

如果我们的 REST API 返回一个文件，我们可以使用`asByteArray()`方法提取响应:

```
File file = new ClassPathResource("test.txt").getFile();
long fileSize = file.length();
when(appService.getFile(1)).thenReturn(file);

byte[] result = get(uri + "/download/1").asByteArray();

assertThat(result.length).isEqualTo(fileSize);
```

这里，我们首先模仿`appService.getFile(1)`返回一个存在于我们的`src/test/resources`路径中的文本文件。然后，我们调用我们的端点，并在一个`byte[]`中提取响应，然后我们断言它具有预期的值。

## 6.结论

在本教程中，我们研究了使用 REST-assured 捕获和验证来自 REST APIs 的响应的不同方法。

和往常一样，这篇文章中的代码可以在 Github 的[上找到。](https://web.archive.org/web/20220901001343/https://github.com/eugenp/tutorials/tree/master/testing-modules/rest-assured)