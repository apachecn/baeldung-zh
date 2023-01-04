# Spring Data REST 中的预测和摘录

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-rest-projections-excerpts>

## 1。概述

在本文中，我们将探索 Spring Data REST 的投影和摘录概念。

我们将学习如何**使用投影来创建模型的定制视图，以及如何使用摘录作为资源集合的默认视图**。

## 2。我们的领域模型

首先，让我们从定义我们的领域模型开始:`Book`和`Author.`

让我们来看看`Book`实体类:

```
@Entity
public class Book {

    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private long id;

    @Column(nullable = false)
    private String title;

    private String isbn;

    @ManyToMany(mappedBy = "books", fetch = FetchType.EAGER)
    private List<Author> authors;
}
```

和`Author`型号:

```
@Entity
public class Author {

    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private long id;

    @Column(nullable = false)
    private String name;

    @ManyToMany(cascade = CascadeType.ALL)
    @JoinTable(
      name = "book_author", 
      joinColumns = @JoinColumn(
        name = "book_id", referencedColumnName = "id"), 
      inverseJoinColumns = @JoinColumn(
        name = "author_id", referencedColumnName = "id"))
    private List<Book> books;
}
```

这两个实体也有多对多的关系。

接下来，让我们为每个模型定义标准的 Spring Data REST 存储库:

```
public interface BookRepository extends CrudRepository<Book, Long> {}
```

```
public interface AuthorRepository extends CrudRepository<Author, Long> {}
```

现在，我们可以访问`Book`端点，使用其在`http://localhost:8080/books/{id}:`的 id 来获取特定的`Book's`细节

```
{
  "title" : "Animal Farm",
  "isbn" : "978-1943138425",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/books/1"
    },
    "book" : {
      "href" : "http://localhost:8080/books/1"
    },
    "authors" : {
      "href" : "http://localhost:8080/books/1/authors"
    }
  }
}
```

注意，由于`Author`模型有自己的存储库，作者的详细信息不是响应的一部分。但是，我们可以找到它们之间的联系—`http://localhost:8080/books/1/authors.`

## 3。创建投影

有时，**我们只对实体属性**的子集或自定义视图感兴趣。对于这种情况，我们可以利用投影。

让我们使用 Spring Data REST 投影为我们的`Book`创建一个定制视图。

我们将从创建一个简单的名为`CustomBook`的`Projection`开始:

```
@Projection(
  name = "customBook", 
  types = { Book.class }) 
public interface CustomBook { 
    String getTitle();
}
```

注意**我们的投影被定义为一个带有`@Projection`注释**的接口。我们可以使用`name`属性来定制投影的名称，以及使用`types`属性来定义它所应用的对象。

在我们的例子中，`CustomBook`投影将只包括一本书的`title`。

在创建了我们的`Projection:`之后，让我们再来看看我们的`Book`表示

```
{
  "title" : "Animal Farm",
  "isbn" : "978-1943138425",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/books/1"
    },
    "book" : {
      "href" : "http://localhost:8080/books/1{?projection}",
      "templated" : true
    },
    "authors" : {
      "href" : "http://localhost:8080/books/1/authors"
    }
  }
}
```

太好了，我们可以看到一个链接到我们的投影。让我们检查一下我们在`http://localhost:8080/books/1?projection=customBook`创建的视图:

```
{
  "title" : "Animal Farm",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/books/1"
    },
    "book" : {
      "href" : "http://localhost:8080/books/1{?projection}",
      "templated" : true
    },
    "authors" : {
      "href" : "http://localhost:8080/books/1/authors"
    }
  }
}
```

在这里，我们可以看到我们只获得了`title`字段，而`isbn`不再出现在自定义视图中。

一般来说，我们可以在`http://localhost:8080/books/1?projection={projection name}.`访问投影的结果

另外，请注意，我们需要在与模型相同的包中定义我们的`Projection`。或者，我们可以使用`RepositoryRestConfigurerAdapter`来显式添加它:

```
@Configuration
public class RestConfig implements RepositoryRestConfigurer {

    @Override
    public void configureRepositoryRestConfiguration(
      RepositoryRestConfiguration repositoryRestConfiguration, CorsRegistry cors) {
        repositoryRestConfiguration.getProjectionConfiguration()
          .addProjection(CustomBook.class);
    }
}
```

## 4。向预测添加新数据

现在，让我们看看如何向我们的投影添加新数据。

正如我们在上一节中讨论的，我们可以使用投影来选择视图中包含哪些属性。更重要的是，我们还可以添加原始视图中没有包含的数据。

