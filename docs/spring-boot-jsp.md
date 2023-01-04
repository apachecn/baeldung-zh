# 带有 JavaServer Pages (JSP)的 Spring Boot

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-boot-jsp>

## 1.概观

在构建 Web 应用程序时，**[【Java server Pages(JSP)】](/web/20220920151330/https://www.baeldung.com/jsp)是我们可以用作 HTML 页面模板机制的一个选项。**

另一方面，**[【Spring Boot】](/web/20220920151330/https://www.baeldung.com/spring-boot)是一个流行的框架，我们可以用它来引导我们的 Web 应用程序。**

在本教程中，我们将看到如何使用 JSP 和 Spring Boot 来构建一个 web 应用程序。

首先，我们将了解如何设置我们的应用程序以在不同的部署场景中工作。然后我们将看看 JSP 的一些常见用法。最后，我们将探索打包应用程序时的各种选项。

这里有一个小提示，JSP 有其自身的局限性，当与 Spring Boot 结合使用时更是如此。因此，我们应该考虑将[百里香](/web/20220920151330/https://www.baeldung.com/thymeleaf-in-spring-mvc)或 [FreeMarker](/web/20220920151330/https://www.baeldung.com/freemarker-in-spring-mvc-tutorial) 作为 JSP 的更好替代方案。

## 2.Maven 依赖性

让我们看看用 JSP 支持 Spring Boot 需要哪些依赖项。

我们还将注意到将我们的应用程序作为独立应用程序运行和在 web 容器中运行之间的微妙之处。

### 2.1.作为独立应用程序运行

首先，让我们把 [`spring-boot-starter-web`](https://web.archive.org/web/20220920151330/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3Aorg.springframework.boot%20a%3Aspring-boot-starter-web) 的依赖关系包括进来。

这种依赖提供了让 web 应用程序与 Spring Boot 一起运行的所有核心需求，以及一个默认的嵌入式 Tomcat Servlet 容器:

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.4.4</version>
</dependency>
```

查看我们的文章[比较 Spring Boot 的嵌入式 Servlet 容器](/web/20220920151330/https://www.baeldung.com/spring-boot-servlet-containers)，了解更多关于如何配置除 Tomcat 之外的嵌入式 Servlet 容器的信息。

我们应该特别注意，当用作嵌入式 Servlet 容器时，**under flow 不支持 JSP。**

接下来，我们需要包含`[tomcat-embed-jasper](https://web.archive.org/web/20220920151330/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3Aorg.apache.tomcat.embed%20a%3Atomcat-embed-jasper) `依赖项，以允许我们的应用程序编译和呈现 JSP 页面:

```java
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
    <version>9.0.44</version>
</dependency>
```

虽然上述两个依赖项可以手动提供，但通常最好让 Spring Boot 管理这些依赖项版本，而我们只需管理 Spring Boot 版本。

这种版本管理可以通过使用 Spring Boot 父 POM 来完成，如我们的文章 [Spring Boot 教程-引导一个简单的应用程序](/web/20220920151330/https://www.baeldung.com/spring-boot-start)中所示，也可以通过使用依赖性管理来完成，如我们的文章 [Spring Boot 依赖性管理和定制父](/web/20220920151330/https://www.baeldung.com/spring-boot-dependency-management-custom-parent)中所示。

最后，我们需要包含`[jstl](https://web.archive.org/web/20220920151330/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3Ajavax.servlet%20a%3Ajstl) `库，它将提供 JSP 页面中所需的 JSTL 标签支持:

```java
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>
```

### 2.2.在 Web 容器中运行(Tomcat)

在 Tomcat web 容器中运行时，我们仍然需要上述依赖关系。

然而，**为了避免我们的应用程序提供的依赖关系与 Tomcat 运行时提供的依赖关系发生冲突，我们需要用`provided `范围**设置两个依赖关系:

```java
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
    <version>9.0.44</version>
    <scope>provided</scope>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <version>2.4.4</version>
    <scope>provided</scope>
</dependency>
```

注意，我们必须显式定义 [`spring-boot-starter-tomcat`](https://web.archive.org/web/20220920151330/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3Aorg.springframework.boot%20a%3Aspring-boot-starter-tomcat) ，并用`provided`范围标记它。这是因为它已经是由`spring-boot-starter-web`提供的可传递依赖项。

## 3.查看解析程序配置

按照惯例，我们将 JSP 文件放在`${project.basedir}/main/webapp/WEB-INF/jsp/ `目录中。

我们需要通过在`application.properties`文件中配置两个属性，让 Spring 知道这些 JSP 文件的位置:

```java
spring.mvc.view.prefix: /WEB-INF/jsp/
spring.mvc.view.suffix: .jsp
```

编译时，Maven 将确保生成的 WAR 文件将上述`jsp`目录放在`WEB-INF`目录中，然后由我们的应用程序提供服务。

## 4.引导我们的应用程序

我们的主应用程序类将受到我们是计划作为独立应用程序运行还是在 web 容器中运行的影响。

当作为**独立应用程序运行时，我们的应用程序类将是一个简单的`@SpringBootApplication`带注释的类以及`main`方法**:

```java
@SpringBootApplication(scanBasePackages = "com.baeldung.boot.jsp")
public class SpringBootJspApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringBootJspApplication.class);
    }
}
```

但是，如果我们需要在 web 容器中部署**，我们需要扩展`SpringBootServletInitializer`。**

**这将我们的应用程序的`Servlet`、`Filter`和`ServletContextInitializer `绑定到运行时服务器**，这是我们的应用程序运行所必需的:

```java
@SpringBootApplication(scanBasePackages = "com.baeldung.boot.jsp")
public class SpringBootJspApplication extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(SpringBootJspApplication.class);
    }

    public static void main(String[] args) {
        SpringApplication.run(SpringBootJspApplication.class);
    }
}
```

## 5.提供一个简单的网页

JSP 页面依赖 JavaServer Pages 标准标记库(JSTL)来提供通用的模板特性，如分支、迭代和格式化，它甚至提供了一组预定义的函数。

让我们创建一个简单的网页，显示应用程序中保存的图书列表。

假设我们有一个`BookService`帮助我们查找所有的`Book`对象:

```java
public class Book {
    private String isbn;
    private String name;
    private String author;

    //getters, setters, constructors and toString
}

public interface BookService {
    Collection<Book> getBooks();
    Book addBook(Book book);
}
```

我们可以编写一个 Spring MVC 控制器，将它公开为一个网页:

```java
@Controller
@RequestMapping("/book")
public class BookController {

    private final BookService bookService;

    public BookController(BookService bookService) {
        this.bookService = bookService;
    }

    @GetMapping("/viewBooks")
    public String viewBooks(Model model) {
        model.addAttribute("books", bookService.getBooks());
        return "view-books";
    }
}
```

注意，上面的`BookController`将返回一个名为`view-books`的视图模板。根据我们之前在`application.properties`的配置，Spring MVC 会在`/WEB-INF/jsp/`目录里面寻找`view-books.jsp `。

我们需要在以下位置创建该文件:

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<html>
    <head>
        <title>View Books</title>
        <link href="<c:url value="/css/common.css"/>" rel="stylesheet" type="text/css">
    </head>
    <body>
        <table>
            <thead>
                <tr>
                    <th>ISBN</th>
                    <th>Name</th>
                    <th>Author</th>
                </tr>
            </thead>
            <tbody>
                <c:forEach items="${books}" var="book">
                    <tr>
                        <td>${book.isbn}</td>
                        <td>${book.name}</td>
                        <td>${book.author}</td>
                    </tr>
                </c:forEach>
            </tbody>
        </table>
    </body>
</html>
```

上面的例子向我们展示了如何使用 JSTL `<c:url>`标签链接到外部资源，比如 JavaScript 和 CSS。我们通常将这些放在`${project.basedir}/main/resources/static/ `目录下。

我们还可以看到如何使用 JSTL `<c:forEach>`标签来迭代由我们的`BookController`提供的`books`模型属性。

## 6.处理表单提交

现在让我们看看如何用 JSP 处理表单提交。

我们的`BookController `将需要提供 MVC 端点来服务表单以添加书籍和处理表单提交:

```java
public class BookController {

    //already existing code

    @GetMapping("/addBook")
    public String addBookView(Model model) {
        model.addAttribute("book", new Book());
        return "add-book";
    }

    @PostMapping("/addBook")
    public RedirectView addBook(@ModelAttribute("book") Book book, RedirectAttributes redirectAttributes) {
        final RedirectView redirectView = new RedirectView("/book/addBook", true);
        Book savedBook = bookService.addBook(book);
        redirectAttributes.addFlashAttribute("savedBook", savedBook);
        redirectAttributes.addFlashAttribute("addBookSuccess", true);
        return redirectView;
    } 
}
```

我们将创建下面的`add-book.jsp`文件(记得把它放在正确的目录中):

```java
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
    <head>
        <title>Add Book</title>
    </head>
    <body>
        <c:if test="${addBookSuccess}">
            <div>Successfully added Book with ISBN: ${savedBook.isbn}</div>
        </c:if>

        <c:url var="add_book_url" value="/book/addBook"/>
        <form:form action="${add_book_url}" method="post" modelAttribute="book">
            <form:label path="isbn">ISBN: </form:label> <form:input type="text" path="isbn"/>
            <form:label path="name">Book Name: </form:label> <form:input type="text" path="name"/>
            <form:label path="author">Author Name: </form:label> <form:input path="author"/>
            <input type="submit" value="submit"/>
        </form:form>
    </body>
</html>
```

我们使用由`<form:form>`标签提供的`modelAttribute`参数将在`BookController`的`addBookView()`方法中添加的`book` 属性绑定到表单，该属性将在提交表单时被填充。

由于使用了这个标签，我们需要单独定义表单动作 URL，因为我们不能把标签放在标签里面。我们还使用在`<form:input>` 标签中找到的`path`属性将每个输入字段绑定到`Book`对象中的一个属性。

请参阅我们的文章[Spring MVC 中的表单入门](/web/20220920151330/https://www.baeldung.com/spring-mvc-form-tutorial)，了解更多关于如何处理表单提交的细节。

## 7.处理错误

由于**对 JSP 使用 Spring Boot 的现有限制，我们不能提供自定义的`error.html `来定制默认的`/error `映射。**相反，我们需要创建定制的错误页面来处理不同的错误。

### 7.1.静态错误页面

如果我们想为不同的 HTTP 错误显示一个定制的错误页面，我们可以提供一个静态错误页面。

假设我们需要为应用程序抛出的所有 4xx 错误提供一个错误页面。我们可以简单地将一个名为`4xx.html`的文件放在`${project.basedir}/main/resources/static/error/ `目录下。

如果我们的应用程序抛出一个 4xx HTTP 错误，Spring 将解决这个错误并返回提供的`4xx.html`页面。

### 7.2.动态错误页面

我们有多种方法可以处理异常，以提供定制的错误页面和上下文相关的信息。让我们看看 Spring MVC 如何使用`@ControllerAdvice`和`@ExceptionHandler`注释为我们提供这种支持。

假设我们的应用程序定义了一个`DuplicateBookException`:

```java
public class DuplicateBookException extends RuntimeException {
    private final Book book;

    public DuplicateBookException(Book book) {
        this.book = book;
    }

    // getter methods
}
```

同样，假设我们的`BookServiceImpl` 类将抛出上面的`DuplicateBookException `,如果我们试图添加两本具有相同 ISBN 的书:

```java
@Service
public class BookServiceImpl implements BookService {

    private final BookRepository bookRepository;

    // constructors, other override methods

    @Override
    public Book addBook(Book book) {
        final Optional<BookData> existingBook = bookRepository.findById(book.getIsbn());
        if (existingBook.isPresent()) {
            throw new DuplicateBookException(book);
        }

        final BookData savedBook = bookRepository.add(convertBook(book));
        return convertBookData(savedBook);
    }

    // conversion logic
}
```

然后，我们的`LibraryControllerAdvice`类将定义我们想要处理的错误，以及我们将如何处理每个错误:

```java
@ControllerAdvice
public class LibraryControllerAdvice {

    @ExceptionHandler(value = DuplicateBookException.class)
    public ModelAndView duplicateBookException(DuplicateBookException e) {
        final ModelAndView modelAndView = new ModelAndView();
        modelAndView.addObject("ref", e.getBook().getIsbn());
        modelAndView.addObject("object", e.getBook());
        modelAndView.addObject("message", "Cannot add an already existing book");
        modelAndView.setViewName("error-book");
        return modelAndView;
    }
}
```

我们需要定义`error-book.jsp`文件，这样上面的错误就会在这里得到解决。确保将它放在`${project.basedir}/main/webapp/WEB-INF/jsp/ `目录下，因为这不再是一个静态 HTML，而是一个需要编译的 JSP 模板。

## 8.创建可执行文件

如果我们计划**在 Tomcat 这样的 Web 容器中部署我们的应用程序，选择很简单，我们将使用`war`打包**来实现这一点。

然而，我们应该注意，如果我们将 JSP 和 Spring Boot 与嵌入式 Servlet 容器一起使用，我们就不能使用`jar`打包。因此，如果作为独立应用程序运行，我们唯一的选择是`war `打包。

无论哪种情况，我们的`pom.xml`都需要将其打包指令设置为`war`:

```java
<packaging>war</packaging>
```

如果我们没有使用 Spring Boot 的父 POM 来管理依赖关系，我们将**需要包含`[spring-boot-maven-plugin](/web/20220920151330/https://www.baeldung.com/spring-boot-run-maven-vs-executable-jar) `以确保生成的`war`文件能够作为独立的应用程序运行。**

我们现在可以使用嵌入式 Servlet 容器运行我们的独立应用程序，或者简单地将生成的`war`文件放到 Tomcat 中，让它为我们的应用程序服务。

## 9.结论

我们已经在本教程中触及了各种主题。让我们回顾一些关键的考虑因素:

*   JSP 包含一些固有的限制。考虑百里香叶或 FreeMarker 代替。
*   如果部署在 Web 容器上，记得将必要的依赖项标记为`provided`。
*   如果用作嵌入式 Servlet 容器，Undertow 将不支持 JSP。
*   如果部署在 web 容器中，我们的`@SpringBootApplication`注释类应该扩展`SpringBootServletInitializer`并提供必要的配置选项。
*   我们不能用 JSP 覆盖默认的`/error`页面。相反，我们需要提供定制的错误页面。
*   如果我们对 Spring Boot 使用 JSP，JAR 打包不是一个选项。

和往常一样，GitHub 上的[提供了我们示例的完整源代码。](https://web.archive.org/web/20220920151330/https://github.com/eugenp/tutorials/tree/master/spring-boot-modules/spring-boot-jsp)