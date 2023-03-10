# 从 Gson 中的序列化中排除字段

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gson-exclude-fields-serialization>

## 1。概述

在这个简短的教程中，我们将探索从 Gson 序列化中排除 Java 类及其子类的一个或多个字段的可用选项。

## 2。初始设置

让我们首先定义我们的类:

```java
@Data
@AllArgsConstructor
public class MyClass {
    private long id;
    private String name;
    private String other;
    private MySubClass subclass;
}

@Data
@AllArgsConstructor
public class MySubClass {
    private long id;
    private String description;
    private String otherVerboseInfo;
} 
```

为了方便起见，我们用 [Lombok](/web/20221205210900/http://www.baeldung.com/intro-to-project-lombok) 对它们进行了注释(getters、setters、constructor……的语法糖)。

现在让我们填充它们:

```java
MySubClass subclass = new MySubClass(42L, "the answer", "Verbose field not to serialize")
MyClass source = new MyClass(1L, "foo", "bar", subclass); 
```

我们的目标是防止`MyClass.other`和`MySubClass.otherVerboseInfo`字段被序列化。

我们期望得到的输出是:

```java
{
  "id":1,
  "name":"foo",
  "subclass":{
    "id":42,
    "description":"the answer"
  }
} 
```

在 Java 中:

```java
String expectedResult = "{\"id\":1,\"name\":\"foo\",\"subclass\":{\"id\":42,\"description\":\"the answer\"}}"; 
```

## 3。瞬变修改器

我们可以用`transient`修饰符标记一个字段:

```java
public class MyClass {
    private long id;
    private String name;
    private transient String other;
    private MySubClass subclass;
}

public class MySubClass {
    private long id;
    private String description;
    private transient String otherVerboseInfo;
} 
```

Gson 序列化程序将忽略每个声明为瞬态的字段:

```java
String jsonString = new Gson().toJson(source);
assertEquals(expectedResult, jsonString); 
```

虽然这非常快，但也带来了严重的负面影响:**每个序列化工具都会考虑瞬态**，而不仅仅是 Gson。

Transient 是 Java 从序列化中排除的方法，那么我们的字段也将被`Serializable`的序列化以及管理我们对象的每个库工具或框架过滤掉。

此外，`transient`关键字始终适用于序列化和反序列化，这可能会受到用例的限制。

## 4。`@Expose`注解

Gson `com.google.gson.annotations [**@Expose**](https://web.archive.org/web/20221205210900/https://static.javadoc.io/com.google.code.gson/gson/2.8.0/com/google/gson/annotations/Expose.html)`注释的工作方式正好相反。

我们可以使用它来声明要序列化的字段，并忽略其他字段:

```java
public class MyClass {
    @Expose 
    private long id;
    @Expose 
    private String name;
    private String other;
    @Expose 
    private MySubClass subclass;
}

public class MySubClass {
    @Expose 
    private long id;
    @Expose 
    private String description;
    private String otherVerboseInfo;
} 
```

为此，我们需要用 GsonBuilder 实例化 Gson:

```java
Gson gson = new GsonBuilder()
  .excludeFieldsWithoutExposeAnnotation()
  .create();
String jsonString = gson.toJson(source);
assertEquals(expectedResult, jsonString); 
```

这一次，我们可以在字段级别控制过滤是针对序列化、反序列化还是两者(默认)。

让我们看看如何防止`MyClass.other`被序列化，但允许它在从 JSON 反序列化的过程中被填充:

```java
@Expose(serialize = false, deserialize = true) 
private String other; 
```

虽然这是 Gson 提供的最简单的方法，并且不影响其他库，但这可能意味着代码中存在冗余。如果我们有一个有一百个字段的类，我们只想排除一个字段，我们需要写九十九个注释，这是大材小用。

## 5。`ExclusionStrategy`

一个高度可定制的解决方案是使用 [`com.google.gson.**ExclusionStrategy**`](https://web.archive.org/web/20221205210900/https://github.com/google/gson/blob/master/gson/src/main/java/com/google/gson/ExclusionStrategy.java) 。

它允许我们定义一个策略(在外部或者使用匿名的内部类)来指示 GsonBuilder 是否使用自定义标准来序列化字段(和/或类)。

```java
Gson gson = new GsonBuilder()
  .addSerializationExclusionStrategy(strategy)
  .create();
String jsonString = gson.toJson(source);

assertEquals(expectedResult, jsonString); 
```

让我们看一些聪明策略的例子。

### 5.1。带有类别和字段名称

当然，我们也可以硬编码一个或多个字段/类名:

```java
ExclusionStrategy strategy = new ExclusionStrategy() {
    @Override
    public boolean shouldSkipField(FieldAttributes field) {
        if (field.getDeclaringClass() == MyClass.class && field.getName().equals("other")) {
            return true;
        }
        if (field.getDeclaringClass() == MySubClass.class && field.getName().equals("otherVerboseInfo")) {
            return true;
        }
        return false;
    }

    @Override
    public boolean shouldSkipClass(Class<?> clazz) {
        return false;
    }
}; 
```

这很快而且直截了当，但是不太可重用，并且在我们重命名属性的情况下容易出错。

### 5.2。符合商业标准

因为我们只需返回一个布尔值，所以我们可以在该方法中实现任何我们喜欢的业务逻辑。

在下面的例子中，我们将把每个以“other”开头的字段标识为不应该序列化的字段，不管它们属于哪个类:

```java
ExclusionStrategy strategy = new ExclusionStrategy() {
    @Override
    public boolean shouldSkipClass(Class<?> clazz) {
        return false;
    }

    @Override
    public boolean shouldSkipField(FieldAttributes field) {
        return field.getName().startsWith("other");
    }
}; 
```

### 5.3。带有自定义注释

另一个聪明的方法是创建自定义注释:

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface Exclude {} 
```

然后我们可以利用`ExclusionStrategy`,让它完全像使用`@Expose`注释一样工作，但是反过来:

```java
public class MyClass {
    private long id;
    private String name;
    @Exclude 
    private String other;
    private MySubClass subclass;
}

public class MySubClass {
    private long id;
    private String description;
    @Exclude 
    private String otherVerboseInfo;
} 
```

策略是这样的:

```java
ExclusionStrategy strategy = new ExclusionStrategy() {
    @Override
    public boolean shouldSkipClass(Class<?> clazz) {
        return false;
    }

    @Override
    public boolean shouldSkipField(FieldAttributes field) {
        return field.getAnnotation(Exclude.class) != null;
    }
}; 
```

[这个 StackOverflow 回答](https://web.archive.org/web/20221205210900/https://stackoverflow.com/a/27986860/1654265)首先描述了这个技巧。

它允许我们一次编写注释和策略，无需进一步修改就可以动态注释我们的字段。

### 5.4。将排除策略扩展到反序列化

无论我们将使用哪种策略，我们总是可以控制它应该应用在哪里。

仅在序列化期间:

```java
Gson gson = new GsonBuilder().addSerializationExclusionStrategy(strategy) 
```

仅在反序列化期间:

```java
Gson gson = new GsonBuilder().addDeserializationExclusionStrategy(strategy) 
```

总是:

```java
Gson gson = new GsonBuilder().setExclusionStrategies(strategy); 
```

## 6。结论

我们已经看到了在 Gson 序列化过程中从类及其子类中排除字段的不同方法。

我们还探讨了每个解决方案的主要优点和缺陷。

和往常一样，完整的源代码可以在 Github 上找到[。](https://web.archive.org/web/20221205210900/https://github.com/eugenp/tutorials/tree/master/json-modules/gson)