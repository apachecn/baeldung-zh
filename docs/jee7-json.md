# Java EE 7 中的 JSON 处理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jee7-json>

## 1。概述

本文将向您展示如何只使用核心 Java EE 来处理 JSON，而不使用像 Jersey 或 Jackson 这样的第三方依赖。我们将使用的几乎所有东西都由 [javax.json](https://web.archive.org/web/20220813065846/https://docs.oracle.com/javaee/7/api/javax/json/package-summary.html) 包提供。

## 2。给 JSON `String`写一个对象

将 Java 对象转换成 JSON `String`非常容易。假设我们有一个简单的`Person`类:

```
public class Person {
    private String firstName;
    private String lastName;
    private Date birthdate;

    // getters and setters
}
```

要将该类的实例转换成 JSON `String`，首先我们需要创建一个`[JsonObjectBuilder](https://web.archive.org/web/20220813065846/https://docs.oracle.com/javaee/7/api/javax/json/JsonObjectBuilder.html)`的实例，并使用`add()`方法添加属性/值对:

```
JsonObjectBuilder objectBuilder = Json.createObjectBuilder()
  .add("firstName", person.getFirstName())
  .add("lastName", person.getLastName())
  .add("birthdate", new SimpleDateFormat("DD/MM/YYYY")
  .format(person.getBirthdate()));
```

注意，`add()`方法有几个重载版本。它可以接收大多数基本类型(以及装箱的对象)作为它的第二个参数。

一旦我们完成了属性的设置，我们只需要将对象写入一个`String`:

```
JsonObject jsonObject = objectBuilder.build();

String jsonString;
try(Writer writer = new StringWriter()) {
    Json.createWriter(writer).write(jsonObject);
    jsonString = writer.toString();
}
```

就是这样！生成的`String`将如下所示:

```
{"firstName":"Michael","lastName":"Scott","birthdate":"06/15/1978"}
```

### 2.1。使用`JsonArrayBuilder`构建数组

现在，为了给我们的示例增加一点复杂性，让我们假设对`Person` 类进行了修改，添加了一个名为`emails` 的新属性，该属性将包含一个电子邮件地址列表:

```
public class Person {
    private String firstName;
    private String lastName;
    private Date birthdate;
    private List<String> emails;

    // getters and setters

}
```

要将列表中的所有值添加到`JsonObjectBuilder`中，我们需要`[JsonArrayBuilder](https://web.archive.org/web/20220813065846/https://docs.oracle.com/javaee/7/api/javax/json/JsonArrayBuilder.html)`的帮助:

```
JsonArrayBuilder arrayBuilder = Json.createArrayBuilder();

for(String email : person.getEmails()) {
    arrayBuilder.add(email);
}

objectBuilder.add("emails", arrayBuilder);
```

请注意，我们使用了另一个重载版本的`add()`方法，该方法将一个`JsonArrayBuilder`对象作为其第二个参数。

因此，让我们来看看为一个有两个电子邮件地址的`Person`对象生成的字符串:

```
{"firstName":"Michael","lastName":"Scott","birthdate":"06/15/1978",
 "emails":["[[email protected]](/web/20220813065846/https://www.baeldung.com/cdn-cgi/l/email-protection)","[[email protected]](/web/20220813065846/https://www.baeldung.com/cdn-cgi/l/email-protection)"]}
```

### 2.2。用`PRETTY_PRINTING` 格式化输出

这样我们就成功地将一个 Java 对象转换成了一个有效的 JSON `String`。现在，在进入下一节之前，让我们添加一些简单的格式，使输出更像“JSON ”,更容易阅读。

在前面的例子中，我们使用简单的 [`Json`创建了一个`JsonWriter` 。`createWriter()`](https://web.archive.org/web/20220813065846/https://docs.oracle.com/javaee/7/api/javax/json/Json.html#createWriter-java.io.Writer-) 静法。为了更好地控制生成的`String`，我们将利用 Java 7 的`[JsonWriterFactory](https://web.archive.org/web/20220813065846/https://docs.oracle.com/javaee/7/api/javax/json/JsonWriterFactory.html)`功能创建一个具有特定配置的写入器。

```
Map<String, Boolean> config = new HashMap<>();

config.put(JsonGenerator.PRETTY_PRINTING, true);

JsonWriterFactory writerFactory = Json.createWriterFactory(config);

String jsonString;

try(Writer writer = new StringWriter()) {
    writerFactory.createWriter(writer).write(jsonObject);
    jsonString = writer.toString();
}
```

代码可能看起来有点冗长，但它实际上没做多少事情。

首先，它创建了一个`JsonWriterFactory`实例，将一个配置图传递给它的构造函数。该映射只包含一个将 PRETTY_PRINTING 属性设置为 true 的条目。然后，我们使用该工厂实例来创建一个编写器，而不是使用`Json.createWriter()`。

新的输出将包含 JSON `String`特有的换行符和表格:

```
{
    "firstName":"Michael",
    "lastName":"Scott",
    "birthdate":"06/15/1978",
    "emails":[
        "[[email protected]](/web/20220813065846/https://www.baeldung.com/cdn-cgi/l/email-protection)",
        "[[email protected]](/web/20220813065846/https://www.baeldung.com/cdn-cgi/l/email-protection)"
    ]
}
```

## 3。从一个`String` 构建一个 Java `Object`

现在让我们做相反的操作:将一个 JSON `String`转换成一个 Java 对象。

转换过程的主要部分围绕着`JsonObject`。要创建这个类的实例，使用静态方法`[Json.createReader()](https://web.archive.org/web/20220813065846/https://docs.oracle.com/javaee/7/api/javax/json/Json.html#createReader-java.io.InputStream-)`后跟`[readObject()](https://web.archive.org/web/20220813065846/https://docs.oracle.com/javaee/7/api/javax/json/JsonReader.html#readObject--)`:

```
JsonReader reader = Json.createReader(new StringReader(jsonString));

JsonObject jsonObject = reader.readObject();
```

`createReader()`方法将一个`InputStream`作为参数。在这个例子中，我们使用了一个`StringReader,`，因为我们的 JSON 包含在一个`String`对象中，但是同样的方法也可以用来从文件中读取内容，例如，使用`[FileInputStream](https://web.archive.org/web/20220813065846/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/FileInputStream.html)`。

手头有了一个`JsonObject`的实例，我们可以使用`getString()` 方法读取属性，并将获得的值赋给我们的`Person` 类的一个新创建的实例:

```
Person person = new Person();

person.setFirstName(jsonObject.getString("firstName"));
person.setLastName(jsonObject.getString("lastName"));
person.setBirthdate(dateFormat.parse(jsonObject.getString("birthdate")));
```

### 3.1。使用`JsonArray`获得`List`值

我们需要使用一个名为`JsonArray`的特殊类从`JsonObject`中提取列表值:

```
JsonArray emailsJson = jsonObject.getJsonArray("emails");

List<String> emails = new ArrayList<>();

for (JsonString j : emailsJson.getValuesAs(JsonString.class)) {
    emails.add(j.getString());
}

person.setEmails(emails);
```

就是这样！我们已经从 Json `String`创建了一个完整的`Person` 实例。

## 4。查询值

现在，让我们假设我们对 JSON `String`中的一段非常具体的数据感兴趣。

考虑下面代表宠物店客户的 JSON。假设出于某种原因，您需要从宠物列表中获取第三只宠物的名字:

```
{
    "ownerName": "Robert",
    "pets": [{
        "name": "Kitty",
        "type": "cat"
    }, {
        "name": "Rex",
        "type": "dog"
    }, {
        "name": "Jake",
        "type": "dog"
    }]
}
```

仅仅为了获得一个值而将整个文本转换成一个 Java 对象不是很有效。因此，让我们检查几个查询 JSON `Strings`的策略，而不必经历整个转换过程。

### 4.1。使用对象模型 API 查询

查询 JSON 结构中已知位置的属性值非常简单。我们可以使用前面例子中使用的同一个类的实例`JsonObject,`:

```
JsonReader reader = Json.createReader(new StringReader(jsonString));

JsonObject jsonObject = reader.readObject();

String searchResult = jsonObject
  .getJsonArray("pets")
  .getJsonObject(2)
  .getString("name"); 
```

这里的要点是使用正确的`get*()`方法序列来浏览`jsonObject`属性。

在这个例子中，我们首先使用`getJsonArray()`获得对“pets”列表的引用，这将返回一个包含 3 条记录的列表。然后，我们使用`getJsonObject()`方法，该方法将一个索引作为参数，返回另一个表示列表中第三项的`JsonObject` 。最后，我们使用`getString()` 来获得我们正在寻找的字符串值。

### 4.2。使用流式 API 查询

在 JSON `String`上执行精确查询的另一种方式是使用流式 API，它将`[JsonParser](https://web.archive.org/web/20220813065846/https://docs.oracle.com/javaee/7/api/javax/json/stream/JsonParser.html)`作为其主类。

`JsonParser` 提供对 JS 的极快的、只读的、向前的访问，缺点是比对象模型稍微复杂一些:

```
JsonParser jsonParser = Json.createParser(new StringReader(jsonString));

int count = 0;
String result = null;

while(jsonParser.hasNext()) {
    Event e = jsonParser.next();

    if (e == Event.KEY_NAME) {
        if(jsonParser.getString().equals("name")) {
            jsonParser.next();

            if(++count == 3) {
                result = jsonParser.getString();
                break;
            }
        }   
    }
}
```

这个例子给出了与上一个例子相同的结果。它从`pets`列表中的第三个宠物返回`name` 。

一旦使用`[Json.createParser()](https://web.archive.org/web/20220813065846/https://docs.oracle.com/javaee/7/api/javax/json/Json.html#createParser-java.io.InputStream-)`创建了`JsonParser` ，我们就需要使用一个迭代器(因此有了`JsonParser`的“向前访问”特性)来导航 JSON 令牌，直到我们到达我们正在寻找的属性。

每次我们遍历迭代器时，我们都会移动到 JSON 数据的下一个标记。因此，我们必须小心检查当前令牌是否具有预期的类型。这是通过检查由`next()` 调用返回的`[Event](https://web.archive.org/web/20220813065846/https://docs.oracle.com/javaee/7/api/javax/json/stream/JsonParser.Event.html)`来完成的。

有许多不同类型的代币。在这个例子中，我们感兴趣的是`[KEY_NAME](https://web.archive.org/web/20220813065846/https://docs.oracle.com/javaee/7/api/javax/json/stream/JsonParser.Event.html#KEY_NAME)`类型，它代表一个属性的名称(例如“ownerName”、“pets”、“name”、“type”)。一旦我们第三次遍历了值为“name”的`KEY_NAME`标记，我们知道下一个标记将包含一个字符串值，表示列表中第三只宠物的名字。

这肯定比使用对象模型 API 更难，尤其是对于更复杂的 JSON 结构。两者之间的选择总是取决于您将要处理的特定场景。

## 5。结论

我们已经用几个简单的例子介绍了 Java EE JSON 处理 API 的许多基础知识。要了解关于 JSON 处理的其他很酷的东西，请查看我们的杰克逊文章系列。

在我们的 [GitHub 库](https://web.archive.org/web/20220813065846/https://github.com/eugenp/tutorials/tree/master/jee-7)中检查本文中使用的类的源代码，以及一些单元测试。