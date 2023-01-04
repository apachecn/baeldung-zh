# Java 数据对象指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jdo>

## 1。概述

Java 数据对象是一个 API，设计用于将面向对象的数据保存到任何数据库中，并使用 Java 语法提供用户友好的查询语言。

在这篇文章中，我们将看到如何使用 JDO API 在数据库中持久化我们的对象。

## 2。Maven 依赖性和设置

我们将使用最新的 DataNucleus JDO API，它提供了对 JDO 3.2 API 的全面支持。

让我们将下面的依赖项添加到我们的`pom.xml`文件中:

```
<dependency>
    <groupId>org.datanucleus</groupId>
    <artifactId>javax.jdo</artifactId>
    <version>3.2.0-m6</version>
</dependency>
<dependency>
    <groupId>org.datanucleus</groupId>
    <artifactId>datanucleus-core</artifactId>
    <version>5.1.0-m2</version>
</dependency>
<dependency>
    <groupId>org.datanucleus</groupId>
    <artifactId>datanucleus-api-jdo</artifactId>
    <version>5.1.0-m2</version>
</dependency>
<dependency>
    <groupId>org.datanucleus</groupId>
    <artifactId>datanucleus-rdbms</artifactId>
    <version>5.1.0-m2</version>
</dependency>
<dependency>
    <groupId>org.datanucleus</groupId>
    <artifactId>datanucleus-xml</artifactId>
    <version>5.0.0-release</version>
</dependency> 
```

