# YAML 和 JSON 的区别

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/yaml-json-differeneces>

## 1.概观

在这篇简短的文章中，我们将通过简单实用的例子来看看 YAML 和 JSON 之间的区别。

## 2.格式

为了有一个更好的形象，让我们先来看看一个简单 POJO 的 JSON 和 YAML 表示:

```java
class Person {
    String name;
    Integer age;
    List<String> hobbies;
    Person manager;
}
```

**首先，我们来看看它的 JSON 表示:**

```java
{
    "name":"John Smith",
    "age":26,
    "hobbies":[
        "sports",
        "cooking"
    ],
    "manager":{
        "name":"Jon Doe",
        "age":45,
        "hobbies":[
            "fishing"
        ],
        "manager":null
    }
}
```

JSON 语法有点麻烦，因为它使用特殊的语法，如花括号`{}`和方括号`[]`来表示对象和数组。

接下来，让我们看看同样的结构在 YAML 会是什么样子:

```java
name: John Smith
age: 26
hobbies:
  - sports
  - cooking
manager:
  name: Jon Doe
  age: 45
  hobbies:
    - fishing
  manager:
```

YAML 的语法看起来更友好一些，因为它用空格来表示对象之间的关系，用'`–`'来表示数组元素。

我们可以看到，尽管两者都易于阅读，但 YAML 更易于阅读。

对 YAML 来说，另一个好处是表示相同信息需要的行数——YAML 只需要 11 行，而 JSON 需要 16 行。

## 3.大小

在上一节中我们已经看到，YAML 用的行数比 JSON 少，但是这是否意味着它占用的空间更少呢？

让我们想象一个深度嵌套的结构，其中一个父节点和五个子节点表示为 JSON:

```java
{
    "child":{
        "child":{
            "child":{
                "child":{
                    "child":{
                        "child":{
                            "child":null
                        }
                    }
                }
            }
        }
    }
}
```

同样的建筑在 YAML 看起来会很相似:

```java
child:
  child:
    child:
      child:
        child:
          child:
            child:
```

乍一看，JSON 似乎占用了更多的空间，但是，实际上，JSON 规范并不关心空格或换行符，它可以简化为:

```java
{"child":{"child":{"child":{"child":{"child":{"child":{"child":null}}}}}}}
```

我们可以看到，第二种格式要短得多，它只占用 74 个字节，而 YAML 格式占用 97 个字节。

## 4.YAML 特色

除了 JSON 提供的基本特性之外，YAML 还提供了额外的功能，我们接下来会看到。

### 4.1.评论

**YAML 允许使用`#`进行注释，这是一个在处理 JSON 文件时经常需要的特性:**

```java
# This is a simple comment
name: John
```

### 4.2.多行字符串

JSON 中缺少的另一个特性是 YAML 的[多行字符串](/web/20220628055216/https://www.baeldung.com/yaml-multi-line):

```java
website: |
  line1
  line2
  line3
```

### 4.3.别名和锚点

我们可以很容易地使用`&`为一个特定的项目分配一个别名，并使用`*`锚定(引用)它:

```java
httpPort: 80
httpsPort: &httpsPort; 443
defaultPort: *httpsPort
```

## 5.表演

由于 JSON 规范的简单性质，它在解析/序列化数据方面的性能比 YAML 好得多。

**我们将使用 [JMH](/web/20220628055216/https://www.baeldung.com/java-microbenchmark-harness) 实现一个简单的基准来比较 YAML 和 JSON 的解析速度。**

对于 YAML 基准，我们将使用众所周知的 [`snake-yaml`](/web/20220628055216/https://www.baeldung.com/java-snake-yaml) 库，对于我们的 JSON 基准，我们将使用 [`org-json`](/web/20220628055216/https://www.baeldung.com/java-org-json) :

```java
@BenchmarkMode(Mode.Throughput)
@OutputTimeUnit(TimeUnit.SECONDS)
@Measurement(batchSize = 10_000, iterations = 5)
@Warmup(batchSize = 10_000, iterations = 5)
@State(Scope.Thread)
class Bench {

    static void main(String[] args) {
        org.openjdk.jmh.Main.main(args);
    }

    @State(Scope.Thread)
    static class YamlState {
        public Yaml yaml = new Yaml();
    }

    @Benchmark
    Object benchmarkYaml(YamlState yamlState) {
        return yamlState.yaml.load("foo: bar");
    }

    @Benchmark
    Object benchmarkJson(Blackhole blackhole) {
        return new JSONObject("{\"foo\": \"bar\"}");
    }
}
```

正如我们所料，JSON 是赢家，速度快了大约 30 倍:

```java
Benchmark             Mode  Cnt    Score   Error  Units
Main2.benchmarkJson  thrpt   50  644.085 ± 9.962  ops/s
Main2.benchmarkYaml  thrpt   50   20.351 ± 0.312  ops/s
```

## 6.图书馆可用性

JavaScript 是 web 的标准，这意味着几乎不可能找到不完全支持 JSON 的语言。

另一方面，YAML 得到了广泛的支持，但它不是一个标准。这意味着大多数流行的编程语言都有库，但由于其复杂性，它们可能无法完全实现规范。

## 7.我应该选择什么？

这可能是一个很难回答的问题，而且在很多情况下是一个主观的问题。

如果我们需要向其他前端或后端应用程序公开一组 REST APIs，我们可能应该使用 JSON，因为它是事实上的行业标准。

如果我们需要创建一个经常被人读取/更新的配置文件，YAML 可能是一个不错的选择。

当然，也可能有 YAML 和 JSON 都适合的用例，这只是个人喜好的问题。

## 8.结论

在这篇简短的文章中，我们了解了 YAML 和 JSON 之间的主要区别，以及在做出明智的选择时应该考虑哪些方面。