# 弹簧座外壳介绍

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-rest-shell>

## 1。概述

在本文中，我们将了解 Spring REST Shell 及其一些特性。

这是一个 Spring Shell 扩展，所以我们建议先阅读关于它的。

## 2。简介

Spring REST Shell 是一个命令行 Shell，旨在方便使用 Spring HATEOAS 兼容的 REST 资源。

我们不再需要通过使用像`curl.` Spring REST Shell 这样的工具来操纵 bash 中的 URL，它提供了一种更方便的与 REST 资源交互的方式。

## 3。安装

如果我们使用带有家酿软件的 macOS 机器，我们可以简单地执行下一个命令:

```
brew install rest-shell
```

对于其他操作系统的用户，我们需要从[官方 GitHub 项目页面](https://web.archive.org/web/20220629004953/https://github.com/spring-projects/rest-shell)下载一个二进制包，解包后找到一个可执行文件运行:

```
tar -zxvf rest-shell-1.2.0.RELEASE.tar.gz
cd rest-shell-1.2.0.RELEASE
bin/rest-shell
```

另一种选择是下载源代码并执行一个梯度任务:

```
git clone git://github.com/spring-projects/rest-shell.git
cd rest-shell
./gradlew installApp
cd build/install/rest-shell-1.2.0.RELEASE
bin/rest-shell
```

如果一切设置正确，我们将看到下面的问候:

```
 ___ ___  __ _____  __  _  _     _ _  __    
| _ \ __/' _/_   _/' _/| || |   / / | \ \   
| v / _|`._`. | | `._`.| >< |  / / /   > >  
|_|_\___|___/ |_| |___/|_||_| |_/_/   /_/   
1.2.1.RELEASE

Welcome to the REST shell. For assistance hit TAB or type "help".
http://localhost:8080:>
```

## 4。入门

我们将使用已经为另一篇文章开发的 API [。`localhost:8080`被用作基本 URL。](https://web.archive.org/web/20220629004953/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-rest-shell)

以下是暴露的端点列表:

*   获取 `/articles`–获取所有的`Article`
*   获取 `/articles/{id}`–通过 id 获取一个`Article`
*   获得 `/articles/search/findByTitle?title={title}`–通过标题获得一个`Article`
*   GET/profile/articles–获取一个`Article`资源的配置文件数据
*   POST `/articles` –创建一个新的`Article`，并提供一个 body

`Article`类有三个字段:`id, title,`和`content.`

### 4.1。创建新资源

让我们增加一个新的条款。我们将使用***post*命令传递带有*–数据*参数**的 JSON `String`。

首先，我们需要`follow`与我们想要添加的资源相关联的 URL。命令 *follow* 获取一个相对 URI，将其与 *baseUri* 连接，并将结果设置为当前位置:

```
http://localhost:8080:> follow articles
http://localhost:8080/articles:> post --data "{title: "First Article"}"
```

该命令的执行结果将是:

```
< 201 CREATED
< Location: http://localhost:8080/articles/1
< Content-Type: application/hal+json;charset=UTF-8
< Transfer-Encoding: chunked
< Date: Sun, 29 Oct 2017 23:04:43 GMT
< 
{
  "title" : "First Article",
  "content" : null,
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/articles/1"
    },
    "article" : {
      "href" : "http://localhost:8080/articles/1"
    }
  }
} 
```

### 4.2。发现资源

现在，当我们有了一些资源，让我们把它们找出来。我们将使用***discover*命令，该命令显示当前 URI** 的所有可用资源:

```
http://localhost:8080/articles:> discover

rel        href                                  
=================================================
self       http://localhost:8080/articles/       
profile    http://localhost:8080/profile/articles
article    http://localhost:8080/articles/1
```

知道了资源 URI，我们可以通过使用 *get* 命令来获取它:

```
http://localhost:8080/articles:> get 1

> GET http://localhost:8080/articles/1

< 200 OK
< Content-Type: application/hal+json;charset=UTF-8
< Transfer-Encoding: chunked
< Date: Sun, 29 Oct 2017 23:25:36 GMT
< 
{
  "title" : "First Article",
  "content" : null,
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/articles/1"
    },
    "article" : {
      "href" : "http://localhost:8080/articles/1"
    }
  }
}
```

### 4.3。添加查询参数

**我们可以使用*–params*参数将查询参数指定为 JSON 片段。**

让我们根据给定的标题来获取一篇文章:

```
http://localhost:8080/articles:> get search/findByTitle \
> --params "{title: "First Article"}"

> GET http://localhost:8080/articles/search/findByTitle?title=First+Article

< 200 OK
< Content-Type: application/hal+json;charset=UTF-8
< Transfer-Encoding: chunked
< Date: Sun, 29 Oct 2017 23:39:39 GMT
< 
{
  "title" : "First Article",
  "content" : null,
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/articles/1"
    },
    "article" : {
      "href" : "http://localhost:8080/articles/1"
    }
  }
}
```

### 4.4。设置标题

**名为`headers`的命令允许管理会话范围**内的报头——每个请求都将使用这些报头发送。*标头集合*采用*–名称*和*–值*参数来确定标头。

我们将添加几个标头，并发出一个包含这些标头的请求:

```
http://localhost:8080/articles:>
  headers set --name Accept --value application/json

{
  "Accept" : "application/json"
}

http://localhost:8080/articles:>
  headers set --name Content-Type --value application/json

{
  "Accept" : "application/json",
  "Content-Type" : "application/json"
}

http://localhost:8080/articles:> get 1

> GET http://localhost:8080/articles/1
> Accept: application/json
> Content-Type: application/json
```

### 4.5。将结果写入文件

将 HTTP 请求的结果打印到屏幕上并不总是可取的。有时，我们需要将结果保存在一个文件中，以便进一步分析。

`–output`参数允许执行这样的操作:

```
http://localhost:8080/articles:> get search/findByTitle \
> --params "{title: "First Article"}" \
> --output first_article.txt

>> first_article.txt
```

### 4.6。从文件中读取 JSON】

通常，JSON 数据太大或太复杂，无法通过控制台使用*–data*参数输入。

此外，我们可以直接在命令行中输入的 JSON 数据的格式也有一些限制。

***–from*参数给出了从文件或目录中读取数据的可能性。**

如果值是一个目录，shell 将读取每个以`“.json”`结尾的文件，并对该文件的内容执行 POST 或 PUT。

如果参数是一个文件，那么 shell 将加载该文件并从该文件中提交/上传数据。

让我们从文件 *second_article.txt* 创建下一篇文章:

```
http://localhost:8080/articles:> post --from second_article.txt

1 files uploaded to the server using POST
```

### 4.7。设置上下文变量

我们还可以在当前会话上下文中定义变量。**命令`var`定义了分别用于获取和设置变量的`get`和`set`参数。**

与`headers`类似，自变量`–name`和`–value`用于给出新变量的名称和值:

```
http://localhost:8080:> var set --name articlesURI --value articles
http://localhost:8080/articles:> var get --name articlesURI

articles 
```

现在，我们将打印出上下文中当前可用变量的列表:

```
http://localhost:8080:> var list

{
  "articlesURI" : "articles"
}
```

确保我们的变量被保存后，我们将使用它和 *follow* 命令切换到给定的 URI:

```
http://localhost:8080:> follow #{articlesURI}
http://localhost:8080/articles:> 
```

### 4.8。查看历史记录

我们走过的路都被记录了下来。**命令`history`按时间顺序显示这些路径**:

```
http://localhost:8080:> history list

1: http://localhost:8080/articles
2: http://localhost:8080 
```

每个 URI 都与一个可用于前往该 URI 的号码相关联:

```
http://localhost:8080:> history go 1
http://localhost:8080/articles:> 
```

## 5。结论

在本教程中，我们关注了 Spring 生态系统中一个有趣而罕见的工具——命令行工具。

你可以在 GitHub 上找到关于项目[的更多信息。](https://web.archive.org/web/20220629004953/https://github.com/spring-projects/rest-shell#commands)

和往常一样，文章中提到的所有代码片段都可以在我们的资源库中找到。