这里可以找到最新版本的依赖项: [javax.jdo](https://web.archive.org/web/20220625175817/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.datanucleus%22%20AND%20a%3A%22javax.jdo%22) 、 [datanucleus-core](https://web.archive.org/web/20220625175817/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.datanucleus%22%20AND%20a%3A%22datanucleus-core%22) 、 [datanucleus-api-jdo](https://web.archive.org/web/20220625175817/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.datanucleus%22%20AND%20a%3A%22datanucleus-api-jdo%22) 、 [datanucleus-rdbms](https://web.archive.org/web/20220625175817/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.datanucleus%22%20AND%20a%3A%22datanucleus-rdbms%22) 和 [datanucleus-xml](https://web.archive.org/web/20220625175817/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.datanucleus%22%20AND%20a%3A%22datanucleus-xml%22) 。

## 3。型号

我们将把数据保存在数据库中，在此之前，我们需要创建一个类，JDO 将使用它来存储我们的数据。

为此，我们需要创建一个具有一些属性的类，并用`@PersistentCapable:`对其进行注释

```
@PersistenceCapable
public class Product {

    @PrimaryKey
    @Persistent(valueStrategy = IdGeneratorStrategy.INCREMENT)
    long id;

    String name;
    Double price = 0.0;

    // standard constructors, getters, setters
}
```

我们还注释了我们的主键和选择的策略。

一旦我们创建了对象，我们需要运行增强器来生成 JDO 所需的字节码。使用 Maven，我们可以运行这个命令:

```
mvn datanucleus:enhance
```

这一步是强制性的。否则，我们会得到编译时错误，即该类没有得到增强。

当然，在 Maven 构建期间可以自动完成这项工作:

```
<plugin>
    <groupId>org.datanucleus</groupId>
    <artifactId>datanucleus-maven-plugin</artifactId>
    <version>5.0.2</version>
    <configuration>
        <api>JDO</api>
        <props>${basedir}/datanucleus.properties</props>
        <log4jConfiguration>${basedir}/log4j.properties</log4jConfiguration>
        <verbose>true</verbose>
    </configuration>
    <executions>
        <execution>
            <phase>process-classes</phase>
            <goals>
                <goal>enhance</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

插件的最新版本可以在这里找到: [datanucleus-maven-plugin](https://web.archive.org/web/20220625175817/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.datanucleus%22%20AND%20a%3A%22datanucleus-maven-plugin%22)

## 4。持久对象

我们使用 JDO 工厂访问数据库，该工厂为我们提供了负责执行事务的事务管理器:

```
PersistenceManagerFactory pmf = new JDOPersistenceManagerFactory(pumd, null);
PersistenceManager pm = pmf.getPersistenceManager(); 
```

事务用于在出现错误时允许回滚:

```
Transaction tx = pm.currentTransaction();
```

我们在一个`try/catch` 区块内进行交易:

```
Product product = new Product("Tablet", 80.0);
pm.makePersistent(product);
```

在我们的`finally`块中，我们定义了在出现故障的情况下要完成的操作。

如果由于任何原因，事务无法完成，我们将进行回滚，并且我们还会关闭与数据库的连接，使用`pm.close()`:

```
finally {
    if (tx.isActive()) {
        tx.rollback();
    }
    pm.close();
}
```

为了将我们的程序连接到数据库，我们需要在运行时创建一个`persistence-unit` 来指定持久类、数据库类型和连接参数:

```
PersistenceUnitMetaData pumd = new PersistenceUnitMetaData(
  "dynamic-unit", "RESOURCE_LOCAL", null);
pumd.addClassName("com.baeldung.jdo.Product");
pumd.setExcludeUnlistedClasses();
pumd.addProperty("javax.jdo.option.ConnectionDriverName", "org.h2.Driver");
pumd
  .addProperty("javax.jdo.option.ConnectionURL", "jdbc:h2:mem:mypersistence");
pumd.addProperty("javax.jdo.option.ConnectionUserName", "sa");
pumd.addProperty("javax.jdo.option.ConnectionPassword", "");
pumd.addProperty("datanucleus.autoCreateSchema", "true");
```

## 5。阅读对象

为了从事务块中的数据库读取数据，我们创建了一个查询。然后，我们将这些项目存储到 Java `List`集合中，该集合将保存来自持久性存储的信息的内存副本。

持久性管理器为我们提供了对查询界面的访问，该界面允许我们与数据库进行交互:

```
Query q = pm.newQuery(
  "SELECT FROM " + Product.class.getName() + " WHERE price < 1");
List<Product> products = (List<Product>) q.execute();
Iterator<Product> iter = products.iterator();
while (iter.hasNext()) {
    Product p = iter.next();
    // show the product information
} 
```

## 6。更新对象

要更新数据库中的对象，我们需要使用查询找到想要更新的对象，然后更新查询结果并提交事务:

```
Query query = pm.newQuery(Product.class, "name == \"Phone\"");
Collection result = (Collection) query.execute();
Product product = (Product) result.iterator().next();
product.setName("Android Phone"); 
```

## 7。删除对象

与更新过程类似，我们首先搜索对象，然后使用持久性管理器删除它。在这些情况下，JDO 更新持久存储:

```
Query query = pm.newQuery(Product.class, "name == \"Android Phone\"");
Collection result = (Collection) query.execute();
Product product = (Product) result.iterator().next();
pm.deletePersistent(product); 
```

## 8。XML 数据存储库

使用 XML 插件，我们可以使用 XML 文件来保存数据。

我们指定我们的`ConnectionURL`,表明这是一个 XML 文件，并指定文件名:

```
pumdXML.addProperty("javax.jdo.option.ConnectionURL", "xml:file:myPersistence.xml");
```

XML 数据存储不支持自动递增属性，因此我们需要创建另一个类:

```
@PersistenceCapable()
public class ProductXML {

    @XmlAttribute
    private long productNumber = 0;
    @PrimaryKey
    private String name = null;
    private Double price = 0.0;

    // standard getters and setters
```

`@XmlAttribute`注释表示这将作为元素的属性出现在 XML 文件中。

让我们创造并坚持我们的产品:

```
ProductXML productXML = new ProductXML(0,"Tablet", 80.0);
pm.makePersistent(productXML);
```

我们得到存储在 XML 文件中的产品:

```
<productXML productNumber="0">
    <name>Tablet</name>
    <price>80.0</price>
</productXML>
```

### 8.1。从 XML 数据存储中恢复对象

我们可以使用查询从 XML 文件中恢复我们的对象:

```
Query q = pm.newQuery("SELECT FROM " + ProductXML.class.getName());
List<ProductXML> products = (List<ProductXML>) q.execute();
```

然后我们用迭代器和每个对象进行交互。

## 9。JDO 询问

JDOQL 是一种基于对象的查询语言，旨在使用 Java 对象执行查询。

### 9.1。声明性 JDOQL

使用声明性查询，我们声明参数并使用 Java 设置它们，这确保了类型安全:

```
Query qDJDOQL = pm.newQuery(Product.class);
qDJDOQL.setFilter("name == 'Tablet' && price == price_value");
qDJDOQL.declareParameters("double price_value");
List<Product> resultsqDJDOQL = qDJDOQL.setParameters(80.0).executeList();
```

### 9.2。SQL

JDO 提供了一种执行标准 SQL 查询的机制:

```
Query query = pm.newQuery("javax.jdo.query.SQL", "SELECT * FROM PRODUCT");
query.setClass(Product.class);
List<Product> results = query.executeList();
```

我们使用`javax.jdo.query.SQL` 作为查询对象的一个参数，第二个参数是 SQL 本身。

### 9.3。JPQL

JDO 也提供了执行 JPA 查询的机制。我们可以使用 JPA 查询语言的完整语法:

```
Query q = pm.newQuery("JPQL",
  "SELECT p FROM " + Product.class.getName() + " p WHERE p.name = 'Laptop'");
List results = (List) q.execute();
```

## 10。总结

在本教程中，我们将:

*   创建了一个使用 JDO 的简单 CRUD 应用程序
*   以 XML 格式保存和检索我们的数据
*   研究了常见的查询机制

和往常一样，你可以在 Github 上找到文章[中的代码。](https://web.archive.org/web/20220625175817/https://github.com/eugenp/tutorials/tree/master/libraries-data-db)