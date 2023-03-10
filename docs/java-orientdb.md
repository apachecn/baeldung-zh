# 面向 java apis 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-orientdb>

## 1。概述

[OrientDB](https://web.archive.org/web/20220629004417/https://orientdb.com/) 是一种开源的多模型 NoSQL 数据库技术，旨在使用[图](https://web.archive.org/web/20220629004417/https://en.wikipedia.org/wiki/Graph_database)、[文档](https://web.archive.org/web/20220629004417/https://en.wikipedia.org/wiki/Document-oriented_database)、[键值](https://web.archive.org/web/20220629004417/https://en.wikipedia.org/wiki/Key-value_database)、[地理空间](https://web.archive.org/web/20220629004417/https://orientdb.com/docs/2.2.x/Spatial-Index.html)和[反应](https://web.archive.org/web/20220629004417/https://orientdb.com/docs/2.2.x/Live-Query.html)模型，同时使用 [SQL](https://web.archive.org/web/20220629004417/https://orientdb.com/docs/2.2.x/SQL.html) 语法管理查询。

在本文中，我们将介绍设置和使用 OrientDB Java APIs。

## 2。安装

首先，我们需要安装二进制包。

让我们下载最新稳定版的 [OrientDB](https://web.archive.org/web/20220629004417/https://orientdb.com/download-previous/) ( [2.2.x](https://web.archive.org/web/20220629004417/https://orientdb.com/download-previous/orientdb-neo4j-importer-2.2.31.tar.gz) 在撰写本文时)。

其次，我们需要解压缩它并将它的内容移动到一个方便的目录中(使用`ORIENTDB_HOME`)。请确保将`bin`文件夹添加到环境变量中，以便于命令行使用。

最后，我们需要编辑位于`$ORIENTDB_HOME/bin`中的`orientdb.sh` 文件，在`ORIENTDB_DIR` 处填入 OrientDB 目录的位置(`ORIENTDB_HOME`)，以及我们希望使用的系统用户，而不是`USER_YOU_WANT_ORIENTDB_RUN_WITH`。

现在我们有了一个完全工作的 OrientDB。我们可以使用带有选项的 `orientdb.sh <option>` 脚本:

*   `start`:启动服务器
*   `status` :检查状态

*   `stop` :停止服务器

**请注意`start`和`stop`操作都需要用户密码(我们在`orientdb.sh`文件中设置的密码)。**

一旦服务器启动，它将占用端口 2480。因此，我们可以使用这个 **[URL](https://web.archive.org/web/20220629004417/http://localhost:2480/studio/index.html) :** 在本地访问它

[![orientdb running](img/79370ac131a304f00944c0b15b561b59.png)](/web/20220629004417/https://www.baeldung.com/wp-content/uploads/2017/12/orientdb-running.png)

手动安装的更多细节可在此处找到[。](https://web.archive.org/web/20220629004417/https://orientdb.com/docs/2.2.x/Tutorial-Installation.html)

注意:OrientDB 要求[Java](https://web.archive.org/web/20220629004417/https://www.java.com/en/download/)1.7 版或更高版本。

以前的版本可在[这里](https://web.archive.org/web/20220629004417/http://orientdb.com/download-previous/)获得。

## 3。OrientDB Java APIs 设置

OrientDB 允许 Java 开发人员使用三种不同的 API，例如:

*   图形 API–图形数据库
*   文档 API–面向文档的数据库
*   对象 API–直接绑定到面向文档的对象

通过集成和使用 OrientDB，我们可以在一个代码库中使用所有这些类型。

让我们看一下可以包含在项目的类路径中的一些可用 jar:

*   `orientdb-core-*.jar`:带来核心库
*   `blueprints-core-*.jar`:带适配器核心部件
*   `orientdb-graphdb-*.jar`:给出图形数据库 API
*   `orientdb-object-*.jar`:提供对象数据库 API
*   `orientdb-distributed-*.jar`:提供分布式数据库插件，与服务器集群协同工作
*   `orientdb-tools-*.jar`:移交控制台命令
*   `orientdb-client-*.jar`:提供远程客户端
*   `orientdb-enterprise-*.jar`:启用客户端和服务器共享的协议和网络类

只有当我们在远程服务器上管理数据时，才需要最后两个。

让我们从一个 Maven 项目开始，并使用以下依赖项:

```java
<dependency>
    <groupId>com.orientechnologies</groupId>
    <artifactId>orientdb-core</artifactId>
    <version>2.2.31</version>
</dependency>
<dependency>
    <groupId>com.orientechnologies</groupId>
    <artifactId>orientdb-graphdb</artifactId>
    <version>2.2.31</version>
</dependency>
<dependency>
    <groupId>com.orientechnologies</groupId>
    <artifactId>orientdb-object</artifactId>
    <version>2.2.31</version>
</dependency>
<dependency>
    <groupId>com.tinkerpop.blueprints</groupId>
    <artifactId>blueprints-core</artifactId>
    <version>2.6.0</version>
</dependency>
```

请检查 Maven 中央存储库中 OrientDB 的最新版本的[核心](https://web.archive.org/web/20220629004417/http://www.maven.org/#search%7Cgav%7C1%7Cg%3A%22com.orientechnologies%22%20AND%20a%3A%22orientdb-core%22)、 [GraphDB](https://web.archive.org/web/20220629004417/http://www.maven.org/#search%7Cgav%7C1%7Cg%3A%22com.orientechnologies%22%20AND%20a%3A%22orientdb-graphdb%22) 、[对象](https://web.archive.org/web/20220629004417/http://www.maven.org/#search|gav|1|g%3A%22com.orientechnologies%22%20AND%20a%3A%22orientdb-object%22)API 和[蓝图-核心](https://web.archive.org/web/20220629004417/http://www.maven.org/#search%7Cgav%7C1%7Cg%3A%22com.tinkerpop.blueprints%22%20AND%20a%3A%22blueprints-core%22)。

## 4.使用

OrientDB 使用 [TinkerPop Blueprints](https://web.archive.org/web/20220629004417/https://tinkerpop.apache.org/) 实现来处理图形。

TinkerPop 是一个图形计算框架，提供了许多构建图形数据库的方法，每种方法都有自己的实现:

*   [蓝图](https://web.archive.org/web/20220629004417/https://github.com/tinkerpop/blueprints/wiki)
*   [管道](https://web.archive.org/web/20220629004417/https://github.com/tinkerpop/pipes/wiki)
*   [小妖精](https://web.archive.org/web/20220629004417/https://github.com/tinkerpop/gremlin/wiki)
*   雷克斯斯特
*   sail 听力

此外，OrientDB 允许使用三种类型的[模式](https://web.archive.org/web/20220629004417/https://orientdb.com/docs/last/java/Graph-Schema.html),而不考虑 API 的类型:

*   启用了 schema-Full-strict 模式，因此所有字段都是在类创建期间指定的
*   无模式——类是在没有特定属性的情况下创建的，所以我们可以根据需要添加它们；**这是默认模式**
*   模式混合——它是全模式和无模式的混合，我们可以创建一个具有预定义字段的类，但让记录定义其他自定义字段

### 4.1。图形 API

由于这是一个基于图形的数据库，数据被表示为一个包含由[边](https://web.archive.org/web/20220629004417/https://orientdb.com/docs/2.2.x/Graph-VE.html#edges)(弧)互连的[顶点](https://web.archive.org/web/20220629004417/https://orientdb.com/docs/2.2.x/Graph-VE.html#vertices)(节点)的网络。

作为第一步，让我们使用 UI 创建一个名为`BaeldungDB`的图形数据库，用户为`admin`，密码为`admin.`

正如我们在后面的图片中看到的，`graph`已经被选为数据库类型，因此它的数据将可以在`GRAPH Tab`中访问:

[![create graph db](img/11359d04d20d455a8d68ea15ff6b66af.png)](/web/20220629004417/https://www.baeldung.com/wp-content/uploads/2017/12/create-graph-db.png)

现在让我们连接到所需的数据库，知道`ORIENTDB_HOME`是一个环境变量，对应于`OrientDB`的安装文件夹:

```java
@BeforeClass
public static void setup() {
    String orientDBFolder = System.getenv("ORIENTDB_HOME");
    graph = new OrientGraphNoTx("plocal:" + orientDBFolder + 
      "/databases/BaeldungDB", "admin", "admin");
}
```

让我们初始化`Article`、`Author`和`Editor`类——同时展示如何向它们的字段添加验证:

```java
@BeforeClass
public static void init() {
    graph.createVertexType("Article");

    OrientVertexType writerType
      = graph.createVertexType("Writer");
    writerType.setStrictMode(true);
    writerType.createProperty("firstName", OType.STRING);
    // ...

    OrientVertexType authorType 
      = graph.createVertexType("Author", "Writer");
    authorType.createProperty("level", OType.INTEGER).setMax("3");

    OrientVertexType editorType
      = graph.createVertexType("Editor", "Writer");
    editorType.createProperty("level", OType.INTEGER).setMin("3");

    Vertex vEditor = graph.addVertex("class:Editor");
    vEditor.setProperty("firstName", "Maxim");
    // ...

    Vertex vAuthor = graph.addVertex("class:Author");
    vAuthor.setProperty("firstName", "Jerome");
    // ...

    Vertex vArticle = graph.addVertex("class:Article");
    vArticle.setProperty("title", "Introduction to ...");
    // ...

    graph.addEdge(null, vAuthor, vEditor, "has");
    graph.addEdge(null, vAuthor, vArticle, "wrote");
}
```

在上面的代码片段中，我们制作了一个简单数据库的简单表示，其中:

*   `Article`是包含文章的无模式类
*   `Writer`是一个全模式的超类，包含必要的编写器信息
*   `Writer`是`Author`的一个子类型，保存它的细节
*   `Editor`是`Writer`的无模式子类型，保存编辑器细节
*   `lastName` 字段尚未填入保存的作者，但仍出现在下图中
*   我们在所有类之间有一个关系:一个`Author`可以写`Article`并且需要有一个`Editor`
*   `Vertex`代表有一些字段的实体
*   `Edge`是连接两个`Vertices`的实体

**请注意，通过尝试向一个完整类的对象添加另一个属性，我们将以 [OValidationException](https://web.archive.org/web/20220629004417/https://orientdb.com/javadoc/latest/com/orientechnologies/orient/core/exception/OValidationException.html) 结束。**

使用 [OrientDB studio](https://web.archive.org/web/20220629004417/https://orientdb.com/docs/2.2.x/Tutorial-Run-the-studio.html) 连接到我们的数据库后，我们将看到数据的图形表示:

[![orientdb graph](img/01798252d92f757c3200a2a057f03f3a.png)](/web/20220629004417/https://www.baeldung.com/wp-content/uploads/2017/12/orientdb-graph.png)

让我们看看如何获得数据库中所有记录(顶点)的数量:

```java
long size = graph.countVertices();
```

现在，让我们只显示`Writer (Author & Editor)`对象的数量:

```java
@Test
public void givenBaeldungDB_checkWeHaveTwoWriters() {
    long size = graph.countVertices("Writer");

    assertEquals(2, size);
}
```

在下一步中，我们可以使用以下语句找到所有`Writer`的数据:

```java
Iterable<Vertex> writers = graph.getVerticesOfClass("Writer");
```

最后，让我们查询所有带有`level` 7 的`Editor`;这里我们只有一个匹配的:

```java
@Test
public void givenBaeldungDB_getEditorWithLevelSeven() {
    String onlyEditor = "";
    for(Vertex v : graph.getVertices("Editor.level", 7)) {
        onlyEditor = v.getProperty("firstName").toString();
    }

    assertEquals("Maxim", onlyEditor);
}
```

请求时，总是指定类名来查找特定的顶点。我们可以在这里找到更多细节[。](https://web.archive.org/web/20220629004417/https://orientdb.com/docs/last/java/Graph-Database-Tinkerpop.html)

### 4.2。文件 API

下一个选项是使用 OrientDB 的文档模型。这通过一个简单的记录来公开数据操作，记录中的信息存储在字段中，类型可以是文本、图片或二进制形式。

让我们再次使用 UI 创建一个名为`BaeldungDBTwo`的数据库，但是现在使用一个 [**文档**](https://web.archive.org/web/20220629004417/https://orientdb.com/javadoc/latest/com/orientechnologies/orient/core/record/impl/ODocument.html) 作为类型:

[![create document db](img/a5967142f57b48e8b8fbc3151f48c835.png)](/web/20220629004417/https://www.baeldung.com/wp-content/uploads/2017/12/create-document-db.png)

注意:同样，这个 API 也可以在全模式、无模式或混合模式下使用。

数据库连接仍然很简单，因为我们只需要实例化一个`ODatabaseDocumentTx`对象，提供数据库 URL 和数据库用户的凭证:

```java
@BeforeClass
public static void setup() {
    String orientDBFolder = System.getenv("ORIENTDB_HOME");
    db = new ODatabaseDocumentTx("plocal:" 
      + orientDBFolder + "/databases/BaeldungDBTwo")
      .open("admin", "admin");
}
```

**让我们从保存一个保存有`Author`信息的简单文档开始。**

在这里，我们可以看到该类已被自动创建:

```java
@Test
public void givenDB_whenSavingDocument_thenClassIsAutoCreated() {
    ODocument doc = new ODocument("Author");
    doc.field("name", "Paul");
    doc.save();

    assertEquals("Author", doc.getSchemaClass().getName());
}
```

相应地，为了计算`Authors`的数量，我们可以使用:

```java
long size = db.countClass("Author");
```

让我们使用一个字段值再次查询文档，搜索带有`level` 7:

```java
@Test
public void givenDB_whenSavingAuthors_thenWeGetOnesWithLevelSeven() {
    for (ODocument author : db.browseClass("Author")) author.delete();

    ODocument authorOne = new ODocument("Author");
    authorOne.field("firstName", "Leo");
    authorOne.field("level", 7);
    authorOne.save();

    ODocument authorTwo = new ODocument("Author");
    authorTwo.field("firstName", "Lucien");
    authorTwo.field("level", 9);
    authorTwo.save();

    List<ODocument> result = db.query(
      new OSQLSynchQuery<ODocument>("select * from Author where level = 7"));

    assertEquals(1, result.size());
}
```

同样，要删除`Author`类的所有记录，我们可以使用:

```java
for (ODocument author : db.browseClass("Author")) {
    author.delete();
}
```

在 OrientDB studio 的`BROWSE Tab`上，我们可以通过查询来获取所有的`Author'`对象:

[![orientdb document](img/9da67c31032a59d1c0004fd23b886a26.png)](/web/20220629004417/https://www.baeldung.com/wp-content/uploads/2017/12/orientdb-document.png)

### 4.3。对象 API

OrientDB 没有数据库的对象类型。因此，对象 API 依赖于文档数据库。

在 Object API 类型中，所有其他概念都保持不变，只有一个增加——[将](https://web.archive.org/web/20220629004417/https://orientdb.com/docs/2.2.x/Object-2-Record-Java-Binding.html)绑定到 POJO。

让我们从使用`OObjectDatabaseTx`类连接到`BaeldungDBThree`开始:

```java
@BeforeClass
public static void setup() {
    String orientDBFolder = System.getenv("ORIENTDB_HOME");
    db = new OObjectDatabaseTx("plocal:" 
      + orientDBFolder + "/databases/BaeldungDBThree")
      .open("admin", "admin");
}
```

接下来，假设`Author`是用于保存`Author`数据的 POJO，我们需要注册它:

```java
db.getEntityManager().registerEntityClass(Author.class);
```

`Author`有以下字段的 getters 和 setters:

*   `firstName`
*   `lastName`
*   `level`

让我们创建一个带有多行指令的`Author`,如果我们确认了一个无参数的构造函数:

```java
Author author = db.newInstance(Author.class);
author.setFirstName("Luke");
author.setLastName("Sky");
author.setLevel(9);
db.save(author);
```

另一方面，如果我们有另一个构造函数，它分别接受`Author`的`firstName`、`lastName`和`level`，那么实例化只有一行:

```java
Author author = db.newInstance(Author.class, "Luke", "Sky", 9);
db.save(author);
```

下面几行用来浏览和删除 Author 类的所有记录:

```java
for (Author author : db.browseClass(Author.class)) {
    db.delete(author);
}
```

为了统计所有作者，我们只需提供类和数据库实例，而无需编写 SQL 查询:

```java
long authorsCount = db.countClass(Author.class);
```

类似地，我们用`level` 7 查询作者，如下所示:

```java
@Test
public void givenDB_whenSavingAuthors_thenWeGetOnesWithLevelSeven() {
    for (Author author : db.browseClass(Author.class)) {
        db.delete(author);
    }

    Author authorOne 
      = db.newInstance(Author.class, "Leo", "Marta", 7);
    db.save(authorOne);

    Author authorTwo
      = db.newInstance(Author.class, "Lucien", "Aurelien", 9);
    db.save(authorTwo);

    List<Author> result
      = db.query(new OSQLSynchQuery<Author>(
      "select * from Author where level = 7"));

    assertEquals(1, result.size());
}
```

最后，这是介绍一些高级对象 API 用途的[官方指南](https://web.archive.org/web/20220629004417/https://orientdb.com/docs/2.2.x/Object-Database.html)。

## 5。结论

在本文中，我们看到了如何使用 OrientDB 作为数据库管理系统及其 Java APIs。我们还学习了如何在字段上添加验证并编写一些简单的查询。

和往常一样，这篇文章的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20220629004417/https://github.com/eugenp/tutorials/tree/master/persistence-modules/orientdb)