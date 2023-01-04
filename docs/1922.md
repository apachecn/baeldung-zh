# 带有自定义设置器的 Lombok 生成器

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/lombok-builder-custom-setter>

## 1。简介

Project Lombok 是一个流行的 Java 库，可以帮助减少开发人员需要编写的样板代码。

在本教程中，我们将看看 Lombok 的 [`@Builder`](https://web.archive.org/web/20220625222624/https://projectlombok.org/features/Builder) 注释是如何工作的，以及我们如何定制它来满足我们的特定需求。

## 2。Maven 依赖关系

让我们从将依赖关系添加到我们的`pom.xml`开始:

```
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.20</version>
</dependency>
```

## 3。龙目岛`Builder`注解

在我们研究定制 Lombok 生成的构建器类之前，让我们快速回顾一下 Lombok `@Builder`注释是如何工作的。我们已经全面介绍了 Lombok 的特性。

**[`@Builder`注释](/web/20220625222624/https://www.baeldung.com/lombok-builder)可以用来为我们的类**自动生成一个构建器。对于我们的例子，我们将使用一个消息传递系统，其中一个用户可以向另一个用户发送消息。该消息要么是一个简单的文本字符串，要么是一个`File`。使用 Lombok，我们可以如下定义我们的`Message`类:

```
@Builder
@Data
public class Message {
    private String sender;
    private String recipient;
    private String text;
    private File file;
}
```

`@Data`生成所有通常与简单 POJO (Plain Old Java Object)相关的样板文件:所有字段的 getters、所有非 final 字段的 setters、适当的`toString`、`equals`和`hashCode`实现，以及一个构造函数。

使用生成的构建器，我们现在可以生成我们的`Message`类的实例:

```
Message message = Message.builder()
  .sender("[[email protected]](/web/20220625222624/https://www.baeldung.com/cdn-cgi/l/email-protection)")
  .recipient("[[email protected]](/web/20220625222624/https://www.baeldung.com/cdn-cgi/l/email-protection)")
  .text("How are you today?")
  .build();
```

`@Builder`注释也支持属性的默认值，但是我们现在不会深入讨论这个。从这个例子中可以清楚地看到,`@Builder`注释非常强大，可以替换很多样板代码。

## 4。定制 Lombok 构建器

上一节展示了我们如何使用 Lombok 来生成一个构建器类。但是可能存在生成的构建器不够用的情况。在我们的示例中，我们有一个约束，即消息只能包含文本或文件。它不能两者兼得。当然，Lombok 并不知道这一点，生成的构建器会很高兴地允许我们进入非法状态。

幸运的是，我们可以通过定制构建器来解决这个问题。

定制 Lombok 构建器简单明了:**我们编写构建器中我们想要定制的部分，Lombok `@Builder`注释将不会生成这些部分**。因此，在我们的示例中，这将是:

```
public static class MessageBuilder {
    private String text;
    private File file;

    public MessageBuilder text(String text) {
        this.text = text;
        verifyTextOrFile();
        return this;
    }

    public MessageBuilder file(File file) {
        this.file = file;
        verifyTextOrFile();
        return this;
    }

    private void verifyTextOrFile() {
        if (text != null && file != null) {
            throw new IllegalStateException("Cannot send 'text' and 'file'.");
        }
    }
}
```

请注意，我们不必声明`sender`和`recipient`成员，或者与它们相关联的构建器方法。龙目岛仍然会为我们产生这些。

如果我们试图用以下代码生成一个既有文本又有文件的`Message`实例:

```
Message message = Message.builder()
  .sender("[[email protected]](/web/20220625222624/https://www.baeldung.com/cdn-cgi/l/email-protection)")
  .recipient("[[email protected]](/web/20220625222624/https://www.baeldung.com/cdn-cgi/l/email-protection)")
  .text("How are you today?")
  .file(new File("/path/to/file"))
  .build();
```

这将导致以下异常:

```
Exception in thread "main" java.lang.IllegalStateException: Cannot send 'text' and 'file'.
```

## 5。结论

在这篇简短的文章中，我们研究了如何定制 Lombok 构建器。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20220625222624/https://github.com/eugenp/tutorials/tree/master/lombok-modules/lombok)