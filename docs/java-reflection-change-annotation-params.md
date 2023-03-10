# 在运行时更改注释参数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-reflection-change-annotation-params>

## 1.概观

*注解* ，一种可以添加到 Java 代码中的元数据形式。这些 *注释* 可以在编译时处理并嵌入到类文件中，或者可以在运行时使用 *反射* 进行保留和访问。

在本文中，我们将讨论如何使用`Reflection`在运行时改变 *标注* 值。对于这个例子，我们将使用类级注释。

## 2.注释

Java 允许使用现有的创建新的`annotations`。在最简单的形式中，注释被表示为`@`符号后跟注释名:

```java
@Override
```

让我们创建自己的注释`Greeter` :

```java
@Retention(RetentionPolicy.RUNTIME)
public @interface Greeter {    
    public String greet() default ""; 
}
```

现在，我们将创建一个 Java 类 *问候* 其中使用了类级 `annotation` *:*

```java
@Greeter(greet="Good morning")
public class Greetings {} 
```

现在，我们将使用反射来访问注释值。Java 类*类*提供了一个方法`getAnnotation`来访问类的注释:

```java
Greeter greetings = Greetings.class.getAnnotation(Greeter.class);
System.out.println("Hello there, " + greetings.greet() + " !!");
```

## 3.改变注释

Java 类 *类* 维护一个用于管理注释的映射—`Annotation`类作为键，`Annotation`对象作为值:

```java
Map<Class<? extends Annotation>, Annotation> map;
```

我们将更新此地图，以便在运行时更改注释。在不同的 JDK 实施中，访问此地图的方法有所不同。我们将针对 JDK7 和 JDK8 进行讨论。

### 3.1。 **JDK 7 号实施**

Java 类*类* 有字段 `annotations`。由于这是一个私有字段，要访问它，我们必须将字段的可访问性设置为`true`。Java 提供了方法*getDeclaredField*来通过名称访问任何字段

```java
Field annotations = Class.class.getDeclaredField(ANNOTATIONS);
annotations.setAccessible(true); 
```

现在，让我们访问类`Greeter`的注释映射:

```java
 Map<Class<? extends Annotation>, Annotation> map = annotations.get(targetClass);
```

现在，这是包含所有注释和它们的值对象的信息的映射。我们想改变`Greeter`注释值，这可以通过更新`Greeter`类的注释对象:来实现

```java
map.put(targetAnnotation, targetValue);
```

### 3.2。 **JDK 8 实施**

Java 8 实现在一个类 *中存储`annotations`信息 AnnotationData* 。我们可以使用`annotationData`方法来访问这个对象。我们将把`annotationData`方法的可访问性设置为`true`，因为它是一个私有方法:

```java
Method method = Class.class.getDeclaredMethod(ANNOTATION_METHOD, null);
method.setAccessible(true);
```

现在，我们可以进入`annotations`字段。由于该字段也是私有字段，我们将可访问性设置为`true` :

```java
Field annotations = annotationData.getClass().getDeclaredField(ANNOTATIONS);
annotations.setAccessible(true);
```

该字段具有存储注释类和值对象的注释缓存映射。 让我们改变一下:

```java
Map<Class<? extends Annotation>, Annotation> map = annotations.get(annotationData); 
map.put(targetAnnotation, targetValue);
```

## 4.应用

让我们举这个例子:

```java
Greeter greetings = Greetings.class.getAnnotation(Greeter.class);
System.err.println("Hello there, " + greetings.greet() + " !!");
```

这将是问候“早上好”，因为这是我们提供给注释的值。
现在，我们将再创建一个`Greeter`类型的对象，其值为“晚上好”:

```java
Greeter targetValue = new DynamicGreeter("Good evening"); 
```

让我们用新值更新注释图:

```java
alterAnnotationValueJDK8(Greetings.class, Greeter.class, targetValue);
```

让我们再次检查问候值:

```java
greetings = Greetings.class.getAnnotation(Greeter.class);
System.err.println("Hello there, " + greetings.greet() + " !!");
```

它会打招呼说“晚上好”。

## 5.结论

Java 实现使用两个数据字段来存储注释数据:`annotations`、`declaredAnnotations`。这两者的区别是:首先存储父类的注释，然后只存储当前类的注释。

为实现*getAnnotation*在 JDK 7 和 JDK 8 中有所不同，我们在这里使用 *标注* 字段映射为简单起见。

和往常一样，Github 上的[提供了实现的源代码。](https://web.archive.org/web/20220625082530/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-reflection)