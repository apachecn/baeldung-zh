# 使用点“.”作为 Java 中的小数点分隔符

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-dot-decimal-separator>

## 1.概观

在这个简短的教程中，我们将看到如何使用点“.”在 Java 中格式化数字输出时作为小数点分隔符。

## 2.使用`String.format()`方法

通常，我们只需要使用`String.format()`方法:

```java
double d = 10.01d;
String.format("%.2f", d);
```

这个方法使用我们 JVM 的默认 [`Locale`](/web/20220627170218/https://www.baeldung.com/java-8-localization) 来选择小数点分隔符。例如，对美国来说是一个点`Locale,`，对德国来说是一个逗号。

如果它不是一个点，我们可以**使用这个方法的重载版本，在这里我们传入我们的自定义`Locale`** :

```java
String.format(Locale.US, "%.2f", d);
```

## 3.使用`DecimalFormat`对象

我们可以用一个 [`DecimalFormat`](/web/20220627170218/https://www.baeldung.com/java-decimalformat) 对象的`format()`方法来达到同样的目的:

```java
DecimalFormatSymbols decimalFormatSymbols = DecimalFormatSymbols.getInstance();
decimalFormatSymbols.setDecimalSeparator('.');
new DecimalFormat("0.00", decimalFormatSymbols).format(d);
```

## 4.使用`Formatter`对象

我们也可以用`format()`的方法得到一个 [`Formatter`](/web/20220627170218/https://www.baeldung.com/java-string-formatter) 的对象:

```java
new Formatter(Locale.US).format("%.2f", d)
```

## 5.使用`Locale.setDefault()`方法

当然，我们可以为我们的应用手动配置`Locale`，但是不建议**更改默认`Locale`**:

```java
Locale.setDefault(Locale.US);
String.format("%.2f", d);
```

## 6.使用虚拟机选项

为我们的应用程序配置`Locale`的另一种方式是设置`user.language`和`user.region`虚拟机选项:

```java
-Duser.language=en -Duser.region=US
```

## 7.使用`printf()`方法

如果我们不需要得到格式化字符串的值而只是把它打印出来，我们可以使用 [`printf()`](/web/20220627170218/https://www.baeldung.com/java-printstream-printf) 方法:

```java
System.out.printf(Locale.US, "%.2f", d);
```

## 8.结论

总之，我们已经学会了使用点“.”的不同方法作为 Java 中的小数点分隔符。