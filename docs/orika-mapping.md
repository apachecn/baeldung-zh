# 使用 Orika 映射

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/orika-mapping>

## 1。概述

Orika 是一个 Java Bean 映射框架，**递归地将数据从一个对象复制到另一个对象。这在开发多层应用程序时非常有用。**

在这些层之间来回移动数据对象时，经常会发现我们需要将对象从一个实例转换到另一个实例，以适应不同的 API。

实现这一点的一些方法是:**硬编码复制逻辑或者实现像[推土机](/web/20220121031604/https://www.baeldung.com/dozer)** 这样的 bean mappers。但是，它可以用来简化一个对象层和另一个对象层之间的映射过程。

Orika **使用字节码生成以最小的开销创建快速映射器**，使其比其他基于反射的映射器如[推土机](/web/20220121031604/https://www.baeldung.com/dozer)快得多。

## 2。简单的例子

映射框架的基础是`MapperFactory`类。这是我们将用来配置映射并获得执行实际映射工作的`MapperFacade` 实例的类。

我们像这样创建一个`MapperFactory`对象:

```java
MapperFactory mapperFactory = new DefaultMapperFactory.Builder().build();
```

然后假设我们有一个源数据对象`Source.java`，它有两个字段:

```java
public class Source {
    private String name;
    private int age;

    public Source(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // standard getters and setters
}
```

以及类似的目的地数据对象，`Dest.java`:

```java
public class Dest {
    private String name;
    private int age;

    public Dest(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // standard getters and setters
}
```

这是使用 Orika 的最基本的 bean 映射:

```java
@Test
public void givenSrcAndDest_whenMaps_thenCorrect() {
    mapperFactory.classMap(Source.class, Dest.class);
    MapperFacade mapper = mapperFactory.getMapperFacade();
    Source src = new Source("Baeldung", 10);
    Dest dest = mapper.map(src, Dest.class);

    assertEquals(dest.getAge(), src.getAge());
    assertEquals(dest.getName(), src.getName());
}
```

正如我们所观察到的，我们简单地通过映射创建了一个与`Source`具有相同字段的`Dest`对象。默认情况下，双向或反向映射也是可能的:

```java
@Test
public void givenSrcAndDest_whenMapsReverse_thenCorrect() {
    mapperFactory.classMap(Source.class, Dest.class).byDefault();
    MapperFacade mapper = mapperFactory.getMapperFacade();
    Dest src = new Dest("Baeldung", 10);
    Source dest = mapper.map(src, Source.class);

    assertEquals(dest.getAge(), src.getAge());
    assertEquals(dest.getName(), src.getName());
}
```

## 3。Maven 设置

为了在我们的 maven 项目中使用 Orika mapper，我们需要在`pom.xml`中拥有`orika-core`依赖关系:

```java
<dependency>
    <groupId>ma.glasnost.orika</groupId>
    <artifactId>orika-core</artifactId>
    <version>1.4.6</version>
</dependency>
```

最新版本总是可以在[这里](https://web.archive.org/web/20220121031604/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22orika-core%22)找到。

## 3。与`MapperFactory`一起工作

用 Orika 映射的一般模式包括创建一个`MapperFactory`对象，配置它以防我们需要调整默认的映射行为，从它那里获得一个`MapperFacade`对象，最后是实际的映射。

我们将在所有的例子中观察这种模式。但是我们的第一个例子显示了映射器的默认行为，我们没有做任何调整。

### 3.1。`BoundMapperFacade`vs`MapperFacade`

需要注意的一点是，我们可以选择使用`BoundMapperFacade`而不是默认的`MapperFacade`，后者非常慢。在这些情况下，我们需要映射一对特定的类型。

因此，我们最初的测试将变成:

```java
@Test
public void givenSrcAndDest_whenMapsUsingBoundMapper_thenCorrect() {
    BoundMapperFacade<Source, Dest> 
      boundMapper = mapperFactory.getMapperFacade(Source.class, Dest.class);
    Source src = new Source("baeldung", 10);
    Dest dest = boundMapper.map(src);

    assertEquals(dest.getAge(), src.getAge());
    assertEquals(dest.getName(), src.getName());
}
```

然而，为了让`BoundMapperFacade`双向映射，我们必须显式地调用`mapReverse`方法，而不是我们在默认`MapperFacade`的情况下看到的 map 方法:

```java
@Test
public void givenSrcAndDest_whenMapsUsingBoundMapperInReverse_thenCorrect() {
    BoundMapperFacade<Source, Dest> 
      boundMapper = mapperFactory.getMapperFacade(Source.class, Dest.class);
    Dest src = new Dest("baeldung", 10);
    Source dest = boundMapper.mapReverse(src);

    assertEquals(dest.getAge(), src.getAge());
    assertEquals(dest.getName(), src.getName());
}
```

否则测试将会失败。

### 3.2。配置字段映射

到目前为止，我们看到的例子涉及到具有相同字段名的源类和目的类。本小节处理两者之间存在差异的情况。

考虑一个源对象`Person`，它有三个字段，即`name`、`nickname`和`age`:

```java
public class Person {
    private String name;
    private String nickname;
    private int age;

    public Person(String name, String nickname, int age) {
        this.name = name;
        this.nickname = nickname;
        this.age = age;
    }

    // standard getters and setters
}
```

然后应用程序的另一层有一个类似的对象，但由法国程序员编写。假设称之为`Personne`，有字段`nom`、`surnom`和`age`，都对应以上三个:

```java
public class Personne {
    private String nom;
    private String surnom;
    private int age;

    public Personne(String nom, String surnom, int age) {
        this.nom = nom;
        this.surnom = surnom;
        this.age = age;
    }

    // standard getters and setters
}
```

**Orika 无法自动解决这些差异。但是我们可以使用`ClassMapBuilder` API 来注册这些唯一的映射。**

我们以前已经使用过它，但是我们还没有利用它的任何强大功能。我们前面每个使用默认`MapperFacade`的测试的第一行是使用`ClassMapBuilder` API 来注册我们想要映射的两个类:

```java
mapperFactory.classMap(Source.class, Dest.class);
```

我们还可以使用默认配置来映射所有字段，以使其更加清晰:

```java
mapperFactory.classMap(Source.class, Dest.class).byDefault()
```

通过添加`byDefault()`方法调用，我们已经使用`ClassMapBuilder` API 配置了映射器的行为。

现在我们希望能够将`Personne`映射到`Person`，所以我们也使用`ClassMapBuilder` API: 将字段映射配置到映射器上

```java
@Test
public void givenSrcAndDestWithDifferentFieldNames_whenMaps_thenCorrect() {
    mapperFactory.classMap(Personne.class, Person.class)
      .field("nom", "name").field("surnom", "nickname")
      .field("age", "age").register();
    MapperFacade mapper = mapperFactory.getMapperFacade();
    Personne frenchPerson = new Personne("Claire", "cla", 25);
    Person englishPerson = mapper.map(frenchPerson, Person.class);

    assertEquals(englishPerson.getName(), frenchPerson.getNom());
    assertEquals(englishPerson.getNickname(), frenchPerson.getSurnom());
    assertEquals(englishPerson.getAge(), frenchPerson.getAge());
}
```

不要忘记调用`register()` API 方法，以便用`MapperFactory`注册配置。

即使只有一个字段不同，沿着这条路走下去意味着我们必须显式地注册所有的字段映射，包括在两个对象中相同的`age`，否则未注册的字段将不会被映射，测试将会失败。

这将很快变得乏味，**如果我们只想映射 20 个**字段中的一个，我们需要配置它们所有的映射吗？

不，在我们没有明确定义映射的情况下，当我们告诉映射器使用它的默认映射配置时不会:

```java
mapperFactory.classMap(Personne.class, Person.class)
  .field("nom", "name").field("surnom", "nickname").byDefault().register();
```

在这里，我们没有为`age`字段定义映射，但是测试将会通过。

### 3.3。排除一个字段

假设我们想从映射中排除`Personne`的`nom`字段，这样`Person`对象只接收未被排除的字段的新值:

```java
@Test
public void givenSrcAndDest_whenCanExcludeField_thenCorrect() {
    mapperFactory.classMap(Personne.class, Person.class).exclude("nom")
      .field("surnom", "nickname").field("age", "age").register();
    MapperFacade mapper = mapperFactory.getMapperFacade();
    Personne frenchPerson = new Personne("Claire", "cla", 25);
    Person englishPerson = mapper.map(frenchPerson, Person.class);

    assertEquals(null, englishPerson.getName());
    assertEquals(englishPerson.getNickname(), frenchPerson.getSurnom());
    assertEquals(englishPerson.getAge(), frenchPerson.getAge());
}
```

请注意我们是如何在`MapperFactory`的配置中排除它的，然后也请注意第一个断言，我们期望`Person`对象中的`name`的值保持为`null`，因为它在映射中被排除了。

## 4。集合映射

有时，目标对象可能有唯一的属性，而源对象只维护集合中的每个属性。

### 4.1。列表和数组

考虑一个只有一个字段的源数据对象，一个人名列表:

```java
public class PersonNameList {
    private List<String> nameList;

    public PersonNameList(List<String> nameList) {
        this.nameList = nameList;
    }
}
```

现在考虑我们的目的数据对象，它将`firstName`和`lastName`分成单独的字段:

```java
public class PersonNameParts {
    private String firstName;
    private String lastName;

    public PersonNameParts(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
}
```

让我们假设我们非常确定，在索引 0 处，总会有这个人的`firstName`，在索引 1 处，总会有他们的`lastName`。

Orika 允许我们使用括号符号来访问集合的成员:

```java
@Test
public void givenSrcWithListAndDestWithPrimitiveAttributes_whenMaps_thenCorrect() {
    mapperFactory.classMap(PersonNameList.class, PersonNameParts.class)
      .field("nameList[0]", "firstName")
      .field("nameList[1]", "lastName").register();
    MapperFacade mapper = mapperFactory.getMapperFacade();
    List<String> nameList = Arrays.asList(new String[] { "Sylvester", "Stallone" });
    PersonNameList src = new PersonNameList(nameList);
    PersonNameParts dest = mapper.map(src, PersonNameParts.class);

    assertEquals(dest.getFirstName(), "Sylvester");
    assertEquals(dest.getLastName(), "Stallone");
}
```

即使不是用`PersonNameList`，而是用`PersonNameArray`，同样的测试也适用于一组名字。

### 4.2。地图

假设我们的源对象有一个值映射。我们知道在那个映射中有一个键，`first`，它的值代表一个人在我们的目的对象中的`firstName`。

同样，我们知道在同一个地图中还有另一个键`last`，它的值代表一个人在目的地对象中的`lastName`。

```java
public class PersonNameMap {
    private Map<String, String> nameMap;

    public PersonNameMap(Map<String, String> nameMap) {
        this.nameMap = nameMap;
    }
}
```

与上一节中的情况类似，我们使用括号符号，但不是传入索引，而是传入我们希望映射到给定目标字段的值的键。

Orika 接受两种检索密钥的方法，这两种方法在下面的测试中都有体现:

```java
@Test
public void givenSrcWithMapAndDestWithPrimitiveAttributes_whenMaps_thenCorrect() {
    mapperFactory.classMap(PersonNameMap.class, PersonNameParts.class)
      .field("nameMap['first']", "firstName")
      .field("nameMap[\"last\"]", "lastName")
      .register();
    MapperFacade mapper = mapperFactory.getMapperFacade();
    Map<String, String> nameMap = new HashMap<>();
    nameMap.put("first", "Leornado");
    nameMap.put("last", "DiCaprio");
    PersonNameMap src = new PersonNameMap(nameMap);
    PersonNameParts dest = mapper.map(src, PersonNameParts.class);

    assertEquals(dest.getFirstName(), "Leornado");
    assertEquals(dest.getLastName(), "DiCaprio");
}
```

我们可以使用单引号或双引号，但我们必须避免后者。

## 5。映射嵌套字段

根据前面的集合示例，假设在我们的源数据对象中，有另一个数据传输对象(DTO ),它保存我们想要映射的值。

```java
public class PersonContainer {
    private Name name;

    public PersonContainer(Name name) {
        this.name = name;
    }
}
```

```java
public class Name {
    private String firstName;
    private String lastName;

    public Name(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
}
```

为了能够访问嵌套 DTO 的属性并将它们映射到我们的目标对象，我们使用点符号，如下所示:

```java
@Test
public void givenSrcWithNestedFields_whenMaps_thenCorrect() {
    mapperFactory.classMap(PersonContainer.class, PersonNameParts.class)
      .field("name.firstName", "firstName")
      .field("name.lastName", "lastName").register();
    MapperFacade mapper = mapperFactory.getMapperFacade();
    PersonContainer src = new PersonContainer(new Name("Nick", "Canon"));
    PersonNameParts dest = mapper.map(src, PersonNameParts.class);

    assertEquals(dest.getFirstName(), "Nick");
    assertEquals(dest.getLastName(), "Canon");
}
```

## 6。映射空值

在某些情况下，您可能希望控制在遇到空值时是映射还是忽略它们。默认情况下，遇到以下情况时，Orika 将映射空值:

```java
@Test
public void givenSrcWithNullField_whenMapsThenCorrect() {
    mapperFactory.classMap(Source.class, Dest.class).byDefault();
    MapperFacade mapper = mapperFactory.getMapperFacade();
    Source src = new Source(null, 10);
    Dest dest = mapper.map(src, Dest.class);

    assertEquals(dest.getAge(), src.getAge());
    assertEquals(dest.getName(), src.getName());
}
```

这种行为可以在不同的层次上定制，这取决于我们想要的具体程度。

### 6.1。全局配置

在创建全局`MapperFactory`之前，我们可以配置我们的映射器来映射空值或者在全局级别忽略它们。还记得我们在第一个例子中是如何创建这个对象的吗？这一次，我们在构建过程中添加了一个额外的调用:

```java
MapperFactory mapperFactory = new DefaultMapperFactory.Builder()
  .mapNulls(false).build();
```

我们可以运行一个测试来确认确实没有映射空值:

```java
@Test
public void givenSrcWithNullAndGlobalConfigForNoNull_whenFailsToMap_ThenCorrect() {
    mapperFactory.classMap(Source.class, Dest.class);
    MapperFacade mapper = mapperFactory.getMapperFacade();
    Source src = new Source(null, 10);
    Dest dest = new Dest("Clinton", 55);
    mapper.map(src, dest);

    assertEquals(dest.getAge(), src.getAge());
    assertEquals(dest.getName(), "Clinton");
}
```

默认情况下，会映射空值。这意味着，即使源对象中的字段值是`null`并且目标对象中相应字段的值有意义，它也会被覆盖。

在我们的例子中，如果对应的源字段有一个`null`值，目标字段不会被覆盖。

### 6.2。本地配置

通过使用`mapNulls(true|false)`或`mapNullsInReverse(true|false)`控制反方向的空值映射，可以在`ClassMapBuilder`上控制`null`值的映射。

通过在一个`ClassMapBuilder`实例上设置该值，在设置该值后，在同一个`ClassMapBuilder`上创建的所有字段映射将采用相同的值。

让我们用一个示例测试来说明这一点:

```java
@Test
public void givenSrcWithNullAndLocalConfigForNoNull_whenFailsToMap_ThenCorrect() {
    mapperFactory.classMap(Source.class, Dest.class).field("age", "age")
      .mapNulls(false).field("name", "name").byDefault().register();
    MapperFacade mapper = mapperFactory.getMapperFacade();
    Source src = new Source(null, 10);
    Dest dest = new Dest("Clinton", 55);
    mapper.map(src, dest);

    assertEquals(dest.getAge(), src.getAge());
    assertEquals(dest.getName(), "Clinton");
}
```

注意我们是如何在注册`name`字段之前调用`mapNulls`的，这将导致在`mapNulls`调用之后的所有字段在它们具有`null`值时被忽略。

双向映射也接受映射的空值:

```java
@Test
public void givenDestWithNullReverseMappedToSource_whenMapsByDefault_thenCorrect() {
    mapperFactory.classMap(Source.class, Dest.class).byDefault();
    MapperFacade mapper = mapperFactory.getMapperFacade();
    Dest src = new Dest(null, 10);
    Source dest = new Source("Vin", 44);
    mapper.map(src, dest);

    assertEquals(dest.getAge(), src.getAge());
    assertEquals(dest.getName(), src.getName());
}
```

我们也可以通过调用`mapNullsInReverse`并传入`false`来防止这种情况:

```java
@Test
public void 
  givenDestWithNullReverseMappedToSourceAndLocalConfigForNoNull_whenFailsToMap_thenCorrect() {
    mapperFactory.classMap(Source.class, Dest.class).field("age", "age")
      .mapNullsInReverse(false).field("name", "name").byDefault()
      .register();
    MapperFacade mapper = mapperFactory.getMapperFacade();
    Dest src = new Dest(null, 10);
    Source dest = new Source("Vin", 44);
    mapper.map(src, dest);

    assertEquals(dest.getAge(), src.getAge());
    assertEquals(dest.getName(), "Vin");
}
```

### 6.3。现场级配置

我们可以使用`fieldMap`在现场级别进行配置，如下所示:

```java
mapperFactory.classMap(Source.class, Dest.class).field("age", "age")
  .fieldMap("name", "name").mapNulls(false).add().byDefault().register();
```

在这种情况下，配置将只影响我们在字段级别称之为的`name`字段:

```java
@Test
public void givenSrcWithNullAndFieldLevelConfigForNoNull_whenFailsToMap_ThenCorrect() {
    mapperFactory.classMap(Source.class, Dest.class).field("age", "age")
      .fieldMap("name", "name").mapNulls(false).add().byDefault().register();
    MapperFacade mapper = mapperFactory.getMapperFacade();
    Source src = new Source(null, 10);
    Dest dest = new Dest("Clinton", 55);
    mapper.map(src, dest);

    assertEquals(dest.getAge(), src.getAge());
    assertEquals(dest.getName(), "Clinton");
}
```

## 7。奥里卡自定义映射

到目前为止，我们已经查看了使用`ClassMapBuilder` API 的简单定制映射示例。我们将仍然使用相同的 API，但是使用 Orika 的`CustomMapper`类定制我们的映射。

假设我们有两个数据对象，每个对象都有一个名为`dtob`的字段，表示一个人出生的日期和时间。

一个数据对象将该值表示为以下 ISO 格式的`datetime String`:

```java
2007-06-26T21:22:39Z
```

另一个表示与以下 unix 时间戳格式中的`long`类型相同:

```java
1182882159000
```

显然，到目前为止，我们讨论的定制都不足以在映射过程中在两种格式之间进行转换，甚至 Orika 的内置转换器也不能处理这项工作。这就是我们必须在映射期间编写一个`CustomMapper`来进行所需转换的地方。

让我们创建第一个数据对象:

```java
public class Person3 {
    private String name;
    private String dtob;

    public Person3(String name, String dtob) {
        this.name = name;
        this.dtob = dtob;
    }
}
```

然后我们的第二个数据对象:

```java
public class Personne3 {
    private String name;
    private long dtob;

    public Personne3(String name, long dtob) {
        this.name = name;
        this.dtob = dtob;
    }
}
```

我们现在不会标记哪个是源，哪个是目的地，因为`CustomMapper`使我们能够满足双向映射。

下面是我们对抽象类`CustomMapper`的具体实现:

```java
class PersonCustomMapper extends CustomMapper<Personne3, Person3> {

    @Override
    public void mapAtoB(Personne3 a, Person3 b, MappingContext context) {
        Date date = new Date(a.getDtob());
        DateFormat format = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss'Z'");
        String isoDate = format.format(date);
        b.setDtob(isoDate);
    }

    @Override
    public void mapBtoA(Person3 b, Personne3 a, MappingContext context) {
        DateFormat format = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss'Z'");
        Date date = format.parse(b.getDtob());
        long timestamp = date.getTime();
        a.setDtob(timestamp);
    }
};
```

注意，我们已经实现了方法`mapAtoB`和`mapBtoA`。实现这两者使得我们的映射函数是双向的。

**每个方法暴露了我们正在映射的数据对象，我们负责将字段值从一个复制到另一个**。

在将源数据写入目标对象之前，我们在这里编写定制代码来根据我们的需求操作源数据。

让我们运行一个测试来确认我们的自定义映射器是否正常工作:

```java
@Test
public void givenSrcAndDest_whenCustomMapperWorks_thenCorrect() {
    mapperFactory.classMap(Personne3.class, Person3.class)
      .customize(customMapper).register();
    MapperFacade mapper = mapperFactory.getMapperFacade();
    String dateTime = "2007-06-26T21:22:39Z";
    long timestamp = new Long("1182882159000");
    Personne3 personne3 = new Personne3("Leornardo", timestamp);
    Person3 person3 = mapper.map(personne3, Person3.class);

    assertEquals(person3.getDtob(), dateTime);
}
```

注意，我们仍然通过`ClassMapBuilder` API 将自定义映射器传递给 Orika 的映射器，就像所有其他简单的自定义一样。

我们也可以确认双向映射是可行的:

```java
@Test
public void givenSrcAndDest_whenCustomMapperWorksBidirectionally_thenCorrect() {
    mapperFactory.classMap(Personne3.class, Person3.class)
      .customize(customMapper).register();
    MapperFacade mapper = mapperFactory.getMapperFacade();
    String dateTime = "2007-06-26T21:22:39Z";
    long timestamp = new Long("1182882159000");
    Person3 person3 = new Person3("Leornardo", dateTime);
    Personne3 personne3 = mapper.map(person3, Personne3.class);

    assertEquals(person3.getDtob(), timestamp);
}
```

## 8。结论

在本文中，我们已经探索了 Orika 映射框架最重要的特性。

肯定有更多的高级特性给了我们更多的控制，但在大多数用例中，这里介绍的已经足够了。

完整的项目代码和所有示例可以在我的 [github 项目](https://web.archive.org/web/20220121031604/https://github.com/eugenp/tutorials/tree/master/orika)中找到。不要忘了查看我们关于[推土机绘图框架](/web/20220121031604/https://www.baeldung.com/dozer)的教程，因为它们都解决了或多或少相同的问题。