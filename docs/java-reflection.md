# Java 反射指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-reflection>

## 1。概述

在本教程中，我们将探索 Java 反射，它允许我们检查和/或修改类、接口、字段和方法的运行时属性。当我们在编译时不知道它们的名字时，这特别有用。

此外，我们可以使用反射实例化新对象、调用方法以及获取或设置字段值。

## 2。项目设置

为了使用 Java 反射，我们不需要包含任何特殊的 jar，任何特殊的配置或者 Maven 依赖。JDK 附带了一组专门用于此目的的类，它们被打包在 [`java.lang.reflect`](https://web.archive.org/web/20220609164400/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/package-summary.html) 包中。

因此，我们需要做的就是在代码中导入以下内容:

```java
import java.lang.reflect.*;
```

我们准备好了。

为了访问实例的类、方法和字段信息，我们调用`getClass`方法，该方法返回对象的运行时类表示。返回的`class`对象提供了访问类信息的方法。

## 3。简单的例子

为了熟悉一下，我们将看一个非常基本的例子，它在运行时检查一个简单 Java 对象的字段。

让我们创建一个简单的`Person`类，只有`name`和`age`字段，没有任何方法。

下面是 Person 类:

```java
public class Person {
    private String name;
    private int age;
}
```

我们现在将使用 Java 反射来发现这个类的所有字段的名称。

为了体会反射的力量，让我们构造一个`Person`对象，并使用 object 作为引用类型:

```java
@Test
public void givenObject_whenGetsFieldNamesAtRuntime_thenCorrect() {
    Object person = new Person();
    Field[] fields = person.getClass().getDeclaredFields();

    List<String> actualFieldNames = getFieldNames(fields);

    assertTrue(Arrays.asList("name", "age")
      .containsAll(actualFieldNames));
}
```

这个测试向我们展示了我们能够从我们的`person`对象中获得一个`F` `ield`对象的数组，即使对该对象的引用是该对象的父类型。

在上面的例子中，我们只对那些字段的名称感兴趣。但是还有更多的事情可以做，我们将在接下来的章节中看到这样的例子。

注意我们如何使用一个助手方法来提取实际的字段名称。

这是一个非常基本的代码:

```java
private static List<String> getFieldNames(Field[] fields) {
    List<String> fieldNames = new ArrayList<>();
    for (Field field : fields)
      fieldNames.add(field.getName());
    return fieldNames;
}
```

## 4。Java 反射用例

在我们讨论 Java 反射的不同特性之前，我们将讨论它的一些常见用途。Java 反射非常强大，可以在很多方面派上用场。

例如，在许多情况下，我们有一个数据库表的命名约定。我们可以选择在表名前面加上前缀`tbl_`来增加一致性，这样包含学生数据的表就叫做`tbl_student_data`。

在这种情况下，我们可以将保存学生数据的 Java 对象命名为`Student`或`StudentData`。然后使用 CRUD 范例，我们为每个操作都有一个入口点，这样`Create`操作只接收一个`Object`参数。

然后，我们使用反射来检索对象名和字段名。此时，我们可以将这些数据映射到一个 DB 表，并将对象字段值分配给适当的 DB 字段名。

## 5。检查 Java 类

在这一节中，我们将探索 Java 反射 API 中最基本的组件。正如我们前面提到的，Java 类对象使我们能够访问任何对象的内部细节。

我们将检查内部细节，如对象的类名、修饰符、字段、方法、实现的接口等。

### 5.1。准备就绪

为了更好地理解应用于 Java 类的反射 API，并有各种各样的例子，让我们创建一个实现了`Eating`接口的抽象`Animal`类。这个接口定义了我们创建的任何具体的`Animal`对象的进食行为。

首先，这里是`Eating`界面:

```java
public interface Eating {
    String eats();
}
```

下面是`Eating`接口的具体`Animal`实现:

```java
public abstract class Animal implements Eating {

    public static String CATEGORY = "domestic";
    private String name;

    protected abstract String getSound();

    // constructor, standard getters and setters omitted 
}
```

让我们也创建另一个名为`Locomotion`的界面来描述动物是如何移动的:

```java
public interface Locomotion {
    String getLocomotion();
}
```

我们现在将创建一个名为`Goat`的具体类，它扩展了`Animal`并实现了`Locomotion`。

由于超类实现了`Eating`，`Goat`也必须实现该接口的方法:

```java
public class Goat extends Animal implements Locomotion {

    @Override
    protected String getSound() {
        return "bleat";
    }

    @Override
    public String getLocomotion() {
        return "walks";
    }

    @Override
    public String eats() {
        return "grass";
    }

    // constructor omitted
}
```

从现在开始，我们将使用 Java 反射来检查出现在上面的类和接口中的 Java 对象的各个方面。

### 5.2。类名

让我们从从`Class`中获取一个对象的名称开始:

```java
@Test
public void givenObject_whenGetsClassName_thenCorrect() {
    Object goat = new Goat("goat");
    Class<?> clazz = goat.getClass();

    assertEquals("Goat", clazz.getSimpleName());
    assertEquals("com.baeldung.reflection.Goat", clazz.getName());
    assertEquals("com.baeldung.reflection.Goat", clazz.getCanonicalName());
}
```

注意，`Class`的`getSimpleName`方法返回对象的基本名称，就像它在声明中出现的那样。然后另外两个方法返回包括包声明的完全限定类名。

让我们看看如何创建一个`Goat`类的对象，如果我们只知道它的完全限定类名:

```java
@Test
public void givenClassName_whenCreatesObject_thenCorrect(){
    Class<?> clazz = Class.forName("com.baeldung.reflection.Goat");

    assertEquals("Goat", clazz.getSimpleName());
    assertEquals("com.baeldung.reflection.Goat", clazz.getName());
    assertEquals("com.baeldung.reflection.Goat", clazz.getCanonicalName()); 
}
```

注意，我们传递给静态`forName`方法的名称应该包含包信息。否则我们会得到一个`ClassNotFoundException`。

### 5.3。类别修饰符

我们可以通过调用返回一个`Integer`的`getModifiers`方法来确定类中使用的修饰符。每个修饰符都是一个置位或清零的标志位。

[`java.lang.reflect.Modifier`](https://web.archive.org/web/20220609164400/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/Modifier.html) 类提供静态方法来分析返回的`Integer`是否存在特定的修饰符。

让我们确认一下上面定义的一些类的修饰符:

```java
@Test
public void givenClass_whenRecognisesModifiers_thenCorrect() {
    Class<?> goatClass = Class.forName("com.baeldung.reflection.Goat");
    Class<?> animalClass = Class.forName("com.baeldung.reflection.Animal");

    int goatMods = goatClass.getModifiers();
    int animalMods = animalClass.getModifiers();

    assertTrue(Modifier.isPublic(goatMods));
    assertTrue(Modifier.isAbstract(animalMods));
    assertTrue(Modifier.isPublic(animalMods));
}
```

我们能够检查位于库 jar 中的任何类的修饰符，这些库 jar 被导入到我们的项目中。

在大多数情况下，我们可能需要使用`forName`方法，而不是完全的实例化，因为在内存密集型类的情况下，这将是一个昂贵的过程。

### 5.4。包装信息

通过使用 Java 反射，我们还能够获得关于任何类或对象的包的信息。这些数据被打包在 [`Package`](https://web.archive.org/web/20220609164400/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Package.html) 类中，通过调用类对象上的`getPackage`方法返回。

让我们运行一个测试来检索包名:

```java
@Test
public void givenClass_whenGetsPackageInfo_thenCorrect() {
    Goat goat = new Goat("goat");
    Class<?> goatClass = goat.getClass();
    Package pkg = goatClass.getPackage();

    assertEquals("com.baeldung.reflection", pkg.getName());
}
```

### 5.5。超类

我们也能够通过使用 Java 反射来获得任何 Java 类的超类。

在许多情况下，特别是在使用库类或 Java 的内置类时，我们可能事先不知道我们正在使用的对象的超类。本小节将说明如何获取这些信息。

让我们继续确定`Goat`的超类。

此外，我们还表明`java.lang.String`类是`java.lang.Object`类的子类:

```java
@Test
public void givenClass_whenGetsSuperClass_thenCorrect() {
    Goat goat = new Goat("goat");
    String str = "any string";

    Class<?> goatClass = goat.getClass();
    Class<?> goatSuperClass = goatClass.getSuperclass();

    assertEquals("Animal", goatSuperClass.getSimpleName());
    assertEquals("Object", str.getClass().getSuperclass().getSimpleName());
}
```

### 5.6。实现的接口

使用 Java 反射，我们还能够**获得给定类实现的接口列表。**

让我们检索由`Goat`类和`Animal`抽象类实现的接口的类类型:

```java
@Test
public void givenClass_whenGetsImplementedInterfaces_thenCorrect(){
    Class<?> goatClass = Class.forName("com.baeldung.reflection.Goat");
    Class<?> animalClass = Class.forName("com.baeldung.reflection.Animal");

    Class<?>[] goatInterfaces = goatClass.getInterfaces();
    Class<?>[] animalInterfaces = animalClass.getInterfaces();

    assertEquals(1, goatInterfaces.length);
    assertEquals(1, animalInterfaces.length);
    assertEquals("Locomotion", goatInterfaces[0].getSimpleName());
    assertEquals("Eating", animalInterfaces[0].getSimpleName());
}
```

从断言中可以注意到，每个类只实现了一个接口。检查这些接口的名称，我们发现`Goat`实现了`Locomotion`，而`Animal`实现了`Eating`，就像它出现在我们的代码中一样。

我们可以看到`Goat`是抽象类`Animal`的子类，实现了接口方法`eats()`。然后，`Goat`也实现了`Eating`接口。

因此值得注意的是，只有那些被类显式声明为用关键字`implements`实现的接口才会出现在返回的数组中。

所以，即使一个类实现了接口方法，因为它的超类实现了那个接口，但是子类没有用`implements`关键字直接声明那个接口，那个接口也不会出现在接口数组中。

### 5.7。构造函数、方法和字段

使用 Java 反射，我们能够检查任何对象类的构造函数以及方法和字段。

稍后，我们将能够看到对一个类的每个组件的更深入的检查。但是现在，只要得到他们的名字，并与我们所期望的进行比较就足够了。

让我们看看如何获得`Goat`类的构造函数:

```java
@Test
public void givenClass_whenGetsConstructor_thenCorrect(){
    Class<?> goatClass = Class.forName("com.baeldung.reflection.Goat");

    Constructor<?>[] constructors = goatClass.getConstructors();

    assertEquals(1, constructors.length);
    assertEquals("com.baeldung.reflection.Goat", constructors[0].getName());
}
```

我们还可以检查`Animal`类的字段:

```java
@Test
public void givenClass_whenGetsFields_thenCorrect(){
    Class<?> animalClass = Class.forName("com.baeldung.reflection.Animal");
    Field[] fields = animalClass.getDeclaredFields();

    List<String> actualFields = getFieldNames(fields);

    assertEquals(2, actualFields.size());
    assertTrue(actualFields.containsAll(Arrays.asList("name", "CATEGORY")));
}
```

我们同样可以检查`Animal`类的方法:

```java
@Test
public void givenClass_whenGetsMethods_thenCorrect(){
    Class<?> animalClass = Class.forName("com.baeldung.reflection.Animal");
    Method[] methods = animalClass.getDeclaredMethods();
    List<String> actualMethods = getMethodNames(methods);

    assertEquals(4, actualMethods.size());
    assertTrue(actualMethods.containsAll(Arrays.asList("getName",
      "setName", "getSound")));
}
```

就像`getFieldNames`一样，我们添加了一个 helper 方法来从一组`Method`对象中检索方法名:

```java
private static List<String> getMethodNames(Method[] methods) {
    List<String> methodNames = new ArrayList<>();
    for (Method method : methods)
      methodNames.add(method.getName());
    return methodNames;
}
```

## 6。检查构造器

有了 Java 反射，我们可以**检查任何类的构造函数**，甚至**在运行时创建类对象。**这是由 [`java.lang.reflect.Constructor`](https://web.archive.org/web/20220609164400/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/Constructor.html) 类实现的。

前面，我们只看了如何获得`Constructor`对象的数组，从中我们可以获得构造函数的名称。

在这一节中，我们将关注如何检索特定的构造函数。

众所周知，在 Java 中，一个类的两个构造函数没有完全相同的方法签名。因此，我们将利用这种唯一性从许多构造函数中获得一个。

为了理解这个类的特性，我们将用三个构造函数创建一个`Animal`的子类`Bird`。

我们不会实现`Locomotion`,因此我们可以使用构造函数参数来指定该行为，以增加更多的多样性:

```java
public class Bird extends Animal {
    private boolean walks;

    public Bird() {
        super("bird");
    }

    public Bird(String name, boolean walks) {
        super(name);
        setWalks(walks);
    }

    public Bird(String name) {
        super(name);
    }

    public boolean walks() {
        return walks;
    }

    // standard setters and overridden methods
}
```

让我们通过使用反射来确认这个类有三个构造函数:

```java
@Test
public void givenClass_whenGetsAllConstructors_thenCorrect() {
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");
    Constructor<?>[] constructors = birdClass.getConstructors();

    assertEquals(3, constructors.length);
}
```

接下来，我们将通过以声明的顺序传递构造函数的参数类类型来检索`Bird`类的每个构造函数:

```java
@Test
public void givenClass_whenGetsEachConstructorByParamTypes_thenCorrect(){
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");

    Constructor<?> cons1 = birdClass.getConstructor();
    Constructor<?> cons2 = birdClass.getConstructor(String.class);
    Constructor<?> cons3 = birdClass.getConstructor(String.class, boolean.class);
}
```

不需要断言，因为我们将得到一个`NoSuchMethodException` ，当具有给定参数类型的构造函数不存在时，测试将自动失败。

在最后一个测试中，我们将看到如何在运行时实例化对象，同时提供它们的参数:

```java
@Test
public void givenClass_whenInstantiatesObjectsAtRuntime_thenCorrect() {
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");
    Constructor<?> cons1 = birdClass.getConstructor();
    Constructor<?> cons2 = birdClass.getConstructor(String.class);
    Constructor<?> cons3 = birdClass.getConstructor(String.class,
      boolean.class);

    Bird bird1 = (Bird) cons1.newInstance();
    Bird bird2 = (Bird) cons2.newInstance("Weaver bird");
    Bird bird3 = (Bird) cons3.newInstance("dove", true);

    assertEquals("bird", bird1.getName());
    assertEquals("Weaver bird", bird2.getName());
    assertEquals("dove", bird3.getName());

    assertFalse(bird1.walks());
    assertTrue(bird3.walks());
}
```

我们通过调用`Constructor`类的`newInstance`方法实例化类对象，并按照声明的顺序传递所需的参数。然后，我们将结果转换为所需的类型。

也可以使用`Class.newInstance()` 方法调用默认的构造函数。然而，这种方法从 Java 9 开始就被弃用了，我们不应该在现代 Java 项目中使用它。

对于`bird1`，我们使用默认的构造函数，它从我们的`Bird`代码中自动将名称设置为 bird，并且我们通过测试来确认这一点。

然后我们用一个名字和测试实例化`bird2`。请记住，当我们不设置移动行为时，它默认为 false，如最后两个断言所示。

## 7。正在检查字段

以前，我们只检查字段的名称。在本节中，**我们将展示** **如何在运行时获取和设置它们的值。**

有两种主要的方法用于在运行时检查类的字段:`getFields()`和`getField(fieldName)`。

`getFields()`方法返回相关类的所有可访问的公共字段。它将返回类和所有超类中的所有公共字段。

例如，当我们在`Bird`类上调用这个方法时，我们将只得到它的超类`Animal`的`CATEGORY`字段，因为`Bird`本身没有声明任何公共字段:

```java
@Test
public void givenClass_whenGetsPublicFields_thenCorrect() {
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");
    Field[] fields = birdClass.getFields();

    assertEquals(1, fields.length);
    assertEquals("CATEGORY", fields[0].getName());
}
```

这个方法还有一个名为`getField`的变体，它通过取字段的名称只返回一个`Field`对象:

```java
@Test
public void givenClass_whenGetsPublicFieldByName_thenCorrect() {
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");
    Field field = birdClass.getField("CATEGORY");

    assertEquals("CATEGORY", field.getName());
}
```

我们不能访问在超类中声明但没有在子类中声明的私有字段。这就是我们无法访问`name`字段的原因。

然而，我们可以通过调用`getDeclaredFields`方法来检查我们正在处理的类中声明的私有字段:

```java
@Test
public void givenClass_whenGetsDeclaredFields_thenCorrect(){
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");
    Field[] fields = birdClass.getDeclaredFields();

    assertEquals(1, fields.length);
    assertEquals("walks", fields[0].getName());
}
```

如果我们知道字段的名称，我们也可以使用它的另一个变体:

```java
@Test
public void givenClass_whenGetsFieldsByName_thenCorrect() {
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");
    Field field = birdClass.getDeclaredField("walks");

    assertEquals("walks", field.getName());
}
```

如果我们弄错了字段的名称或者输入了一个不存在的字段，我们将得到一个`NoSuchFieldException`。

现在我们将得到字段类型:

```java
@Test
public void givenClassField_whenGetsType_thenCorrect() {
    Field field = Class.forName("com.baeldung.reflection.Bird")
      .getDeclaredField("walks");
    Class<?> fieldClass = field.getType();

    assertEquals("boolean", fieldClass.getSimpleName());
}
```

接下来，让我们看看如何访问和修改字段值。

要获取一个字段的值，更不用说设置它了，我们必须首先通过调用`Field`对象上的`setAccessible`方法并向其传递布尔`true`来设置它的可访问性:

```java
@Test
public void givenClassField_whenSetsAndGetsValue_thenCorrect() {
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");
    Bird bird = (Bird) birdClass.getConstructor().newInstance();
    Field field = birdClass.getDeclaredField("walks");
    field.setAccessible(true);

    assertFalse(field.getBoolean(bird));
    assertFalse(bird.walks());

    field.set(bird, true);

    assertTrue(field.getBoolean(bird));
    assertTrue(bird.walks());
}
```

在上面的测试中，我们在将字段`walks`的值设置为真之前，确定它确实是假的。

请注意我们如何使用`Field`对象来设置和获取值，方法是向它传递我们正在处理的类的实例，以及我们希望该对象中的字段具有的新值。

关于`Field`对象需要注意的一件重要事情是，当它被声明为`public static`时，我们不需要包含它们的类的实例。

我们可以只传递`null`来代替它，并且仍然获得该字段的默认值:

```java
@Test
public void givenClassField_whenGetsAndSetsWithNull_thenCorrect(){
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");
    Field field = birdClass.getField("CATEGORY");
    field.setAccessible(true);

    assertEquals("domestic", field.get(null));
}
```

## 8。检查方法

在前面的例子中，我们只使用反射来检查方法名。然而，Java 反射比这更强大。

有了 Java 反射，我们可以**在** **运行时**调用方法，并向它们传递所需的参数，就像我们对构造函数所做的那样。类似地，我们也可以通过指定每个重载方法的参数类型来调用重载方法。

就像字段一样，我们有两个主要的方法用于检索类方法。`getMethods`方法返回该类和超类的所有公共方法的数组。

这意味着通过这个方法，我们可以得到`java.lang.Object`类的公共方法，比如`toString`、`hashCode`和`notifyAll`:

```java
@Test
public void givenClass_whenGetsAllPublicMethods_thenCorrect(){
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");
    Method[] methods = birdClass.getMethods();
    List<String> methodNames = getMethodNames(methods);

    assertTrue(methodNames.containsAll(Arrays
      .asList("equals", "notifyAll", "hashCode",
        "walks", "eats", "toString")));
}
```

为了只获得我们感兴趣的类的公共方法，我们必须使用`getDeclaredMethods`方法:

```java
@Test
public void givenClass_whenGetsOnlyDeclaredMethods_thenCorrect(){
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");
    List<String> actualMethodNames
      = getMethodNames(birdClass.getDeclaredMethods());

    List<String> expectedMethodNames = Arrays
      .asList("setWalks", "walks", "getSound", "eats");

    assertEquals(expectedMethodNames.size(), actualMethodNames.size());
    assertTrue(expectedMethodNames.containsAll(actualMethodNames));
    assertTrue(actualMethodNames.containsAll(expectedMethodNames));
}
```

这些方法中的每一个都有一个单独的变体，返回一个我们知道名字的`Method`对象:

```java
@Test
public void givenMethodName_whenGetsMethod_thenCorrect() throws Exception {
    Bird bird = new Bird();
    Method walksMethod = bird.getClass().getDeclaredMethod("walks");
    Method setWalksMethod = bird.getClass().getDeclaredMethod("setWalks", boolean.class);

    assertTrue(walksMethod.canAccess(bird));
    assertTrue(setWalksMethod.canAccess(bird));
}
```

请注意我们是如何检索单个方法并指定它们所采用的参数类型的。那些没有参数类型的方法用一个空的变量参数检索，留给我们的只有一个参数，方法名。

接下来，我们将展示如何在运行时调用方法。

我们默认知道`Bird`类的`walks`属性是`false`。

我们想调用它的`setWalks`方法，并将其设置为`true`:

```java
@Test
public void givenMethod_whenInvokes_thenCorrect() {
    Class<?> birdClass = Class.forName("com.baeldung.reflection.Bird");
    Bird bird = (Bird) birdClass.getConstructor().newInstance();
    Method setWalksMethod = birdClass.getDeclaredMethod("setWalks", boolean.class);
    Method walksMethod = birdClass.getDeclaredMethod("walks");
    boolean walks = (boolean) walksMethod.invoke(bird);

    assertFalse(walks);
    assertFalse(bird.walks());

    setWalksMethod.invoke(bird, true);

    boolean walks2 = (boolean) walksMethod.invoke(bird);
    assertTrue(walks2);
    assertTrue(bird.walks());
}
```

注意我们如何首先调用`walks`方法，将返回类型转换为适当的数据类型，然后检查它的值。我们稍后调用`setWalks`方法来更改该值并再次测试。

## 9。结论

在本文中，我们介绍了 Java 反射 API，并研究了如何在编译时不了解类、接口、字段和方法的内部结构的情况下，在运行时使用它来检查它们。

本文的完整源代码和示例可以在 GitHub 的[中找到。](https://web.archive.org/web/20220609164400/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-11-2)