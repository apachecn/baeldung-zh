# 地理工具简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/geo-tools>

## 1。概述

在本文中，我们将介绍 **[地理工具](https://web.archive.org/web/20221107031402/http://geotools.org/)开源 Java 库的基础知识——用于处理地理空间数据**。该库提供了实现地理信息系统(GIS)的兼容方法，并实现和支持许多开放地理空间联盟(OGC)标准。

随着 OGC 开发新标准，它们由地理工具实现，这使得地理空间工作非常方便。

## 2。依赖性

我们需要将地理工具依赖项添加到我们的`pom.xml`文件中。由于这些依赖项不在 Maven Central 上托管，我们还需要声明它们的存储库，以便 Maven 可以下载它们:

```java
<repositories>
    <repository>
        <id>osgeo</id>
        <name>Open Source Geospatial Foundation Repository</name>
        <url>http://download.osgeo.org/webdav/geotools/</url>
    </repository>
    <repository>
        <id>opengeo</id>
        <name>OpenGeo Maven Repository</name>
        <url>http://repo.opengeo.org</url>
    </repository>
</repositories>
```

之后，我们可以添加依赖项:

```java
<dependency>
    <groupId>org.geotools</groupId>
    <artifactId>gt-shapefile</artifactId>
    <version>15.2</version>
</dependency>
<dependency>
    <groupId>org.geotools</groupId>
    <artifactId>gt-epsg-hsql</artifactId>
    <version>15.2</version>
</dependency>
```

## 3。GIS 和 shape file

要实际使用 GeoTools 库，我们需要了解一些关于地理信息系统和`shapefiles`的知识。

### 3.1。GIS

如果我们想处理地理数据，我们需要一个地理信息系统(GIS)。该系统可用于呈现、捕捉、存储、操作、分析或管理地理数据。

部分地理数据是空间数据——它引用了地球上的具体位置。空间数据通常伴随着属性数据。属性数据可以是关于每个空间要素的任何附加信息。

地理数据的一个例子是城市。城市的实际位置是空间数据。城市名称和人口等附加数据将构成属性数据。

### 3.2。形状文件

不同的格式可用于处理地理空间数据。栅格和矢量是两种主要的数据类型。

在这篇文章中，我们将看到如何处理向量数据类型 **e** 。这种数据类型可以表示为点、线或多边形。

为了在文件中存储矢量数据，我们将使用一个`shapefile`。使用地理空间矢量数据类型时，会使用这种文件格式。此外，它与各种 GIS 软件兼容。

我们可以使用地理工具为`shapefiles`添加城市、学校和地标等特征。

## 4。创建特征

文档规定一个特征是可以在地图上绘制的任何东西，比如一个城市或一些地标。而且，正如我们提到的，一旦创建，特征就可以保存到名为`shapefiles`的文件中。

### 4.1。保存地理空间数据

在创建一个特征之前，我们需要知道它的地理空间数据或者它在地球上的位置的经纬度坐标。至于属性数据，我们需要知道我们想要创建的特征的名称。

这些信息可以在网上找到。一些网站，如 simplemaps.com、maxmind.com 和 T2，提供免费的地理空间数据数据库。

当我们知道一个城市的经度和纬度时，我们可以很容易地将它们存储在某个对象中。我们可以使用一个`Map`对象来保存城市名称及其坐标列表。

让我们创建一个助手方法来简化我们的`Map`对象中的数据存储:

```java
private static void addToLocationMap(
  String name,
  double lat,
  double lng,
  Map<String, List<Double>> locations) {
    List<Double> coordinates = new ArrayList<>();

    coordinates.add(lat);
    coordinates.add(lng);
    locations.put(name, coordinates);
}
```

现在让我们填写我们的`Map`对象:

```java
Map<String, List<Double>> locations = new HashMap<>();

addToLocationMap("Bangkok", 13.752222, 100.493889, locations);
addToLocationMap("New York", 53.083333, -0.15, locations);
addToLocationMap("Cape Town", -33.925278, 18.423889, locations);
addToLocationMap("Sydney", -33.859972, 151.211111, locations);
addToLocationMap("Ottawa", 45.420833, -75.69, locations);
addToLocationMap("Cairo", 30.07708, 31.285909, locations);
```

如果我们下载一些包含这些数据的 CSV 数据库，我们可以很容易地创建一个读取器来检索数据，而不是像这里这样将数据保存在一个对象中。

### 4.2。定义特征类型

现在我们有了一张城市地图。为了能够用这些数据创建特征，我们需要首先定义它们的类型。 GeoTools 提供了两种定义要素类型的方法。

一种方法是使用`DataUtilites`类的`createType`方法:

```java
SimpleFeatureType TYPE = DataUtilities.createType(
  "Location", "location:Point:srid=4326," + "name:String");
```

另一种方法是**使用`SimpleFeatureTypeBuilder`，这提供了更大的灵活性**。例如，我们可以为类型设置坐标参考系统，并且可以为名称字段设置最大长度:

```java
SimpleFeatureTypeBuilder builder = new SimpleFeatureTypeBuilder();
builder.setName("Location");
builder.setCRS(DefaultGeographicCRS.WGS84);

builder
  .add("Location", Point.class);
  .length(15)
  .add("Name", String.class);

SimpleFeatureType CITY = builder.buildFeatureType();
```

**两种类型存储相同的信息。**城市的位置存储为一个`Point`，城市的名称存储为一个`String`。

你可能注意到了类型变量`TYPE`和`CITY`都是用大写字母命名的，就像常量一样。**类型变量应该被视为`final`变量，并且在它们被创建**之后不应该被改变，所以这种命名方式可以用来表明这一点。

### 4.3。特征创建和特征集合

一旦定义了要素类型，并且有了包含创建要素所需数据的对象，我们就可以开始使用构建器创建要素了。

让我们实例化一个`SimpleFeatureBuilder`来提供我们的特征类型:

```java
SimpleFeatureBuilder featureBuilder = new SimpleFeatureBuilder(CITY);
```

我们还需要一个集合来存储所有创建的特征对象:

```java
DefaultFeatureCollection collection = new DefaultFeatureCollection();
```

因为我们在我们的特征类型中声明了为位置保存一个`Point`，我们将需要**根据他们的坐标**为我们的城市创建点。我们可以用`GeoTools's JTSGeometryFactoryFinder`来做这件事:

```java
GeometryFactory geometryFactory
  = JTSFactoryFinder.getGeometryFactory(null);
```

注意**我们也可以使用其他的`Geometry`类，比如`Line`和`Polygon`** 。

我们可以创建一个`function`来帮助我们将功能放入系列中:

```java
private static Function<Map.Entry<String, List<Double>>, SimpleFeature>
  toFeature(SimpleFeatureType CITY, GeometryFactory geometryFactory) {
    return location -> {
        Point point = geometryFactory.createPoint(
           new Coordinate(location.getValue()
             .get(0), location.getValue().get(1)));

        SimpleFeatureBuilder featureBuilder
          = new SimpleFeatureBuilder(CITY);
        featureBuilder.add(point);
        featureBuilder.add(location.getKey());
        return featureBuilder.buildFeature(null);
    };
}
```

一旦我们有了构建器和集合，通过使用先前创建的`function`，我们可以**创建特征并将它们存储在我们的集合**中:

```java
locations.entrySet().stream()
  .map(toFeature(CITY, geometryFactory))
  .forEach(collection::add);
```

该集合现在包含了基于保存地理空间数据的`Map`对象创建的所有特性。

## 5。创建数据存储库

`GeoTools`包含一个用于表示地理空间数据源的`DataStore API`。这个源可以是文件、数据库或一些返回数据的服务。我们可以使用一个`DataStoreFactory`来创建我们的`DataStore`，它将包含我们的特性。

让我们设置将包含特性的文件:

```java
File shapeFile = new File(
  new File(".").getAbsolutePath() + "shapefile.shp");
```

现在，让我们设置参数，我们将使用这些参数来告诉`DataStoreFactory`使用哪个文件，并指示我们需要在创建`DataStore`时存储一个空间索引:

```java
Map<String, Serializable> params = new HashMap<>();
params.put("url", shapeFile.toURI().toURL());
params.put("create spatial index", Boolean.TRUE);
```

让我们使用刚刚创建的参数来创建`DataStoreFactory`，并使用该工厂来创建`DataStore`:

```java
ShapefileDataStoreFactory dataStoreFactory
  = new ShapefileDataStoreFactory();

ShapefileDataStore dataStore 
  = (ShapefileDataStore) dataStoreFactory.createNewDataStore(params);
dataStore.createSchema(CITY);
```

## 6。写入形状文件

我们需要做的最后一步是将数据写入一个`shapefile`。为了安全起见，我们将**使用`Transaction` 接口**，它是`GeoTools` API 的一部分。

这个接口让我们有可能**轻松`commit`我们对文件**的修改。它还提供了一种方法，如果在写入文件时出现问题，则**执行不成功更改的`rollback`:**

```java
Transaction transaction = new DefaultTransaction("create");

String typeName = dataStore.getTypeNames()[0];
SimpleFeatureSource featureSource
  = dataStore.getFeatureSource(typeName);

if (featureSource instanceof SimpleFeatureStore) {
    SimpleFeatureStore featureStore
      = (SimpleFeatureStore) featureSource;

    featureStore.setTransaction(transaction);
    try {
        featureStore.addFeatures(collection);
        transaction.commit();

    } catch (Exception problem) {
        transaction.rollback();
    } finally {
        transaction.close();
    }
}
```

`SimpleFeatureSource`用于读取特征，`SimpleFeatureStore`用于读/写访问。在`GeoTools`文档中规定，使用`instanceof`方法检查我们是否可以写入文件是正确的做法。

稍后可以用任何支持`shapefile`的 GIS 查看器打开这个`shapefile`。

## 7。结论

在本文中，我们看到了如何利用`GeoTools`库来做一些非常有趣的地理空间工作。

尽管这个例子很简单，但它可以被扩展并用于创建各种用途的 rich `shapefiles`。

我们应该记住,`GeoTools`是一个充满活力的库，本文只是对该库的一个基本介绍。此外，`GeoTools`不仅限于创建矢量数据类型，它还可以用于创建或处理栅格数据类型。

您可以在我们的 [GitHub 项目](https://web.archive.org/web/20221107031402/https://github.com/eugenp/tutorials/tree/master/geotools)中找到本文中使用的完整示例代码。这是一个 Maven 项目，所以您应该能够导入它并按原样运行它。