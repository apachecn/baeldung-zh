# Java @SuppressWarnings 批注

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-suppresswarnings>

## 1.概观

在这个快速教程中，我们将看看如何使用`@SuppressWarnings`注释。

## 2。`@SuppressWarnings`注解

编译器警告消息通常很有帮助。不过，有时候警告可能会很吵。

尤其是当我们不能或不想解决这些问题时:

```java
public class Machine {
    private List versions;

    public void addVersion(String version) {
        versions.add(version);
    }
}
```

编译器将发出关于此方法的警告。它会警告我们正在使用一个原始类型的集合。如果我们不想修复警告，那么**我们可以用`@SuppressWarnings`注释**来抑制它。

这个注释允许我们决定忽略哪种警告。虽然警告类型因[编译器供应商](https://web.archive.org/web/20221026040511/https://stackoverflow.com/questions/1205995/what-is-the-list-of-valid-suppresswarnings-warning-names-in-java)、**而异，但最常见的两种是`deprecation`和`unchecked`。**

`deprecatio` `n` 告诉编译器在我们使用不推荐的方法或类型时忽略它。

告诉编译器在我们使用原始类型时忽略它。

因此，在上面的例子中，**我们可以取消与我们的原始类型用法**相关联的警告:

```java
public class Machine {
    private List versions;

    @SuppressWarnings("unchecked")
    // or
    @SuppressWarnings({"unchecked"})
    public void addVersion(String version) {
        versions.add(version);
    }
}
```

为了抑制多个警告的列表，我们设置了一个包含相应警告列表的`String`数组:

```java
@SuppressWarnings({"unchecked", "deprecated"})
```

## 3.结论

在本指南中，我们看到了如何在 Java 中使用`@SuppressWarnings`注释。

示例的完整源代码可以在 GitHub 的[中找到。](https://web.archive.org/web/20221026040511/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-annotations)