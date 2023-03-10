# JPA 属性转换器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-attribute-converters>

## 1。简介

在这篇简短的文章中，我们将介绍 JPA 2.1 中可用的属性转换器的用法——简单地说，它允许我们将 JDBC 类型映射到 Java 类。

这里我们将使用 Hibernate 5 作为我们的 JPA 实现。

## 2。创建转换器

我们将展示如何为一个定制的 Java 类实现一个属性转换器。

首先，让我们创建一个`PersonName`类——稍后将被转换:

```java
public class PersonName implements Serializable {

    private String name;
    private String surname;

    // getters and setters
}
```

然后，我们将把类型为`PersonName`的属性添加到一个`@Entity`类中:

```java
@Entity(name = "PersonTable")
public class Person {

    private PersonName personName;

    //...
}
```

现在我们需要创建一个转换器，将`PersonName`属性转换成数据库列，反之亦然。在我们的例子中，我们将属性转换为包含姓名字段的`String`值。

为此，**我们必须用`@Converter` 注释我们的转换器类，并实现`AttributeConverter` 接口。**我们将用类的类型和数据库列来参数化接口，顺序如下:

```java
@Converter
public class PersonNameConverter implements 
  AttributeConverter<PersonName, String> {

    private static final String SEPARATOR = ", ";

    @Override
    public String convertToDatabaseColumn(PersonName personName) {
        if (personName == null) {
            return null;
        }

        StringBuilder sb = new StringBuilder();
        if (personName.getSurname() != null && !personName.getSurname()
            .isEmpty()) {
            sb.append(personName.getSurname());
            sb.append(SEPARATOR);
        }

        if (personName.getName() != null 
          && !personName.getName().isEmpty()) {
            sb.append(personName.getName());
        }

        return sb.toString();
    }

    @Override
    public PersonName convertToEntityAttribute(String dbPersonName) {
        if (dbPersonName == null || dbPersonName.isEmpty()) {
            return null;
        }

        String[] pieces = dbPersonName.split(SEPARATOR);

        if (pieces == null || pieces.length == 0) {
            return null;
        }

        PersonName personName = new PersonName();        
        String firstPiece = !pieces[0].isEmpty() ? pieces[0] : null;
        if (dbPersonName.contains(SEPARATOR)) {
            personName.setSurname(firstPiece);

            if (pieces.length >= 2 && pieces[1] != null 
              && !pieces[1].isEmpty()) {
                personName.setName(pieces[1]);
            }
        } else {
            personName.setName(firstPiece);
        }

        return personName;
    }
}
```

**注意，我们必须实现两个方法:`convertToDatabaseColumn()`和`convertToEntityAttribute().`**

这两种方法用于从属性转换到数据库列，反之亦然。

## 3。使用转换器

**要使用我们的转换器，我们只需要给属性添加`@Convert`注释，并指定我们想要使用的转换器类**:

```java
@Entity(name = "PersonTable")
public class Person {

    @Convert(converter = PersonNameConverter.class)
    private PersonName personName;

    // ...
}
```

最后，让我们创建一个单元测试，看看它是否真的有效。

为此，我们将首先在数据库中存储一个`Person`对象:

```java
@Test
public void givenPersonName_whenSaving_thenNameAndSurnameConcat() {
    String name = "name";
    String surname = "surname";

    PersonName personName = new PersonName();
    personName.setName(name);
    personName.setSurname(surname);

    Person person = new Person();
    person.setPersonName(personName);

    Long id = (Long) session.save(person);

    session.flush();
    session.clear();
}
```

接下来，我们将测试`PersonName`是否按照我们在转换器中定义的那样存储——通过从数据库表中检索该字段:

```java
@Test
public void givenPersonName_whenSaving_thenNameAndSurnameConcat() {
    // ...

    String dbPersonName = (String) session.createNativeQuery(
      "select p.personName from PersonTable p where p.id = :id")
      .setParameter("id", id)
      .getSingleResult();

    assertEquals(surname + ", " + name, dbPersonName);
}
```

让我们通过编写一个检索整个`Person`类的查询来测试从数据库中存储的值到`PersonName`类的转换是否如转换器中定义的那样工作:

```java
@Test
public void givenPersonName_whenSaving_thenNameAndSurnameConcat() {
    // ...

    Person dbPerson = session.createNativeQuery(
      "select * from PersonTable p where p.id = :id", Person.class)
        .setParameter("id", id)
        .getSingleResult();

    assertEquals(dbPerson.getPersonName()
      .getName(), name);
    assertEquals(dbPerson.getPersonName()
      .getSurname(), surname);
}
```

## 4。结论

在这篇简短的教程中，我们展示了如何使用 JPA 2.1 中新引入的属性转换器。

与往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220626200821/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-jpa)