# 使用 NullAway 避免 NullPointerExceptions

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-nullaway>

## 1.概观

多年来，我们已经采取了许多策略，从 Elvis 运营商到`[Optional](/web/20221206185737/https://www.baeldung.com/java-optional)`，来帮助从我们的应用程序中删除`NullPointerException`。在本教程中，我们将了解优步对会话的贡献，null away，以及如何使用它。

NullAway 是一个构建工具，它帮助我们消除 Java 代码中的 npe。

该工具执行一系列基于类型的局部检查，以确保在代码中被解引用的任何指针都不能是`null`。它具有较低的构建时开销，并且可以配置为在代码的每个构建中运行。

## 2.装置

让我们看看如何安装 NullAway 及其依赖项。在这个例子中，我们将使用 Gradle 配置 NullAway。

空值依赖于容易出错的 T2。因此，我们将添加`errorprone`插件:

```
plugins {
  id "net.ltgt.errorprone" version "1.1.1"
}
```

我们还将在不同的范围中添加四个依赖项:`annotationProcessor`、`compileOnly`、`errorprone,`和`errorproneJavac`:

```
dependencies {
  annotationProcessor "com.uber.nullaway:nullaway:0.7.9"
  compileOnly "com.google.code.findbugs:jsr305:3.0.2"
  errorprone "com.google.errorprone:error_prone_core:2.3.4"
  errorproneJavac "com.google.errorprone:javac:9+181-r4173-1"
} 
```

最后，我们将添加 Gradle 任务，该任务配置 NullAway 在编译期间的工作方式:

```
import net.ltgt.gradle.errorprone.CheckSeverity

tasks.withType(JavaCompile) {
    options.errorprone {
        check("NullAway", CheckSeverity.ERROR)
        option("NullAway:AnnotatedPackages", "com.baeldung")
    }
}
```

上面的任务将 NullAway 严重性设置为错误级别，这意味着我们可以配置 NullAway 来停止有错误的构建。默认情况下，NullAway 只会在编译时警告用户。

此外，该任务设置要检查的包是否为空解引用。

就这样，我们现在可以在 Java 代码中使用这个工具了。

同样，我们可以使用[其他构建系统](https://web.archive.org/web/20221206185737/https://github.com/uber/NullAway/wiki/Configuration#other-build-systems)、Maven 或 Bazel `,`来集成该工具。

## 3.使用

假设我们有一个`Person`类，包含一个`age`属性。此外，我们有一个`getAge`方法，它将一个`Person`实例作为参数:

```
Integer getAge(Person person) {
    return person.getAge();
}
```

此时我们可以看到，如果`person`是`null`，那么`getAge`会抛出一个`NullPointerException` 。

NullAway 假设每个方法参数、返回值和字段都是非`-null.` 的，因此，它希望`person`实例是非`null`的。

让我们假设在我们的代码中有一个地方，实际上，传递一个空引用到`getAge`:

```
Integer yearsToRetirement() {
    Person p = null;
    // ... p never gets set correctly...
    return 65 - getAge(p);
}
```

然后，运行构建将产生以下错误:

```
error: [NullAway] passing @Nullable parameter 'null' where @NonNull is required
    getAge(p);
```

我们可以通过给我们的参数添加一个`@Nullable`注释来修复这个错误:

```
Integer getAge(@Nullable Person person) { 
    // ... same as earlier
}
```

现在，当我们运行构建时，我们会看到一个新的错误:

```
error: [NullAway] dereferenced expression person is @Nullable
    return person.getAge();
            ^
```

这是在告诉我们，`person`实例有成为`null`的可能性。我们可以通过添加一个标准的空值检查来解决这个问题:

```
Integer getAge(@Nullable Person person) {
    if (person != null) {
        return person.getAge();
    } else {
        return 0;
    }
}
```

## 4.结论

在本教程中，我们已经了解了如何使用 NullAway 来限制遇到`NullPointerException` s 的可能性。

像往常一样，所有的源代码都可以在 GitHub 上获得[。](https://web.archive.org/web/20221206185737/https://github.com/eugenp/tutorials/tree/master/libraries-3)