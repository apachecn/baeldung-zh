# 在 Java 中验证 UUID 字符串

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-validate-uuid-string>

## 1.概观

在本教程中，我们将了解一些在 Java 中验证 UUID(通用唯一标识符)字符串的方法。

我们将经历一个 [`UUID`](/web/20220928160749/https://www.baeldung.com/java-uuid) 类方法，然后我们将使用正则表达式。

## 2.使用`UUID.fromString()`

检查一个`String`是否是一个`UUID`的最快方法之一是尝试使用属于`UUID`类的静态方法`fromString`来映射它。让我们试一试:

```
@Test
public void whenValidUUIDStringIsValidated_thenValidationSucceeds() {
    String validUUID = "26929514-237c-11ed-861d-0242ac120002";
    Assertions.assertEquals(UUID.fromString(validUUID).toString(), validUUID);

    String invalidUUID = "invalid-uuid";
    Assertions.assertThrows(IllegalArgumentException.class, () -> UUID.fromString(invalidUUID));
}
```

在上面的代码片段中，我们可以看到，如果我们试图验证的字符串不代表 UUID，那么就会抛出一个`IllegalArgumentException`。然而，对于“1-1-1-1-1-1”这样的字符串，方法`fromString`将返回“000000001-0001-0001-00000001”。所以我们包括了字符串比较来处理这种情况。

有些人可能会认为使用异常对于流控制来说不是一个好的实践，所以我们将看到实现相同结果的不同方法。

## 3.使用正则表达式

验证 UUID 的另一种方式是使用与格式完全匹配的正则表达式。

首先，我们需要定义一个用于匹配字符串的`Pattern`。

```
Pattern UUID_REGEX =
  Pattern.compile("^[0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{12}$");
```

然后，我们可以使用此模式尝试将其与一个字符串进行匹配，以验证它是否是一个 UUID:

```
@Test
public void whenUUIDIsValidatedUsingRegex_thenValidationSucceeds() {
    Pattern UUID_REGEX =
      Pattern.compile("^[0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{12}$");

    Assertions.assertTrue(UUID_REGEX.matcher("26929514-237c-11ed-861d-0242ac120002").matches());

    Assertions.assertFalse(UUID_REGEX.matcher("invalid-uuid").matches());
}
```

## 4.结论

在本文中，我们学习了如何通过使用正则表达式或者利用`UUID`类的静态方法来验证 UUID 字符串。

和往常一样，这些例子的代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220928160749/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-uuid)