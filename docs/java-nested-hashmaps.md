# Java 中的嵌套 HashMaps 示例

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-nested-hashmaps>

## 1.概观

在本教程中，我们将看看如何在 Java 中处理嵌套的`[HashMaps](/web/20220525124848/https://www.baeldung.com/java-hashmap)`。我们还将了解如何创建和比较它们。最后，我们还将了解如何在内部映射中删除和添加记录。

## 2.用例

嵌套的`HashMap`对于存储 JSON 或类似 JSON 的结构非常有帮助，在这种结构中，对象相互嵌入。例如，类似于以下内容的结构或 JSON:

```java
{
    "type": "donut",
    "batters":
    {
        “batter”:
        [
            { "id": "1001", "type": "Regular" },
            { "id": "1002", "type": "Chocolate" },
            { "id": "1003", "type": "Blueberry" },
            { "id": "1004", "type": "Devil's Food" }
        ]
    }
} 
```

是嵌套的`HashMap`的完美候选。一般来说，每当我们需要将一个对象嵌入另一个对象时，我们都可以使用它们。

## 3.创建一个`HashMap`

[创建`HashMap`](/web/20220525124848/https://www.baeldung.com/java-initialize-hashmap) 有多种方式，比如手动构建地图或者使用`[Streams](/web/20220525124848/https://www.baeldung.com/java-streams)`和分组功能。 [`Map`](/web/20220525124848/https://www.baeldung.com/java-hashmap) 结构既可以是基元类型，也可以是 [`Objects`](/web/20220525124848/https://www.baeldung.com/java-classes-objects) 。

### 3.1.使用`p` `ut()`的方法

我们可以通过手动创建内部地图，然后使用 put 方法将它们插入到外部的`Map`来构建一个嵌套的`HashMap`:

```java
public Map<Integer, String> buildInnerMap(List<String> batterList) {
     Map<Integer, String> innerBatterMap = new HashMap<Integer, String>();
     int index = 1;
     for (String item : batterList) {
         innerBatterMap.put(index, item);
         index++;
     }
     return innerBatterMap;
} 
```

我们可以用以下方法进行测试:

```java
assertThat(mUtil.buildInnerMap(batterList), is(notNullValue()));
Assert.assertEquals(actualBakedGoodsMap.keySet().size(), 2);
Assert.assertThat(actualBakedGoodsMap, IsMapContaining.hasValue(equalTo(mUtil.buildInnerMap(batterList))));
```

### 3.2.使用流

如果我们想要将一个`List`转换成一个`Map`，我们可以创建一个流，然后使用 [`Collectors.toMap`](/web/20220525124848/https://www.baeldung.com/java-collectors-tomap) 方法将其转换成一个`Map`。这里，我们有两个例子:一个有一个`Strings`的内部`Map`，另一个是一个有`Integer`和`Object`值的`Map`。

在第一个例子中，`Employee`中嵌套了`Address`对象。然后我们建立一个嵌套的`HashMap`:

```java
Map<Integer, Map<String, String>> employeeAddressMap = listEmployee.stream()
  .collect(Collectors.groupingBy(e -> e.getAddress().getAddressId(),
    Collectors.toMap(f -> f.getAddress().getAddressLocation(), Employee::getEmployeeName)));
return employeeAddressMap;
```

在第二个例子中，我们正在构建一个类型为`<Employee id <Address id, Address object>>`的对象:

```java
Map<Integer, Map<Integer, Address>> employeeMap = new HashMap<>();
employeeMap = listEmployee.stream().collect(Collectors.groupingBy((Employee emp) -> emp.getEmployeeId(),
  Collectors.toMap((Employee emp) -> emp.getAddress().getAddressId(), fEmpObj -> fEmpObj.getAddress())));
return employeeMap;
```

## 4.遍历嵌套的`HashMap`

遍历嵌套的`Hashmap`与遍历常规的或未嵌套的`HashMap`没有什么不同。嵌套的和常规的`Map`的唯一区别是嵌套的`HashMap`的值是`Map `类型:

```java
for (Map.Entry<String, Map<Integer, String>> outerBakedGoodsMapEntrySet : outerBakedGoodsMap.entrySet()) {
    Map<Integer, String> valueMap = outerBakedGoodsMapEntrySet.getValue();
    System.out.println(valueMap.entrySet());
}

for (Map.Entry<Integer, Map<String, String>> employeeEntrySet : employeeAddressMap.entrySet()) {
    Map<String, String> valueMap = employeeEntrySet.getValue();
    System.out.println(valueMap.entrySet());
}
```

## 5.比较嵌套的`HashMap`

Java 中比较`HashMap`的方法有很多种。我们可以使用`equals()`方法来比较它们。默认实现比较每个值。

如果我们改变内部映射的内容，相等性检查就会失败。如果在用户定义对象的情况下，内部对象每次都是新的实例，则相等性检查也将失败。类似地，如果我们改变外部`Map`的内容，相等性检查也会失败:

```java
assertNotEquals(outerBakedGoodsMap2, actualBakedGoodsMap);

outerBakedGoodsMap3.put("Donut", mUtil.buildInnerMap(batterList));
assertNotEquals(outerBakedGoodsMap2, actualBakedGoodsMap);

Map<Integer, Map<String, String>> employeeAddressMap1 = mUtil.createNestedMapfromStream(listEmployee);
assertNotEquals(employeeAddressMap1, actualEmployeeAddressMap);
```

对于将用户定义的对象作为值的`Map`，我们需要使用在[比较`HashMap`的文章](/web/20220525124848/https://www.baeldung.com/java-compare-hashmaps)中提到的方法之一来定制等式方法。否则，检查将失败:

```java
//Comparing a Map<Integer, Map<String, String>> and Map<Integer, Map<Integer, Address>> map
assertNotSame(employeeMap1, actualEmployeeMap);
assertNotEquals(employeeMap1, actualEmployeeMap);
Map<Integer, Map<Integer, Address>> expectedMap = setupAddressObjectMap();
assertNotSame(expectedMap, actualEmployeeMap);
assertNotEquals(expectedMap, actualEmployeeMap);
```

如果两个映射相同，则相等性检查成功。对于用户定义的映射，如果所有相同的对象都被移动到另一个映射中，则相等性检查会成功:

```java
Map<String, Map<Integer, String>> outerBakedGoodsMap4 = new HashMap<>();
outerBakedGoodsMap4.putAll(actualBakedGoodsMap);
assertEquals(actualBakedGoodsMap, outerBakedGoodsMap4);
Map<Integer, Map<Integer, Address>> employeeMap1 = new HashMap<>();
employeeMap1.putAll(actualEmployeeMap);
assertEquals(actualEmployeeMap, employeeMap1);
```

## 6.向嵌套的`HashMap`添加元素

要向嵌套的`HashMap`的内部`Map`添加元素，我们首先必须检索它。我们可以使用`get()`方法来检索内部对象。然后我们可以在内部的`Map`对象上使用`put()`方法并插入新的值:

```java
assertEquals(actualBakedGoodsMap.get("Cake").size(), 5);
actualBakedGoodsMap.get("Cake").put(6, "Cranberry");
assertEquals(actualBakedGoodsMap.get("Cake").size(), 6);
```

如果我们必须向外部`Map`添加一个条目，我们还需要为内部`Map`提供正确的条目:

```java
outerBakedGoodsMap.put("Eclair", new HashMap<Integer, String>() {
    {
        put(1, "Dark Chocolate");
    }
});
```

## 7.**从嵌套的`HashMap` s 中删除记录**

要从内部`Map`删除记录，首先，我们需要检索它，然后使用`remove()`方法删除它。如果内部`Map`中只有一个值，那么剩下一个`null`对象作为值:

```java
assertNotEquals(actualBakedGoodsMap.get("Cake").get(5), null);
actualBakedGoodsMap.get("Cake").remove(5);
assertEquals(actualBakedGoodsMap.get("Cake").get(5), null);
```

```java
assertNotEquals(actualBakedGoodsMap.get("Eclair").get(1), null);
actualBakedGoodsMap.get("Eclair").remove(1);
assertEquals(actualBakedGoodsMap.get("Eclair").get(1), null);
actualBakedGoodsMap.put("Eclair", new HashMap<Integer, String>() {
    {
        put(1, "Dark Chocolate");
    }
});
```

如果我们从外部的`Map`中删除一个记录，Java 会删除内部和外部的`Map`记录，这是显而易见的，因为内部的`Map`是外部的`Map`的“值”:

```java
assertNotEquals(actualBakedGoodsMap.get("Eclair"), null);
actualBakedGoodsMap.remove("Eclair");
assertEquals(actualBakedGoodsMap.get("Eclair"), null);
```

## 8.展平嵌套的`HashMap`

嵌套`HashMap`的一种替代方法是使用组合键。组合键通常将嵌套结构中的两个键连接起来，中间有一个点。例如，组合键可以是`Donut.1`、`Donut.2`等等。我们可以“扁平化”，即从嵌套的`Map`结构转换成单一的`Map`结构:

```java
var flattenedBakedGoodsMap = mUtil.flattenMap(actualBakedGoodsMap);
assertThat(flattenedBakedGoodsMap, IsMapContaining.hasKey("Donut.2"));
var flattenedEmployeeAddressMap = mUtil.flattenMap(actualEmployeeAddressMap);
assertThat(flattenedEmployeeAddressMap, IsMapContaining.hasKey("200.Bag End"));
```

组合键方法克服了嵌套`HashMaps`带来的额外内存存储缺点。然而，组合键方法并不擅长缩放。

## 9.结论

在本文中，我们看到了如何创建、比较、更新和展平嵌套的`HashMap`。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220525124848/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-4)