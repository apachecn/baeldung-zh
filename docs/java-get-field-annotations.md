# 使用反射获取字段的注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-get-field-annotations>

## 1.概观

在本教程中，我们将学习如何获得一个字段的注释。另外，我们将解释保留元注释是如何工作的。之后，我们将展示返回字段注释的两种方法之间的区别。

## 2.注释的保留策略

首先，我们来看看`[Retention](/web/20220529023153/https://www.baeldung.com/java-default-annotations#2-retention) `标注。它定义了注释的生命周期。这个元注释带有一个`RetentionPolicy` 属性`.` ，也就是说`,` 属性定义了注释可见的生命周期:

*   `RetentionPolicy.SOURCE – `只在源代码中可见
*   `RetentionPolicy.CLASS`–编译时对编译器可见
*   对编译器和运行时可见

因此，**只有`RUNTIME`保留策略允许我们以编程方式读取注释**。

## 3.使用反射获取字段的注释

现在，让我们创建一个带有注释字段的示例类。我们将定义三个注释，其中只有两个在运行时可见。

第一个注释在运行时可见:

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface FirstAnnotation {
}
```

第二个具有相同的保持力:

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface SecondAnnotation {
}
```

最后，让我们创建仅在源代码中可见的第三个注释:

```java
@Retention(RetentionPolicy.SOURCE)
public @interface ThirdAnnotation {
}
```

现在，让我们用字段`classMember`定义一个类，用我们所有的三个注释进行注释:

```java
public class ClassWithAnnotations {

    @FirstAnnotation
    @SecondAnnotation
    @ThirdAnnotation
    private String classMember;
}
```

之后，让我们在运行时检索所有可见的注释。**我们将使用 [Java 反射](/web/20220529023153/https://www.baeldung.com/java-reflection)，它允许我们检查字段的属性:**

```java
@Test
public void whenCallingGetDeclaredAnnotations_thenOnlyRuntimeAnnotationsAreAvailable() throws NoSuchFieldException {
    Field classMemberField = ClassWithAnnotations.class.getDeclaredField("classMember");
    Annotation[] annotations = classMemberField.getDeclaredAnnotations();
    assertThat(annotations).hasSize(2);
}
```

因此，我们只检索到两个在运行时可用的注释。方法`getDeclaredAnnotations` 返回一个长度为零的数组，以防字段上没有注释。

我们可以用同样的方式读取超类字段的注释:[检索超类的字段](/web/20220529023153/https://www.baeldung.com/java-reflection-class-fields#inherited)并调用相同的`getDeclaredAnnotations`方法。

## 4.检查字段是否用特定类型进行了注释

现在让我们看看如何检查字段中是否存在特定的注释。`[Field](https://web.archive.org/web/20220529023153/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/Field.html) `类有一个方法`isAnnotationPresent` ，当指定类型的注释出现在元素上时，该方法返回`true` 。让我们在我们的`classMember` 场地上测试一下:

```java
@Test
public void whenCallingIsAnnotationPresent_thenOnlyRuntimeAnnotationsAreAvailable() throws NoSuchFieldException {
    Field classMemberField = ClassWithAnnotations.class.getDeclaredField("classMember");
    assertThat(classMemberField.isAnnotationPresent(FirstAnnotation.class)).isTrue();
    assertThat(classMemberField.isAnnotationPresent(SecondAnnotation.class)).isTrue();
    assertThat(classMemberField.isAnnotationPresent(ThirdAnnotation.class)).isFalse();
}
```

正如所料，`ThirdAnnotation` 不存在，因为它有一个为`Retention` 元注释`.`指定的`SOURCE`保留策略

## 5.`Field`方法`getAnnotations`和`getDeclaredAnnnotations`

现在让我们看看由`Field` 类公开的两个方法，`getAnnotations`和`getDeclaredAnnotations`。根据 Javadoc， [`getDeclaredAnnotations`](https://web.archive.org/web/20220529023153/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/AccessibleObject.html#getDeclaredAnnotations()) 方法返回直接出现在元素上的**注释。另一方面，Javadoc 对 [`getAnnotations`](https://web.archive.org/web/20220529023153/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/reflect/AccessibleObject.html#getAnnotations()) 说它返回**元素**上出现的所有注释。**

类中的字段包含位于其定义上方的注释。因此，根本不涉及注释的继承。所有注释必须与字段定义一起定义。因此，方法`getAnnotations`和`getDeclaredAnnotations` **总是返回相同的结果**。

让我们通过一个简单的测试来展示它:

```java
@Test
public void whenCallingGetDeclaredAnnotationsOrGetAnnotations_thenSameAnnotationsAreReturned() throws NoSuchFieldException {
    Field classMemberField = ClassWithAnnotations.class.getDeclaredField("classMember");
    Annotation[] declaredAnnotations = classMemberField.getDeclaredAnnotations();
    Annotation[] annotations = classMemberField.getAnnotations();
    assertThat(declaredAnnotations).containsExactly(annotations);
}
```

而且，在`Field` 类中，我们可以发现`getAnnotations`方法调用了`getDeclaredAnnotations`方法:

```java
@Override
public Annotation[] getAnnotations() {
    return getDeclaredAnnotations();
}
```

## 6.结论

在这篇短文中，我们解释了保留策略元注释在检索注释中的作用。然后我们展示了如何阅读一个字段的注释。最后，我们证明了一个字段没有注释的继承。

和往常一样，这个例子的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220529023153/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-annotations)