# 将 BufferedReader 转换为 JSONObject

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-bufferedreader-to-jsonobject>

## 1.概观

在这个快速教程中，我们将展示**如何使用两种不同的方法**将一个 [`BufferedReader`](/web/20220630134032/https://www.baeldung.com/java-buffered-reader) 转换成一个 [`JSONObject`](/web/20220630134032/https://www.baeldung.com/java-org-json#jsonobject) **。**

## 2.属国

在我们开始之前，我们需要**将 [`org.json`](https://web.archive.org/web/20220630134032/https://search.maven.org/search?q=g:org.json%20AND%20a:json&core=gav) 依赖**添加到我们的`pom.xml`中:

```java
<dependency>
    <groupId>org.json</groupId>
    <artifactId>json</artifactId>
    <version>20200518</version>
</dependency>
```

## 3.`JSONTokener`

`org.json`库的最新版本**带有一个`JSONTokener`构造函数。它直接接受一个`Reader`** 作为参数。

所以，让我们用下面的方法把一个`BufferedReader`转换成一个`JSONObject`:

```java
@Test
public void givenValidJson_whenUsingBufferedReader_thenJSONTokenerConverts() {
    byte[] b = "{ \"name\" : \"John\", \"age\" : 18 }".getBytes(StandardCharsets.UTF_8);
    InputStream is = new ByteArrayInputStream(b);
    BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(is));
    JSONTokener tokener = new JSONTokener(bufferedReader);
    JSONObject json = new JSONObject(tokener);

    assertNotNull(json);
    assertEquals("John", json.get("name"));
    assertEquals(18, json.get("age"));
} 
```

## 4.首先转换为`String`

现在，让我们看看**另一种获得`JSONObject` 的方法，首先将 `BufferedReader` 转换为 `String`** 。

在旧版本的`org.json`中工作时可以使用这种方法:

```java
@Test
public void givenValidJson_whenUsingString_thenJSONObjectConverts()
  throws IOException {
    // ... retrieve BufferedReader<br />
    StringBuilder sb = new StringBuilder();
    String line;
    while ((line = bufferedReader.readLine()) != null) {
        sb.append(line);
    }
    JSONObject json = new JSONObject(sb.toString());

    // ... same checks as before
} 
```

在这里，我们用**将一个`BufferedReader`转换成一个`String`，然后我们用`JSONObject`构造函数将一个`String`转换成一个`JSONObject`** 。

## 5.结论

在本文中，我们已经通过简单的例子看到了将`BufferedReader`转换为`JSONObject`的两种不同方式。毫无疑问，`org.json`的最新版本提供了一种简洁明了的方式，用更少的代码将`BufferedReader`转换成`JSONObject`。

和往常一样，这个例子的完整源代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220630134032/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-io-conversions-2)