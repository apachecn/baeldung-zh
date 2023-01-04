# 用 Java 将文件读入映射

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-read-file-into-map>

## 1.概观

我们知道在 Java 中,`[Map](/web/20221208143917/https://www.baeldung.com/tag/java-map/)`保存键值对。有时，我们可能希望加载一个文本文件的内容，并将其转换成 Java `Map`。

在这个快速教程中，让我们来探索如何实现它。

## 2.问题简介

由于`Map`存储键值条目，如果我们想要将文件的内容导入到 Java `Map `对象中，文件应该遵循特定的格式。

一个示例文件可以快速解释这一点:

```
$ cat theLordOfRings.txt
title:The Lord of the Rings: The Return of the King
director:Peter Jackson
actor:Sean Astin
actor:Ian McKellen
Gandalf and Aragorn lead the World of Men against Sauron's
army to draw his gaze from Frodo and Sam as they approach Mount Doom with the One Ring.
```

正如我们在`theLordOfRings.txt`文件中看到的，如果我们将冒号字符视为分隔符，大多数行都遵循“`KEY:VALUE`”的模式，比如“`director:Peter Jackson`”。

因此，我们可以读取每一行，解析键和值，并将它们放在一个`Map`对象中。

但是，我们需要处理一些特殊情况:

*   包含分隔符–Value 的值不应被截断。比如第一行“`title:The Lord of the Rings: The Return …`”
*   重复键——三种策略:覆盖现有的键，丢弃后者，根据需要将值聚合到一个`List`中。例如，我们在文件中有两个“`actor`”键。
*   不符合“`KEY:VALUE`”模式的行–应跳过该行。例如，查看文件中的最后两行。

接下来，让我们读取这个文件，并将它存储在一个 Java `Map`对象中。

## 3.`DupKeyOption`枚举

正如我们所讨论的，对于重复键的情况，我们有三种选择:覆盖、丢弃和聚集。

此外，如果我们使用覆盖或丢弃选项，我们将返回类型为`Map<String, String>`的`Map`。然而，如果我们想要聚合重复键的值，我们将得到结果为`Map<String, List<String>>`。

所以，让我们首先来探讨覆盖和丢弃场景。最后，我们将在一个独立的部分中讨论聚合选项。

