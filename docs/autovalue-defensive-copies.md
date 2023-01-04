# 使用 AutoValue 的集合的防御性副本

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/autovalue-defensive-copies>

## 1.概观

创建不可变的值对象会引入一些不必要的样板文件。另外， [Java 的标准集合类型](/web/20220628095029/https://www.baeldung.com/java-collections)有可能给值对象引入可变性，而这种特性是不可取的。

在本教程中，我们将演示如何在使用 [AutoValue](/web/20220628095029/https://www.baeldung.com/introduction-to-autovalue) 时创建集合的防御性副本，这是一个有用的工具，可以减少定义不可变值对象的样板代码。

## 2.价值对象和防御性副本

Java 社区通常认为值对象是表示不可变数据记录的一类类型。当然，这种类型可能包含对标准 Java 集合类型的引用，如`java.util.List`。

例如，考虑一个`Person`值对象:

```
class Person {
    private final String name;
    private final List<String> favoriteMovies;

    // accessors, constructor, toString, equals, hashcode omitted
}
```

因为 Java 的标准集合类型可能是可变的，所以不可变的`Person`类型必须保护自己，防止调用者在创建新的`Person`后修改`favoriteMovies`列表:

```
var favoriteMovies = new ArrayList<String>();
favoriteMovies.add("Clerks"); // fine
var person = new Person("Katy", favoriteMovies);
favoriteMovies.add("Dogma"); // oh, no!
```

`Person`类必须制作`favoriteMovies`集合的防御副本。通过这样做，`Person`类捕获了当`Person`被创建时`favoriteMovies`列表的状态。

`Person`类构造函数可以使用`List.copyOf`静态工厂方法来制作`favoriteMovies`列表的防御性副本:

```
public Person(String name, List<String> favoriteMovies) {
    this.name = name;
    this.favoriteMovies = List.copyOf(favoriteMovies);
}
```

Java 10 引入了`List.copyOf`等防御性复制静态工厂方法。使用旧版本 Java 的应用程序可能会使用复制构造函数和`Collections`类上的一个“不可修改的”静态工厂方法创建一个防御性副本:

```
public Person(String name, List<String> favoriteMovies) {
    this.name = name;
    this.favoriteMovies = Collections.unmodifiableList(new ArrayList<>(favoriteMovies));
}
```

注意，由于`String`实例是不可变的，所以不需要制作`String name`参数的防御性副本。

## 3.自动增值和防御性拷贝

AutoValue 是一个注释处理工具，用于生成定义值对象类型的样板代码。然而， **AutoValue 在构造值对象时并不做防御性的复制。**

`@AutoValue`注释指示 AutoValue 生成一个类`AutoValue_Person`，它扩展了`Person`，并包含了我们之前从`Person`类中省略的访问器、构造器、`toString`、`equals`和`hashCode`方法。

最后，我们向`Person`类添加一个静态工厂方法，并调用生成的`AutoValue_Person`构造函数:

```
@AutoValue
public abstract class Person {

    public static Person of(String name, List<String> favoriteMovies) {
        return new AutoValue_Person(name, favoriteMovies);
    }

    public abstract String name();
    public abstract List<String> favoriteMovies();
}
```

AutoValue 生成的构造函数不会自动创建任何防御副本，包括一个用于`favoriteMovies`集合的副本。

因此，我们需要**在我们定义的静态工厂方法**中创建一个`favoriteMovies`集合的防御副本:

```
public abstract class Person {

    public static Person of(String name, List<String> favoriteMovies) {
        // create defensive copy before calling constructor
        var favoriteMoviesCopy = List.copyOf(favoriteMovies);
        return new AutoValue_Person(name, favoriteMoviesCopy);
    }

    public abstract String name();
    public abstract List<String> favoriteMovies();
}
```

## 4.自动价值构建器和防御性副本

如果需要，我们可以使用`@AutoValue.Builder`注释，它指示 AutoValue 生成一个`Builder`类:

```
@AutoValue
public abstract class Person {

    public abstract String name();
    public abstract List<String> favoriteMovies();

    public static Builder builder() {
        return new AutoValue_Person.Builder();
    }

    @AutoValue.Builder
    public static class Builder {
        public abstract Builder name(String value);
        public abstract Builder favoriteMovies(List<String> value);
        public abstract Person build();
    }
}
```

因为 AutoValue 生成所有抽象方法的实现，所以不清楚如何创建`List`的防御性副本。我们需要混合使用自动值生成代码和定制代码，在构建器构建新的`Person`实例之前制作集合的防御副本。

首先，我们将用两个新的包私有抽象方法来补充我们的构建器:`favoriteMovies()`和`autoBuild()`。这些方法是包私有的，因为我们想在`build()`方法的自定义实现中使用它们，但是我们不希望这个 API 的消费者使用它们。

```
@AutoValue.Builder
public static abstract class Builder {

    public abstract Builder name(String value);
    public abstract Builder favoriteMovies(List<String> value);

    abstract List<String> favoriteMovies();
    abstract Person autoBuild();

    public Person build() {
        // implementation omitted
    }
}
```

最后，我们将提供一个`build()`方法的**定制实现，它在构造`Person`之前创建列表的防御副本**。我们将使用`favoriteMovies()`方法来检索用户设置的`List`。接下来，在调用`autoBuild()`来构造`Person`之前，我们将用一个新的副本替换这个列表:

```
public Person build() {
    List<String> favoriteMovies = favoriteMovies();
    List<String> copy = Collections.unmodifiableList(new ArrayList<>(favoriteMovies));
    favoriteMovies(copy);
    return autoBuild();
}
```

## 5.结论

在本教程中，我们了解到 AutoValue 不会自动创建防御性副本，这对 Java 集合来说非常重要。

我们演示了在构造 AutoValue 生成的类的实例之前，如何在静态工厂方法中创建防御性副本。接下来，我们展示了在使用 AutoValue 的`Builder`类时，如何结合定制和生成的代码来创建防御性副本。

和往常一样，本教程中使用的代码片段可以从 GitHub 上的[处获得。](https://web.archive.org/web/20220628095029/https://github.com/eugenp/tutorials/tree/master/code-generation)