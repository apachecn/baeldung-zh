# 在 Spring Data REST 中处理关系

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-rest-relationships>

## 1。概述

在本教程中，我们将学习**如何在 Spring Data REST** 中处理实体之间的关系。

我们将关注 Spring Data REST 为存储库公开的关联资源，考虑我们可以定义的每种类型的关系。

为了避免任何额外的设置，我们将使用`H2`嵌入式数据库作为示例。我们可以在我们的[Spring Data REST 介绍](/web/20220625071334/https://www.baeldung.com/spring-data-rest-intro)文章中找到所需依赖项的列表。

## 2。一对一的关系

### 2.1。数据模型

让我们通过使用`@OneToOne`注释来定义两个实体类，`Library`和`Address,` 具有一对一的关系。关联归关联的`Library`端所有:

```
@Entity
public class Library {

    @Id
    @GeneratedValue
    private long id;

    @Column
    private String name;

    @OneToOne
    @JoinColumn(name = "address_id")
    @RestResource(path = "libraryAddress", rel="address")
    private Address address;

    // standard constructor, getters, setters
}
```

```
@Entity
public class Address {

    @Id
    @GeneratedValue
    private long id;

    @Column(nullable = false)
    private String location;

    @OneToOne(mappedBy = "address")
    private Library library;

    // standard constructor, getters, setters
}
```

`@RestResource`注释是可选的，我们可以用它来定制端点。

**我们还必须注意每个关联资源有不同的名称**。否则，我们会遇到带有消息`“Detected multiple association links with same relation type! Disambiguate association.”`的`JsonMappingException`

关联名默认为属性名，我们可以使用`@RestResource`注释的`rel`属性对其进行定制:

```
@OneToOne
@JoinColumn(name = "secondary_address_id")
@RestResource(path = "libraryAddress", rel="address")
private Address secondaryAddress;
```

如果我们将上面的`secondaryAddress`属性添加到`Library`类中，我们将有两个名为`address`的资源，因此会遇到冲突。

我们可以通过为`rel`属性指定一个不同的值来解决这个问题，或者省略`RestResource`注释，使资源名默认为`secondaryAddress`。

### 2.2。仓库

为了**将这些实体作为资源**公开，我们将通过扩展`CrudRepository`接口为每个实体创建两个存储库接口:

```
public interface LibraryRepository extends CrudRepository<Library, Long> {}
```

```
public interface AddressRepository extends CrudRepository<Address, Long> {}
```

### 2.3。创建资源

首先，我们将添加一个`Library`实例来处理:

```
curl -i -X POST -H "Content-Type:application/json" 
  -d '{"name":"My Library"}' http://localhost:8080/libraries
```

然后 API 返回 JSON 对象:

```
{
  "name" : "My Library",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/libraries/1"
    },
    "library" : {
      "href" : "http://localhost:8080/libraries/1"
    },
    "address" : {
      "href" : "http://localhost:8080/libraries/1/libraryAddress"
    }
  }
}
```

注意，如果我们在 Windows 上使用`curl`，我们必须对代表`JSON`主体的`String`中的双引号字符进行转义:

```
-d "{\"name\":\"My Library\"}"
```

我们可以在响应体中看到，一个关联资源已经在`libraries/{libraryId}/address`端点公开。

在我们创建关联之前，向这个端点发送 GET 请求将返回一个空对象。

但是，如果我们想要添加一个关联，我们必须首先创建一个`Address`实例:

```
curl -i -X POST -H "Content-Type:application/json" 
  -d '{"location":"Main Street nr 5"}' http://localhost:8080/addresses
```

POST 请求的结果是一个包含`Address`记录的 JSON 对象:

```
{
  "location" : "Main Street nr 5",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/addresses/1"
    },
    "address" : {
      "href" : "http://localhost:8080/addresses/1"
    },
    "library" : {
      "href" : "http://localhost:8080/addresses/1/library"
    }
  }
}
```

### 2.4。创建关联

**在持久化两个实例之后，我们可以通过使用其中一个关联资源**来建立关系。

这是使用 HTTP 方法 PUT 完成的，该方法支持媒体类型`text/uri-list`和包含绑定到关联的资源的`URI`的主体。

由于`Library`实体是关联的所有者，我们将向库添加一个地址:

```
curl -i -X PUT -d "http://localhost:8080/addresses/1" 
  -H "Content-Type:text/uri-list" http://localhost:8080/libraries/1/libraryAddress
```

如果成功，它将返回状态 204。为了验证这一点，我们可以检查`address`的`library`关联资源:

```
curl -i -X GET http://localhost:8080/addresses/1/library
```

它应该返回名为`“My Library.”`的`Library` JSON 对象

为了**删除关联**，我们可以使用 DELETE 方法调用端点，确保使用关系所有者的关联资源:

```
curl -i -X DELETE http://localhost:8080/libraries/1/libraryAddress
```

## 3。一对多关系

我们使用`@OneToMany`和`@ManyToOne`注释定义一对多关系。我们还可以添加可选的`@RestResource`注释来定制关联资源。

### 3.1。数据模型

为了举例说明一对多关系，我们将添加一个新的`Book`实体，它表示与`Library`实体的关系的“多”端:

```
@Entity
public class Book {

    @Id
    @GeneratedValue
    private long id;

    @Column(nullable=false)
    private String title;

    @ManyToOne
    @JoinColumn(name="library_id")
    private Library library;

    // standard constructor, getter, setter
}
```

然后我们也将关系添加到`Library`类中:

```
public class Library {

    //...

    @OneToMany(mappedBy = "library")
    private List<Book> books;

    //...

}
```

### 3.2。储存库

我们还需要创建一个`BookRepository`:

```
public interface BookRepository extends CrudRepository<Book, Long> { }
```

### 3.3。协会资源

为了**向图书馆**添加一本书，我们需要首先使用/ `books`集合资源创建一个`Book`实例:

```
curl -i -X POST -d "{\"title\":\"Book1\"}" 
  -H "Content-Type:application/json" http://localhost:8080/books
```

这是帖子请求的回复:

```
{
  "title" : "Book1",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/books/1"
    },
    "book" : {
      "href" : "http://localhost:8080/books/1"
    },
    "bookLibrary" : {
      "href" : "http://localhost:8080/books/1/library"
    }
  }
}
```

在响应体中，我们可以看到关联端点`/books/{bookId}/library,`已经创建。

现在让我们**通过向包含图书馆资源`URI`的关联资源发送一个 PUT 请求，将这本书与我们在上一节中创建的图书馆**关联起来:

```
curl -i -X PUT -H "Content-Type:text/uri-list" 
-d "http://localhost:8080/libraries/1" http://localhost:8080/books/1/library
```

我们可以通过使用图书馆/ `books`关联资源上的 GET 方法来**验证图书馆**中的图书:

```
curl -i -X GET http://localhost:8080/libraries/1/books
```

返回的 JSON 对象将包含一个`books`数组:

```
{
  "_embedded" : {
    "books" : [ {
      "title" : "Book1",
      "_links" : {
        "self" : {
          "href" : "http://localhost:8080/books/1"
        },
        "book" : {
          "href" : "http://localhost:8080/books/1"
        },
        "bookLibrary" : {
          "href" : "http://localhost:8080/books/1/library"
        }
      }
    } ]
  },
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/libraries/1/books"
    }
  }
}
```

为了**删除一个关联**，我们可以在关联资源上使用 DELETE 方法:

```
curl -i -X DELETE http://localhost:8080/books/1/library
```

## 4。多对多关系

我们使用`@ManyToMany`注释定义了一个多对多的关系，我们还可以向其中添加`@RestResource`。

### 4.1。数据模型

为了创建一个多对多关系的例子，我们将添加一个新的模型类`Author,` ，它与`Book`实体有多对多关系:

```
@Entity
public class Author {

    @Id
    @GeneratedValue
    private long id;

    @Column(nullable = false)
    private String name;

    @ManyToMany(cascade = CascadeType.ALL)
    @JoinTable(name = "book_author", 
      joinColumns = @JoinColumn(name = "book_id", referencedColumnName = "id"), 
      inverseJoinColumns = @JoinColumn(name = "author_id", 
      referencedColumnName = "id"))
    private List<Book> books;

    //standard constructors, getters, setters
}
```

然后我们也将关联添加到`Book`类中:

```
public class Book {

    //...

    @ManyToMany(mappedBy = "books")
    private List<Author> authors;

    //...
}
```

### 4.2。储存库

接下来，我们将创建一个存储库接口来管理`Author`实体:

```
public interface AuthorRepository extends CrudRepository<Author, Long> { }
```

### 4.3。协会资源

如前几节所述，在建立关联之前，我们必须首先**创建资源**。

我们将通过向/ `authors` 集合资源发送 POST 请求来创建一个`Author`实例:

```
curl -i -X POST -H "Content-Type:application/json" 
  -d "{\"name\":\"author1\"}" http://localhost:8080/authors
```

接下来，我们将向数据库添加第二条`Book`记录:

```
curl -i -X POST -H "Content-Type:application/json" 
  -d "{\"title\":\"Book 2\"}" http://localhost:8080/books
```

然后，我们将对我们的`Author`记录执行 GET 请求，以查看关联 URL:

```
{
  "name" : "author1",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/authors/1"
    },
    "author" : {
      "href" : "http://localhost:8080/authors/1"
    },
    "books" : {
      "href" : "http://localhost:8080/authors/1/books"
    }
  }
}
```

现在，我们可以使用端点`authors/1/books`和 PUT 方法**在两个`Book`记录和`Author`记录之间创建一个关联**，它支持媒体类型`text/uri-list`，并且可以接收多个`URI`。

要发送多个`URI`,我们必须用换行符将它们分开:

```
curl -i -X PUT -H "Content-Type:text/uri-list" 
  --data-binary @uris.txt http://localhost:8080/authors/1/books
```

`uris.txt`文件包含书籍的`URI`,每一个都在单独的一行:

```
http://localhost:8080/books/1
http://localhost:8080/books/2
```

为了**验证两本书都是与作者**相关联的 **，我们可以向关联端点发送一个 GET 请求:**

```
curl -i -X GET http://localhost:8080/authors/1/books
```

我们会收到这样的回复:

```
{
  "_embedded" : {
    "books" : [ {
      "title" : "Book 1",
      "_links" : {
        "self" : {
          "href" : "http://localhost:8080/books/1"
        }
      //...
      }
    }, {
      "title" : "Book 2",
      "_links" : {
        "self" : {
          "href" : "http://localhost:8080/books/2"
        }
      //...
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

为了**删除一个关联**，我们可以用 DELETE 方法向关联资源的 URL 发送一个请求，后跟`{bookId}`:

```
curl -i -X DELETE http://localhost:8080/authors/1/books/1
```

## 5。用`TestRestTemplate` 测试端点

让我们创建一个测试类，它注入一个`TestRestTemplate`实例，并定义我们将使用的常数:

```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = SpringDataRestApplication.class, 
  webEnvironment = WebEnvironment.DEFINED_PORT)