为了使我们的解决方案更加灵活，让我们创建一个 [`enum`](/web/20221208143917/https://www.baeldung.com/a-guide-to-java-enums) 类，这样我们就可以将选项作为参数传递给我们的解决方法:

```
enum DupKeyOption {
    OVERWRITE, DISCARD
} 
```

## 4.使用`BufferedReader`和`FileReader`类

**我们可以结合 [`BufferedReader`](/web/20221208143917/https://www.baeldung.com/java-buffered-reader) 和`[FileReader](/web/20221208143917/https://www.baeldung.com/java-filereader)`从文件中逐行读取内容**。

### 4.1.创建`byBufferedReader`方法

让我们基于`BufferedReader`和`FileReader`创建一个方法:

```
public static Map<String, String> byBufferedReader(String filePath, DupKeyOption dupKeyOption) {
    HashMap<String, String> map = new HashMap<>();
    String line;
    try (BufferedReader reader = new BufferedReader(new FileReader(filePath))) {
        while ((line = reader.readLine()) != null) {
            String[] keyValuePair = line.split(":", 2);
            if (keyValuePair.length > 1) {
                String key = keyValuePair[0];
                String value = keyValuePair[1];
                if (DupKeyOption.OVERWRITE == dupKeyOption) {
                    map.put(key, value);
                } else if (DupKeyOption.DISCARD == dupKeyOption) {
                    map.putIfAbsent(key, value);
                }
            } else {
                System.out.println("No Key:Value found in line, ignoring: " + line);
            }
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
    return map;
} 
```

`byBufferedReader`方法接受两个参数:输入文件路径和决定如何处理具有重复键的条目的`dupKeyOption`对象。

如上面的代码所示，我们已经定义了一个`BufferedReader`对象来从给定的输入文件中读取行。然后，我们在一个`while`循环中解析和处理每一行。让我们浏览并理解它是如何工作的:

*   我们创建一个`BufferedReader`对象，**使用 [`try-with-resources`](/web/20221208143917/https://www.baeldung.com/java-try-with-resources) 来确保`reader`对象自动关闭**
*   我们使用带有 limit 参数的`[split](/web/20221208143917/https://www.baeldung.com/string/split)`方法来保持值部分不变，如果它包含冒号的话
*   然后，`if`检查过滤掉不匹配“`KEY:VALUE`”模式的行
*   在有重复键的情况下，如果我们想采取“覆盖”策略，我们可以简单地调用`map.put(key, value)`
*   否则，**调用 [`putIfAbsent`](/web/20221208143917/https://www.baeldung.com/java-map-computeifabsent) 方法允许我们忽略后面出现的带有重复键的条目**

接下来，让我们测试该方法是否如预期的那样工作。

### 4.2.测试解决方案

在我们编写相应的测试方法之前，让我们[初始化两个包含预期条目的 map 对象](/web/20221208143917/https://www.baeldung.com/java-initialize-hashmap):

```
private static final Map<String, String> EXPECTED_MAP_DISCARD = Stream.of(new String[][]{
    {"title", "The Lord of the Rings: The Return of the King"},
    {"director", "Peter Jackson"},
    {"actor", "Sean Astin"}
  }).collect(Collectors.toMap(data -> data[0], data -> data[1]));

private static final Map<String, String> EXPECTED_MAP_OVERWRITE = Stream.of(new String[][]{
...
    {"actor", "Ian McKellen"}
  }).collect(Collectors.toMap(data -> data[0], data -> data[1]));
```

正如我们所看到的，我们已经初始化了两个`Map` 对象来帮助测试断言。一个是针对我们丢弃重复键的情况，另一个是针对我们覆盖它们的情况。

接下来，让我们测试我们的方法，看看我们是否能得到预期的`Map`对象:

```
@Test
public void givenInputFile_whenInvokeByBufferedReader_shouldGetExpectedMap() {
    Map<String, String> mapOverwrite = FileToHashMap.byBufferedReader(filePath, FileToHashMap.DupKeyOption.OVERWRITE);
    assertThat(mapOverwrite).isEqualTo(EXPECTED_MAP_OVERWRITE);

    Map<String, String> mapDiscard = FileToHashMap.byBufferedReader(filePath, FileToHashMap.DupKeyOption.DISCARD);
    assertThat(mapDiscard).isEqualTo(EXPECTED_MAP_DISCARD);
} 
```

如果我们试一下，测试就通过了。所以，我们已经解决了这个问题。

## 5.使用 Java `Stream`

[`Stream`](/web/20221208143917/https://www.baeldung.com/java-streams) 自爪哇八世以来一直存在。同样，**`Files.lines`方法可以方便地返回一个包含文件**中所有行的`Stream`对象。

现在，让我们创建一个使用`Stream`来解决问题的方法:

```
public static Map<String, String> byStream(String filePath, DupKeyOption dupKeyOption) {
    Map<String, String> map = new HashMap<>();
    try (Stream<String> lines = Files.lines(Paths.get(filePath))) {
        lines.filter(line -> line.contains(":"))
            .forEach(line -> {
                String[] keyValuePair = line.split(":", 2);
                String key = keyValuePair[0];
                String value = keyValuePair[1];
                if (DupKeyOption.OVERWRITE == dupKeyOption) {
                    map.put(key, value);
                } else if (DupKeyOption.DISCARD == dupKeyOption) {
                    map.putIfAbsent(key, value);
                }
            });
    } catch (IOException e) {
        e.printStackTrace();
    }
    return map;
} 
```

如上面的代码所示，主要逻辑与我们的`byBufferedReader`方法非常相似。让我们快速通过:

*   我们仍然在对`Stream`对象使用 try-with-resources，因为`Stream`对象包含对打开文件的引用。**我们应该通过关闭流来关闭文件。**
*   `filter`方法跳过所有不遵循“`KEY:VALUE`”模式的行。
*   `forEach`方法与`byBufferedReader`解决方案中的`while`块做得非常相似。

最后，让我们测试一下`byStream`解决方案:

```
@Test
public void givenInputFile_whenInvokeByStream_shouldGetExpectedMap() {
    Map<String, String> mapOverwrite = FileToHashMap.byStream(filePath, FileToHashMap.DupKeyOption.OVERWRITE);
    assertThat(mapOverwrite).isEqualTo(EXPECTED_MAP_OVERWRITE);

    Map<String, String> mapDiscard = FileToHashMap.byStream(filePath, FileToHashMap.DupKeyOption.DISCARD);
    assertThat(mapDiscard).isEqualTo(EXPECTED_MAP_DISCARD);
} 
```

当我们执行测试时，它也通过了。

## 6.按键聚合值

到目前为止，我们已经看到了覆盖和丢弃场景的解决方案。但是，正如我们已经讨论过的，如果需要的话，我们也可以通过键来聚合值。因此，最终，我们将拥有一个类型为`Map<String, List<String>>`的`Map`对象。现在，让我们构建一个方法来实现这个需求:

```
public static Map<String, List<String>> aggregateByKeys(String filePath) {
    Map<String, List<String>> map = new HashMap<>();
    try (Stream<String> lines = Files.lines(Paths.get(filePath))) {
        lines.filter(line -> line.contains(":"))
          .forEach(line -> {
              String[] keyValuePair = line.split(":", 2);
              String key = keyValuePair[0];
              String value = keyValuePair[1];
              if (map.containsKey(key)) {
                  map.get(key).add(value);
              } else {
                  map.put(key, Stream.of(value).collect(Collectors.toList()));
              }
          });
    } catch (IOException e) {
        e.printStackTrace();
    }
    return map;
} 
```

我们使用了`Stream`方法来读取输入文件中的所有行。实现非常简单。一旦我们解析了输入行中的键和值，我们就检查这个键是否已经存在于结果`map`对象中。如果它确实存在，我们将把这个值追加到现有的列表中。否则，我们[将包含当前值的`List`](/web/20221208143917/https://www.baeldung.com/java-init-list-one-line#create-from-a-stream-java-8) 初始化为单个元素:`Stream.of(value).collect(Collectors.toList()). `

值得一提的是，我们不应该使用`Collections.singletonList(value)` 或`List.of(value)`来初始化`List`。这是因为**的`[Collections.singletonList](https://web.archive.org/web/20221208143917/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Collections.html#singletonList(T))`和 [`List.of`](https://web.archive.org/web/20221208143917/https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/util/List.html#of(E)) (Java 9+)方法都返回一个不可变的`L` `**ist**.`** 也就是说，如果同一个键再次出现，我们不能将值追加到列表中。

接下来，让我们测试一下我们的方法，看看它是否有效。像往常一样，我们首先创建预期的结果:

```
private static final Map<String, List<String>> EXPECTED_MAP_AGGREGATE = Stream.of(new String[][]{
      {"title", "The Lord of the Rings: The Return of the King"},
      {"director", "Peter Jackson"},
      {"actor", "Sean Astin", "Ian McKellen"}
  }).collect(Collectors.toMap(arr -> arr[0], arr -> Arrays.asList(Arrays.copyOfRange(arr, 1, arr.length))));
```

然后，测试方法本身非常简单:

```
@Test
public void givenInputFile_whenInvokeAggregateByKeys_shouldGetExpectedMap() {
    Map<String, List<String>> mapAgg = FileToHashMap.aggregateByKeys(filePath);
    assertThat(mapAgg).isEqualTo(EXPECTED_MAP_AGGREGATE);
}
```

如果我们试一试，测试就会通过。这意味着我们的解决方案如预期一样有效。

## 7.结论

在本文中，我们学习了两种从文本文件中读取内容并将其保存在 Java `Map`对象中的方法:使用`BufferedReader`类和使用`Stream`。

此外，我们已经讨论了实现三种策略来处理重复键:覆盖、丢弃和聚集。

和往常一样，GitHub 上有完整版本的代码[。](https://web.archive.org/web/20221208143917/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-4)