### 4.1。隐藏数据

默认情况下，id 不包含在原始资源视图中。

为了在结果中看到 id，我们可以显式地包含`id`字段:

```
@Projection(
  name = "customBook", 
  types = { Book.class }) 
public interface CustomBook {
    @Value("#{target.id}")
    long getId(); 

    String getTitle();
}
```

现在在`http://localhost:8080/books/1?projection={projection name}` 的输出将是:

```
{
  "id" : 1,
  "title" : "Animal Farm",
  "_links" : {
     ...
  }
}
```

请注意，我们还可以用`@JsonIgnore.`包含原始视图中隐藏的数据

### 4.2。计算数据

我们还可以包括根据资源属性计算的新数据。

例如，我们可以在预测中包括作者数量:

```
@Projection(name = "customBook", types = { Book.class }) 
public interface CustomBook {

    @Value("#{target.id}")
    long getId(); 

    String getTitle();

    @Value("#{target.getAuthors().size()}")
    int getAuthorCount();
}
```

我们可以在`http://localhost:8080/books/1?projection=customBook`查看:

```
{
  "id" : 1,
  "title" : "Animal Farm",
  "authorCount" : 1,
  "_links" : {
     ...
  }
}
```

### 4.3。轻松访问相关资源

最后，如果我们经常需要访问相关资源——就像我们的例子中的一本书的作者，我们可以通过显式包含它来避免额外的请求:

```
@Projection(
  name = "customBook", 
  types = { Book.class }) 
public interface CustomBook {

    @Value("#{target.id}")
    long getId(); 

    String getTitle();

    List<Author> getAuthors();

    @Value("#{target.getAuthors().size()}")
    int getAuthorCount();
}
```

最终的`Projection`输出将是:

```
{
  "id" : 1,
  "title" : "Animal Farm",
  "authors" : [ {
    "name" : "George Orwell"
  } ],
  "authorCount" : 1,
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/books/1"
    },
    "book" : {
      "href" : "http://localhost:8080/books/1{?projection}",
      "templated" : true
    },
    "authors" : {
      "href" : "http://localhost:8080/books/1/authors"
    }
  }
}
```

接下来，我们来看看摘录。

## 5。摘录

摘录是我们作为默认视图应用于资源集合的预测。

让我们定制我们的`BookRepository`来自动使用`customBook` `Projection`进行收集响应。

为此，我们将使用`@RepositoryRestResource`注释的`excerptProjection`属性:

```
@RepositoryRestResource(excerptProjection = CustomBook.class)
public interface BookRepository extends CrudRepository<Book, Long> {}
```

现在我们可以通过调用`http://localhost:8080/books`来确保`customBook`是图书收藏的默认视图:

```
{
  "_embedded" : {
    "books" : [ {
      "id" : 1,
      "title" : "Animal Farm",
      "authors" : [ {
        "name" : "George Orwell"
      } ],
      "authorCount" : 1,
      "_links" : {
        "self" : {
          "href" : "http://localhost:8080/books/1"
        },
        "book" : {
          "href" : "http://localhost:8080/books/1{?projection}",
          "templated" : true
        },
        "authors" : {
          "href" : "http://localhost:8080/books/1/authors"
        }
      }
    } ]
  },
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/books"
    },
    "profile" : {
      "href" : "http://localhost:8080/profile/books"
    }
  }
}
```

这同样适用于在`http://localhost:8080/authors/1/books`查看特定作者的书籍:

```
{
  "_embedded" : {
    "books" : [ {
      "id" : 1,
      "authors" : [ {
        "name" : "George Orwell"
      } ],
      "authorCount" : 1,
      "title" : "Animal Farm",
      "_links" : {
        "self" : {
          "href" : "http://localhost:8080/books/1"
        },
        "book" : {
          "href" : "http://localhost:8080/books/1{?projection}",
          "templated" : true
        },
        "authors" : {
          "href" : "http://localhost:8080/books/1/authors"
        }
      }
    } ]
  },
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/authors/1/books"
    }
  }
}
```

如上所述，摘录仅自动应用于集合资源。对于单个资源，我们必须使用前面几节中显示的`projection`参数。

这是因为如果我们将预测应用为单个资源的默认视图，将很难知道如何从局部视图更新资源。

最后一点，重要的是要记住**预测和摘录是只读的**。

## 6。结论

我们学习了如何使用 Spring Data REST 投影来创建模型的定制视图。我们还学习了如何使用摘录作为资源集合的默认视图。

示例的完整源代码可以在 GitHub 的[中找到。](https://web.archive.org/web/20220625234410/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-rest)