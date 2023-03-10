# 弹性搜索中的地理空间支持

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/elasticsearch-geo-spatial>

## 1。 **简介**

Elasticsearch 最出名的是它的全文搜索功能，但它也有完整的地理空间支持。

我们可以在[上一篇文章](/web/20221129015827/https://www.baeldung.com/elasticsearch-java)中找到更多关于设置 Elasticsearch 和开始使用的信息。

让我们看看如何在 Elasticsearch 中保存地理数据，以及如何使用地理查询来搜索这些数据。

## 2。地理数据类型

为了启用地理查询，我们需要手动创建索引的映射，并显式设置字段映射。

**为地理类型设置映射时，动态映射将不起作用。**

Elasticsearch 提供了两种表示地理数据的方式:

1.  使用地理点字段类型的经纬度对
2.  使用地理形状字段类型在`GeoJSON`中定义的复杂形状

让我们更深入地看看上面的每一个类别:

### 2.1。地理点数据类型

地理点字段类型接受经纬度对，可用于:

*   寻找中心点一定距离内的点
*   在一个盒子或多边形内寻找点
*   按地理位置或距中心点的距离聚集文档
*   按距离分类文件

下面是保存地理点数据的字段映射示例:

```java
PUT /index_name
{
    "mappings": {
        "TYPE_NAME": {
            "properties": {
                "location": { 
                    "type": "geo_point" 
                } 
            } 
        } 
    } 
}
```

从上面的例子我们可以看出，为 *位置* 字段为*geo _ point*。因此，我们现在可以在位置字段中的 *位置* 中提供经纬度对。

### 2.2。地理形状数据类型

与`geo-point`不同，`geo shape`提供了保存和搜索多边形和矩形等复杂形状的功能。`Geo shape`当我们想要搜索包含形状而不是地理点的文档时，必须使用数据类型。

让我们来看看地理形状数据类型的映射:

```java
PUT /index_name
{
    "mappings": {
        "TYPE_NAME": {
            "properties": {
                "location": {
                    "type": "geo_shape"
                }
            }
        }
    }
}
```

**最近版本的 Elasticsearch 将提供的地理形状分解成一个三角形网格**。根据官方[文档](https://web.archive.org/web/20221129015827/https://www.elastic.co/guide/en/elasticsearch/reference/current/geo-shape.html#geo-shape-mapping-options)，这提供了近乎完美的空间分辨率。

## 3。保存地理点数据的不同方法

### 3.1。纬度经度对象

```java
PUT index_name/index_type/1
{
    "location": { 
        "lat": 23.02,
        "lon": 72.57
    }
}
```

这里，地理点 *位置* 保存为对象，以 *纬度* 和 *经度* 为关键字。

### 3.2。纬度经度对

```java
{
    "location": "23.02,72.57"
}
```

在这里， *位置* 用普通字符串格式表示为经纬度对。请注意，字符串格式的经纬度序列。

### 3.3。地理哈希

```java
{
    "location": "tsj4bys"
}
```

我们还可以以地理哈希的形式提供地理点数据，如上例所示。我们可以使用[在线工具](https://web.archive.org/web/20221129015827/http://www.movable-type.co.uk/scripts/geohash.html)将经纬度转换为地理哈希。

### 3.4。经度纬度数组

```java
{
    "location": [72.57, 23.02]
}
```

当纬度和经度作为数组提供时，纬度-经度的顺序是相反的。最初，纬度-经度对被用在字符串和数组中，但是后来为了匹配 [GeoJSON](https://web.archive.org/web/20221129015827/https://en.wikipedia.org/wiki/GeoJSON) 使用的格式，它被颠倒了。

## 4。保存地理形状数据的不同方法

### 4.1。`Point`

```java
POST /index/type
{
    "location" : {
        "type" : "point",
        "coordinates" : [72.57, 23.02]
    }
}
```

这里，我们试图插入的地理形状类型是一个`point`。请看一下`location`字段，我们有一个由字段`type`和`coordinates`组成的嵌套对象。这些元字段帮助 Elasticsearch 识别地理形状及其实际数据。

### 4.2。`LineString`

```java
POST /index/type
{
    "location" : {
        "type" : "linestring",
        "coordinates" : [[77.57, 23.02], [77.59, 23.05]]
    }
}
```

在这里，我们插入`linestring`地理形状。`linestring`的坐标由两点组成，即起点和终点。`LineString` geo shape 对导航用例非常有帮助。

### 4.3。`Polygon`

```java
POST /index/type
{
    "location" : {
        "type" : "polygon",
        "coordinates" : [
            [ [10.0, 0.0], [11.0, 0.0], [11.0, 1.0], [10.0, 1.0], [10.0, 0.0] ]
        ]
    }
}
```

在这里，我们插入`polygon`地理形状。请看上面例子中的`coordinates`，`first`和`last`在多边形中的坐标应该总是匹配的，即一个闭合的多边形。

**Elasticsearch 也支持其他 GeoJSON 结构。其他支持格式的完整列表如下:**

*   **T2`MultiPoint`**
*   **T2`MultiLineString`**
*   **T2`MultiPolygon`**
*   **T2`GeometryCollection`**
*   **T2`Envelope`**
*   **T2`Circle`**

我们可以在官方网站上找到上述支持格式的例子。

对于所有结构，内部的`type`和`coordinates`是必填字段。此外，由于其复杂的结构，分类和检索地理形状场目前在弹性搜索中是不可能的。因此，检索 geo 字段的唯一方法是从源字段中检索。

## 5 号。弹性大地查询〔t1〕

现在，我们知道了如何插入包含地理形状的文档，让我们开始使用地理形状查询获取这些记录。但是在我们开始使用地理查询之前，我们需要以下 maven 依赖项来支持用于地理查询的 Java API:

```java
<dependency>
    <groupId>org.locationtech.spatial4j</groupId>
    <artifactId>spatial4j</artifactId>
    <version>0.7</version> 
</dependency>
<dependency>
    <groupId>com.vividsolutions</groupId>
    <artifactId>jts</artifactId>
    <version>1.13</version>
    <exclusions>
        <exclusion>
            <groupId>xerces</groupId>
            <artifactId>xercesImpl</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

我们也可以在 [Maven 中央存储库](https://web.archive.org/web/20221129015827/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22org.locationtech.spatial4j%22%20AND%20a%3A%22spatial4j%22)中搜索上述依赖项。

Elasticsearch 支持不同类型的地理查询，具体如下:

### 5.1。地理形状查询

这需要`geo_shape`映射。

与`geo_shape`类型类似，`geo_shape`使用 GeoJSON 结构查询文档。

下面是一个示例查询，用于获取给定左上和右下坐标的所有落入`within`的文档:

```java
{
    "query":{
        "bool": {
            "must": {
                "match_all": {}
            },
            "filter": {
                "geo_shape": {
                    "region": {
                        "shape": {
                            "type": "envelope",
                            "coordinates" : [[75.00, 25.0], [80.1, 30.2]]
                        },
                        "relation": "within"
                    }
                }
            }
        }
    }
}
```

这里，`relation`决定了在搜索时使用的**空间关系操作符**。

以下是支持的运算符列表:

*   **`INTERSECTS`**–(默认)返回其`geo_shape`字段与查询几何相交的所有文档
*   **`DISJOINT`**–检索其`geo_shape`字段与查询几何毫无共同之处的所有文档
*   **`WITHIN`** –获取其`geo_shape` 字段在查询几何内的所有文档
*   **`CONTAINS`** –返回其`geo_shape` 字段包含查询几何的所有文档

同样，我们可以使用不同的 GeoJSON 形状进行查询。

上述查询的 Java 代码如下:

```java
Coordinate topLeft = new Coordinate(74, 31.2);
Coordinate bottomRight = new Coordinate(81.1, 24);

GeoShapeQueryBuilder qb = QueryBuilders.geoShapeQuery("region",
    new EnvelopeBuilder(topLeft, bottomRight).buildGeometry());
qb.relation(ShapeRelation.INTERSECTS);
```

### 5.2。地理边界框查询

地理包围盒查询用于基于点位置提取所有文档。下面是一个边界框查询示例:

```java
{
    "query": {
        "bool" : {
            "must" : {
                "match_all" : {}
            },
            "filter" : {
                "geo_bounding_box" : {
                    "location" : {
                        "bottom_left" : [28.3, 30.5],
                        "top_right" : [31.8, 32.12]
                    }
                }
            }
        }
    }
}
```

上述边界框查询的 Java 代码如下:

```java
QueryBuilders
  .geoBoundingBoxQuery("location").setCorners(31.8, 30.5, 28.3, 32.12);
```

地理边界框查询支持与`geo_point`数据类型类似的格式。有关支持格式的示例查询可在[官方网站](https://web.archive.org/web/20221129015827/https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-geo-bounding-box-query.html#_accepted_formats)上找到。

### 5.3。地理距离查询

地理距离查询用于过滤具有指定范围的点的所有文档。

下面是一个示例`geo_distance`查询:

```java
{
    "query": {
        "bool" : {
            "must" : {
                "match_all" : {}
            },
            "filter" : {
                "geo_distance" : {
                    "distance" : "10miles",
                    "location" : [31.131,29.976]
                }
            }
        }
    }
}
```

下面是上述查询的 Java 代码:

```java
QueryBuilders
  .geoDistanceQuery("location")
  .point(29.976, 31.131)
  .distance(10, DistanceUnit.MILES);
```

与`geo_point,`类似，地理距离查询也支持多种格式来传递位置坐标。更多关于支持格式的细节可以在[官方网站](https://web.archive.org/web/20221129015827/https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-geo-distance-query.html#_accepted_formats_2)找到。

### 5.4。地理位置`Polygon`查询

一种查询，用于过滤所有点位于给定点多边形内的所有记录。

让我们快速看一下一个示例查询:

```java
{
    "query": {
        "bool" : {
            "must" : {
                "match_all" : {}
            },
            "filter" : {
                "geo_polygon" : {
                    "location" : {
                        "points" : [
                        {"lat" : 22.733, "lon" : 68.859},
                        {"lat" : 24.733, "lon" : 68.859},
                        {"lat" : 23, "lon" : 70.859}
                        ]
                    }
                }
            }
        }
    }
}
```

这个查询的 Java 代码:

```java
List<GeoPoint> allPoints = new ArrayList<GeoPoint>(); 
allPoints.add(new GeoPoint(22.733, 68.859)); 
allPoints.add(new GeoPoint(24.733, 68.859)); 
allPoints.add(new GeoPoint(23, 70.859));

QueryBuilders.geoPolygonQuery("location", allPoints);
```

地理多边形查询还支持以下格式:

*   lat-long 作为数组:[lon，lat]
*   lat-long 字符串:" lat，lon "
*   地理哈希

为了使用此查询，数据类型是必需的。

## 6。结论

在本文中，我们讨论了索引地理数据的不同映射选项，即`geo_point`和`geo_shape`。

我们还通过不同的方式来存储`geo-data`，最后，我们观察了地理查询和 Java API 来使用地理查询过滤结果。

与往常一样，在这个 GitHub 项目中可以获得代码[。](https://web.archive.org/web/20221129015827/https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-elasticsearch)