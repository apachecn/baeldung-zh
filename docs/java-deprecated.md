# Java @Deprecated 注释

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-deprecated>

## 1.概观

在这个快速教程中，我们将看看 Java 中被否决的 API 以及如何使用`@Deprecated`注释。

## 2。`@Deprecated` 注解

随着项目的发展，它的 API 也会改变。随着时间的推移，我们不希望人们再使用某些构造函数、字段、类型或方法。

我们可以用`@Deprecated`注释`.`来标记这些元素，而不是破坏项目 API 的向后兼容性

**`@Deprecated`告诉其他开发者******标记的元素不能再用****。通常还会在`@Deprecated`注释旁边添加一些 Javadoc 来解释什么是服务于正确行为的更好选择:****

```java
public class Worker {
    /**
     * Calculate period between versions
     * @deprecated
     * This method is no longer acceptable to compute time between versions.
     * <p> Use {@link Utils#calculatePeriod(Machine)} instead.
     *
     * @param machine instance
     * @return computed time
     */
    @Deprecated
    public int calculate(Machine machine) {
        return machine.exportVersions().size() * 10;
    }
}
```

请记住，只有在代码中的某个地方使用了带注释的 Java 元素时，编译器才会显示不推荐使用的 API 警告。因此，在这种情况下，它只会显示是否有代码调用了`calculate` 方法。

此外，**我们还可以通过使用 Javadoc `@deprecated`标签**来传达文档中被否决的状态。

## 3.Java 9 中添加的可选属性

**Java 9 给`@Deprecated`注释增加了一些可选属性:`since`和`forRemoval`。**

`since`属性需要一个字符串，让我们定义该元素在哪个版本中被弃用。默认值是一个空字符串。

而`forRemoval`是一个`boolean`，它允许我们指定该元素是否会在下一个版本中被移除。其默认值为`false:`

```java
public class Worker {
    /**
     * Calculate period between versions
     * @deprecated
     * This method is no longer acceptable to compute time between versions.
     * <p> Use {@link Utils#calculatePeriod(Machine)} instead.
     *
     * @param machine instance
     * @return computed time
     */
    @Deprecated(since = "4.5", forRemoval = true)
    public int calculate(Machine machine) {
        return machine.exportVersions().size() * 10;
    }
}
```

简而言之，上面的用法意味着`calculate`从我们的库的 4.5 版本开始就被弃用了，并且计划在下一个主要版本中移除。

添加这个对我们很有帮助，因为**如果编译器发现我们正在使用具有那个值的方法，它会给我们一个更强烈的警告**。

**并且 IDEs** **已经支持检测标记有`forRemoval=true.`** 的方法的使用，例如，用红线而不是黑线删除代码[。](https://web.archive.org/web/20220926193203/https://www.vojtechruzicka.com/java-9-enhanced-deprecation/)

## 4.结论

在这篇简短的文章中，我们看到了如何使用`@Deprecated`注释及其[可选属性](https://web.archive.org/web/20220926193203/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Deprecated.html)来标记不应该再使用的代码。

示例的完整源代码可以在 GitHub 的[中找到。](https://web.archive.org/web/20220926193203/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-annotations)****