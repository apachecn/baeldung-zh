# JDO 问题介绍 2/2

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jdo-queries>

## 1。概述

在本系列的前一篇文章中，我们展示了如何将 Java 对象持久化到不同的数据存储中。有关更多详细信息，请查看[Java 数据对象指南](/web/20220707143819/https://www.baeldung.com/jdo)。

JDO 支持不同的查询语言，为开发人员使用他最熟悉的查询语言提供了灵活性。

## 2。JDO 查询语言

JDO 支持以下查询语言:

*   JDOQL——使用 Java 语法的查询语言
*   类型化 JDOQL——遵循 JDOQL 语法，但提供了一个 API 来简化查询的使用。
*   SQL–仅用于 RDBMS。
*   JPQL 由 Datanucleus 提供，但不是 JDO 规范的一部分。

## 3。查询 API

### 3.1。创建查询

要创建一个查询，我们需要指定语言和查询`String:`

```java
Query query = pm.newQuery(
  "javax.jdo.query.SQL",
  "select * from product_item where price < 10");
```

如果我们不指定语言，它默认为 JDOQL:

```java
Query query = pm.newQuery(
  "SELECT FROM com.baeldung.jdo.query.ProductItem WHERE price < 10");
```

### 3.2。创建命名查询

我们还可以定义查询，并通过其保存的名称来引用它。

为此，我们首先创建一个`ProductItem`类:

```java
@PersistenceCapable
public class ProductItem {

    @PrimaryKey
    @Persistent(valueStrategy = IdGeneratorStrategy.INCREMENT)
    int id;
    String name;
    String status;
    String description;
    double price;

    //standard getters, setters & constructors
}
```

接下来，我们向`META-INF/package.jdo` 文件添加一个类配置来定义一个查询，并将其命名为:

```java
<jdo>
    <package name="com.baeldung.jdo.query">
        <class name="ProductItem" detachable="true" table="product_item">
            <query name="PriceBelow10" language="javax.jdo.query.SQL">
            <![CDATA[SELECT * FROM PRODUCT_ITEM WHERE PRICE < 10]]>
            </query>
        </class>
    </package>
</jdo>
```

我们定义了一个名为“`PriceBelow10″.` 的查询

我们可以在代码中使用它:

```java
Query<ProductItem> query = pm.newNamedQuery(
  ProductItem.class, "PriceBelow10");
List<ProductItem> items = query.executeList();
```

### 3.3。`Query`闭幕一

为了节省资源，我们可以关闭查询:

```java
query.close();
```

同样，我们可以通过将特定的结果集作为参数传递给`close()`方法来关闭它:

```java
query.close(ResultSet);
```

### 3.4。`Query`编译一个

如果我们想验证一个查询，我们可以调用`compile()`方法:

```java
query.compile();
```

如果查询无效，那么该方法将抛出一个`JDOException.`

## 4。JDOQL

JDOQL 是一种基于对象的查询语言，旨在提供 SQL 语言的强大功能，并保留应用程序模型中的 Java 对象关系。

JDOQL 查询可以用`single-String`的形式定义。

在我们深入探讨之前，让我们回顾一些基本概念:

### 4.1。候选类别

JDOQL 中的候选类必须是一个持久的类。在 SQL 语言中，我们使用完整的类名而不是表名:

```java
Query query = pm.newQuery("SELECT FROM com.baeldung.jdo.query.ProductItem");
List<ProductItem> r = query.executeList();
```

正如我们在上面的例子中看到的，`com.baeldung.jdo.query.ProductItem`是这里的候选类。

### 4.2。过滤器

过滤器可以用 Java 编写，但必须计算为布尔值:

```java
Query query = pm.newQuery("SELECT FROM com.baeldung.jdo.query.ProductItem");
query.setFilter("status == 'SoldOut'");
List<ProductItem> result = query.executeList();
```

### 4.3。方法

JDOQL 并不支持所有的 Java 方法，但是它支持各种我们可以从查询中调用的方法，并且可以在广泛的范围内使用:

```java
query.setFilter("this.name.startsWith('supported')");
```

有关支持方法的更多详细信息，请查看此[链接](https://web.archive.org/web/20220707143819/http://www.datanucleus.org/products/accessplatform_5_1/jdo/query.html#jdoql_methods)。

### 4.4。参数

我们可以将值作为参数传递给查询。我们可以显式或隐式地定义参数。

要显式定义参数:

```java
Query query = pm.newQuery(
  "SELECT FROM com.baeldung.jdo.query.ProductItem "
  + "WHERE price < threshold PARAMETERS double threshold");
List<ProductItem> result = (List<ProductItem>) query.execute(10);
```

这也可以通过使用`setParameters`方法来实现:

```java
Query query = pm.newQuery(
  "SELECT FROM com.baeldung.jdo.query.ProductItem "
  + "WHERE price < :threshold");
query.setParameters("double threshold");
List<ProductItem> result = (List<ProductItem>) query.execute(10);
```

我们可以通过不定义参数类型来隐式实现:

```java
Query query = pm.newQuery(
  "SELECT FROM com.baeldung.jdo.query.ProductItem "
  + "WHERE price < :threshold");
List<ProductItem> result = (List<ProductItem>) query.execute(10); 
```

## 5。JDOQL 类型化

要使用 JDOQLTypedQueryAPI，我们需要准备环境。

### 5.1。Maven 设置

```java
<dependency>
    <groupId>org.datanucleus</groupId>
    <artifactId>datanucleus-jdo-query</artifactId>
    <version>5.0.2</version>
</dependency>
...
<plugins>
    <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
            <source>1.8</source>
            <target>1.8</target>
        </configuration>
    </plugin>
</plugins>
```

这些依赖项的最新版本是 [datanucleus-jdo-query](https://web.archive.org/web/20220707143819/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.datanucleus%22%20AND%20a%3A%22datanucleus-jdo-query%22) 和 [maven-compiler-plugin。](https://web.archive.org/web/20220707143819/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.apache.maven.plugins%22%20AND%20a%3A%22maven-compiler-plugin%22)

### 5.2。启用注释处理

对于 Eclipse，我们可以按照下面的步骤来启用带注释的处理:

1.  转到`Java Compiler`并确保编译器兼容级别为 1.8 或更高
2.  转到`Java Compiler → Annotation Processing`，启用项目特定设置，并启用注释处理
3.  转到`Java Compiler → Annotation Processing → Factory Path`，启用项目特定设置，然后将以下 jar 添加到列表:`javax.jdo.jar,` `datanucleus-jdo-query.jar`

上述准备意味着每当我们编译持久类时，datanucleus-jdo-query.jar 中的注释处理器将为每个由`@PersistenceCapable.`注释的类生成一个查询类

在我们的例子中，处理器生成一个`QProductItem`类。生成的类几乎与持久类同名，尽管前缀是 q。

### 5.3。创建 JDOQL 类型化查询:

```java
JDOQLTypedQuery<ProductItem> tq = pm.newJDOQLTypedQuery(ProductItem.class);
QProductItem cand = QProductItem.candidate();
tq = tq.filter(cand.price.lt(10).and(cand.name.startsWith("pro")));
List<ProductItem> results = tq.executeList();
```

我们可以利用查询类来访问候选字段，并使用其可用的 Java 方法。

## 6。SQL

JDO 支持 SQL 语言，以防我们使用 RDBMS。

让我们创建 SQL 查询:

```java
Query query = pm.newQuery("javax.jdo.query.SQL","select * from "
  + "product_item where price < ? and status = ?");
query.setClass(ProductItem.class);
query.setParameters(10,"InStock");
List<ProductItem> results = query.executeList();
```

当我们执行查询时，我们使用查询的`setClass()` 来检索`ProductItem`对象。否则，它检索一个`Object`类型。

## 7。JPQL

JDO DataNucleus 提供 JPQL 语言。

让我们使用 JPQL 创建一个查询:

```java
Query query = pm.newQuery("JPQL","select i from "
  + "com.baeldung.jdo.query.ProductItem i where i.price < 10"
  + " and i.status = 'InStock'");
List<ProductItem> results = (List<ProductItem>) query.execute();
```

这里的实体名是`com.baeldung.jdo.query.ProductItem.` 我们不能只使用类名。这是因为 JDO 没有像 JPA `.`那样定义实体名称的元数据，我们定义了一个`ProductItem` `p`，之后，我们可以使用`p`作为别名来引用`ProductItem.`

关于 JPQL 语法的更多细节，请查看这个[链接](https://web.archive.org/web/20220707143819/http://www.datanucleus.org/products/accessplatform_5_1/jpa/getting_started.html)。

## 8。结论

在本文中，我们展示了 JDO 支持的不同查询语言。我们展示了如何保存命名查询以便重用，解释了 JDOQL 的概念，并展示了如何将 SQL 和 JPQL 用于 JDO。

文章中的代码示例可以在 GitHub 上找到[。](https://web.archive.org/web/20220707143819/https://github.com/eugenp/tutorials/tree/master/libraries-data-db)