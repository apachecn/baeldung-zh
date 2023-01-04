# 用 Java 获取当前日期和时间

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/current-date-time-and-timestamp-in-java-8>

## 1。简介

这篇简短的文章描述了我们如何在 Java 8 中获得当前日期、当前时间和当前时间戳。

## 2。当前日期

首先，让我们使用`java.time.LocalDate`来获取当前系统日期:

```java
LocalDate localDate = LocalDate.now();
```

要获得任何其他时区的日期，我们可以使用`LocalDate.now(ZoneId)`:

```java
LocalDate localDate = LocalDate.now(ZoneId.of("GMT+02:30"));
```

我们也可以使用`java.time.LocalDateTime`来获得`LocalDate:`的一个实例

```java
LocalDateTime localDateTime = LocalDateTime.now();
LocalDate localDate = localDateTime.toLocalDate();
```

## 3。当前时间

使用`java.time.LocalTime`，让我们检索当前系统时间:

```java
LocalTime localTime = LocalTime.now();
```

要获得特定时区的当前时间，我们可以使用`LocalTime.now(ZoneId)`:

```java
LocalTime localTime = LocalTime.now(ZoneId.of("GMT+02:30"));
```

我们也可以使用`java.time.LocalDateTime`来获得`LocalTime:`的一个实例

```java
LocalDateTime localDateTime = LocalDateTime.now();
LocalTime localTime = localDateTime.toLocalTime();
```

## 4。当前时间戳

使用`java.time.Instant`从 Java epoch 中获取一个时间戳。根据 [JavaDoc](https://web.archive.org/web/20221208143856/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/Instant.html) ，“纪元秒从标准的 Java 纪元 1970-01-01T00:00:00Z 开始计算，其中纪元后的瞬间具有正值:

```java
Instant instant = Instant.now();
long timeStampMillis = instant.toEpochMilli();
```

我们可以获得纪元秒数:

```java
Instant instant = Instant.now();
long timeStampSeconds = instant.getEpochSecond();
```

## 5。结论

在本教程中，我们主要使用`java.time.*`来获取当前日期、时间和时间戳。

和往常一样，这篇文章的代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221208143856/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-8-datetime)