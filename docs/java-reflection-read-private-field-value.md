# 从 Java 的不同类中读取“私有”字段的值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-reflection-read-private-field-value>

## 1.概观

在这个快速教程中，我们将讨论如何从 Java 的不同类中访问一个`[private](/web/20221208143832/https://www.baeldung.com/java-private-keyword)`字段的值。

在开始本教程之前，我们需要理解,`private`访问修饰符可以防止意外误用字段。然而，如果我们希望访问它们，我们可以通过使用[反射](/web/20221208143832/https://www.baeldung.com/java-reflection) API 来实现。

## 2.例子

让我们用一些`private`字段定义一个样本类`Person`:

```java
public class Person {

    private String name = "John";
    private byte age = 30;
    private short uidNumber = 5555;
    private int pinCode = 452002;
    private long contactNumber = 123456789L;
    private float height = 6.1242f;
    private double weight = 75.2564;
    private char gender = 'M';
    private boolean active = true;

    // getters and setters
}
```

## 3.使`private`字段可访问

为了使任何`private`字段可访问，**我们必须调用`Field#setAccessible`方法:**

```java
Person person = new Person(); 
Field nameField = person.getClass().getDeclaredField("name"); 
nameField.setAccessible(true);
```

在上面的例子中，我们首先通过使用 **`Class#getDeclaredField`** 方法来指定我们想要检索的字段——`name`。然后，我们使用`nameField.setAccessible(true)`使字段可访问。

## 4.访问`private`原始字段

我们可以通过使用`Field#getXxx`方法来**访问作为原语的`private`字段。**

### 4.1.访问整数字段

我们可以使用`getByte,` `getShort`、`getInt`和`getLong`方法分别访问`byte` `,` `short`、`int`和`long`字段:

```java
@Test
public void whenGetIntegerFields_thenSuccess() 
  throws Exception {
    Person person = new Person();

    Field ageField = person.getClass().getDeclaredField("age");
    ageField.setAccessible(true);

    byte age = ageField.getByte(person);
    Assertions.assertEquals(30, age);

    Field uidNumberField = person.getClass().getDeclaredField("uidNumber");
    uidNumberField.setAccessible(true);

    short uidNumber = uidNumberField.getShort(person);
    Assertions.assertEquals(5555, uidNumber);

    Field pinCodeField = person.getClass().getDeclaredField("pinCode");
    pinCodeField.setAccessible(true);

    int pinCode = pinCodeField.getInt(person);
    Assertions.assertEquals(452002, pinCode);

    Field contactNumberField = person.getClass().getDeclaredField("contactNumber");
    contactNumberField.setAccessible(true);

    long contactNumber = contactNumberField.getLong(person);
    Assertions.assertEquals(123456789L, contactNumber);
}
```

**也可以用原始类型**执行[自动装箱](/web/20221208143832/https://www.baeldung.com/java-wrapper-classes#autoboxing-and-unboxing)

```java
@Test
public void whenDoAutoboxing_thenSuccess() 
  throws Exception {
    Person person = new Person();

    Field pinCodeField = person.getClass().getDeclaredField("pinCode");
    pinCodeField.setAccessible(true);

    Integer pinCode = pinCodeField.getInt(person);
    Assertions.assertEquals(452002, pinCode);
}
```

**原始数据类型的`getXxx`方法也支持[扩展](/web/20221208143832/https://www.baeldung.com/java-primitive-conversions#widening-primitive-conversions) :**

```java
@Test
public void whenDoWidening_thenSuccess() 
  throws Exception {
    Person person = new Person();

    Field pinCodeField = person.getClass().getDeclaredField("pinCode");
    pinCodeField.setAccessible(true);

    Long pinCode = pinCodeField.getLong(person);
    Assertions.assertEquals(452002L, pinCode);
}
```

### 4.2.访问浮点型字段

为了访问`float`和`double`字段，我们需要分别使用`getFloat`和`getDouble`方法:

```java
@Test
public void whenGetFloatingTypeFields_thenSuccess() 
  throws Exception {
    Person person = new Person();

    Field heightField = person.getClass().getDeclaredField("height");
    heightField.setAccessible(true);

    float height = heightField.getFloat(person);
    Assertions.assertEquals(6.1242f, height);

    Field weightField = person.getClass().getDeclaredField("weight");
    weightField.setAccessible(true);

    double weight = weightField.getDouble(person);
    Assertions.assertEquals(75.2564, weight);
}
```

### 4.3.访问字符字段

要访问`char`字段，我们可以使用`getChar`方法:

```java
@Test
public void whenGetCharacterFields_thenSuccess() 
  throws Exception {
    Person person = new Person();

    Field genderField = person.getClass().getDeclaredField("gender");
    genderField.setAccessible(true);

    char gender = genderField.getChar(person);
    Assertions.assertEquals('M', gender);
}
```

### 4.4.访问布尔字段

类似地，我们可以使用`getBoolean`方法来访问`boolean`字段:

```java
@Test
public void whenGetBooleanFields_thenSuccess() 
  throws Exception {
    Person person = new Person();

    Field activeField = person.getClass().getDeclaredField("active");
    activeField.setAccessible(true);

    boolean active = activeField.getBoolean(person);
    Assertions.assertTrue(active);
}
```

## 5.访问作为对象的`private`字段

我们可以通过使用`Field#get`方法来**访问作为对象的`private`字段。需要注意的是，通用的`get`方法**返回一个`Object`，所以我们需要将它转换成目标类型以利用值**:**

```java
@Test
public void whenGetObjectFields_thenSuccess() 
  throws Exception {
    Person person = new Person();

    Field nameField = person.getClass().getDeclaredField("name");
    nameField.setAccessible(true);

    String name = (String) nameField.get(person);
    Assertions.assertEquals("John", name);
}
```

## 6.例外

现在，让我们讨论 JVM 在访问`private`字段时可能抛出的异常。

### 6.1.`IllegalArgumentException`

如果我们使用与目标字段的类型不兼容的`getXxx`访问器，JVM 将抛出`IllegalArgumentException` **。在我们的例子中，如果我们写`nameField.getInt(person)`，JVM 抛出这个异常，因为字段的类型是`String`，而不是`int`或`Integer`:**

```java
@Test
public void givenInt_whenSetStringField_thenIllegalArgumentException() 
  throws Exception {
    Person person = new Person();
    Field nameField = person.getClass().getDeclaredField("name");
    nameField.setAccessible(true);

    Assertions.assertThrows(IllegalArgumentException.class, () -> nameField.getInt(person));
}
```

正如我们已经看到的，`getXxx`方法支持基本类型的扩展。需要注意的是**我们需要提供正确的扩展目标，这样才能取得成功**。否则，JVM 抛出一个`IllegalArgumentException`:

```java
@Test
public void givenInt_whenGetLongField_thenIllegalArgumentException() 
  throws Exception {
    Person person = new Person();
    Field contactNumberField = person.getClass().getDeclaredField("contactNumber");
    contactNumberField.setAccessible(true);

    Assertions.assertThrows(IllegalArgumentException.class, () -> contactNumberField.getInt(person));
}
```

### 6.2.`IllegalAccessException`

如果我们试图访问一个没有访问权限的字段，JVM 将抛出一个`IllegalAccessException` **。在上面的例子中，如果我们不写语句`nameField.setAccessible(true)`，那么 JVM 抛出异常:**

```java
@Test
public void whenFieldNotSetAccessible_thenIllegalAccessException() 
  throws Exception {
    Person person = new Person();
    Field nameField = person.getClass().getDeclaredField("name");

    Assertions.assertThrows(IllegalAccessException.class, () -> nameField.get(person));
}
```

### 6.3.`NoSuchFieldException`

**如果我们试图访问一个在`Person`类中不存在的字段**，那么 JVM 可能会抛出`NoSuchFieldException`:

```java
Assertions.assertThrows(NoSuchFieldException.class,
  () -> person.getClass().getDeclaredField("firstName"));
```

### 6.4.`NullPointerException`

最后，如您所料，如果我们将字段名作为`null` 传递，JVM 会抛出一个`NullPointerException` **:**

```java
Assertions.assertThrows(NullPointerException.class,
  () -> person.getClass().getDeclaredField(null));
```

## 7.结论

在本教程中，我们已经看到了如何在另一个类中访问一个类的`private`字段。我们还看到了 JVM 可能抛出的异常以及导致这些异常的原因。

和往常一样，这个例子的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-reflection-2)