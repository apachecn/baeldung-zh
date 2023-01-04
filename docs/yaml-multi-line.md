# 打破多行 YAML 字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/yaml-multi-line>

## 1.概观

在这篇文章中，我们将学习如何在多行中断开 YAML 字符串。

为了解析和测试我们的 YAML 文件，我们将利用 [SnakeYAML 库](/web/20220801194623/https://www.baeldung.com/java-snake-yaml)。

## 2.多行字符串

在我们开始之前，让我们创建一个方法，简单地将一个 YAML 键从一个文件读入一个`String`:

```java
String parseYamlKey(String fileName, String key) {
    InputStream inputStream = this.getClass()
      .getClassLoader()
      .getResourceAsStream(fileName);
    Map<String, String> parsed = yaml.load(inputStream);
    return parsed.get(key);
}
```

在接下来的小节中，我们将会看到一些将字符串拆分成多行的策略。

我们还将学习 YAML 如何处理由块的开头和结尾的空行表示的开头和结尾的换行符。

## 3.文字风格

文本运算符由管道符号(“|”)表示。它保留了我们的换行符，但是将字符串末尾的空行减少到一个换行符。

让我们来看看 YAML 的档案`literal.yaml`:

```java
key: |
  Line1
  Line2
  Line3
```

我们可以看到我们的换行符保留了下来:

```java
String key = parseYamlKey("literal.yaml", "key");
assertEquals("Line1\nLine2\nLine3", key);
```

接下来我们来看看`literal2.yaml`，里面有**一些开头和结尾的换行符:**

```java
key: |

  Line1

  Line2

  Line3

...
```

我们可以看到，除了结尾换行符，每个换行符都存在，结尾换行符减少为一个:

```java
String key = parseYamlKey("literal2.yaml", "key");
assertEquals("\n\nLine1\n\nLine2\n\nLine3\n", key);
```

接下来，我们将讨论块 chomping 以及它如何让我们更好地控制开始和结束换行符。

我们可以通过使用**两个 chop 方法来改变默认行为:keep 和 strip** 。

### 3.1.保持

Keep 由“+”表示，正如我们在`literal_keep.yaml`中看到的:

```java
key: |+
  Line1
  Line2
  Line3

...
```

通过覆盖默认行为，我们可以看到**每个结束空行都被保留**:

```java
String key = parseYamlKey("literal_keep.yaml", "key");
assertEquals("Line1\nLine2\nLine3\n\n", key);
```

### 3.2.剥夺

正如我们在`literal_strip.yaml`中看到的，该条用“-”表示:

```java
key: |-
  Line1
  Line2
  Line3

...
```

正如我们可能已经预料到的，这会导致**删除每个结束空行**:

```java
String key = parseYamlKey("literal_strip.yaml", "key");
assertEquals("Line1\nLine2\nLine3", key);
```

## 4.折叠风格

折叠运算符由“>”表示，正如我们在`folded.yaml`中看到的:

```java
key: >
  Line1
  Line2
  Line3
```

**默认情况下，对于连续的非空行，换行符由空格字符替换:**

```java
String key = parseYamlKey("folded.yaml", "key");
assertEquals("Line1 Line2 Line3", key);
```

让我们看一个类似的文件`folded2.yaml`，它有几个结束空行:

```java
key: >
  Line1
  Line2

  Line3

...
```

我们可以看到**空行被保留，但是结束换行符也被减少到一个**:

```java
String key = parseYamlKey("folded2.yaml", "key");
assertEquals("Line1 Line2\n\nLine3\n", key);
```

我们应该记住，**块咬影响折叠风格，就像它影响字面风格**一样。

## 5.引用

让我们快速地看一下在双引号和单引号的帮助下拆分字符串。

### 5.1.双引号

有了双引号，我们可以通过使用“`\n`”轻松地创建多行字符串:

```java
key: "Line1\nLine2\nLine3"
```

```java
String key = parseYamlKey("plain_double_quotes.yaml", "key");
assertEquals("Line1\nLine2\nLine3", key);
```

### 5.2.单引号

另一方面，单引号将"`\n`"视为字符串的一部分，因此插入换行符的唯一方法是使用空行:

```java
key: 'Line1\nLine2

  Line3'
```

```java
String key = parseYamlKey("plain_single_quotes.yaml", "key");
assertEquals("Line1\\nLine2\nLine3", key);
```

## 6.结论

在这个快速教程中，我们通过快速和实用的例子查看了在多行上断开 YAML 弦的多种方法。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220801194623/https://github.com/eugenp/tutorials/tree/master/libraries-data-io)