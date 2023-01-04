# Java 泛型中的超类型标记

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-super-type-tokens>

## 1.概观

在本教程中，我们将熟悉超类型标记，看看它们如何帮助我们在运行时保存[泛型类型信息](/web/20221208143841/https://www.baeldung.com/java-generics)。

## 2.擦除

**有时我们需要向方法**传递特定的类型信息。例如，这里我们期望从 Jackson 将 JSON 字节数组转换成一个`String:`

```java
byte[] data = // fetch json from somewhere
String json = objectMapper.readValue(data, String.class);
```

我们通过一个文字类标记来传达这种期望，在本例中是`String.class. `

然而，我们不能轻易地为泛型类型设置相同的期望:

```java
Map<String, String> json = objectMapper.readValue(data, Map<String, String>.class); // won't compile
```

Java 在编译过程中删除泛型类型信息。因此，**泛型类型参数仅仅是源代码的一个工件，在运行时是不存在的。**

### 2.1.具体化

从技术上讲，泛型类型在 Java 中没有具体化。在编程语言的术语中，当一个类型出现在运行时，我们说这个类型被具体化了。

Java 中的具体化类型如下:

*   简单的原始类型，如`long`
*   非通用抽象，如`String `或`Runnable`
*   原始类型，如`List `或`HashMap`
*   所有类型都是无界通配符的泛型类型，如`List<?> `或`HashMap<?, ?>`
*   其他具体化类型的数组，如`String[], int[], List[],` 或 `Map<?, ?>[]`

因此，我们不能使用类似于`Map<String, String>.class `的东西，因为`Map<String, String> `不是具体化的类型。

## 3.超级类型令牌

事实证明，我们可以利用 Java 中的[匿名内部类](/web/20221208143841/https://www.baeldung.com/java-anonymous-classes)在编译时保存类型信息:

```java
public abstract class TypeReference<T> {

    private final Type type;

    public TypeReference() {
        Type superclass = getClass().getGenericSuperclass();
        type = ((ParameterizedType) superclass).getActualTypeArguments()[0];
    }

    public Type getType() {
        return type;
    }
}
```

这个类是抽象的，所以我们只能从它派生子类。

例如，我们可以创建一个匿名的内部:

```java
TypeReference<Map<String, Integer>> token = new TypeReference<Map<String, String>>() {};
```

构造函数执行以下步骤来保留类型信息:

*   首先，它获取这个特定实例的通用超类元数据——在本例中，通用超类是`TypeReference<Map<String, Integer>>`
*   然后，它获取并存储泛型超类的实际类型参数——在本例中，它将是`Map<String, Integer>`

**这种保存泛型类型信息的方法通常被称为** **超类型令牌**:

```java
TypeReference<Map<String, Integer>> token = new TypeReference<Map<String, Integer>>() {};
Type type = token.getType();

assertEquals("java.util.Map<java.lang.String, java.lang.Integer>", type.getTypeName());

Type[] typeArguments = ((ParameterizedType) type).getActualTypeArguments();
assertEquals("java.lang.String", typeArguments[0].getTypeName());
assertEquals("java.lang.Integer", typeArguments[1].getTypeName());
```

使用超类型标记，我们知道容器类型是`Map, `，它的类型参数是`String `和`Integer. `

这种模式非常有名，像 Jackson 这样的库和 Spring 这样的框架都有自己的实现。将一个 JSON 对象解析成一个`Map<String, String>` 可以通过用一个超类型标记定义该类型来完成:

```java
TypeReference<Map<String, String>> token = new TypeReference<Map<String, String>>() {};
Map<String, String> json = objectMapper.readValue(data, token);
```

## 4.结论

在本教程中，我们学习了如何使用超类型标记在运行时保存泛型类型信息。

像往常一样，所有的例子都可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143841/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-oop-generics)