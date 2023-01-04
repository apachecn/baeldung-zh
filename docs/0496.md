# 使用 Tablesaw 处理表格数据

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/tablesaw>

## 1.介绍

在本文中，我们将学习使用 Tablesaw 库来处理表格数据。首先，我们将导入一些数据。然后，我们将通过处理数据获得一些洞察力。

我们将使用[鳄梨价格](https://web.archive.org/web/20221125192801/https://www.kaggle.com/datasets/neuromusic/avocado-prices)数据集。简而言之，它包含多个美国市场上鳄梨价格和销量的历史数据。

## 2.在 Tablesaw 中导入数据

首先，我们需要导入数据。Tablesaw 支持各种格式，包括我们数据集的格式 CSV。因此，让我们从数据集的 CSV 文件加载数据集开始:

```
CsvReadOptions csvReadOptions =
    CsvReadOptions.builder(file)
        .separator(',')
        .header(true)
        .dateFormat(formatter)
        .build();
table = Table.read().usingOptions(csvReadOptions);
```

上面，我们通过将`File`对象传递给[构建器](/web/20221125192801/https://www.baeldung.com/creational-design-patterns#builder)来创建`CsvReadOptions`类。然后，我们描述如何通过正确配置 options 对象来读取 CSV 文件。

首先，我们使用`separator()`方法设置列分隔符。第二，我们读取文件的第一行作为标题。第三，我们提供了一个`DateTimeFormatter`来正确解析日期和时间。最后，我们使用新创建的`CsvReadOptions`来读取表格数据。

### 2.1.验证导入的数据

让我们使用`structure()`方法来检查表格的设计。它返回另一个包含列名、索引和数据类型的表:

```
 Structure of avocado.csv         
 Index  |  Column Name   |  Column Type  |
------------------------------------------
     0  |            C0  |      INTEGER  |
     1  |          Date  |   LOCAL_DATE  |
     2  |  AveragePrice  |       DOUBLE  |
     3  |  Total Volume  |       DOUBLE  |
    ... |       ...      |       ...     |
```

接下来，让我们通过使用`shape()`方法来检查它的形状:

```
assertThat(table.shape()).isEqualTo("avocado.csv: 18249 rows X 14 cols");
```

这个方法返回一个`String`,文件名后跟行数和列数。我们的数据集总共包含 18249 行数据和 14 列。

## 3.表内数据表示法

Tablesaw 主要处理表格和列，它们构成了数据框的基础。简而言之，表是一组列，其中每一列都有固定的类型。表中的一行是一组值，每个值都被分配给其匹配的列。

Tablesaw 支持多种[列类型](https://web.archive.org/web/20221125192801/https://www.javadoc.io/static/tech.tablesaw/tablesaw-core/0.43.1/tech/tablesaw/api/ColumnType.html)。除了扩展 Java 中的基本类型之外，它还提供了文本列和时态列。

### 3.1.文本类型

在 Tablesaw 中，有两种文本类型:`TextColumn`和`StringColumn`。第一种类型是通用类型，它按原样保存任何文本。另一方面， **`StringColumn`在存储它们之前将值编码成类似字典的数据结构**。这样可以有效地保存数据，而不是在列中重复值。

例如，在 avocado 数据集中，region 和 type 列属于类型`StringColumn`。它们在列向量中的重复值被更有效地存储，并指向文本的同一个实例:

```
StringColumn type = table.stringColumn("type");
List<String> conventional = type.where(type.isEqualTo("conventional")).asList().stream()
    .limit(2)
    .toList();
assertThat(conventional.get(0)).isSameAs(conventional.get(1));
```

### 3.2.时间类型

Tablesaw 中有四种可用的时态类型。它们映射到对应的 Java 对象:`DateColumn`、`DateTimeColumn`、`TimeColumn`和`InstantColumn`。如上所述，我们可以在导入时配置如何解析这些值。

## 4.使用列

接下来，让我们看看如何处理导入的数据并从中提取见解。例如，在 Tablesaw 中，我们可以转换单个列或处理整个表。

### 4.1.创建新列

让我们通过调用在每种类型的可用列上定义的静态方法`.create()`来创建一个新列。例如，要使一个`TimeColumn`命名为`time`，我们写:

```
TimeColumn time = TimeColumn.create("Time");
```

然后可以使用`.addColumns()` 方法将该列添加到表中:

```
Table table = Table.create("test");
table.addColumns(time);
assertThat(table.columnNames()).contains("time");
```

### 4.2.添加或修改列数据

让我们使用`.append()`方法将数据添加到列的末尾:

```
DoubleColumn averagePrice = table.doubleColumn("AveragePrice");
averagePrice.append(1.123);
assertThat(averagePrice.get(averagePrice.size() - 1)).isEqualTo(1.123);
```

对于表，我们必须为每一列提供一个值，以确保所有列至少有一个值。否则，当创建具有不同大小的列的表时，它将抛出一个`IllegalArgumentException`:

```
DoubleColumn averagePrice2 = table.doubleColumn("AveragePrice").copy();
averagePrice2.setName("AveragePrice2");
averagePrice2.append(1.123);
assertThatExceptionOfType(IllegalArgumentException.class).isThrownBy(() -> table.addColumns(averagePrice2));
```

我们使用`the .set()`方法来改变列向量中的特定值。要使用它，我们必须知道我们想要改变的值的索引:

```
stringColumn.set(2, "Baeldung");
```

从列中删除数据可能会有问题，尤其是在表的情况下。因此， **Tablesaw 不允许从列向量**中删除值。相反，让我们使用`.setMissing()`将希望删除的值标记为缺失，并将每个值的索引传递给这个方法:

```
DoubleColumn averagePrice = table.doubleColumn("AveragePrice").setMissing(0);
assertThat(averagePrice.get(0)).isNull();
```

因此，它不会从 vector 中移除值 holder，而是将它设置为`null`。因此，向量的大小保持不变。

## 5.整理数据

接下来，让我们对之前导入的数据进行排序。首先，我们将根据一组列对表格行进行排序。为此，我们**使用了`.sortAscending()`和`.sortDescending()`方法**，它们接受列的名称。让我们排序以获取数据集中最早和最近的日期:

```
Table ascendingDateSortedTable = table.sortAscendingOn("Date");
assertThat(ascendingDateSortedTable.dateColumn("Date").get(0)).isEqualTo(LocalDate.parse("2015-01-04"));
Table descendingDateSortedTable = table.sortDescendingOn("Date");
assertThat(descendingDateSortedTable.dateColumn("Date").get(0)).isEqualTo(LocalDate.parse("2018-03-25"));
```

然而，**这些方法都非常局限**。例如，我们不能混合升序和降序排序。**为了解决这些限制，我们使用了`.sortOn()`方法**。默认情况下，它接受一组列名并对它们进行排序。为了对特定的列进行降序排序，我们在列名前面加了一个减号“-”。例如，让我们按年份和最高平均价格降序排列数据:

```
Table ascendingYearAndAveragePriceSortedTable = table.sortOn("year", "-AveragePrice");
assertThat(ascendingYearAndAveragePriceSortedTable.intColumn("year").get(0)).isEqualTo(2015);
assertThat(ascendingYearAndAveragePriceSortedTable.numberColumn("AveragePrice").get(0)).isEqualTo(2.79);
```

**这些方法并不适合所有的用例。对于这种情况，Tablesaw 接受一个针对`.sortOn()`方法**的自定义实现`Comparator<VRow>`。

## 6.过滤数据

过滤器允许我们从原始表中获取数据的子集。过滤一个表返回另一个表，**我们使用`.where()`和`.dropWhere()`方法来应用过滤器**。第一个方法将返回符合我们指定标准的值或行。相反，第二种方法会删除它们。

要指定过滤标准，我们首先需要了解`Selections.`

### 6.1.选择

**A `Selection`是一个逻辑位图**。换句话说，它是一个包含布尔值的数组，这些值屏蔽了列向量上的值。例如，将选择应用于一列将产生另一列，其中包含已筛选的值，例如，删除给定索引的掩码为 0 的值。此外，选择向量将与其原始列的大小相同。

让我们通过获得 2017 年平均价格仅高于 2 美元的数据表来实践这一点:

```
DateColumn dateTable = table.dateColumn("Date");
DoubleColumn averagePrice = table.doubleColumn("AveragePrice");
Selection selection = dateTable.isInYear(2017).and(averagePrice.isGreaterThan(2D));
Table table2017 = table.where(selection);
assertThat(table2017.intColumn("year")).containsOnly(2017);
assertThat(table2017.doubleColumn("AveragePrice")).allMatch(avrgPrice -> avrgPrice > 2D);
```

上面，我们使用了在`DateColumn`上定义的方法`.isInYear()`和在`DoubleColumn`上定义的`.isGreaterThan()`。我们用一种类似查询的语言和`.and()`方法将它们结合起来。Tablesaw 提供了许多这样的内置助手方法。因此，对于简单的任务，我们很少需要自己构建一个自定义选择。对于复杂的任务，我们使用`.and(), .andNot(), or()`和其他[列过滤器](https://web.archive.org/web/20221125192801/https://jtablesaw.github.io/tablesaw/userguide/filters#:~:text=Current%20list%20of%20provided%20column%20filters)来组合它们。

或者，我们通过创建一个`Predicate`并将其传递给每一列上可用的`.eval()`方法来编写自定义过滤器。这个方法返回一个我们用来过滤表格或列的`Selection`对象。

## 7.汇总数据

处理完数据后，我们希望从中提取一些真知灼见。我们使用。`summarize()`汇总数据的方法来了解它。例如，从鳄梨数据集中，我们提取平均价格的最小值、最大值、平均值和标准差:

```
Table summary = table.summarize("AveragePrice", max, min, mean, stdDev).by("year");
System.out.println(summary.print());
```

首先，我们将想要聚合的列名和列表`AggregateFunction`传递给`.summarize()`方法。接下来，我们使用。`by()`法。最后，我们在标准输出中打印结果:

```
 avocado.csv summary                                               
 year  |  Mean [AveragePrice]  |  Max [AveragePrice]  |  Min [AveragePrice]  |  Std. Deviation [AveragePrice]  |
----------------------------------------------------------------------------------------------------------------
 2015  |    1.375590382902939  |                2.79  |                0.49  |            0.37559477067238917  |
 2016  |   1.3386396011396013  |                3.25  |                0.51  |            0.39370799476072077  |
 2017  |   1.5151275777700104  |                3.17  |                0.44  |             0.4329056466203253  |
 2018  |   1.3475308641975308  |                 2.3  |                0.56  |             0.3058577391135024  | 
```

Tablesaw 为最常见的操作提供了 [`AggregateFunction`](https://web.archive.org/web/20221125192801/https://www.javadoc.io/static/tech.tablesaw/tablesaw-core/0.43.1/tech/tablesaw/aggregate/AggregateFunctions.html) 。或者，我们可以实现一个定制的`AggregateFunction`对象，但是由于这超出了本文的范围，我们将保持事情简单。

## 8.保存数据

到目前为止，我们一直将数据打印到标准输出。在运行中验证我们的结果时，打印到控制台是很好的，但是我们需要将数据保存到文件中，以便其他人可以重用这些结果。所以，让我们直接在表上使用`.write()`方法:

```
summary.write().csv("summary.csv");
```

上面，我们使用了`.csv()`方法将数据保存为 CSV 格式。目前， **Tablesaw 只支持 CSV 格式和固定宽度格式**，类似于`.print()`方法在控制台上显示的内容。此外，我们使用`CsvWriterOptions`定制数据的 CSV 输出。

## 9.结论

在本文中，我们探讨了如何使用 Tablesaw 库处理表格数据。

首先，我们解释了如何导入数据。然后，我们描述了数据的内部表示以及如何使用它。接下来，我们探索了修改导入的表的结构，并创建过滤器，以便在聚合之前提取必要的数据。最后，我们将其保存到一个 CSV 文件中。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221125192801/https://github.com/eugenp/tutorials/tree/master/tablesaw)