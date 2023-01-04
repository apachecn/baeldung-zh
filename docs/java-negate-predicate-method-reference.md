# 用 Java 11 否定谓词方法引用

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-negate-predicate-method-reference>

## 1.概观

在这个简短的教程中，我们将看到如何使用 Java 11 否定一个`Predicate`方法引用。

为了在 Java 11 之前实现这一点，我们将从遇到的限制开始。然后我们将看到`Predicate.not() `方法是如何帮助我们的。

## 2.Java 11 之前

首先，让我们看看在 Java 11 之前我们是如何设法否定一个`Predicate`的。

首先，让我们用一个`age `字段和一个`isAdult()`方法创建一个`Person`类:

```java
public class Person {
    private static final int ADULT_AGE = 18;

    private int age;

    public Person(int age) {
        this.age = age;
    }

    public boolean isAdult() {
        return age >= ADULT_AGE;
    }
}
```

现在，假设我们有一个人员列表:

```java
List<Person> people = Arrays.asList(
  new Person(1),
  new Person(18),
  new Person(2)
);
```

我们想找回所有成年的。为了在 Java 8 中实现这一点，我们可以:

```java
people.stream()                      
  .filter(Person::isAdult)           
  .collect(Collectors.toList());
```

但是，如果我们要检索的是未成年人呢？那么我们必须否定这个谓词:

```java
people.stream()                       
  .filter(person -> !person.isAdult())
  .collect(Collectors.toList());
```

**不幸的是，我们被迫放弃方法引用，尽管我们发现它更容易阅读。**一个可能的解决方法是在`Person`类上创建一个`isNotAdult()`方法，然后使用对该方法的引用:

```java
people.stream()                 
  .filter(Person::isNotAdult)   
  .collect(Collectors.toList());
```

但是也许我们不想把这个方法添加到我们的 API 中，或者也许我们就是不能，因为这个类不是我们的。这时 Java 11 带来了`Predicate.not()`方法，我们将在下一节看到。

## 3.`Predicate.not()`法

**静态方法`Predicate.not() `已经被添加到 Java 11 中，以便否定现有的`Predicate`。**

让我们看一下前面的例子，看看这意味着什么。我们可以不使用 lambda 或者在`Person`类上创建一个新方法，而是使用这个新方法:

```java
people.stream()                          
  .filter(Predicate.not(Person::isAdult))
  .collect(Collectors.toList());
```

这样，我们不必修改我们的 API，仍然可以依赖方法引用的可读性。

我们可以通过静态导入使这一点更加清楚:

```java
people.stream()                  
  .filter(not(Person::isAdult))  
  .collect(Collectors.toList());
```

## 4.结论

在这篇短文中，我们看到了如何利用`Predicate` `.not()`方法来维护谓词的方法引用，即使它们被否定。

和往常一样，这篇文章的完整代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221206060852/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-11)