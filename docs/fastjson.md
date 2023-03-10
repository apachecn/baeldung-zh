# FastJson 指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/fastjson>

## 1。概述

[**FastJson**](https://web.archive.org/web/20220815040838/https://github.com/alibaba/fastjson) 是一个轻量级的 Java 库，用于有效地将 Json 字符串转换成 Java 对象，反之亦然。

在本文中，我们将深入研究 FastJson 库的几个具体和实际的应用程序。

## 2。Maven 配置

为了开始使用 FastJson，我们首先需要将它添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.13</version>
</dependency> 
```

快速说明一下—[这是 Maven Central 上库的最新版本](https://web.archive.org/web/20220815040838/https://search.maven.org/classic/#search%7Cgav%7C1%7Cg%3A%22com.alibaba%22%20AND%20a%3A%22fastjson%22)。

## 3。将 Java 对象转换成 JSON 格式

让我们定义下面的`Person` Java bean:

```java
public class Person {

    @JSONField(name = "AGE")
    private int age;

    @JSONField(name = "FULL NAME")
    private String fullName;

    @JSONField(name = "DATE OF BIRTH")
    private Date dateOfBirth;

    public Person(int age, String fullName, Date dateOfBirth) {
        super();
        this.age = age;
        this.fullName= fullName;
        this.dateOfBirth = dateOfBirth;
    }

    // standard getters & setters
}
```

我们可以使用 **`JSON.toJSONString()`** 将 Java 对象转换成 JSON 字符串:

```java
private List<Person> listOfPersons = new ArrayList<Person>();

@Before
public void setUp() {
    listOfPersons.add(new Person(15, "John Doe", new Date()));
    listOfPersons.add(new Person(20, "Janette Doe", new Date()));
}

@Test
public void whenJavaList_thanConvertToJsonCorrect() {
    String jsonOutput= JSON.toJSONString(listOfPersons);
}
```

这是结果:

```java
[  
    {  
        "AGE":15,
        "DATE OF BIRTH":1468962431394,
        "FULL NAME":"John Doe"
    },
    {  
        "AGE":20,
        "DATE OF BIRTH":1468962431394,
        "FULL NAME":"Janette Doe"
    }
]
```

我们还可以更进一步，开始定制输出并控制诸如**排序**，日期**格式化**，或者**序列化**标志之类的事情。

例如，让我们更新 bean 并添加几个字段:

```java
@JSONField(name="AGE", serialize=false)
private int age;

@JSONField(name="LAST NAME", ordinal = 2)
private String lastName;

@JSONField(name="FIRST NAME", ordinal = 1)
private String firstName;

@JSONField(name="DATE OF BIRTH", format="dd/MM/yyyy", ordinal = 3)
private Date dateOfBirth;
```

这里列出了我们可以和`**@JSONField**`注释一起使用的最基本的参数，以便定制转换过程:

*   参数 **`format`** 用于正确格式化`date`属性
*   默认情况下，FastJson 库完全序列化 Java bean，但是我们可以使用参数 **`serialize`** 来忽略特定字段的序列化
*   参数 **`ordinal`** 用于指定字段顺序

这是新的结果:

```java
[
    {
        "FIRST NAME":"Doe",
        "LAST NAME":"Jhon",
        "DATE OF BIRTH":"19/07/2016"
    },
    {
        "FIRST NAME":"Doe",
        "LAST NAME":"Janette",
        "DATE OF BIRTH":"19/07/2016"
    }
]
```

FastJson 还支持一个非常有趣的 **`BeanToArray`序列化**特性:

```java
String jsonOutput= JSON.toJSONString(listOfPersons, SerializerFeature.BeanToArray);
```

这种情况下的输出如下所示:

```java
[
    [
        15,
        1469003271063,
        "John Doe"
    ],
    [
        20,
        1469003271063,
        "Janette Doe"
    ]
]
```

## 4。创建 JSON 对象

像[的其他 JSON 库](/web/20220815040838/https://www.baeldung.com/java-json)一样，从头开始创建一个 JSON 对象非常简单，只需要组合 **`JSONObject`** 和`**JSONArray**` 对象:

```java
@Test
public void whenGenerateJson_thanGenerationCorrect() throws ParseException {
    JSONArray jsonArray = new JSONArray();
    for (int i = 0; i < 2; i++) {
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("AGE", 10);
        jsonObject.put("FULL NAME", "Doe " + i);
        jsonObject.put("DATE OF BIRTH", "2016/12/12 12:12:12");
        jsonArray.add(jsonObject);
    }
    String jsonOutput = jsonArray.toJSONString();
}
```

这里是输出的样子:

```java
[
   {
      "AGE":"10",
      "DATE OF BIRTH":"2016/12/12 12:12:12",
      "FULL NAME":"Doe 0"
   },
   {
      "AGE":"10",
      "DATE OF BIRTH":"2016/12/12 12:12:12",
      "FULL NAME":"Doe 1"
   }
]
```

## 5。将 JSON 字符串解析成 Java 对象

既然我们已经知道了如何从头开始创建 JSON 对象，以及如何将 Java 对象转换成它们的 JSON 表示，那么让我们把重点放在如何解析 JSON 表示上:

```java
@Test
public void whenJson_thanConvertToObjectCorrect() {
    Person person = new Person(20, "John", "Doe", new Date());
    String jsonObject = JSON.toJSONString(person);
    Person newPerson = JSON.parseObject(jsonObject, Person.class);

    assertEquals(newPerson.getAge(), 0); // if we set serialize to false
    assertEquals(newPerson.getFullName(), listOfPersons.get(0).getFullName());
}
```

我们可以使用 **`JSON.parseObject()`** 从 JSON 字符串中获取 Java 对象。

请注意，如果您已经声明了自己的参数化构造函数，则必须定义一个**无参数或默认构造函数**，否则将抛出一个 **`com.alibaba.fastjson.JSONException`** 。

下面是这个简单测试的输出:

```java
Person [age=20, fullName=John Doe, dateOfBirth=Wed Jul 20 08:51:12 WEST 2016]
```

通过使用`@JSONField`注释中的选项`deserialize`，我们可以忽略特定字段的反序列化，在这种情况下，默认值将自动应用于被忽略的字段:

```java
@JSONField(name = "DATE OF BIRTH", deserialize=false)
private Date dateOfBirth;
```

这是新创建的对象:

```java
Person [age=20, fullName=John Doe, dateOfBirth=null]
```

## 6。使用`ContextValueFilter` 配置 JSON 转换

在某些场景中，我们可能需要对从 Java 对象到 JSON 格式的转换过程有更多的控制。

在这种情况下，我们可以利用 **`ContextValueFilter`** 对象对转换流进行额外的过滤和自定义处理:

```java
@Test
public void givenContextFilter_whenJavaObject_thanJsonCorrect() {
    ContextValueFilter valueFilter = new ContextValueFilter () {
        public Object process(
          BeanContext context, Object object, String name, Object value) {
            if (name.equals("DATE OF BIRTH")) {
                return "NOT TO DISCLOSE";
            }
            if (value.equals("John")) {
                return ((String) value).toUpperCase();
            } else {
                return null;
            }
        }
    };
    String jsonOutput = JSON.toJSONString(listOfPersons, valueFilter);
}
```

在这个例子中，我们隐藏了`DATE OF BIRTH`字段，通过强制一个常数值，我们也忽略了所有不是`John`或`Doe:` 的字段

```java
[
    {
        "FULL NAME":"JOHN DOE",
        "DATE OF BIRTH":"NOT TO DISCLOSE"
    }
]
```

正如您所看到的，这是一个非常基本的示例，但是您当然也可以将相同的概念用于更复杂的场景——将 FastJson 提供的这些强大而轻量级的工具集结合到一个真实的项目中。

## 7。使用`NameFilter`和`SerializeConfig`和

FastJson 提供了一套工具，用于在处理任意对象时定制 Json 操作——我们没有这些对象的源代码。

假设我们有一个编译版本的`Person` Java bean，最初在本文中声明，我们需要对字段命名和基本格式进行一些增强:

```java
@Test
public void givenSerializeConfig_whenJavaObject_thanJsonCorrect() {
    NameFilter formatName = new NameFilter() {
        public String process(Object object, String name, Object value) {
            return name.toLowerCase().replace(" ", "_");
        }
    };

    SerializeConfig.getGlobalInstance().addFilter(Person.class,  formatName);
    String jsonOutput = 
      JSON.toJSONStringWithDateFormat(listOfPersons, "yyyy-MM-dd");
}
```

我们已经使用 **`NameFilter`** 匿名类声明了`formatName`过滤器来处理字段名。新创建的过滤器与`Person`类相关联，然后被添加到一个全局实例中——这基本上是`**SerializeConfig**`类中的一个静态属性。

现在我们可以轻松地将对象转换成 JSON 格式，如本文前面所示。

请注意，我们使用了 **`toJSONStringWithDateFormat()`** 而不是`toJSONString()`来快速对日期字段应用相同的格式规则。

这是输出结果:

```java
[  
    {  
        "full_name":"John Doe",
        "date_of_birth":"2016-07-21"
    },
    {  
        "full_name":"Janette Doe",
        "date_of_birth":"2016-07-21"
    }
]
```

如您所见–**字段名发生了变化**，日期值的格式也变得正确。

将 **`SerializeFilter`** 与 **`ContextValueFilter`** 结合使用，可以对任意复杂的 Java 对象的转换过程进行全面控制。

## 8。结论

在本文中，我们展示了如何使用 FastJson 将 Java beans 转换为 Json 字符串，以及如何反过来。我们还展示了如何使用 FastJson 的一些核心特性来定制 Json 输出。

正如你所看到的，库提供了一个相对简单但仍然非常强大的 API。 **`JSON.toJSONString`** 和 **`JSON.parseObject`** 都是为了满足你的大部分需求而需要使用的——如果不是全部的话。

你可以在 [**链接的 GitHub 项目**](https://web.archive.org/web/20220815040838/https://github.com/eugenp/tutorials/tree/master/json-modules/json-2) 中查看本文提供的例子。