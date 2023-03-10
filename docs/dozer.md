# 推土机绘图指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/dozer>

## 1。概述

[Dozer](https://web.archive.org/web/20220709012055/http://dozer.sourceforge.net/) 是一个 **Java Bean 到 Java Bean 映射器**，它递归地将数据从一个对象复制到另一个对象，一个属性接一个属性。

该库不仅支持 Java Beans 的属性名之间的映射，而且**会自动在类型**之间转换——如果它们不同的话。

大多数转换场景都支持开箱即用，但是 Dozer 还允许您通过 XML 指定自定义转换。

## 2。简单的例子

对于我们的第一个例子，让我们假设源和目标数据对象都共享相同的公共属性名。

这是推土机可以完成的最基本的绘图:

```java
public class Source {
    private String name;
    private int age;

    public Source() {}

    public Source(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // standard getters and setters
}
```

然后我们的目的地文件，`Dest.java`:

```java
public class Dest {
    private String name;
    private int age;

    public Dest() {}

    public Dest(String name, int age) {
        this.name = name;
        this.age = age;
    }

    // standard getters and setters
}
```

我们需要确保**包含默认或零参数构造函数**，因为 Dozer 使用了反射。

为了提高性能，让我们将映射器设为全局的，并创建一个将在整个测试中使用的对象:

```java
DozerBeanMapper mapper;

@Before
public void before() throws Exception {
    mapper = new DozerBeanMapper();
}
```

现在，让我们运行我们的第一个测试来确认当我们创建一个`Source`对象时，我们可以将它直接映射到一个`Dest`对象上:

```java
@Test
public void givenSourceObjectAndDestClass_whenMapsSameNameFieldsCorrectly_
  thenCorrect() {
    Source source = new Source("Baeldung", 10);
    Dest dest = mapper.map(source, Dest.class);

    assertEquals(dest.getName(), "Baeldung");
    assertEquals(dest.getAge(), 10);
}
```

正如我们所看到的，在 Dozer 映射之后，结果将是一个新的`Dest` 对象实例，它包含与`Source`对象具有相同字段名的所有字段的值。

或者，我们可以只创建`Dest`对象并将它的引用传递给`mapper`,而不是传递给`mapper`类`Dest`:

```java
@Test
public void givenSourceObjectAndDestObject_whenMapsSameNameFieldsCorrectly_
  thenCorrect() {
    Source source = new Source("Baeldung", 10);
    Dest dest = new Dest();
    mapper.map(source, dest);

    assertEquals(dest.getName(), "Baeldung");
    assertEquals(dest.getAge(), 10);
}
```

## 3。Maven 设置

现在我们对 Dozer 的工作原理有了基本的了解，让我们给`pom.xml`添加以下依赖项:

```java
<dependency>
    <groupId>net.sf.dozer</groupId>
    <artifactId>dozer</artifactId>
    <version>5.5.1</version>
</dependency>
```

最新版本可从[这里](https://web.archive.org/web/20220709012055/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22dozer%22)获得。

## 4。数据转换示例

我们已经知道，Dozer 可以将一个现有对象映射到另一个对象，只要它在两个类中找到同名的属性。

然而，情况并非总是如此；因此，如果任何映射的属性是不同的数据类型，推土机映射引擎将**自动执行数据类型转换**。

让我们来看看这个新概念的实际应用:

```java
public class Source2 {
    private String id;
    private double points;

    public Source2() {}

    public Source2(String id, double points) {
        this.id = id;
        this.points = points;
    }

    // standard getters and setters
}
```

目的地类别:

```java
public class Dest2 {
    private int id;
    private int points;

    public Dest2() {}

    public Dest2(int id, int points) {
        super();
        this.id = id;
        this.points = points;
    }

    // standard getters and setters
}
```

请注意，属性名称相同，但是**它们的数据类型不同**。

在源类中，`id`是一个`String`，而`points`是一个`double`，而在目的类中，`id`和`points`都是`integer`

现在让我们看看 Dozer 如何正确处理转换:

```java
@Test
public void givenSourceAndDestWithDifferentFieldTypes_
  whenMapsAndAutoConverts_thenCorrect() {
    Source2 source = new Source2("320", 15.2);
    Dest2 dest = mapper.map(source, Dest2.class);

    assertEquals(dest.getId(), 320);
    assertEquals(dest.getPoints(), 15);
}
```

我们将`“320”`和`15.2`、一个`String`和一个`double`传递给源对象，结果在目的对象中有`320`和`15,`两个`integer` s。

## 5。通过 XML 的基本自定义映射

在我们前面看到的所有例子中，源和目标数据对象都有相同的字段名称，这使得我们可以轻松地进行映射。

然而，在现实世界的应用程序中，我们映射的两个数据对象会有无数次没有共享一个公共属性名的字段。

为了解决这个问题，Dozer 给了我们一个选项，在 XML 中创建一个**自定义映射配置。**

在这个 XML 文件中，我们可以定义类映射条目，推土机映射引擎将使用这些条目来决定将哪个源属性映射到哪个目的属性。

让我们来看一个例子，让我们试着将一个法国程序员构建的应用程序中的数据对象解组为一种英国风格的对象命名。

我们有一个带有`name`、`nickname`和`age`字段的`Person`对象:

```java
public class Person {
    private String name;
    private String nickname;
    private int age;

    public Person() {}

    public Person(String name, String nickname, int age) {
        super();
        this.name = name;
        this.nickname = nickname;
        this.age = age;
    }

    // standard getters and setters
}
```

我们要解组的对象名为`Personne`，有字段`nom`、`surnom`和`age`:

```java
public class Personne {
    private String nom;
    private String surnom;
    private int age;

    public Personne() {}

    public Personne(String nom, String surnom, int age) {
        super();
        this.nom = nom;
        this.surnom = surnom;
        this.age = age;
    }

    // standard getters and setters
}
```

这些物品确实达到了同样的目的，但是我们有语言障碍。为了帮助解决这个障碍，我们可以使用 Dozer 将法国的`Personne`对象映射到我们的`Person`对象。

我们只需要创建一个定制的映射文件来帮助 Dozer 完成这项工作，我们称之为`dozer_mapping.xml`:

```java
<?xml version="1.0" encoding="UTF-8"?>
<mappings  
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://dozer.sourceforge.net
      http://dozer.sourceforge.net/schema/beanmapping.xsd">
    <mapping>
        <class-a>com.baeldung.dozer.Personne</class-a>
        <class-b>com.baeldung.dozer.Person</class-b>
        <field>
            <a>nom</a>
            <b>name</b>
        </field>
        <field>
            <a>surnom</a>
            <b>nickname</b>
        </field>
    </mapping>
</mappings>
```

这是我们可以拥有的定制 XML 映射文件的最简单的例子。

现在，注意到我们有`<mappings>`作为我们的根元素就足够了，它有一个子元素`<mapping>`，我们可以在`<mappings>`中有尽可能多的这些子元素，只要有需要定制映射的类对出现。

还要注意我们如何在`<mapping></mapping>`标签中指定源类和目的类。接下来是需要定制映射的每个源和目标字段对的`<field></field>` 。

最后，注意我们没有在自定义映射文件中包含字段`age`。法语中表示年龄的单词仍然是 age，这让我们想到了 Dozer 的另一个重要特征。

**同名的属性不需要在映射 XML 文件**中指定。Dozer 自动将所有具有相同属性名称的字段从源对象映射到目标对象。

然后，我们将自定义 XML 文件放在类路径中的`src`文件夹下。然而，无论我们把它放在类路径的什么地方，Dozer 都会搜索整个类路径来寻找指定的文件。

让我们创建一个助手方法，将映射文件添加到我们的`mapper`:

```java
public void configureMapper(String... mappingFileUrls) {
    mapper.setMappingFiles(Arrays.asList(mappingFileUrls));
}
```

现在让我们测试代码:

```java
@Test
public void givenSrcAndDestWithDifferentFieldNamesWithCustomMapper_
  whenMaps_thenCorrect() {
    configureMapper("dozer_mapping.xml");
    Personne frenchAppPerson = new Personne("Sylvester Stallone", "Rambo", 70);
    Person englishAppPerson = mapper.map(frenchAppPerson, Person.class);

    assertEquals(englishAppPerson.getName(), frenchAppPerson.getNom());
    assertEquals(englishAppPerson.getNickname(), frenchAppPerson.getSurnom());
    assertEquals(englishAppPerson.getAge(), frenchAppPerson.getAge());
}
```

如测试所示，`DozerBeanMapper`接受定制 XML 映射文件的列表，并在运行时决定何时使用每个文件。

假设我们现在开始在英语应用程序和法语应用程序之间来回解组这些数据对象。我们不需要在 XML 文件中创建另一个映射， **Dozer 很聪明，只需要一个映射配置就可以双向映射对象**:

```java
@Test
public void givenSrcAndDestWithDifferentFieldNamesWithCustomMapper_
  whenMapsBidirectionally_thenCorrect() {
    configureMapper("dozer_mapping.xml");
    Person englishAppPerson = new Person("Dwayne Johnson", "The Rock", 44);
    Personne frenchAppPerson = mapper.map(englishAppPerson, Personne.class);

    assertEquals(frenchAppPerson.getNom(), englishAppPerson.getName());
    assertEquals(frenchAppPerson.getSurnom(),englishAppPerson.getNickname());
    assertEquals(frenchAppPerson.getAge(), englishAppPerson.getAge());
}
```

因此，这个示例测试使用了 Dozer 的另一个特性——Dozer 映射引擎**是双向的**,因此，如果我们想要将目的对象映射到源对象，我们不需要向 XML 文件添加另一个类映射。

我们也可以从类路径外部加载一个定制的映射文件，如果我们需要，可以在资源名中使用前缀“`file:`”。

在 Windows 环境中(比如下面的测试)，我们当然会使用 Windows 特定的文件语法。

在 Linux 机器上，我们可以将文件存储在`/home` 下，然后:

```java
configureMapper("file:/home/dozer_mapping.xml");
```

在 Mac OS 上:

```java
configureMapper("file:/Users/me/dozer_mapping.xml");
```

如果您正在运行来自 [github 项目](https://web.archive.org/web/20220709012055/https://github.com/eugenp/tutorials/tree/master/dozer)的单元测试(您应该这样做)，您可以将映射文件复制到适当的位置，并更改`configureMapper`方法的输入。

该映射文件位于 GitHub 项目的 test/resources 文件夹下:

```java
@Test
public void givenMappingFileOutsideClasspath_whenMaps_thenCorrect() {
    configureMapper("file:E:\\dozer_mapping.xml");
    Person englishAppPerson = new Person("Marshall Bruce Mathers III","Eminem", 43);
    Personne frenchAppPerson = mapper.map(englishAppPerson, Personne.class);

    assertEquals(frenchAppPerson.getNom(), englishAppPerson.getName());
    assertEquals(frenchAppPerson.getSurnom(),englishAppPerson.getNickname());
    assertEquals(frenchAppPerson.getAge(), englishAppPerson.getAge());
}
```

## 6。通配符和进一步的 XML 定制

让我们创建第二个名为`dozer_mapping2.xml`的定制映射文件:

```java
<?xml version="1.0" encoding="UTF-8"?>
<mappings  
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://dozer.sourceforge.net 
      http://dozer.sourceforge.net/schema/beanmapping.xsd">
    <mapping wildcard="false">
        <class-a>com.baeldung.dozer.Personne</class-a>
        <class-b>com.baeldung.dozer.Person</class-b>
        <field>
            <a>nom</a>
            <b>name</b>
        </field>
        <field>
            <a>surnom</a>
            <b>nickname</b>
        </field>
    </mapping>
</mappings>
```

请注意，我们已经为元素`<mapping></mapping>`添加了一个属性`wildcard`，这是之前没有的。

默认情况下，`wildcard`为`true`。它告诉推土机引擎，我们希望源对象中的所有字段都映射到相应的目标字段。

当我们将它设置为`false,`时，我们告诉 Dozer 只映射我们在 XML 中明确指定的字段。

所以在上面的配置中，我们只想要映射两个字段，省略了`age`:

```java
@Test
public void givenSrcAndDest_whenMapsOnlySpecifiedFields_thenCorrect() {
    configureMapper("dozer_mapping2.xml");
    Person englishAppPerson = new Person("Shawn Corey Carter","Jay Z", 46);
    Personne frenchAppPerson = mapper.map(englishAppPerson, Personne.class);

    assertEquals(frenchAppPerson.getNom(), englishAppPerson.getName());
    assertEquals(frenchAppPerson.getSurnom(),englishAppPerson.getNickname());
    assertEquals(frenchAppPerson.getAge(), 0);
}
```

正如我们在最后一个断言中看到的，目的地`age`字段仍然是`0`。

## 7。通过注释自定义映射

对于简单的映射情况和我们对想要映射的数据对象也有写访问权的情况，我们可能不需要使用 XML 映射。

通过注释映射不同命名的字段非常简单，我们必须编写比 XML 映射少得多的代码，但只能在简单的情况下帮助我们。

让我们将数据对象复制到`Person2.java`和`Personne2.java`中，而完全不改变字段。

为了实现这一点，我们只需要在源对象中的`getter`方法上添加@ `mapper(“destinationFieldName”)`注释。像这样:

```java
@Mapping("name")
public String getNom() {
    return nom;
}

@Mapping("nickname")
public String getSurnom() {
    return surnom;
}
```

这次我们将`Personne2`视为源头，但由于推土机发动机的**双向性质**，这并不重要。

现在去掉了所有与 XML 相关的代码，我们的测试代码更短了:

```java
@Test
public void givenAnnotatedSrcFields_whenMapsToRightDestField_thenCorrect() {
    Person2 englishAppPerson = new Person2("Jean-Claude Van Damme", "JCVD", 55);
    Personne2 frenchAppPerson = mapper.map(englishAppPerson, Personne2.class);

    assertEquals(frenchAppPerson.getNom(), englishAppPerson.getName());
    assertEquals(frenchAppPerson.getSurnom(), englishAppPerson.getNickname());
    assertEquals(frenchAppPerson.getAge(), englishAppPerson.getAge());
}
```

我们也可以测试双向的:

```java
@Test
public void givenAnnotatedSrcFields_whenMapsToRightDestFieldBidirectionally_
  thenCorrect() {
    Personne2 frenchAppPerson = new Personne2("Jason Statham", "transporter", 49);
    Person2 englishAppPerson = mapper.map(frenchAppPerson, Person2.class);

    assertEquals(englishAppPerson.getName(), frenchAppPerson.getNom());
    assertEquals(englishAppPerson.getNickname(), frenchAppPerson.getSurnom());
    assertEquals(englishAppPerson.getAge(), frenchAppPerson.getAge());
}
```

## 8。自定义 API 映射

在前面的例子中，我们从一个法语应用程序中解组数据对象，我们使用 XML 和注释来定制映射。

Dozer 中另一个类似于注释映射的选择是 API 映射。它们是相似的，因为我们消除了 XML 配置并严格使用 Java 代码。

在这种情况下，我们使用`BeanMappingBuilder`类，在我们最简单的情况下定义如下:

```java
BeanMappingBuilder builder = new BeanMappingBuilder() {
    @Override
    protected void configure() {
        mapping(Person.class, Personne.class)
          .fields("name", "nom")
            .fields("nickname", "surnom");
    }
};
```

如我们所见，我们有一个抽象方法`configure()`，我们必须覆盖它来定义我们的配置。然后，就像我们在 XML 中的`<mapping></mapping>`标签一样，我们根据需要定义尽可能多的`TypeMappingBuilder`。

这些构建器告诉推土机我们正在映射哪个源到目的地字段。然后，我们将 XML 映射文件从`BeanMappingBuilder`传递到`DozerBeanMapper`，只是使用了不同的 API:

```java
@Test
public void givenApiMapper_whenMaps_thenCorrect() {
    mapper.addMapping(builder);

    Personne frenchAppPerson = new Personne("Sylvester Stallone", "Rambo", 70);
    Person englishAppPerson = mapper.map(frenchAppPerson, Person.class);

    assertEquals(englishAppPerson.getName(), frenchAppPerson.getNom());
    assertEquals(englishAppPerson.getNickname(), frenchAppPerson.getSurnom());
    assertEquals(englishAppPerson.getAge(), frenchAppPerson.getAge());
}
```

映射 API 也是双向的:

```java
@Test
public void givenApiMapper_whenMapsBidirectionally_thenCorrect() {
    mapper.addMapping(builder);

    Person englishAppPerson = new Person("Sylvester Stallone", "Rambo", 70);
    Personne frenchAppPerson = mapper.map(englishAppPerson, Personne.class);

    assertEquals(frenchAppPerson.getNom(), englishAppPerson.getName());
    assertEquals(frenchAppPerson.getSurnom(), englishAppPerson.getNickname());
    assertEquals(frenchAppPerson.getAge(), englishAppPerson.getAge());
}
```

或者，我们可以选择仅使用此构建器配置来映射显式指定的字段:

```java
BeanMappingBuilder builderMinusAge = new BeanMappingBuilder() {
    @Override
    protected void configure() {
        mapping(Person.class, Personne.class)
          .fields("name", "nom")
            .fields("nickname", "surnom")
              .exclude("age");
    }
};
```

我们的`age==0`测试回来了:

```java
@Test
public void givenApiMapper_whenMapsOnlySpecifiedFields_thenCorrect() {
    mapper.addMapping(builderMinusAge); 
    Person englishAppPerson = new Person("Sylvester Stallone", "Rambo", 70);
    Personne frenchAppPerson = mapper.map(englishAppPerson, Personne.class);

    assertEquals(frenchAppPerson.getNom(), englishAppPerson.getName());
    assertEquals(frenchAppPerson.getSurnom(), englishAppPerson.getNickname());
    assertEquals(frenchAppPerson.getAge(), 0);
}
```

## 9。定制转换器

我们在映射中可能面临的另一个场景是，我们希望**在两个对象**之间执行自定义映射。

我们已经看了源和目的字段名不同的场景，就像法语的`Personne`对象。本节解决了一个不同的问题。

如果我们要解组的数据对象表示一个日期和时间字段，比如一个`long`或 Unix 时间，如下所示:

```java
1182882159000
```

但是我们自己的等价数据对象以这种 ISO 格式表示相同的日期和时间字段和值，比如一个`String:`

```java
2007-06-26T21:22:39Z
```

默认的转换器会简单地将长值映射到一个`String`,如下所示:

```java
"1182882159000"
```

这肯定会影响我们的应用程序。那么我们如何解决这个问题呢？我们通过**在映射 XML 文件中添加一个配置块**和**指定我们自己的转换器**来解决这个问题。

首先，让我们用一个`name,`字段复制远程应用程序的`Person` DTO，然后是出生日期和时间，`dtob`字段:

```java
public class Personne3 {
    private String name;
    private long dtob;

    public Personne3(String name, long dtob) {
        super();
        this.name = name;
        this.dtob = dtob;
    }

    // standard getters and setters
}
```

这是我们自己的:

```java
public class Person3 {
    private String name;
    private String dtob;

    public Person3(String name, String dtob) {
        super();
        this.name = name;
        this.dtob = dtob;
    }

    // standard getters and setters
}
```

请注意源和目标 dto 中`dtob`的类型差异。

让我们也创建我们自己的`CustomConverter`以在映射 XML 中传递给 Dozer:

```java
public class MyCustomConvertor implements CustomConverter {
    @Override
    public Object convert(Object dest, Object source, Class<?> arg2, Class<?> arg3) {
        if (source == null) 
            return null;

        if (source instanceof Personne3) {
            Personne3 person = (Personne3) source;
            Date date = new Date(person.getDtob());
            DateFormat format = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss'Z'");
            String isoDate = format.format(date);
            return new Person3(person.getName(), isoDate);

        } else if (source instanceof Person3) {
            Person3 person = (Person3) source;
            DateFormat format = new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss'Z'");
            Date date = format.parse(person.getDtob());
            long timestamp = date.getTime();
            return new Personne3(person.getName(), timestamp);
        }
    }
}
```

我们只需要覆盖`convert()`方法，然后返回我们想返回的任何内容。我们可以利用源和目标对象以及它们的类类型。

注意我们是如何通过假设源可以是我们正在映射的两个类中的任何一个来处理双向的。

为了清楚起见，我们将创建一个新的映射文件，`dozer_custom_convertor.xml`:

```java
<?xml version="1.0" encoding="UTF-8"?>
<mappings  
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://dozer.sourceforge.net
      http://dozer.sourceforge.net/schema/beanmapping.xsd">
    <configuration>
        <custom-converters>
            <converter type="com.baeldung.dozer.MyCustomConvertor">
                <class-a>com.baeldung.dozer.Personne3</class-a>
                <class-b>com.baeldung.dozer.Person3</class-b>
            </converter>
        </custom-converters>
    </configuration>
</mappings>
```

这是我们在前面几节中看到的普通映射文件，我们只添加了一个`<configuration></configuration>`块，在这个块中，我们可以根据需要定义任意数量的自定义转换器，以及它们各自的源和目标数据类。

让我们测试一下新的`CustomConverter`代码:

```java
@Test
public void givenSrcAndDestWithDifferentFieldTypes_whenAbleToCustomConvert_
  thenCorrect() {

    configureMapper("dozer_custom_convertor.xml");
    String dateTime = "2007-06-26T21:22:39Z";
    long timestamp = new Long("1182882159000");
    Person3 person = new Person3("Rich", dateTime);
    Personne3 person0 = mapper.map(person, Personne3.class);

    assertEquals(timestamp, person0.getDtob());
}
```

我们还可以测试以确保它是双向的:

```java
@Test
public void givenSrcAndDestWithDifferentFieldTypes_
  whenAbleToCustomConvertBidirectionally_thenCorrect() {
    configureMapper("dozer_custom_convertor.xml");
    String dateTime = "2007-06-26T21:22:39Z";
    long timestamp = new Long("1182882159000");
    Personne3 person = new Personne3("Rich", timestamp);
    Person3 person0 = mapper.map(person, Person3.class);

    assertEquals(dateTime, person0.getDtob());
}
```

## 10。结论

在本教程中，我们已经**介绍了推土机映射库**的大部分基础知识，以及如何在我们的应用中使用它。

所有这些例子和代码片段的完整实现可以在 Dozer [github 项目](https://web.archive.org/web/20220709012055/https://github.com/eugenp/tutorials/tree/master/dozer)中找到。