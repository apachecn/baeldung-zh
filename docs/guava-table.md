# 番石榴餐桌指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/guava-table>

## 1。概述

在本教程中，我们将展示如何使用 Google Guava 的`Table`界面及其多种实现。

Guava 的`Table`是一个集合，表示一个类似表格的结构，包含行、列和相关的单元格值。行和列充当一对有序的键。

## 2。谷歌番石榴的`Table`

让我们来看看如何使用`Table`类。

### 2.1。Maven 依赖关系

让我们从在`pom.xml`中添加 Google 的番石榴库依赖项开始:

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

依赖关系的最新版本可以在[这里](https://web.archive.org/web/20220816215415/https://search.maven.org/classic/#search|gav|1|g%3A%22com.google.guava%22%20AND%20a%3A%22guava%22)查看。

### 2.2。关于

**如果我们使用 core Java 中的`Collections`来表示 Guava 的`Table`，那么这个结构将是一个行的映射，其中每一行包含一个列的映射以及相关的单元格值。**

`Table`表示一个特殊的映射，其中两个键可以以组合的方式被指定来引用一个值。

类似于创建地图的地图，比如`Map<UniversityName, Map<CoursesOffered, SeatAvailable>>`。`Table`也是表现战列舰游戏棋盘的完美方式。

## 3。创建

您可以通过多种方式创建`Table`的实例:

*   从内部使用`LinkedHashMap`的类`HashBasedTable`中使用`create`方法:

    ```java
    Table<String, String, Integer> universityCourseSeatTable 
      = HashBasedTable.create();
    ```

*   如果我们需要一个`Table`，它的行键和列键需要按照它们的自然顺序或者通过提供比较器来排序，那么您可以从一个名为`TreeBasedTable`的类中使用`create`方法创建一个`Table`的实例，该类在内部使用【T4:

    ```java
    Table<String, String, Integer> universityCourseSeatTable
      = TreeBasedTable.create(); 
    ```

*   如果我们预先知道行键和列键，并且表的大小是固定的，那么使用来自类`ArrayTable` :

    ```java
    List<String> universityRowTable 
      = Lists.newArrayList("Mumbai", "Harvard");
    List<String> courseColumnTables 
      = Lists.newArrayList("Chemical", "IT", "Electrical");
    Table<String, String, Integer> universityCourseSeatTable
      = ArrayTable.create(universityRowTable, courseColumnTables); 
    ```

    的`create`方法
*   如果我们打算创建一个内部数据永远不会改变的`Table`的不可变实例，使用`ImmutableTable`类(按照构建器模式创建):

    ```java
    Table<String, String, Integer> universityCourseSeatTable
      = ImmutableTable.<String, String, Integer> builder()
      .put("Mumbai", "Chemical", 120).build(); 
    ```

## 4。使用

让我们从一个简单的例子开始，展示`Table`的用法。

### 4.1。检索

如果我们知道行键和列键，那么我们可以得到与行键和列键相关的值:

```java
@Test
public void givenTable_whenGet_returnsSuccessfully() {
    Table<String, String, Integer> universityCourseSeatTable 
      = HashBasedTable.create();
    universityCourseSeatTable.put("Mumbai", "Chemical", 120);
    universityCourseSeatTable.put("Mumbai", "IT", 60);
    universityCourseSeatTable.put("Harvard", "Electrical", 60);
    universityCourseSeatTable.put("Harvard", "IT", 120);

    int seatCount 
      = universityCourseSeatTable.get("Mumbai", "IT");
    Integer seatCountForNoEntry 
      = universityCourseSeatTable.get("Oxford", "IT");

    assertThat(seatCount).isEqualTo(60);
    assertThat(seatCountForNoEntry).isEqualTo(null);
}
```

### 4.2。检查条目

我们可以根据以下内容检查`Table`中条目的存在:

*   行键
*   列键
*   行键和列键
*   价值

让我们看看如何检查条目的存在:

```java
@Test
public void givenTable_whenContains_returnsSuccessfully() {
    Table<String, String, Integer> universityCourseSeatTable 
      = HashBasedTable.create();
    universityCourseSeatTable.put("Mumbai", "Chemical", 120);
    universityCourseSeatTable.put("Mumbai", "IT", 60);
    universityCourseSeatTable.put("Harvard", "Electrical", 60);
    universityCourseSeatTable.put("Harvard", "IT", 120);

    boolean entryIsPresent
      = universityCourseSeatTable.contains("Mumbai", "IT");
    boolean courseIsPresent 
      = universityCourseSeatTable.containsColumn("IT");
    boolean universityIsPresent 
      = universityCourseSeatTable.containsRow("Mumbai");
    boolean seatCountIsPresent 
      = universityCourseSeatTable.containsValue(60);

    assertThat(entryIsPresent).isEqualTo(true);
    assertThat(courseIsPresent).isEqualTo(true);
    assertThat(universityIsPresent).isEqualTo(true);
    assertThat(seatCountIsPresent).isEqualTo(true);
}
```

### 4.3。移除

我们可以通过提供行键和列键从`Table`中删除一个条目:

```java
@Test
public void givenTable_whenRemove_returnsSuccessfully() {
    Table<String, String, Integer> universityCourseSeatTable
      = HashBasedTable.create();
    universityCourseSeatTable.put("Mumbai", "Chemical", 120);
    universityCourseSeatTable.put("Mumbai", "IT", 60);

    int seatCount 
      = universityCourseSeatTable.remove("Mumbai", "IT");

    assertThat(seatCount).isEqualTo(60);
    assertThat(universityCourseSeatTable.remove("Mumbai", "IT")).
      isEqualTo(null);
} 
```

### 4.4。单元格值映射的行键

通过提供列键，我们可以得到一个键为行、值为 T1 的`Map`表示:

```java
@Test
public void givenTable_whenColumn_returnsSuccessfully() {
    Table<String, String, Integer> universityCourseSeatTable 
      = HashBasedTable.create();
    universityCourseSeatTable.put("Mumbai", "Chemical", 120);
    universityCourseSeatTable.put("Mumbai", "IT", 60);
    universityCourseSeatTable.put("Harvard", "Electrical", 60);
    universityCourseSeatTable.put("Harvard", "IT", 120);

    Map<String, Integer> universitySeatMap 
      = universityCourseSeatTable.column("IT");

    assertThat(universitySeatMap).hasSize(2);
    assertThat(universitySeatMap.get("Mumbai")).isEqualTo(60);
    assertThat(universitySeatMap.get("Harvard")).isEqualTo(120);
} 
```

### 4.5。`Table`地图表示一个

我们可以通过使用`columnMap`方法得到一个`Map<UniversityName, Map<CoursesOffered, SeatAvailable>>`表示:

```java
@Test
public void givenTable_whenColumnMap_returnsSuccessfully() {
    Table<String, String, Integer> universityCourseSeatTable 
      = HashBasedTable.create();
    universityCourseSeatTable.put("Mumbai", "Chemical", 120);
    universityCourseSeatTable.put("Mumbai", "IT", 60);
    universityCourseSeatTable.put("Harvard", "Electrical", 60);
    universityCourseSeatTable.put("Harvard", "IT", 120);

    Map<String, Map<String, Integer>> courseKeyUniversitySeatMap 
      = universityCourseSeatTable.columnMap();

    assertThat(courseKeyUniversitySeatMap).hasSize(3);
    assertThat(courseKeyUniversitySeatMap.get("IT")).hasSize(2);
    assertThat(courseKeyUniversitySeatMap.get("Electrical")).hasSize(1);
    assertThat(courseKeyUniversitySeatMap.get("Chemical")).hasSize(1);
} 
```

### 4.6。单元格值映射的列键

通过提供行键，我们可以得到一个键为列、值为 T1 的`Map`表示:

```java
@Test
public void givenTable_whenRow_returnsSuccessfully() {
    Table<String, String, Integer> universityCourseSeatTable 
      = HashBasedTable.create();
    universityCourseSeatTable.put("Mumbai", "Chemical", 120);
    universityCourseSeatTable.put("Mumbai", "IT", 60);
    universityCourseSeatTable.put("Harvard", "Electrical", 60);
    universityCourseSeatTable.put("Harvard", "IT", 120);

    Map<String, Integer> courseSeatMap 
      = universityCourseSeatTable.row("Mumbai");

    assertThat(courseSeatMap).hasSize(2);
    assertThat(courseSeatMap.get("IT")).isEqualTo(60);
    assertThat(courseSeatMap.get("Chemical")).isEqualTo(120);
} 
```

### 4.7。获取不同的行键

我们可以使用`rowKeySet`方法从一个表中获取所有的行键:

```java
@Test
public void givenTable_whenRowKeySet_returnsSuccessfully() {
    Table<String, String, Integer> universityCourseSeatTable
      = HashBasedTable.create();
    universityCourseSeatTable.put("Mumbai", "Chemical", 120);
    universityCourseSeatTable.put("Mumbai", "IT", 60);
    universityCourseSeatTable.put("Harvard", "Electrical", 60);
    universityCourseSeatTable.put("Harvard", "IT", 120);

    Set<String> universitySet = universityCourseSeatTable.rowKeySet();

    assertThat(universitySet).hasSize(2);
} 
```

### 4.8。获取不同的列键

我们可以使用`columnKeySet`方法从一个表中获取所有的列键:

```java
@Test
public void givenTable_whenColKeySet_returnsSuccessfully() {
    Table<String, String, Integer> universityCourseSeatTable
      = HashBasedTable.create();
    universityCourseSeatTable.put("Mumbai", "Chemical", 120);
    universityCourseSeatTable.put("Mumbai", "IT", 60);
    universityCourseSeatTable.put("Harvard", "Electrical", 60);
    universityCourseSeatTable.put("Harvard", "IT", 120);

    Set<String> courseSet = universityCourseSeatTable.columnKeySet();

    assertThat(courseSet).hasSize(3);
} 
```

## 5。结论

在本教程中，我们展示了来自 Guava 库的`Table`类的方法。`Table`类提供了一个集合，表示一个类似表格的结构，包含行、列和相关的单元格值。

属于上述例子的代码可以在 GitHub 项目中找到——这是一个基于 Maven 的项目，所以它应该很容易导入和运行。