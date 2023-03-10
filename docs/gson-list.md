# 用 Gson 序列化和反序列化列表

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/gson-list>

## 1。简介

在本教程中，我们将探索一些高级的[序列化](/web/20221107092027/https://www.baeldung.com/gson-serialization-guide)和[反序列化](/web/20221107092027/https://www.baeldung.com/gson-deserialization-guide)案例，用于使用 [Google 的 Gson 库](https://web.archive.org/web/20221107092027/https://github.com/google/gson)的`List`。

## 2。对象列表

一个常见的用例是序列化和反序列化 POJOs 列表。

考虑一下这个类:

```java
public class MyClass {
    private int id;
    private String name;

    public MyClass(int id, String name) {
        this.id = id;
        this.name = name;
    }

    // getters and setters
}
```

下面是我们如何连载`List<MyClass>`:

```java
@Test
public void givenListOfMyClass_whenSerializing_thenCorrect() {
    List<MyClass> list = Arrays.asList(new MyClass(1, "name1"), new MyClass(2, "name2"));

    Gson gson = new Gson();
    String jsonString = gson.toJson(list);
    String expectedString = "[{\"id\":1,\"name\":\"name1\"},{\"id\":2,\"name\":\"name2\"}]";

    assertEquals(expectedString, jsonString);
}
```

正如我们所见，序列化相当简单。

然而，反序列化是棘手的。这是一种不正确的做法:

```java
@Test(expected = ClassCastException.class)
public void givenJsonString_whenIncorrectDeserializing_thenThrowClassCastException() {
    String inputString = "[{\"id\":1,\"name\":\"name1\"},{\"id\":2,\"name\":\"name2\"}]";

    Gson gson = new Gson();
    List<MyClass> outputList = gson.fromJson(inputString, ArrayList.class);

    assertEquals(1, outputList.get(0).getId());
}
```

在这里，**尽管在反序列化后我们会得到一个大小为 2 的`List`，但它不会是`MyClass`** 的`List`。因此，6 号线抛出`ClassCastException`。

**Gson 可以序列化任意对象的集合，但是在没有附加信息的情况下无法反序列化数据。这是因为用户没有办法指出结果对象的类型。**相反，在反序列化时，`Collection`必须是特定的泛型类型。

反序列化`List`的正确方式应该是:

```java
@Test
public void givenJsonString_whenDeserializing_thenReturnListOfMyClass() {
    String inputString = "[{\"id\":1,\"name\":\"name1\"},{\"id\":2,\"name\":\"name2\"}]";
    List<MyClass> inputList = Arrays.asList(new MyClass(1, "name1"), new MyClass(2, "name2"));

    Type listOfMyClassObject = new TypeToken<ArrayList<MyClass>>() {}.getType();

    Gson gson = new Gson();
    List<MyClass> outputList = gson.fromJson(inputString, listOfMyClassObject);

    assertEquals(inputList, outputList);
}
```

这里，**我们使用 Gson 的`TypeToken`来确定要反序列化的正确类型——`ArrayList<MyClass>`**。用于获取`listOfMyClassObject`的习语实际上定义了一个匿名的本地内部类，包含一个返回完全参数化类型的方法`getType()`。

## 3。多态对象列表

### 3.1。问题

考虑一个动物等级的例子:

```java
public abstract class Animal {
    // ...
}

public class Dog extends Animal {
    // ...
}

public class Cow extends Animal {
    // ...
}
```

我们如何序列化和反序列化`List<Animal>`？我们可以像在上一节中一样使用`TypeToken<ArrayList<Animal>>`。然而，Gson 仍然无法确定存储在列表中的对象的具体数据类型。

### 3.2。使用自定义反序列化器

解决这个问题的一个方法是向序列化的 JSON 添加类型信息。我们在 JSON 反序列化过程中尊重类型信息。为此，我们需要编写自己的自定义序列化程序和反序列化程序。

首先，我们将在基类`Animal`中引入一个名为 `type`的新`String`字段。它存储它所属的类的简单名称。

让我们看一下我们的示例类:

```java
public abstract class Animal {
    public String type = "Animal";
}
```

```java
public class Dog extends Animal {
    private String petName;

    public Dog() {
        petName = "Milo";
        type = "Dog";
    }

    // getters and setters
}
```

```java
public class Cow extends Animal {
    private String breed;

    public Cow() {
        breed = "Jersey";
        type = "Cow";
    }

    // getters and setters
}
```

序列化将像以前一样继续工作，没有任何问题:

```java
@Test 
public void givenPolymorphicList_whenSerializeWithTypeAdapter_thenCorrect() {
    String expectedString
      = "[{\"petName\":\"Milo\",\"type\":\"Dog\"},{\"breed\":\"Jersey\",\"type\":\"Cow\"}]";

    List<Animal> inList = new ArrayList<>();
    inList.add(new Dog());
    inList.add(new Cow());

    String jsonString = new Gson().toJson(inList);

    assertEquals(expectedString, jsonString);
}
```

为了反序列化列表，我们必须提供一个定制的反序列化器:

```java
public class AnimalDeserializer implements JsonDeserializer<Animal> {
    private String animalTypeElementName;
    private Gson gson;
    private Map<String, Class<? extends Animal>> animalTypeRegistry;

    public AnimalDeserializer(String animalTypeElementName) {
        this.animalTypeElementName = animalTypeElementName;
        this.gson = new Gson();
        this.animalTypeRegistry = new HashMap<>();
    }

    public void registerBarnType(String animalTypeName, Class<? extends Animal> animalType) {
        animalTypeRegistry.put(animalTypeName, animalType);
    }

    public Animal deserialize(JsonElement json, Type typeOfT, JsonDeserializationContext context) {
        JsonObject animalObject = json.getAsJsonObject();
        JsonElement animalTypeElement = animalObject.get(animalTypeElementName);

        Class<? extends Animal> animalType = animalTypeRegistry.get(animalTypeElement.getAsString());
        return gson.fromJson(animalObject, animalType);
    }
}
```

这里，`animalTypeRegistry`映射维护类名和类类型之间的映射。

在反序列化过程中，我们首先提取出新添加的`type`字段。使用这个值，我们在`animalTypeRegistry`映射上进行查找，以获得具体的数据类型。这个数据类型然后被传递给`fromJson()`。

让我们看看如何使用我们的自定义反序列化器:

```java
@Test
public void givenPolymorphicList_whenDeserializeWithTypeAdapter_thenCorrect() {
    String inputString
      = "[{\"petName\":\"Milo\",\"type\":\"Dog\"},{\"breed\":\"Jersey\",\"type\":\"Cow\"}]";

    AnimalDeserializer deserializer = new AnimalDeserializer("type");
    deserializer.registerBarnType("Dog", Dog.class);
    deserializer.registerBarnType("Cow", Cow.class);
    Gson gson = new GsonBuilder()
      .registerTypeAdapter(Animal.class, deserializer)
      .create();

    List<Animal> outList = gson.fromJson(inputString, new TypeToken<List<Animal>>(){}.getType());

    assertEquals(2, outList.size());
    assertTrue(outList.get(0) instanceof Dog);
    assertTrue(outList.get(1) instanceof Cow);
}
```

### 3.3。使用`RuntimeTypeAdapterFactory`

编写定制反序列化器的另一种方法是使用 [Gson 源代码](https://web.archive.org/web/20221107092027/https://github.com/google/gson/blob/master/extras/src/main/java/com/google/gson/typeadapters/RuntimeTypeAdapterFactory.java)中的`RuntimeTypeAdapterFactory`类。然而，**并没有被图书馆公开给用户使用**。因此，我们必须在 Java 项目中创建该类的副本。

完成后，我们可以用它来反序列化我们的列表:

```java
@Test
public void givenPolymorphicList_whenDeserializeWithRuntimeTypeAdapter_thenCorrect() {
    String inputString
      = "[{\"petName\":\"Milo\",\"type\":\"Dog\"},{\"breed\":\"Jersey\",\"type\":\"Cow\"}]";

    Type listOfAnimals = new TypeToken<ArrayList<Animal>>(){}.getType();

    RuntimeTypeAdapterFactory<Animal> adapter = RuntimeTypeAdapterFactory.of(Animal.class, "type")
      .registerSubtype(Dog.class)
      .registerSubtype(Cow.class);

    Gson gson = new GsonBuilder().registerTypeAdapterFactory(adapter).create();

    List<Animal> outList = gson.fromJson(inputString, listOfAnimals);

    assertEquals(2, outList.size());
    assertTrue(outList.get(0) instanceof Dog);
    assertTrue(outList.get(1) instanceof Cow);
}
```

请注意，底层机制仍然是相同的。

我们仍然需要在序列化过程中引入类型信息。类型信息可以在以后的反序列化过程中使用。**因此，这个解决方案的每个类中仍然需要字段`type`。**我们不需要编写自己的反序列化器。

`RuntimeTypeAdapterFactory`根据传递给它的字段名和注册的子类型提供正确的类型适配器。

## 4。结论

在本文中，我们看到了如何使用 Gson 序列化和反序列化对象列表。

像往常一样，代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221107092027/https://github.com/eugenp/tutorials/tree/master/json-modules/gson)