# MongoDB 中的地理空间支持

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/mongodb-geospatial-support>

## 1。概述

在本教程中，我们将探索 MongoDB 中的地理空间支持。

我们将讨论如何存储地理空间数据、地理索引和地理空间搜索。我们还将使用多个地理空间搜索查询，如`near`、`geoWithin`和`geoIntersects`。

## 2。存储地理空间数据

首先，让我们看看如何在 MongoDB 中存储地理空间数据。

**MongoDB 支持多种`GeoJSON`类型存储地理空间数据。**在我们的例子中，我们将主要使用`Point`和`Polygon`类型。

### 2.1。`Point`

这是最基本和最常见的`GeoJSON`类型，**用来表示网格上的一个特定点**。

这里，我们有一个简单的对象，在我们的`places`集合`,`中，字段`location`作为`Point`:

```
{
  "name": "Big Ben",
  "location": {
    "coordinates": [-0.1268194, 51.5007292],
    "type": "Point"
  }
}
```

注意首先是经度值，然后是纬度值。

### 2.2。`Polygon`

`Polygon`是稍微复杂一点的`GeoJSON`类型。

**我们可以使用`Polygon`来定义一个区域的外部边界**以及内部孔洞(如果需要的话)。

让我们看看另一个对象，它的位置被定义为一个`Polygon`:

```
{
  "name": "Hyde Park",
  "location": {
    "coordinates": [
      [
        [-0.159381, 51.513126],
        [-0.189615, 51.509928],
        [-0.187373, 51.502442],
        [-0.153019, 51.503464],
        [-0.159381, 51.513126]
      ]
    ],
    "type": "Polygon"
  }
}
```

在这个例子中，我们定义了一个表示外部边界的点的数组。我们还必须封闭边界，使最后一个点等于第一个点。

注意，我们需要在逆时针方向定义外部边界点，在顺时针方向定义球洞边界。

除了这些类型，还有许多其他类型，如`LineString,` `MultiPoint,` `MultiPolygon,` `MultiLineString,`和`GeometryCollection.`

## 3。地理空间索引

**为了对我们存储的地理空间数据执行搜索查询，我们需要在我们的`location`字段上创建一个地理空间索引。**

我们基本上有两种选择:`2d`和`2dsphere`。

但是首先，让我们定义我们的位置:

```
MongoClient mongoClient = new MongoClient();
MongoDatabase db = mongoClient.getDatabase("myMongoDb");
collection = db.getCollection("places");
```

### 3.1。`2d`地理空间索引

`2d`索引使我们能够执行基于 2d 平面计算的搜索查询。

我们可以在 Java 应用程序的`location`字段上创建一个`2d`索引，如下所示:

```
collection.createIndex(Indexes.geo2d("location"));
```

当然，我们可以在`mongo` shell 中做同样的事情:

```
db.places.createIndex({location:"2d"})
```

### 3.2。`2dsphere`地理空间索引

`2dsphere`索引支持基于球体计算的查询。

类似地，我们可以使用与上面相同的`Indexes`类在 Java 中创建一个`2dsphere`索引:

```
collection.createIndex(Indexes.geo2dsphere("location"));
```

或者在`mongo`外壳中:

```
db.places.createIndex({location:"2dsphere"})
```

## 4。使用地理空间查询进行搜索

现在，最激动人心的部分是，让我们使用地理空间查询根据对象的位置来搜索对象。

### 4.1。近查询

让我们从`near. ` **开始，我们可以使用`near`查询来搜索给定距离内的地点。**

`near` 查询同时适用于 `2d` 和 `2dsphere` 索引`.`

在下一个示例中，我们将搜索距离给定位置不到 1 公里但超过 10 米的地点:

```
@Test
public void givenNearbyLocation_whenSearchNearby_thenFound() {
    Point currentLoc = new Point(new Position(-0.126821, 51.495885));

    FindIterable<Document> result = collection.find(
      Filters.near("location", currentLoc, 1000.0, 10.0));

    assertNotNull(result.first());
    assertEquals("Big Ben", result.first().get("name"));
}
```

以及在`mongo` shell 中的相应查询:

```
db.places.find({
  location: {
    $near: {
      $geometry: {
        type: "Point",
        coordinates: [-0.126821, 51.495885]
      },
      $maxDistance: 1000,
      $minDistance: 10
    }
  }
})
```

请注意，结果是从最近到最远排序的。

同样，如果我们使用一个非常远的位置，我们不会找到任何附近的地方:

```
@Test
public void givenFarLocation_whenSearchNearby_thenNotFound() {
    Point currentLoc = new Point(new Position(-0.5243333, 51.4700223));

    FindIterable<Document> result = collection.find(
      Filters.near("location", currentLoc, 5000.0, 10.0));

    assertNull(result.first());
}
```

我们还有`nearSphere`方法，它的作用与`near,` 完全一样，只是它使用球面几何来计算距离。

