# Apache Commons Collections MapUtils

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/apache-commons-map-utils>

[This article is part of a series:](javascript:void(0);)[• Apache Commons Collections Bag](/web/20221208143855/https://www.baeldung.com/apache-commons-bag)
[• Apache Commons Collections SetUtils](/web/20221208143855/https://www.baeldung.com/apache-commons-setutils)
[• Apache Commons Collections OrderedMap](/web/20221208143855/https://www.baeldung.com/apache-commons-ordered-map)
[• Apache Commons Collections BidiMap](/web/20221208143855/https://www.baeldung.com/commons-collections-bidi-map)
[• A Guide to Apache Commons Collections CollectionUtils](/web/20221208143855/https://www.baeldung.com/apache-commons-collection-utils)
• Apache Commons Collections MapUtils (current article)[• Guide to Apache Commons CircularFifoQueue](/web/20221208143855/https://www.baeldung.com/commons-circular-fifo-queue)

## 1。简介

`MapUtils`是 Apache Commons Collections 项目中可用的工具之一。

简单地说，它提供了实用方法和装饰器来处理`java.util.Map`和`java.util.SortedMap`实例。

## 2。设置

让我们从添加依赖关系开始:

```
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-collections4</artifactId>
    <version>4.1</version>
</dependency>
```

## 3。实用方法

### 3.1。从`Array` 创建`Map`

现在，让我们设置将用于创建地图的数组:

```
public class MapUtilsTest {
    private String[][] color2DArray = new String[][] {
        {"RED", "#FF0000"},
        {"GREEN", "#00FF00"},
        {"BLUE", "#0000FF"}
    };
    private String[] color1DArray = new String[] {
        "RED", "#FF0000",
        "GREEN", "#00FF00",
        "BLUE", "#0000FF"
    };
    private Map<String, String> colorMap;

    //...
}
```

让我们看看如何从二维数组创建地图:

```
@Test
public void whenCreateMapFrom2DArray_theMapIsCreated() {
    this.colorMap = MapUtils.putAll(
      new HashMap<>(), this.color2DArray);

    assertThat(
      this.colorMap, 
      is(aMapWithSize(this.color2DArray.length)));

    assertThat(this.colorMap, hasEntry("RED", "#FF0000"));
    assertThat(this.colorMap, hasEntry("GREEN", "#00FF00"));
    assertThat(this.colorMap, hasEntry("BLUE", "#0000FF"));
}
```

我们也可以使用一维数组。在这种情况下，数组被视为备用索引中的键和值:

```
@Test
public void whenCreateMapFrom1DArray_theMapIsCreated() {
    this.colorMap = MapUtils.putAll(
      new HashMap<>(), this.color1DArray);

    assertThat(
      this.colorMap, 
      is(aMapWithSize(this.color1DArray.length / 2)));

    assertThat(this.colorMap, hasEntry("RED", "#FF0000"));
    assertThat(this.colorMap, hasEntry("GREEN", "#00FF00"));
    assertThat(this.colorMap, hasEntry("BLUE", "#0000FF"));
}
```

### 3.2。`Map`印刷的内容

很多时候，在调试或调试日志中，我们希望打印整个地图:

```
@Test
public void whenVerbosePrintMap_thenMustPrintFormattedMap() {
    MapUtils.verbosePrint(System.out, "Optional Label", this.colorMap);
}
```

结果是:

```
Optional Label = 
{
    RED = #FF0000
    BLUE = #0000FF
    GREEN = #00FF00
}
```

我们也可以使用`debugPrint()`，它额外打印值的数据类型。

### 3.3。获取值

`MapUtils`提供了一些方法，用于以一种`null`安全的方式从一个给定键的映射中提取值。

例如，`getString()`从`Map`得到一个`String`。通过`toString()`获得`String`值。如果值为`null`或转换失败，我们可以选择指定要返回的默认值:

```
@Test
public void whenGetKeyNotPresent_thenMustReturnDefaultValue() {
    String defaultColorStr = "COLOR_NOT_FOUND";
    String color = MapUtils
      .getString(this.colorMap, "BLACK", defaultColorStr);

    assertEquals(color, defaultColorStr);
}
```

请注意，这些方法是`null`安全的，即它们可以安全地处理`null` map 参数:

```
@Test
public void whenGetOnNullMap_thenMustReturnDefaultValue() {
    String defaultColorStr = "COLOR_NOT_FOUND";
    String color = MapUtils.getString(null, "RED", defaultColorStr);

    assertEquals(color, defaultColorStr);
}
```

这里,`color`将得到值为`COLOR_NOT_FOUND`,即使地图是`null`。

### 3.4。反相`Map`

我们还可以轻松地反转地图:

```
@Test
public void whenInvertMap_thenMustReturnInvertedMap() {
    Map<String, String> invColorMap = MapUtils.invertMap(this.colorMap);

    int size = invColorMap.size();
    Assertions.assertThat(invColorMap)
      .hasSameSizeAs(colorMap)
      .containsKeys(this.colorMap.values().toArray(new String[] {}))
      .containsValues(this.colorMap.keySet().toArray(new String[] {}));
}
```

这将把`colorMap`反转为`:`

```
{
    #00FF00 = GREEN
    #FF0000 = RED
    #0000FF = BLUE
}
```

如果源映射为多个键关联相同的值，那么在反转后，其中一个值将随机成为一个键。

### 3.5。无效支票

如果一个`Map`为`null`或为空，则`isEmpty()`方法返回`true`。

`safeAddToMap()`方法防止将空元素添加到`Map.`

## 4。装修工

这些方法为`Map.`增加了额外的功能

在大多数情况下，最好不要存储对修饰图`.`的引用

### 4.1。固定尺寸`Map`

`fixedSizeMap()`返回由给定地图支持的固定大小的地图。可以更改元素，但不能添加或删除:

```
@Test(expected = IllegalArgumentException.class)
public void whenCreateFixedSizedMapAndAdd_thenMustThrowException() {
    Map<String, String> rgbMap = MapUtils
      .fixedSizeMap(MapUtils.putAll(new HashMap<>(), this.color1DArray));

    rgbMap.put("ORANGE", "#FFA500");
}
```

### 4.2。`Map`断言

`predicatedMap()`方法返回一个`Map`,确保所有保存的元素都匹配所提供的谓词:

```
@Test(expected = IllegalArgumentException.class)
public void whenAddDuplicate_thenThrowException() {
    Map<String, String> uniqValuesMap 
      = MapUtils.predicatedMap(this.colorMap, null, 
        PredicateUtils.uniquePredicate());

    uniqValuesMap.put("NEW_RED", "#FF0000");
}
```

这里，我们使用`PredicateUtils.uniquePredicate()`为值指定了谓词。任何将重复值插入此映射的尝试都将导致`java.lang.` `IllegalArgumentException`。

我们可以通过实现`Predicate`接口来实现定制谓词。

### 4.3。`Map` 懒惰

`lazyMap()`返回一个映射，当被请求时，该映射中的值被初始化。

如果传递给此映射的`Map.get(Object)`方法的键在映射中不存在，则`Transformer`实例将用于创建一个新对象，该对象将与所请求的键相关联:

```
@Test
public void whenCreateLazyMap_theMapIsCreated() {
    Map<Integer, String> intStrMap = MapUtils.lazyMap(
      new HashMap<>(),
      TransformerUtils.stringValueTransformer());

    assertThat(intStrMap, is(anEmptyMap()));

    intStrMap.get(1);
    intStrMap.get(2);
    intStrMap.get(3);

    assertThat(intStrMap, is(aMapWithSize(3)));
}
```

## 5。结论

在这个快速教程中，我们已经探索了 Apache Commons Collections `MapUtils`类，并研究了可以简化各种常见地图操作的各种实用方法和装饰器。

像往常一样，代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221208143855/https://github.com/eugenp/tutorials/tree/master/libraries-apache-commons-collections)

Next **»**[Guide to Apache Commons CircularFifoQueue](/web/20221208143855/https://www.baeldung.com/commons-circular-fifo-queue)**«** Previous[A Guide to Apache Commons Collections CollectionUtils](/web/20221208143855/https://www.baeldung.com/apache-commons-collection-utils)