public class SpringDataRelationshipsTest {

    @Autowired
    private TestRestTemplate template;

    private static String BOOK_ENDPOINT = "http://localhost:8080/books/";
    private static String AUTHOR_ENDPOINT = "http://localhost:8080/authors/";
    private static String ADDRESS_ENDPOINT = "http://localhost:8080/addresses/";
    private static String LIBRARY_ENDPOINT = "http://localhost:8080/libraries/";

    private static String LIBRARY_NAME = "My Library";
    private static String AUTHOR_NAME = "George Orwell";
}
```

### 5.1。测试一对一关系

我们将创建一个`@Test`方法，通过向集合资源发出 POST 请求来保存`Library`和`Address`对象。

然后，它通过对关联资源的 PUT 请求保存关系，并验证它是通过对同一资源的 GET 请求建立的:

```
@Test
public void whenSaveOneToOneRelationship_thenCorrect() {
    Library library = new Library(LIBRARY_NAME);
    template.postForEntity(LIBRARY_ENDPOINT, library, Library.class);

    Address address = new Address("Main street, nr 1");
    template.postForEntity(ADDRESS_ENDPOINT, address, Address.class);

    HttpHeaders requestHeaders = new HttpHeaders();
    requestHeaders.add("Content-type", "text/uri-list");
    HttpEntity<String> httpEntity 
      = new HttpEntity<>(ADDRESS_ENDPOINT + "/1", requestHeaders);
    template.exchange(LIBRARY_ENDPOINT + "/1/libraryAddress", 
      HttpMethod.PUT, httpEntity, String.class);

    ResponseEntity<Library> libraryGetResponse 
      = template.getForEntity(ADDRESS_ENDPOINT + "/1/library", Library.class);
    assertEquals("library is incorrect", 
      libraryGetResponse.getBody().getName(), LIBRARY_NAME);
}
```

### 5.2。测试一对多关系

现在我们将创建一个保存一个`Library`实例和两个`Book`实例的`@Test`方法，向每个`Book`对象的`/library`关联资源发送一个 PUT 请求，并验证关系是否已经保存:

```
@Test
public void whenSaveOneToManyRelationship_thenCorrect() {
    Library library = new Library(LIBRARY_NAME);
    template.postForEntity(LIBRARY_ENDPOINT, library, Library.class);

    Book book1 = new Book("Dune");
    template.postForEntity(BOOK_ENDPOINT, book1, Book.class);

    Book book2 = new Book("1984");
    template.postForEntity(BOOK_ENDPOINT, book2, Book.class);

    HttpHeaders requestHeaders = new HttpHeaders();
    requestHeaders.add("Content-Type", "text/uri-list");    
    HttpEntity<String> bookHttpEntity 
      = new HttpEntity<>(LIBRARY_ENDPOINT + "/1", requestHeaders);
    template.exchange(BOOK_ENDPOINT + "/1/library", 
      HttpMethod.PUT, bookHttpEntity, String.class);
    template.exchange(BOOK_ENDPOINT + "/2/library", 
      HttpMethod.PUT, bookHttpEntity, String.class);

    ResponseEntity<Library> libraryGetResponse = 
      template.getForEntity(BOOK_ENDPOINT + "/1/library", Library.class);
    assertEquals("library is incorrect", 
      libraryGetResponse.getBody().getName(), LIBRARY_NAME);
}
```

### 5.3。测试多对多关系

为了测试`Book`和`Author`实体之间的多对多关系，我们将创建一个保存一个`Author`记录和两个`Book`记录的测试方法。

然后，它向带有两个`Books` ' `URI`的`/books`关联资源发送一个 PUT 请求，并验证关系已经建立:

```
@Test
public void whenSaveManyToManyRelationship_thenCorrect() {
    Author author1 = new Author(AUTHOR_NAME);
    template.postForEntity(AUTHOR_ENDPOINT, author1, Author.class);

    Book book1 = new Book("Animal Farm");
    template.postForEntity(BOOK_ENDPOINT, book1, Book.class);

    Book book2 = new Book("1984");
    template.postForEntity(BOOK_ENDPOINT, book2, Book.class);

    HttpHeaders requestHeaders = new HttpHeaders();
    requestHeaders.add("Content-type", "text/uri-list");
    HttpEntity<String> httpEntity = new HttpEntity<>(
      BOOK_ENDPOINT + "/1\n" + BOOK_ENDPOINT + "/2", requestHeaders);
    template.exchange(AUTHOR_ENDPOINT + "/1/books", 
      HttpMethod.PUT, httpEntity, String.class);

    String jsonResponse = template
      .getForObject(BOOK_ENDPOINT + "/1/authors", String.class);
    JSONObject jsonObj = new JSONObject(jsonResponse).getJSONObject("_embedded");
    JSONArray jsonArray = jsonObj.getJSONArray("authors");
    assertEquals("author is incorrect", 
      jsonArray.getJSONObject(0).getString("name"), AUTHOR_NAME);
}
```

## 6。结论

在本文中，我们演示了不同类型的关系在 Spring Data REST 中的使用。

这些例子的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220625071334/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-rest)