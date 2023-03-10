# Hibernate Spatial 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-spatial>

## 1。简介

在本文中，我们将看看 Hibernate 的空间扩展， [hibernate-spatial](https://web.archive.org/web/20221126233811/http://www.hibernatespatial.org/) 。

从版本 5 开始， **Hibernate Spatial 提供了一个处理地理数据的标准接口**。

## 2.Hibernate Spatial 的背景

地理数据包括类似于`Point, Line, Polygon`的实体表示。这种数据类型不是 JDBC 规范的一部分，因此 [JTS (JTS 拓扑套件)](https://web.archive.org/web/20221126233811/http://www.tsusiatsoftware.net/jts/main.html)已经成为表示空间数据类型的标准。

除了 JTS，Hibernate spatial 还支持[geo latte-geom](https://web.archive.org/web/20221126233811/https://github.com/GeoLatte/geolatte-geom)——这是一个最新的库，具有一些 JTS 没有的功能。

这两个库都已经包含在 hibernate-spatial 项目中。用一个库代替另一个库只是一个我们从哪个 jar 导入数据类型的问题。

尽管 Hibernate spatial 支持不同的数据库，如 Oracle、MySQL、PostgreSQL/PostGIS 和其他一些数据库，但对数据库特定函数的支持并不统一。

最好参考最新的 hibernate 文档，查看 Hibernate 为给定数据库提供支持的函数列表。

在本文中，我们将使用内存中的[Maria db4j](https://web.archive.org/web/20221126233811/https://github.com/vorburger/MariaDB4j)——它保持了 MySQL 的全部功能。

Mariadb4j 和 MySql 的配置是相似的，甚至 mysql-connector 库也适用于这两种数据库。

## 3 **。Maven 依赖关系**

让我们来看看建立一个简单的 hibernate-spatial 项目所需的 Maven 依赖性:

```java
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.2.12.Final</version>
</dependency>
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-spatial</artifactId>
    <version>5.2.12.Final</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>6.0.6</version>
</dependency>
<dependency>
    <groupId>ch.vorburger.mariaDB4j</groupId>
    <artifactId>mariaDB4j</artifactId>
    <version>2.2.3</version>
</dependency> 
```

`hibernate-spatial`依赖项将为空间数据类型提供支持。最新版本的 [hibernate-core](https://web.archive.org/web/20221126233811/https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.hibernate%22%20AND%20a%3A%22hibernate-core%22) 、 [hibernate-spatial](https://web.archive.org/web/20221126233811/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.hibernate%22%20AND%20a%3A%22hibernate-spatial%22) 、 [mysql-connector-java](https://web.archive.org/web/20221126233811/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22mysql%22%20AND%20a%3A%22mysql-connector-java%22) 和 [mariaDB4j](https://web.archive.org/web/20221126233811/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22ch.vorburger.mariaDB4j%22%20AND%20a%3A%22mariaDB4j%22) 可以从 Maven Central 获得。

## 4.配置休眠空间

第一步是在`resources`目录中创建一个`hibernate.properties`:

```java
hibernate.dialect=org.hibernate.spatial.dialect.mysql.MySQL56SpatialDialect
// ...
```

**冬眠空间唯一特有的是`MySQL56SpatialDialect`方言**。这种方言扩展了`MySQL55Dialect`方言，并提供了与空间数据类型相关的附加功能。

特定于加载属性文件、创建`SessionFactory`和实例化 Mariadb4j 实例的代码与标准 hibernate 项目中的相同。

## 5 **。了解`Geometry`类型**

`Geometry`是 JTS 所有空间类型的基础类型。这意味着其他类型如`Point`、`Polygon`等从`Geometry`延伸而来。java 中的`Geometry`类型也对应于 MySql 中的`GEOMETRY`类型。

通过解析类型的一个`String`表示，我们得到了一个`Geometry`的实例。JTS 提供的实用程序类`WKTReader`可用于将任何[知名文本](https://web.archive.org/web/20221126233811/https://en.wikipedia.org/wiki/Well-known_text)表示转换为`Geometry`类型；

```java
public Geometry wktToGeometry(String wellKnownText) 
  throws ParseException {

    return new WKTReader().read(wellKnownText);
}
```

现在，让我们来看看这个方法的实际应用:

```java
@Test
public void shouldConvertWktToGeometry() {
    Geometry geometry = wktToGeometry("POINT (2 5)");

    assertEquals("Point", geometry.getGeometryType());
    assertTrue(geometry instanceof Point);
}
```

正如我们看到的，即使方法的返回类型是`read()`方法是`Geometry`，实际的实例是一个`Point`的实例。

## 6.将点存储在 DB 中

既然我们对什么是`Geometry`类型以及如何从`String`中获得`Point`有了很好的了解，那么让我们来看看`PointEntity`:

```java
@Entity
public class PointEntity {

    @Id
    @GeneratedValue
    private Long id;

    private Point point;

    // standard getters and setters
}
```

注意，实体`PointEntity`包含一个空间类型`Point`。如前所述，`Point`由两个坐标表示:

```java
public void insertPoint(String point) {
    PointEntity entity = new PointEntity();
    entity.setPoint((Point) wktToGeometry(point));
    session.persist(entity);
}
```

方法`insertPoint()`接受一个众所周知的`Point`的文本(WKT)表示，将其转换成一个`Point`实例，并保存在 DB 中。

提醒一下，`session`不是 hibernate-spatial 特有的，它是以类似于另一个 hibernate 项目的方式创建的。

我们可以注意到，一旦我们创建了一个`Point`的实例，存储`PointEntity`的过程类似于任何常规实体。

让我们来看一些测试:

```java
@Test
public void shouldInsertAndSelectPoints() {
    PointEntity entity = new PointEntity();
    entity.setPoint((Point) wktToGeometry("POINT (1 1)"));

    session.persist(entity);
    PointEntity fromDb = session
      .find(PointEntity.class, entity.getId());

    assertEquals("POINT (1 1)", fromDb.getPoint().toString());
    assertTrue(geometry instanceof Point);
}
```

在一个`Point`上调用`toString()`会返回一个`Point`的 WKT 表示。这是因为`Geometry`类覆盖了`toString()`方法，并在内部使用了`WKTWriter,`一个我们之前看到的`WKTReader`的补充类。

一旦我们运行这个测试，hibernate 将为我们创建`PointEntity`表。

让我们看看那张表:

```java
desc PointEntity;
Field    Type          Null    Key
id       bigint(20)    NO      PRI
point    geometry      YES
```

果然，`Field` `Point`的`Type`是`GEOMETRY`。因此，在使用我们的 SQL 编辑器(如 MySql workbench)获取数据时，我们需要将这种几何类型转换为人类可读的文本:

```java
select id, astext(point) from PointEntity;

id      astext(point)
1       POINT(2 4)
```

然而，当我们在`Geometry`或它的任何子类上调用`toString()`方法时，hibernate 已经返回了 WKT 表示，我们不需要为这个转换而烦恼。

## 7。使用空间功能

### 7.1。`ST_WITHIN()`例子

我们现在来看一下处理空间数据类型的数据库函数的用法。

MySQL 中的一个这样的函数是`ST_WITHIN()`,它告诉我们一个`Geometry`是否在另一个内。一个很好的例子是找出给定半径内的所有点。

让我们先来看看如何创建一个圆:

```java
public Geometry createCircle(double x, double y, double radius) {
    GeometricShapeFactory shapeFactory = new GeometricShapeFactory();
    shapeFactory.setNumPoints(32);
    shapeFactory.setCentre(new Coordinate(x, y));
    shapeFactory.setSize(radius * 2);
    return shapeFactory.createCircle();
}
```

圆由`setNumPoints()`方法指定的一组有限的点表示。在调用`setSize()`方法之前，`radius`被加倍，因为我们需要在两个方向上围绕中心画一个圆。

现在让我们向前看，看看如何获取给定半径内的点:

```java
@Test
public void shouldSelectAllPointsWithinRadius() throws ParseException {
    insertPoint("POINT (1 1)");
    insertPoint("POINT (1 2)");
    insertPoint("POINT (3 4)");
    insertPoint("POINT (5 6)");

    Query query = session.createQuery("select p from PointEntity p where 
      within(p.point, :circle) = true", PointEntity.class);
    query.setParameter("circle", createCircle(0.0, 0.0, 5));

    assertThat(query.getResultList().stream()
      .map(p -> ((PointEntity) p).getPoint().toString()))
      .containsOnly("POINT (1 1)", "POINT (1 2)");
    }
```

Hibernate 将其`within()`函数映射到 MySql 的`ST_WITHIN()`函数。

这里一个有趣的观察是点(3，4)正好落在圆上。但是，查询没有返回这一点。这是因为只有当给定的`Geometry`完全在另一个`Geometry` 内时，`within()`函数才会返回 true。

### 7.2。`ST_TOUCHES()`例子

这里，我们将给出一个例子，在数据库中插入一组`Polygon`并选择与给定的`Polygon`相邻的`Polygon`。让我们快速浏览一下`PolygonEntity`类:

```java
@Entity
public class PolygonEntity {

    @Id
    @GeneratedValue
    private Long id;

    private Polygon polygon;

    // standard getters and setters
}
```

这里与之前的`PointEntity`唯一不同的是我们使用了类型`Polygon`而不是`Point`。

现在让我们开始测试:

```java
@Test
public void shouldSelectAdjacentPolygons() throws ParseException {
    insertPolygon("POLYGON ((0 0, 0 5, 5 5, 5 0, 0 0))");
    insertPolygon("POLYGON ((3 0, 3 5, 8 5, 8 0, 3 0))");
    insertPolygon("POLYGON ((2 2, 3 1, 2 5, 4 3, 3 3, 2 2))");

    Query query = session.createQuery("select p from PolygonEntity p 
      where touches(p.polygon, :polygon) = true", PolygonEntity.class);
    query.setParameter("polygon", wktToGeometry("POLYGON ((5 5, 5 10, 10 10, 10 5, 5 5))"));
    assertThat(query.getResultList().stream()
      .map(p -> ((PolygonEntity) p).getPolygon().toString())).containsOnly(
      "POLYGON ((0 0, 0 5, 5 5, 5 0, 0 0))", "POLYGON ((3 0, 3 5, 8 5, 8 0, 3 0))");
}
```

`insertPolygon()`方法类似于我们之前看到的`insertPoint()`方法。源代码包含此方法的完整实现。

我们使用`touches()`函数来寻找与给定的`Polygon`相邻的`Polygon`。显然，结果中没有返回第三个`Polygon`，因为没有边接触给定的`Polygon`。

## 8.结论

在本文中，我们已经看到 hibernate-spatial 使得处理空间数据类型变得更加简单，因为它处理了底层细节。

即使本文使用了 Mariadb4j，我们也可以用 MySql 代替它，无需修改任何配置。

和往常一样，本文的完整源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221126233811/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-enterprise)