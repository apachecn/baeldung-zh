# 在 Groovy 中将字符串转换为整数

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/groovy-convert-string-to-integer>

## 1.概观

在这个简短的教程中，我们将展示在 Groovy 中从`String`转换到`Integer `的不同方法。

## 2.用`as`铸造

我们可以用于转换的第一个方法是`as `关键字，它与调用**类的`asType()`方法**相同:

```java
@Test
void givenString_whenUsingAsInteger_thenConvertToInteger() {
    def stringNum = "123"
    Integer expectedInteger = 123
    Integer integerNum = stringNum as Integer

    assertEquals(integerNum, expectedInteger)
}
```

和上面一样，我们可以用`as int`:

```java
@Test
void givenString_whenUsingAsInt_thenConvertToInt() {
    def stringNum = "123"
    int expectedInt = 123
    int intNum = stringNum as int

    assertEquals(intNum, expectedInt)
}
```

## 3.`toInteger`

另一种方法是从 Groovy 的 JDK 扩展为 [`java.lang.CharSequence`](https://web.archive.org/web/20221221232606/https://docs.groovy-lang.org/latest/html/groovy-jdk/java/lang/CharSequence.html#toInteger()) :

```java
@Test
void givenString_whenUsingToInteger_thenConvertToInteger() {
    def stringNum = "123"
    int expectedInt = 123
    int intNum = stringNum.toInteger()

    assertEquals(intNum, expectedInt)
}
```

## 4.`Integer#parseInt`

第三种方法是使用 **Java 的静态方法`Integer.parseInt()`** :

```java
@Test
void givenString_whenUsingParseInt_thenConvertToInteger() {
    def stringNum = "123"
    int expectedInt = 123
    int intNum = Integer.parseInt(stringNum)

    assertEquals(intNum, expectedInt)
}
```

## 5.`Integer#intValue`

另一种方法是创建一个新的`Integer`对象并调用它的`intValue`方法:

```java
@Test
void givenString_whenUsingIntValue_thenConvertToInteger() {
    def stringNum = "123"
    int expectedInt = 123
    int intNum = new Integer(stringNum).intValue()

    assertEquals(intNum, expectedInt)
}
```

或者，在这种情况下，我们也可以只使用`new Integer(stringNum)`:

```java
@Test
void givenString_whenUsingNewInteger_thenConvertToInteger() {
    def stringNum = "123"
    int expectedInt = 123
    int intNum = new Integer(stringNum)

    assertEquals(intNum, expectedInt)
}
```

## 6.`Integer#valueOf`

类似于`Integer.parseInt()`，我们也可以使用 Java 的静态方法`Integer#valueOf`:

```java
@Test
void givenString_whenUsingValueOf_thenConvertToInteger() {
    def stringNum = "123"
    int expectedInt = 123
    int intNum = Integer.valueOf(stringNum)

    assertEquals(intNum, expectedInt)
}
```

## 7.`DecimalFormat`

对于我们的最后一个方法，我们可以应用 Java 的`DecimalFormat`类:

```java
@Test
void givenString_whenUsingDecimalFormat_thenConvertToInteger() {
    def stringNum = "123"
    int expectedInt = 123
    DecimalFormat decimalFormat = new DecimalFormat("#")
    int intNum = decimalFormat.parse(stringNum).intValue()

    assertEquals(intNum, expectedInt)
}
```

## 8.异常处理

所以，**如果转换失败，比如有非数字字符，就会抛出`NumberFormatException`**。另外，在`String`为`null`的情况下，会抛出`NullPointerException`:

```java
@Test(expected = NumberFormatException.class)
void givenInvalidString_whenUsingAs_thenThrowNumberFormatException() {
    def invalidString = "123a"
    invalidString as Integer
}

@Test(expected = NullPointerException.class)
void givenNullString_whenUsingToInteger_thenThrowNullPointerException() {
    def invalidString = null
    invalidString.toInteger()
}
```

为了防止这种情况发生，我们可以使用`isInteger method`**:**

```java
@Test
void givenString_whenUsingIsInteger_thenCheckIfCorrectValue() {
    def invalidString = "123a"
    def validString = "123"
    def invalidNum = invalidString?.isInteger() ? invalidString as Integer : false
    def correctNum = validString?.isInteger() ? validString as Integer : false

    assertEquals(false, invalidNum)
    assertEquals(123, correctNum)
}
```

## 9.摘要

在这篇短文中，我们展示了一些在 Groovy 中从`String `对象切换到`Integer`对象的有效方法。

当谈到选择转换对象类型的最佳方法时，**以上所有方法都同样适用。**最重要的是避免错误，首先**检查我们的应用程序中`String`的值是否可以是非数字、空或者`null`T5。**

像往常一样，所有代码示例都可以在 GitHub 上找到[。](https://web.archive.org/web/20221221232606/https://github.com/eugenp/tutorials/tree/master/core-groovy-modules/core-groovy)