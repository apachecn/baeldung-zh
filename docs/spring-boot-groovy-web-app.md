# 用 Spring Boot 和 Groovy 构建简单的 Web 应用程序

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-groovy-web-app>

## 1.概观

[**Groovy**](/web/20221208143845/https://www.baeldung.com/groovy-language) **有许多我们可能想在 Spring web 应用程序中使用的功能。**

因此，在本教程中，我们将使用 Spring Boot 和 Groovy 构建一个简单的 todo 应用程序。此外，我们将探索它们的集成点。

## 2.待办事项应用程序

我们的应用程序将具有以下特性:

*   创建任务
*   编辑任务
*   删除任务
*   查看特定任务
*   查看所有任务

这将是一个基于 REST 的应用程序，我们将使用 T2 Maven 作为我们的构建工具。

### 2.1.Maven 依赖性

让我们在`pom.xml` 文件中包含所有需要的依赖项:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>2.5.1</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.5.1</version>
</dependency>
<dependency>
    <groupId>org.codehaus.groovy</groupId>
    <artifactId>groovy</artifactId>
    <version>3.0.3</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <version>2.5.1</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.200</version>
    <scope>runtime</scope>
</dependency>
```

在这里，我们用**包括 [`spring-boot-starter-web`](https://web.archive.org/web/20221208143845/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-web) 来构建 REST 端点**，用**导入 [`groovy`](https://web.archive.org/web/20221208143845/https://search.maven.org/search?q=g:org.codehaus.groovy%20AND%20a:groovy) 依赖来为我们的项目**提供 Groovy 支持。

**对于持久层，我们使用的是 [`spring-boot-starter-data-jpa`](https://web.archive.org/web/20221208143845/https://search.maven.org/search?q=g:org.springframework.boot%20AND%20a:spring-boot-starter-data-jpa) ， [`h2`](https://web.archive.org/web/20221208143845/https://search.maven.org/search?q=g:com.h2database%20AND%20a:h2) 是嵌入式数据库**。

此外，我们必须将**包括 [`gmavenplus-plugin`](https://web.archive.org/web/20221208143845/https://search.maven.org/search?q=gmavenplus-plugin)** 以及`pom.xml:`中的所有[目标](https://web.archive.org/web/20221208143845/https://github.com/groovy/GMavenPlus/wiki/Usage#why-do-i-need-so-many-goals)

```java
<build>
    <plugins>
        //...
        <plugin>
            <groupId>org.codehaus.gmavenplus</groupId>
            <artifactId>gmavenplus-plugin</artifactId>
            <version>1.9.0</version>
            <executions>
                <execution>
                    <goals>
                        <goal>addSources</goal>
                        <goal>addTestSources</goal>
                        <goal>generateStubs</goal>
                        <goal>compile</goal>
                        <goal>generateTestStubs</goal>
                        <goal>compileTests</goal>
                        <goal>removeStubs</goal>
                        <goal>removeTestStubs</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### 2.2.JPA 实体类

让我们编写一个简单的 **`Todo` Groovy 类，它有三个字段——`id`、`task,`和`isCompleted`** :

```java
@Entity
@Table(name = 'todo')
class Todo {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    Integer id

    @Column
    String task

    @Column
    Boolean isCompleted
}
```

这里，`id` 字段是任务的唯一标识符。`task`包含任务的细节，`isCompleted`显示任务是否完成。

注意，**当我们不为该字段提供访问修饰符时，Groovy 编译器会将该字段设为私有，并为其生成 [getter 和 setter](https://web.archive.org/web/20221208143845/https://groovy-lang.org/style-guide.html#_getters_and_setters) 方法**。

### 2.3.持久层

让**创建一个 Groovy 接口——`TodoRepository`** ，它实现了 [`JpaRepository`](/web/20221208143845/https://www.baeldung.com/spring-data-repositories#jprepository) 。它将负责我们应用程序中的所有 CRUD 操作:

```java
@Repository
interface TodoRepository extends JpaRepository<Todo, Integer> {}
```

### 2.4.服务层

**`TodoService` 接口包含了我们的 CRUD 操作**所需的所有抽象方法:

```java
interface TodoService {

    List<Todo> findAll()

    Todo findById(Integer todoId)

    Todo saveTodo(Todo todo)

    Todo updateTodo(Todo todo)

    Todo deleteTodo(Integer todoId)
}
```

**`TodoServiceImpl`是一个实现类**，它实现了`TodoService:`的所有方法

```java
@Service
class TodoServiceImpl implements TodoService {

    //...

    @Override
    List<Todo> findAll() {
        todoRepository.findAll()
    }

    @Override
    Todo findById(Integer todoId) {
        todoRepository.findById todoId get()
    }

    @Override
    Todo saveTodo(Todo todo){
        todoRepository.save todo
    }

    @Override
    Todo updateTodo(Todo todo){
        todoRepository.save todo
    }

    @Override
    Todo deleteTodo(Integer todoId){
        todoRepository.deleteById todoId
    }
}
```

### 2.5.控制器层

现在，让我们用**来定义`TodoController`** 中所有其余的 API，也就是我们的 [`@RestController`](/web/20221208143845/https://www.baeldung.com/spring-controller-vs-restcontroller#spring-mvc-rest-controller) :

```java
@RestController
@RequestMapping('todo')
public class TodoController {

    @Autowired
    TodoService todoService

    @GetMapping
    List<Todo> getAllTodoList(){
        todoService.findAll()
    }

    @PostMapping
    Todo saveTodo(@RequestBody Todo todo){
        todoService.saveTodo todo
    }

    @PutMapping
    Todo updateTodo(@RequestBody Todo todo){
        todoService.updateTodo todo
    }

    @DeleteMapping('/{todoId}')
    deleteTodo(@PathVariable Integer todoId){
        todoService.deleteTodo todoId
    }

    @GetMapping('/{todoId}')
    Todo getTodoById(@PathVariable Integer todoId){
        todoService.findById todoId
    }
}
```

这里，我们定义了五个端点，用户可以调用它们来执行 CRUD 操作。

### 2.6.引导 Spring Boot 应用程序

现在，让我们用将用于启动我们的应用程序的 main 方法编写一个类:

```java
@SpringBootApplication
class SpringBootGroovyApplication {
    static void main(String[] args) {
        SpringApplication.run SpringBootGroovyApplication, args
    }
}
```

注意，在 Groovy 中，**在通过传递参数调用方法时，括号的[使用是可选的](https://web.archive.org/web/20221208143845/https://groovy-lang.org/style-guide.html#_omitting_parentheses)**——这就是我们在上面的例子中所做的。

另外，Groovy 中的任何类都不需要**的[后缀`.class`](https://web.archive.org/web/20221208143845/https://groovy-lang.org/style-guide.html#_classes_as_first_class_citizens)，这就是为什么我们直接使用`SpringBootGroovyApplication`的原因。**

现在，让我们将`pom.xml` 中的这个类定义为 [`start-class`](/web/20221208143845/https://www.baeldung.com/spring-boot-main-class#maven) :

```java
<properties>
    <start-class>com.baeldung.app.SpringBootGroovyApplication</start-class>
</properties>
```

## 3.运行应用程序

最后，我们的应用程序可以运行了。我们应该简单地将`SpringBootGroovyApplication`类作为 Java 应用程序运行，或者运行 Maven build:

```java
spring-boot:run
```

这应该在`[http://localhost:8080](https://web.archive.org/web/20221208143845/http://localhost:8080/)`上启动应用程序，我们应该能够访问它的端点。

## 4.测试应用程序

我们的应用程序已经可以测试了。让我们创建一个 Groovy 类–`TodoAppTest`来测试我们的应用程序。

### 4.1.初始设置

让我们在我们的类中定义三个静态变量——`API_ROOT`、`readingTodoId,`和`writingTodoId` :

```java
static API_ROOT = "http://localhost:8080/todo"
static readingTodoId
static writingTodoId
```

这里，`API_ROOT` 包含我们应用程序的根 URL。`readingTodoId`和`writingTodoId`是我们测试数据的主键，稍后我们将使用它们来执行测试。

现在，让我们通过使用注释 [`@BeforeClass`](/web/20221208143845/https://www.baeldung.com/junit-before-beforeclass-beforeeach-beforeall#beforeclass) 来创建另一个方法——`populateDummyData()` 来填充测试数据:

```java
@BeforeClass
static void populateDummyData() {
    Todo readingTodo = new Todo(task: 'Reading', isCompleted: false)
    Todo writingTodo = new Todo(task: 'Writing', isCompleted: false)

    final Response readingResponse = 
      RestAssured.given()
        .contentType(MediaType.APPLICATION_JSON_VALUE)
        .body(readingTodo).post(API_ROOT)

    Todo cookingTodoResponse = readingResponse.as Todo.class
    readingTodoId = cookingTodoResponse.getId()

    final Response writingResponse = 
      RestAssured.given()
        .contentType(MediaType.APPLICATION_JSON_VALUE)
        .body(writingTodo).post(API_ROOT)

    Todo writingTodoResponse = writingResponse.as Todo.class
    writingTodoId = writingTodoResponse.getId()
}
```

我们还将使用相同的方法填充变量–`readingTodoId`和`writingTodoId` ,以存储我们正在保存的记录的主键。

注意，**在 Groovy 中，我们也可以通过使用[命名参数和默认构造函数](https://web.archive.org/web/20221208143845/https://groovy-lang.org/style-guide.html#_initializing_beans_with_named_parameters_and_the_default_constructor)** 来初始化 bean，就像我们对上面代码片段中的`readingTodo`和`writingTodo`这样的 bean 所做的那样。

### 4.2.测试 CRUD 操作

接下来，让我们从待办事项列表中找到所有任务:

```java
@Test
void whenGetAllTodoList_thenOk(){
    final Response response = RestAssured.get(API_ROOT)

    assertEquals HttpStatus.OK.value(),response.getStatusCode()
    assertTrue response.as(List.class).size() > 0
}
```

然后，让我们通过传递之前填充的`readingTodoId` 来找到一个特定的任务:

```java
@Test
void whenGetTodoById_thenOk(){
    final Response response = 
      RestAssured.get("$API_ROOT/$readingTodoId")

    assertEquals HttpStatus.OK.value(),response.getStatusCode()
    Todo todoResponse = response.as Todo.class
    assertEquals readingTodoId,todoResponse.getId()
}
```

这里，我们使用了[插值](https://web.archive.org/web/20221208143845/https://groovy-lang.org/style-guide.html#_gstrings_interpolation_multiline)来连接 URL 字符串。

此外，让我们尝试使用`readingTodoId` `:`来更新待办事项列表中的任务

```java
@Test
void whenUpdateTodoById_thenOk(){
    Todo todo = new Todo(id:readingTodoId, isCompleted: true)
    final Response response = 
      RestAssured.given()
        .contentType(MediaType.APPLICATION_JSON_VALUE)
        .body(todo).put(API_ROOT)

    assertEquals HttpStatus.OK.value(),response.getStatusCode()
    Todo todoResponse = response.as Todo.class
    assertTrue todoResponse.getIsCompleted()
}
```

然后使用`writingTodoId`删除待办事项列表中的任务:

```java
@Test
void whenDeleteTodoById_thenOk(){
    final Response response = 
      RestAssured.given()
        .delete("$API_ROOT/$writingTodoId")

    assertEquals HttpStatus.OK.value(),response.getStatusCode()
}
```

最后，我们可以保存一个新任务:

```java
@Test
void whenSaveTodo_thenOk(){
    Todo todo = new Todo(task: 'Blogging', isCompleted: false)
    final Response response = 
      RestAssured.given()
        .contentType(MediaType.APPLICATION_JSON_VALUE)
        .body(todo).post(API_ROOT)

    assertEquals HttpStatus.OK.value(),response.getStatusCode()
}
```

## 5.结论

在本文中，我们使用 Groovy 和 Spring Boot 构建了一个简单的应用程序。我们还看到了它们是如何集成在一起的，并用例子展示了 Groovy 的一些很酷的特性。

和往常一样，这个例子的完整源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221208143845/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-groovy)