# Spring 数据库中的多桶和空间视图查询

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/spring-data-couchbase-buckets-and-spatial-view-queries>

## 1。简介

在关于 Spring Data Couchbase 的第三篇教程中，我们演示了支持跨多个存储桶的 Couchbase 数据模型所需的配置，并介绍了使用空间视图来查询多维数据。

## 2。数据模型

除了来自我们第一个教程的[的`Person`实体和来自我们第二个教程](/web/20220628144739/https://www.baeldung.com/spring-data-couchbase)的[的`Student`实体，我们为这个教程定义了一个`Campus`实体:](/web/20220628144739/https://www.baeldung.com/entity-validation-locking-and-query-consistency-in-spring-data-couchbase)

```java
@Document
public class Campus {
    @Id
    private String id;

    @Field
    @NotNull
    private String name;

    @Field
    @NotNull
    private Point location;

    // standard getters and setters
}
```

## 3。多个 Couchbase 存储桶的 Java 配置

为了在您的项目中使用多个 bucket，您将需要使用 Spring Data Couchbase 模块的 2.0.0 或更高版本，并且您将需要使用基于 Java 的配置，因为基于 XML 的配置只支持单个 bucket 场景。

下面是我们包含在 Maven `pom.xml`文件中的依赖关系:

```java
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-couchbase</artifactId>
    <version>2.1.1.RELEASE</version>
</dependency>
```

### 3.1。定义桶 Bean

在我们的[Spring Data couch base](/web/20220628144739/https://www.baeldung.com/spring-data-couchbase)简介教程中，我们指定`“baeldung”`作为默认 Couchbase bucket 的名称，用于 Spring 数据。

我们将在`“baeldung2”`桶中存储`Campus`个实体。

为了使用第二个 bucket，我们首先必须在 Couchbase 配置类中为`Bucket`本身定义一个`@Bean`:

```java
@Bean
public Bucket campusBucket() throws Exception {
    return couchbaseCluster().openBucket("baeldung2", "");
}
```

### 3.2。配置模板 Bean

接下来，我们为这个桶使用的`CouchbaseTemplate`定义一个`@Bean`:

```java
@Bean
public CouchbaseTemplate campusTemplate() throws Exception {
    CouchbaseTemplate template = new CouchbaseTemplate(
      couchbaseClusterInfo(), campusBucket(),
      mappingCouchbaseConverter(), translationService());
    template.setDefaultConsistency(getDefaultConsistency());
    return template;
}
```

### 3.3。映射存储库

最后，我们定义了 Couchbase 存储库操作的定制映射，以便`Campus`实体类将使用新的模板和存储桶，而其他实体类将继续使用默认的模板和存储桶:

```java
@Override
public void configureRepositoryOperationsMapping(
  RepositoryOperationsMapping baseMapping) {
    try {
        baseMapping.mapEntity(Campus.class, campusTemplate());
    } catch (Exception e) {
        //custom Exception handling
    }
}
```

## 4。查询空间或多维数据

Couchbase 提供了对二维数据(如地理数据)运行边界框查询的本机支持，使用一种称为`Spatial view`的特殊类型的视图。

边界框查询是一种范围查询，它使用框的最西南的`[x,y]`点作为其`startRange`参数，使用最西北的`[x,y]`点作为其`endRange`参数。

Spring Data 使用一种寻求消除假阳性匹配的算法，将 Couchbase 的原生边界框查询功能扩展到涉及圆形和多边形的查询，并且它还为涉及两个以上维度的查询提供支持。

Spring Data 通过一组关键字简化了多维查询的创建，这些关键字可用于定义 Couchbase 存储库中的派生查询。

### 4.1。支持的数据类型

Spring Data Couchbase 存储库查询支持来自`org.springframework.data.geo`包的数据类型，包括`Point, Box, Circle, Polygon,` 和 `Distance`。

### 4.2。派生查询关键字

除了标准的 Spring 数据存储库关键字，Couchbase 存储库在涉及两个维度的派生查询中支持以下关键字:

*   `Within`、`InWithin` (用两个`Point`参数定义一个边界框)
*   `Near`、`IsNear`(以`Point`和`Distance`为参数)

并且以下关键字可以用于涉及两个以上维度的查询:

*   `Between`(用于将单个数值加到`startRange`和`endRange`上)
*   `GreaterThan, GreaterThanEqual, After`(用于向`startRange`添加单个数值)
*   `LessThan, LessThanEqual, Before`(用于向`endRange`添加单个数值)

以下是使用这些关键字的派生查询方法的一些示例:

*   `findByLocationNear`
*   `findByLocationWithin`
*   `findByLocationNearAndPopulationGreaterThan`
*   `findByLocationWithinAndAreaLessThan`
*   `findByLocationNearAndTuitionBetween`

## 5。定义存储库

由空间视图支持的存储库方法必须用`@Dimensional`注释来修饰，该注释指定了设计文档名称、视图名称和用于定义视图关键字的维数(如果没有另外指定，则默认为 2)。

### 5.1。校园仓库界面

在我们的`CampusRepository`接口中，我们声明了两个方法——一个使用传统的 Spring 数据关键字，由 MapReduce 视图支持，另一个使用维度 Spring 数据关键字，由空间视图支持:

```java
public interface CampusRepository extends CrudRepository<Campus, String> {

    @View(designDocument="campus", viewName="byName")
    Set<Campus> findByName(String name);

    @Dimensional(dimensions=2, designDocument="campus_spatial",
      spatialViewName="byLocation")
    Set<Campus> findByLocationNear(Point point, Distance distance);
}
```

### 5.2。空间视图

空间视图被写成 JavaScript 函数，很像 MapReduce 视图。与 MapReduce 视图不同，MapReduce 视图由一个`map`函数和一个`reduce`函数组成，空间视图仅由一个`spatial`函数组成，并且不能与 MapReduce 视图共存于同一个 Couchbase 设计文档中。

对于我们的`Campus`实体，我们将创建一个名为`“campus_spatial”`的设计文档，其中包含一个名为`“byLocation”`的空间视图，其功能如下:

```java
function (doc) {
  if (doc.location &&
      doc._class == "com.baeldung.spring.data.couchbase.model.Campus") {
    emit([doc.location.x, doc.location.y], null);
  }
}
```

如本例所示，当您编写空间视图函数时，`emit`函数调用中使用的键必须是两个或更多值的数组。

### 5.3。MapReduce 视图

为了给我们的存储库提供充分的支持，我们必须创建一个名为`“campus”`的设计文档，其中包含两个 MapReduce 视图:`“all”`和`“byName”`。

以下是`“all”`视图的地图功能:

```java
function (doc, meta) {
  if(doc._class == "com.baeldung.spring.data.couchbase.model.Campus") {    
    emit(meta.id, null);
  }
}
```

这里是`“byName”`视图的地图功能:

```java
function (doc, meta) {
  if(doc._class == "com.baeldung.spring.data.couchbase.model.Campus" &&
     doc.name) {    
    emit(doc.name, null);
  }
}
```

## 6。结论

我们展示了如何配置 Spring Data Couchbase 项目来支持多个存储桶的使用，以及如何使用存储库抽象来编写针对多维数据的空间视图查询。

你可以在 github 项目中查看本教程的完整源代码。

要了解更多关于 Spring Data Couchbase 的信息，请访问官方的 [Spring Data Couchbase](https://web.archive.org/web/20220628144739/https://projects.spring.io/spring-data-couchbase/) 项目网站。