### 4.2。在查询内

接下来，我们将探索`geoWithin`查询。

**`geoWithin`查询使我们能够搜索在给定的`Geometry`** 内完全存在的地方，比如圆、盒子或多边形。这也适用于 `2d` 和 `2dsphere` 指数。

在本例中，我们要查找距离给定中心位置 5 km 半径范围内的地点:

```
@Test
public void givenNearbyLocation_whenSearchWithinCircleSphere_thenFound() {
    double distanceInRad = 5.0 / 6371;

    FindIterable<Document> result = collection.find(
      Filters.geoWithinCenterSphere("location", -0.1435083, 51.4990956, distanceInRad));

    assertNotNull(result.first());
    assertEquals("Big Ben", result.first().get("name"));
}
```

注意，我们需要将距离从 km 转换为弧度(只需除以地球半径)。

结果查询:

```
db.places.find({
  location: {
    $geoWithin: {
      $centerSphere: [
        [-0.1435083, 51.4990956],
        0.0007848061528802386
      ]
    }
  }
})
```

接下来，我们将搜索矩形“盒子”中存在的所有地方。我们需要定义盒子的左下位置和右上位置:

```
@Test
public void givenNearbyLocation_whenSearchWithinBox_thenFound() {
    double lowerLeftX = -0.1427638;
    double lowerLeftY = 51.4991288;
    double upperRightX = -0.1256209;
    double upperRightY = 51.5030272;

    FindIterable<Document> result = collection.find(
      Filters.geoWithinBox("location", lowerLeftX, lowerLeftY, upperRightX, upperRightY));

    assertNotNull(result.first());
    assertEquals("Big Ben", result.first().get("name"));
}
```

下面是在`mongo` shell 中的相应查询:

```
db.places.find({
  location: {
    $geoWithin: {
      $box: [
        [-0.1427638, 51.4991288],
        [-0.1256209, 51.5030272]
      ]
    }
  }
})
```

最后，**如果我们想要搜索的区域不是矩形或圆形，我们可以使用多边形来定义一个更具体的区域**:

```
@Test
public void givenNearbyLocation_whenSearchWithinPolygon_thenFound() {
    ArrayList<List<Double>> points = new ArrayList<List<Double>>();
    points.add(Arrays.asList(-0.1439, 51.4952));
    points.add(Arrays.asList(-0.1121, 51.4989));
    points.add(Arrays.asList(-0.13, 51.5163));
    points.add(Arrays.asList(-0.1439, 51.4952));

    FindIterable<Document> result = collection.find(
      Filters.geoWithinPolygon("location", points));

    assertNotNull(result.first());
    assertEquals("Big Ben", result.first().get("name"));
}
```

下面是相应的查询:

```
db.places.find({
  location: {
    $geoWithin: {
      $polygon: [
        [-0.1439, 51.4952],
        [-0.1121, 51.4989],
        [-0.13, 51.5163],
        [-0.1439, 51.4952]
      ]
    }
  }
})
```

我们只定义了一个有外部边界的多边形，但是我们也可以给它添加洞。每个孔将是一个`Point`的`List`;

```
geoWithinPolygon("location", points, hole1, hole2, ...)
```

### 4.3。相交查询

最后，我们来看一下`geoIntersects`查询。

**`geoIntersects`查询查找至少与给定的`Geometry.`** 相交的对象，相比之下，`geoWithin`查找完全存在于给定的`Geometry`内的对象。

该查询仅适用于 `the 2dsphere` 索引。

让我们在实践中看到这一点，以寻找与`Polygon`相交的任何地方为例:

```
@Test
public void givenNearbyLocation_whenSearchUsingIntersect_thenFound() {
    ArrayList<Position> positions = new ArrayList<Position>();
    positions.add(new Position(-0.1439, 51.4952));
    positions.add(new Position(-0.1346, 51.4978));
    positions.add(new Position(-0.2177, 51.5135));
    positions.add(new Position(-0.1439, 51.4952));
    Polygon geometry = new Polygon(positions);

    FindIterable<Document> result = collection.find(
      Filters.geoIntersects("location", geometry));

    assertNotNull(result.first());
    assertEquals("Hyde Park", result.first().get("name"));
}
```

结果查询:

```
db.places.find({
  location:{
    $geoIntersects:{
      $geometry:{
        type:"Polygon",
          coordinates:[
          [
            [-0.1439, 51.4952],
            [-0.1346, 51.4978],
            [-0.2177, 51.5135],
            [-0.1439, 51.4952]
          ]
        ]
      }
    }
  }
})
```

## 5。结论

在本文中，我们学习了如何在 MongoDB 中存储地理空间数据，并查看了`2d`和`2dsphere`地理空间索引之间的差异。我们还学习了如何使用地理空间查询在 MongoDB 中进行搜索。

像往常一样，这些例子的完整源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220628161904/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-mongodb-2)