# Java 中带有 Play 框架的 REST API

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/rest-api-with-play>

## 1。概述

本教程的目的是探索 Play 框架，并学习如何使用 Java 构建 REST 服务。

我们将构建一个 REST API 来创建、检索、更新和删除学生记录。

在这样的应用程序中，我们通常会有一个数据库来存储学生记录。Play 框架有一个内置的 H2 数据库，并支持带有 Hibernate 和其他持久性框架的 JPA。

然而，为了使事情简单并专注于最重要的东西，我们将使用一个简单的映射来存储具有唯一 id 的学生对象。

## 2。创建新的应用程序

一旦我们安装了 Play 框架，就像我们对 Play 框架的[介绍中描述的那样，我们就准备好创建我们的应用程序了。](/web/20220815031421/https://www.baeldung.com/java-intro-to-the-play-framework)

让我们使用`sbt`命令通过`play-java-seed`创建一个名为`student-api`的新应用程序:

```java
sbt new playframework/play-java-seed.g8
```

## 3。型号

有了我们的应用程序框架，让我们导航到`student-api/app/models`并创建一个用于处理学生信息的 Java bean:

```java
public class Student {
    private String firstName;
    private String lastName;
    private int age;
    private int id;

    // standard constructors, getters and setters
}
```

我们现在将创建一个简单的数据存储——由一个用于学生数据的`HashMap –`支持，使用助手方法来执行 CRUD 操作:

```java
public class StudentStore {
    private Map<Integer, Student> students = new HashMap<>();

    public Optional<Student> addStudent(Student student) {
        int id = students.size();
        student.setId(id);
        students.put(id, student);
        return Optional.ofNullable(student);
    }

    public Optional<Student> getStudent(int id) {
        return Optional.ofNullable(students.get(id));
    }

    public Set<Student> getAllStudents() {
        return new HashSet<>(students.values());
    }

    public Optional<Student> updateStudent(Student student) {
        int id = student.getId();
        if (students.containsKey(id)) {
            students.put(id, student);
            return Optional.ofNullable(student);
        }
        return null;
    }

    public boolean deleteStudent(int id) {
        return students.remove(id) != null;
    }
}
```

## 4。控制器

让我们前往`student-api/app/controllers`并创建一个名为`StudentController.java`的新控制器。我们将逐步执行代码。

首先，我们需要**配置一个** `**HttpExecutionContext**.`我们将使用异步、非阻塞代码实现我们的动作。这意味着我们的动作方法将返回`CompletionStage<Result> `，而不仅仅是`Result`。这样做的好处是允许我们编写长时间运行的任务而不会阻塞。

在 Play Framework 控制器中处理异步编程时，只有一个警告:我们必须提供一个`HttpExecutionContext.`。如果我们不提供 HTTP 执行上下文，在调用 action 方法时，我们将得到臭名昭著的错误“这里没有可用的 HTTP 上下文”。

让我们注射它:

```java
private HttpExecutionContext ec;
private StudentStore studentStore;

@Inject
public StudentController(HttpExecutionContext ec, StudentStore studentStore) {
    this.studentStore = studentStore;
    this.ec = ec;
}
```

注意，我们还添加了`StudentStore `，并使用`@Inject `注释将这两个字段注入控制器的构造函数中。完成这些之后，我们现在可以继续执行操作方法了。

注意, **Play 随 Jackson 一起发布，允许数据处理**——所以我们可以导入任何我们需要的 Jackson 类，而不需要外部依赖。

让我们定义一个实用程序类来执行重复操作。在这种情况下，构建 HTTP 响应。

因此，让我们创建`student-api/app/utils`包并在其中添加`Util.java`:

```java
public class Util {
    public static ObjectNode createResponse(Object response, boolean ok) {
        ObjectNode result = Json.newObject();
        result.put("isSuccessful", ok);
        if (response instanceof String) {
            result.put("body", (String) response);
        } else {
            result.putPOJO("body", response);
        }
        return result;
    }
}
```

使用这种方法，我们将创建带有布尔`isSuccessful`键和响应体的标准 JSON 响应。

我们现在可以单步执行控制器类的操作。

### 4.1.`create`行动

映射为一个`POST `动作，该方法处理`Student`对象的创建:

```java
public CompletionStage<Result> create(Http.Request request) {
    JsonNode json = request.body().asJson();
    return supplyAsync(() -> {
        if (json == null) {
            return badRequest(Util.createResponse("Expecting Json data", false));
        }

        Optional<Student> studentOptional = studentStore.addStudent(Json.fromJson(json, Student.class));
        return studentOptional.map(student -> {
            JsonNode jsonObject = Json.toJson(student);
            return created(Util.createResponse(jsonObject, true));
        }).orElse(internalServerError(Util.createResponse("Could not create data.", false)));
    }, ec.current());
}
```

我们使用来自注入的`Http.Request `类的调用将请求体放入 Jackson 的`JsonNode`类。请注意，如果身体是`null`，我们如何使用效用方法来创建响应。

我们还返回了一个`CompletionStage<Result>`，这使我们能够使用`CompletedFuture.supplyAsync `方法编写非阻塞代码。

我们可以给它传递任何一个`String`或一个`JsonNode`，以及一个`boolean`标志来表示状态。

还要注意我们如何使用`Json.fromJson()`将传入的 JSON 对象转换成一个`Student`对象，然后返回 JSON 进行响应。

最后，我们使用了来自`play.mvc.results`包的`created`助手方法，而不是我们习惯的`ok()`。其思想是使用一种方法，为在特定上下文中执行的操作提供正确的 HTTP 状态。例如，`ok()`表示 HTTP OK 200 状态，而`created()`表示 HTTP 创建 201 时的结果状态，如上所述。这个概念将会在接下来的行动中出现。

### 4.2.`update`行动

对`http://localhost:9000/` 的`PUT `请求命中`StudentController.` `update`方法，该方法通过调用`StudentStore`的`updateStudent `方法更新学生信息:

```java
public CompletionStage<Result> update(Http.Request request) {
    JsonNode json = request.body().asJson();
    return supplyAsync(() -> {
        if (json == null) {
            return badRequest(Util.createResponse("Expecting Json data", false));
        }
        Optional<Student> studentOptional = studentStore.updateStudent(Json.fromJson(json, Student.class));
        return studentOptional.map(student -> {
            if (student == null) {
                return notFound(Util.createResponse("Student not found", false));
            }
            JsonNode jsonObject = Json.toJson(student);
            return ok(Util.createResponse(jsonObject, true));
        }).orElse(internalServerError(Util.createResponse("Could not create data.", false)));
    }, ec.current());
}
```

### 4.3.`retrieve`行动

为了检索学生，我们在对`http://localhost:9000/:id`的`GET `请求中传递学生的 id 作为路径参数。这将打击`retrieve `行动:

```java
public CompletionStage<Result> retrieve(int id) {
    return supplyAsync(() -> {
        final Optional<Student> studentOptional = studentStore.getStudent(id);
        return studentOptional.map(student -> {
            JsonNode jsonObjects = Json.toJson(student);
            return ok(Util.createResponse(jsonObjects, true));
        }).orElse(notFound(Util.createResponse("Student with id:" + id + " not found", false)));
    }, ec.current());
}
```

### 4.4.`delete`行动

`delete `动作被映射到`http://localhost:9000/:id`。我们提供了`id`来标识要删除的记录:

```java
public CompletionStage<Result> delete(int id) {
    return supplyAsync(() -> {
        boolean status = studentStore.deleteStudent(id);
        if (!status) {
            return notFound(Util.createResponse("Student with id:" + id + " not found", false));
        }
        return ok(Util.createResponse("Student with id:" + id + " deleted", true));
    }, ec.current());
}
```

### 4.5.`listStudents`行动

最后，`listStudents`动作返回到目前为止存储的所有学生的列表。它作为一个`GET`请求被映射到`http://localhost:9000/`:

```java
public CompletionStage<Result> listStudents() {
    return supplyAsync(() -> {
        Set<Student> result = studentStore.getAllStudents();
        ObjectMapper mapper = new ObjectMapper();
        JsonNode jsonData = mapper.convertValue(result, JsonNode.class);
        return ok(Util.createResponse(jsonData, true));
    }, ec.current());
}
```

## 5。映射

设置好控制器动作后，我们现在可以通过打开文件`student-api/conf/routes`并添加这些路线来映射它们:

```java
GET     /                           controllers.StudentController.listStudents()
GET     /:id                        controllers.StudentController.retrieve(id:Int)
POST    /                           controllers.StudentController.create(request: Request)
PUT     /                           controllers.StudentController.update(request: Request)
DELETE  /:id                        controllers.StudentController.delete(id:Int)
GET     /assets/*file               controllers.Assets.versioned(path="/public", file: Asset)
```

**`/assets`端点必须始终存在，以便下载静态资源。**

在这之后，我们完成了构建`Student` API。

要了解有关定义路由映射的更多信息，请访问我们的[Play 应用中的路由](/web/20220815031421/https://www.baeldung.com/routing-in-play)教程。

## 6。测试

我们现在可以通过向`http://localhost:9000/`发送请求并添加适当的上下文来对我们的 API 运行测试。从浏览器运行基本路径应该会输出:

```java
{
     "isSuccessful":true,
     "body":[]
}
```

正如我们所看到的，主体是空的，因为我们还没有添加任何记录。使用`curl`，让我们运行一些测试(或者，我们可以使用像 Postman 这样的 REST 客户端)。

让我们打开一个终端窗口，执行 curl 命令来**添加一个学生**:

```java
curl -X POST -H "Content-Type: application/json" \
 -d '{"firstName":"John","lastName":"Baeldung","age": 18}' \
 http://localhost:9000/
```

这将返回新创建的学生:

```java
{ 
    "isSuccessful":true,
    "body":{ 
        "firstName":"John",
        "lastName":"Baeldung",
        "age":18,
        "id":0
    }
}
```

运行上述测试后，从浏览器加载`http://localhost:9000`应该会给出:

```java
{ 
    "isSuccessful":true,
    "body":[ 
        { 
            "firstName":"John",
            "lastName":"Baeldung",
            "age":18,
            "id":0
        }
    ]
} 
```

对于我们添加的每个新记录,`id`属性将增加。

为了**删除一条记录**我们发送一个`DELETE` 请求:

```java
curl -X DELETE http://localhost:9000/0
{ 
    "isSuccessful":true,
    "body":"Student with id:0 deleted"
} 
```

在上面的测试中，我们删除了在第一次测试中创建的记录，现在让**再次创建它，以便我们可以测试`update`方法**:

```java
curl -X POST -H "Content-Type: application/json" \
-d '{"firstName":"John","lastName":"Baeldung","age": 18}' \
http://localhost:9000/
{ 
    "isSuccessful":true,
    "body":{ 
        "firstName":"John",
        "lastName":"Baeldung",
        "age":18,
        "id":0
    }
}
```

现在让我们**更新记录**，将名字设置为“Andrew ”,年龄设置为 30:

```java
curl -X PUT -H "Content-Type: application/json" \
-d '{"firstName":"Andrew","lastName":"Baeldung","age": 30,"id":0}' \
http://localhost:9000/
{ 
    "isSuccessful":true,
    "body":{ 
        "firstName":"Andrew",
        "lastName":"Baeldung",
        "age":30,
        "id":0
    }
}
```

上面的测试演示了更新记录后`firstName `和`age`字段值的变化。

让我们创建一些额外的虚拟记录，我们将添加两个:John Doe 和 Sam Baeldung:

```java
curl -X POST -H "Content-Type: application/json" \
-d '{"firstName":"John","lastName":"Doe","age": 18}' \
http://localhost:9000/
```

```java
curl -X POST -H "Content-Type: application/json" \
-d '{"firstName":"Sam","lastName":"Baeldung","age": 25}' \
http://localhost:9000/
```

现在，让我们得到所有的记录:

```java
curl -X GET http://localhost:9000/
{ 
    "isSuccessful":true,
    "body":[ 
        { 
            "firstName":"Andrew",
            "lastName":"Baeldung",
            "age":30,
            "id":0
        },
        { 
            "firstName":"John",
            "lastName":"Doe",
            "age":18,
            "id":1
        },
        { 
            "firstName":"Sam",
            "lastName":"Baeldung",
            "age":25,
            "id":2
        }
    ]
}
```

通过上述测试，我们确定了`listStudents`控制器动作的正常功能。

## 7。结论

在本文中，我们展示了如何使用 Play 框架构建一个成熟的 REST API。

像往常一样，本教程的源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20220815031421/https://github.com/eugenp/tutorials/tree/master/play-framework)