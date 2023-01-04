# 设置带反射的字段值

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-set-private-field-value>

## 1.概观

在我们的[上一篇文章](/web/20220630014027/https://www.baeldung.com/java-reflection-read-private-field-value)中，我们讨论了如何从 Java 的不同类中读取`private`字段的值。然而，有些情况下我们需要设置字段的值，比如在一些库中我们没有访问字段的权限。

在这个快速教程中，我们将讨论如何通过使用[反射](/web/20220630014027/https://www.baeldung.com/java-reflection) API 来设置 Java 中不同类的字段值。

**注意，我们将在这里的例子中使用与我们在[上一篇文章](/web/20220630014027/https://www.baeldung.com/java-reflection-read-private-field-value)中使用的相同的`Person`类。**

## 2.设置原始字段

我们可以通过使用`Field#setXxx`方法来**设置作为原语的字段。**

### 2.1.设置整数字段

我们可以使用`setByte,` `setShort`、s `etInt`和`setLong`方法分别设置`byte` `,` `short`、`int`和`long`字段:

```java
@Test
public void whenSetIntegerFields_thenSuccess() 
  throws Exception {
    Person person = new Person();

    Field ageField = person.getClass()
        .getDeclaredField("age");
    ageField.setAccessible(true);

    byte age = 26;
    ageField.setByte(person, age);
    Assertions.assertEquals(age, person.getAge());

    Field uidNumberField = person.getClass()
        .getDeclaredField("uidNumber");
    uidNumberField.setAccessible(true);

    short uidNumber = 5555;
    uidNumberField.setShort(person, uidNumber);
    Assertions.assertEquals(uidNumber, person.getUidNumber());

    Field pinCodeField = person.getClass()
        .getDeclaredField("pinCode");
    pinCodeField.setAccessible(true);

    int pinCode = 411057;
    pinCodeField.setInt(person, pinCode);
    Assertions.assertEquals(pinCode, person.getPinCode());

    Field contactNumberField = person.getClass()
        .getDeclaredField("contactNumber");
    contactNumberField.setAccessible(true);

    long contactNumber = 123456789L;
    contactNumberField.setLong(person, contactNumber);
    Assertions.assertEquals(contactNumber, person.getContactNumber());

}
```

**也可以对原始类型**执行[拆箱](/web/20220630014027/https://www.baeldung.com/java-wrapper-classes#autoboxing-and-unboxing)

```java
@Test
public void whenDoUnboxing_thenSuccess() 
  throws Exception {
    Person person = new Person();

    Field pinCodeField = person.getClass()
        .getDeclaredField("pinCode");
    pinCodeField.setAccessible(true);

    Integer pinCode = 411057;
    pinCodeField.setInt(person, pinCode);
    Assertions.assertEquals(pinCode, person.getPinCode());
}
```

**原始数据类型的 s `etXxx`方法也支持[缩小](/web/20220630014027/https://www.baeldung.com/java-primitive-conversions#widening-primitive-conversions) :**

```java
@Test
public void whenDoNarrowing_thenSuccess() 
  throws Exception {
    Person person = new Person();

    Field pinCodeField = person.getClass()
        .getDeclaredField("pinCode");
    pinCodeField.setAccessible(true);

    short pinCode = 4110;
    pinCodeField.setInt(person, pinCode);
    Assertions.assertEquals(pinCode, person.getPinCode());
}
```

### 2.2.设置浮动类型字段

为了设置`float`和`double`字段，我们需要分别使用`setFloat`和`setDouble`方法:

```java
@Test
public void whenSetFloatingTypeFields_thenSuccess() 
  throws Exception {
    Person person = new Person();

    Field heightField = person.getClass()
        .getDeclaredField("height");
    heightField.setAccessible(true);

    float height = 6.1242f;
    heightField.setFloat(person, height);
    Assertions.assertEquals(height, person.getHeight());

    Field weightField = person.getClass()
        .getDeclaredField("weight");
    weightField.setAccessible(true);

    double weight = 75.2564;
    weightField.setDouble(person, weight);
    Assertions.assertEquals(weight, person.getWeight());
}
```

### 2.3.设置字符字段

要设置`char`字段，我们可以使用`setChar`方法:

```java
@Test
public void whenSetCharacterFields_thenSuccess() 
  throws Exception {
    Person person = new Person();

    Field genderField = person.getClass()
        .getDeclaredField("gender");
    genderField.setAccessible(true);

    char gender = 'M';
    genderField.setChar(person, gender);
    Assertions.assertEquals(gender, person.getGender());
}
```

### 2.4.设置布尔字段

类似地，我们可以使用`setBoolean`方法来设置`boolean`字段:

```java
@Test
public void whenSetBooleanFields_thenSuccess() 
  throws Exception {
    Person person = new Person();

    Field activeField = person.getClass()
        .getDeclaredField("active");
    activeField.setAccessible(true);

    activeField.setBoolean(person, true);
    Assertions.assertTrue(person.isActive());
}
```

## 3.设置作为对象的字段

我们可以通过使用`Field#set`方法来**设置作为对象的字段:**

```java
@Test
public void whenSetObjectFields_thenSuccess() 
  throws Exception {
    Person person = new Person();

    Field nameField = person.getClass()
        .getDeclaredField("name");
    nameField.setAccessible(true);

    String name = "Umang Budhwar";
    nameField.set(person, name);
    Assertions.assertEquals(name, person.getName());
}
```

## 4.例外

现在，让我们讨论 JVM 在设置字段时可能抛出的异常。

### 4.1.`IllegalArgumentException`

如果我们使用与目标字段的类型不兼容的`setXxx`赋值函数，JVM 将抛出`IllegalArgumentException` **。在我们的例子中，如果我们写`nameField.setInt(person, 26)`，JVM 抛出这个异常，因为字段的类型是`String`，而不是`int`或`Integer`:**

```java
@Test
public void givenInt_whenSetStringField_thenIllegalArgumentException() 
  throws Exception {
    Person person = new Person();
    Field nameField = person.getClass()
        .getDeclaredField("name");
    nameField.setAccessible(true);

    Assertions.assertThrows(IllegalArgumentException.class, () -> nameField.setInt(person, 26));
}
```

正如我们已经看到的，s `etXxx`方法支持基本类型的缩小。需要注意的是**我们需要提供正确的目标来成功缩小范围**。否则，JVM 抛出一个`IllegalArgumentException`:

```java
@Test
public void givenInt_whenSetLongField_thenIllegalArgumentException() 
  throws Exception {
    Person person = new Person();

    Field pinCodeField = person.getClass()
        .getDeclaredField("pinCode");
    pinCodeField.setAccessible(true);

    long pinCode = 411057L;

    Assertions.assertThrows(IllegalArgumentException.class, () -> pinCodeField.setLong(person, pinCode));
}
```

### 4.2.`IllegalAccessException`

**如果我们试图设置一个没有访问权限**的`private` 字段，那么 JVM 将抛出一个`IllegalAccessException`。在上面的例子中，如果我们不写语句`nameField.setAccessible(true)`，那么 JVM 抛出异常:

```java
@Test
public void whenFieldNotSetAccessible_thenIllegalAccessException() 
  throws Exception {
    Person person = new Person();
    Field nameField = person.getClass()
        .getDeclaredField("name");

    Assertions.assertThrows(IllegalAccessException.class, () -> nameField.set(person, "Umang Budhwar"));
}
```

## 5.结论

在本教程中，我们看到了如何在 Java 中修改或设置一个类的私有字段的值。我们还看到了 JVM 可能抛出的异常以及导致这些异常的原因。

和往常一样，这个例子的完整代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220630014027/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-reflection-2)