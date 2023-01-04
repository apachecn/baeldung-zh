# 弹簧可选路径变量

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-optional-path-variables>

## 1.概观

在本教程中，我们将学习如何在 Spring 中使路径变量可选。首先，我们将描述[Spring 如何在处理程序方法中绑定`@PathVariable`](/web/20221207175422/https://www.baeldung.com/spring-requestmapping) 参数。然后，我们将展示在不同的 Spring 版本中使路径变量可选的不同方法。

要快速浏览路径变量，请阅读我们的春季 MVC 文章。

## 2.Spring 如何绑定`@PathVariable`参数

默认情况下，Spring 会尝试将处理程序方法中所有用`@PathVariable`标注的参数与 URI 模板中相应的变量绑定在一起。如果 Spring 失败了，它不会将我们的请求传递给那个处理程序方法。

例如，考虑下面的`getArticle`方法，它试图(不成功地)使`id`路径变量可选:

```
@RequestMapping(value = {"/article", "/article/{id}"})
public Article getArticle(@PathVariable(name = "id") Integer articleId) {
    if (articleId != null) {
        //...
    } else {
        //...
    }
}
```

这里，`getArticle`方法应该服务于对`/article`和`/article/{id}`的请求。如果存在的话，Spring 会尝试将`articleId`参数绑定到`id`路径变量。

例如，向`/article/123`发送请求会将`articleId`的值设置为 123。

另一方面，如果我们向`/article`发送一个请求，Spring 由于以下异常返回状态代码 500:

```
org.springframework.web.bind.MissingPathVariableException:
  Missing URI template variable 'id' for method parameter of type Integer
```

这是因为 Spring 无法为参数`articleId`设置值，因为`id`丢失了。

因此，我们需要某种方法来告诉 Spring，如果它没有相应的路径变量，就忽略绑定特定的`@PathVariable`参数，我们将在下面几节中看到。

## 3.使路径变量可选

### 3.1.使用`@PathVariable`的`required`属性

从 Spring 4.3.3 开始，`@PathVariable`注释为我们定义了布尔属性`required`,以指示路径变量对于处理程序方法是否是强制的。

例如，以下版本的`getArticle`使用了`required`属性:

```
@RequestMapping(value = {"/article", "/article/{id}"})
public Article getArticle(@PathVariable(required = false) Integer articleId) {
   if (articleId != null) {
       //...
   } else {
       //...
   }
}
```

由于`required`属性是`false`，如果请求中没有发送`id`路径变量，Spring 不会抱怨。也就是说，如果发送了，Spring 会将`articleId`设置为`id`，否则设置为`null`。

另一方面，如果`required`是`true`，Spring 会抛出一个异常，以防`id`丢失。

### 3.2.使用`Optional` 参数类型

下面的实现展示了 Spring 4.1 如何与 [JDK 8 的`Optional`类](/web/20221207175422/https://www.baeldung.com/java-optional)一起，提供了另一种使`articleId`可选的方法:

```
@RequestMapping(value = {"/article", "/article/{id}"}")
public Article getArticle(@PathVariable Optional<Integer> optionalArticleId) {
    if (optionalArticleId.isPresent()) {
        Integer articleId = optionalArticleId.get();
        //...
    } else {
        //...
    }
}
```

这里，Spring 创建了`Optional<Integer>`实例`optionalArticleId`，来保存`id`的值。如果`id`存在，`optionalArticleId`将包装其值，否则，`optionalArticleId`将包装一个`null`值。然后，我们可以使用`Optional`的`isPresent(),` `get(), `或`orElse()`方法来处理该值。

### 3.3.使用`Map`参数类型

另一种定义可选路径变量的方法是用@ `PathVariable`参数的`M` `ap`:

```
@RequestMapping(value = {"/article", "/article/{id}"})
public Article getArticle(@PathVariable Map<String, String> pathVarsMap) {
    String articleId = pathVarsMap.get("id");
    if (articleId != null) {
        Integer articleIdAsInt = Integer.valueOf(articleId);
        //...
    } else {
        //...
    }
}
```

在本例中，`Map<String, String>` `pathVarsMap`参数收集 URI 中作为键/值对的所有路径变量。然后，我们可以使用`get()`方法得到一个特定的路径变量。

注意，因为 Spring 提取路径变量的值作为`String`，我们使用`Integer.valueOf()`方法将其转换为`Integer`。

### 3.4.使用两种处理程序方法

如果我们使用的是遗留的 Spring 版本，我们可以将`getArticle` handler 方法分成两个方法。

第一种方法将处理对`/article/{id}`的请求:

```
@RequestMapping(value = "/article/{id}")
public Article getArticle(@PathVariable(name = "id") Integer articleId) {
    //...        
} 
```

而第二种方法将处理对`/article`的请求:

```
@RequestMapping(value = "/article")
public Article getDefaultArticle() {
    //...
}
```

## 4.结论

总而言之，我们已经讨论了如何在不同的 Spring 版本中使路径变量可选。

和往常一样，这篇文章的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221207175422/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-basics-4)