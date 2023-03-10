# 在龙目语中省略 Getter 或 Setter

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/lombok-omit-getter-setter>

## 1.概观

有时，我们想隐藏在对象中获取或设置字段值的能力。但是 Lombok 会自动生成默认的 getter/setter。在这个快速教程中，我们将展示如何省略 Lombok 生成的 getters 和 setters。关于 Lombok 项目库的详细信息也可以在[Lombok 项目简介](/web/20220525134849/https://www.baeldung.com/intro-to-project-lombok)中找到。

在继续之前，我们应该[在我们的 IDE](/web/20220525134849/https://www.baeldung.com/lombok-ide) 中安装 Lombok 插件。

## 2.属国

首先，我们需要将[龙目岛](https://web.archive.org/web/20220525134849/https://search.maven.org/search?q=g:org.projectlombok%20AND%20a:lombok)添加到我们的 `pom.xml`文件中:

```java
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.22</version>
    <scope>provided</scope>
</dependency>
```

## 3.默认行为

在我们进入如何省略 getters 和 setters 生成的细节之前，**让我们回顾一下负责生成它们的注释的默认行为**。

### 3.1.@ `Getter`和@ `Setter`注释

Lombok 提供了两个访问器注释，`@Getter`和`@Setter`。我们可以注释每个字段，或者简单地用它们标记整个类。默认情况下生成的方法会是`public`。但是，我们可以将访问级别更改为`protected`、包或`private`。让我们看一个例子:

```java
@Setter
@Getter
public class User {
    private long id;
    private String login;
    private int age;
}
```

我们可以使用 IDE 插件中的`delombok`选项，查看 Lombok 生成的代码:

```java
public class User {
    private long id;
    private String login;
    private int age;

    public long getId() {
        return this.id;
    }

    public String getLogin() {
        return this.login;
    }

    public int getAge() {
        return this.age;
    }

    public void setId(long id) {
        this.id = id;
    }

    public void setLogin(String login) {
        this.login = login;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

正如我们所观察到的，所有的 getters 和 setters 都是被创建的。所有字段的 Getter 和 setter 都是公共的，因为我们没有明确指定任何字段的访问级别。

### 3.2.@ *数据*标注

`@Data`结合了一些其他注释的特性，包括`@Getter`和`@Setter.`，因此，在这种情况下，默认的访问器方法也将生成为`public`:

```java
@Data
public class Employee {
    private String name;
    private String workplace;
    private int workLength;
}
```

## 4.使用`AccessLevel.NONE `省略 Getter 或 Setter

要禁用特定字段的默认 getter/setter 生成，我们应该使用特定的访问级别:

```java
@Getter(AccessLevel.NONE)
@Setter(AccessLevel.NONE)
```

这个访问级别**让我们可以覆盖类**上的`@Getter`、`@Setter`或`@Data`注释的行为。要覆盖访问级别，用显式的`@Setter`或`@Getter`注释来注释字段或类。

### 4.1.覆盖`@Getter`和`@Setter`注释

让我们将`age`字段的 getter 和`id`字段的 setter 上的`AccessLevel`改为`NONE`:

```java
@Getter
@Setter
public class User {
    @Setter(AccessLevel.NONE)
    private  long id;

    private String login;

    @Getter(AccessLevel.NONE)
    private int age;
}
```

让我们`delombok`这个代码:

```java
public class User {
    private  long id;

    private String login;

    private int age;

    public long getId() {
        return this.id;
    }

    public String getLogin() {
        return this.login;
    }

    public void setLogin(String login) {
        this.login = login;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

正如我们所看到的，对于`id `字段没有 setter，对于*年龄*字段没有 go getter。

### 4.2.覆盖`@Data`注释

让我们看另一个例子，在这个例子中，我们用`@Data`注释将类上的`AccessLevel`改为`NONE`:

```java
@Data
public class Employee {

    @Setter(AccessLevel.NONE)
    private String name;

    private String workplace;

    @Getter(AccessLevel.NONE)
    private int workLength;
}
```

我们向*工作长度*字段添加了显式`@Getter`注释，向*名称*字段添加了显式`@Setter`注释。两个存取器的`AccessLevel`都被设置为`NONE`。让我们看看`delombok`的代码:

```java
public class Employee {

    private String name;

    private String workplace;

    private int workLength;

    public String getName() {
        return this.name;
    }

    public String getWorkplace() {
        return this.workplace;
    }

    public void setWorkplace(String workplace) {
        this.workplace = workplace;
    }

    public void setWorkLength(int workLength) {
        this.workLength = workLength;
    }
}
```

正如我们所料，我们对`@Getter`和`@Setter`的显式设置覆盖了由`@Data`注释生成的 getters 和 setters。没有为*名称*字段生成 setter，也没有为*工作长度*字段生成 getter。

## 5.结论

在本文中，我们探讨了如何通过 Lombok 为对象中的特定字段省略 getter 和 setter 生成。此外，我们看到了 `@Getter`、`@Setter`和`@Data`注释的例子。接下来，我们看到了 Lombok 为我们的注释设置生成的代码。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220525134849/https://github.com/eugenp/tutorials/tree/master/lombok)