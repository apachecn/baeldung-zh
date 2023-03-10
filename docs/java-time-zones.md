# 在 Java 中用 GMT 和 UTC 显示所有时区

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-time-zones>

## 1。概述

每当我们处理时间和日期时，我们都需要一个参照系。标准时间是 UTC T1，但是在一些应用中我们也会看到 T2 GMT T3。

简而言之，UTC 是标准，而 GMT 是时区。

这是维基百科告诉我们的使用方法:

> 在大多数情况下，UTC 被认为可以与格林威治标准时间(GMT)互换，但 GMT 不再由科学界精确定义。

换句话说，一旦我们用 UTC 的时区偏移量编译了一个列表，我们也将得到 GMT 的时区偏移量。

首先，我们将看看 Java 8 实现这一点的方法，然后我们将看看如何在 Java 7 中得到同样的结果。

## 2。获取区域列表

首先，我们需要检索所有已定义时区的列表。

为此，`ZoneId`类有一个方便的静态方法:

```java
Set<String> availableZoneIds = ZoneId.getAvailableZoneIds();
```

然后，我们可以使用`Set`来生成一个时区的排序列表及其相应的偏移量:

```java
public List<String> getTimeZoneList(OffsetBase base) {

    LocalDateTime now = LocalDateTime.now();
    return ZoneId.getAvailableZoneIds().stream()
      .map(ZoneId::of)
      .sorted(new ZoneComparator())
      .map(id -> String.format(
        "(%s%s) %s", 
        base, getOffset(now, id), id.getId()))
      .collect(Collectors.toList());
}
```

上面的方法使用了一个`enum`参数，它代表了我们想要看到的偏移:

```java
public enum OffsetBase {
    GMT, UTC
}
```

现在让我们更详细地检查一下代码。

一旦我们获取了所有可用的时区 id，我们就需要一个实际的时间参考，用`LocalDateTime.now().`表示

之后，我们使用 Java 的`Stream` API 来迭代我们的时区`String` id 集合中的每个条目，并将其转换为带有相应偏移量的格式化时区列表。

对于这些条目中的每一个，我们用`map(ZoneId::of).` 生成一个`ZoneId`实例

## 3。获取偏移量

我们还需要找到实际的 UTC 偏移量。例如，在中欧时间的情况下，偏移量将是`+01:00.`

**要获得任何给定区域的 UTC 偏移量，我们可以使用`LocalDateTime's getOffset()`方法。**

还要注意，Java 将`+00:00`偏移量表示为`Z`。

因此，为了使零偏移时区的`String`看起来一致，我们将把`Z`替换为`+00:00:`

```java
private String getOffset(LocalDateTime dateTime, ZoneId id) {
    return dateTime
      .atZone(id)
      .getOffset()
      .getId()
      .replace("Z", "+00:00");
}
```

## 4。制作专区`Comparable`

或者，我们也可以根据偏移量对时区进行排序。

为此，我们将使用一个`ZoneComparator`类:

```java
private class ZoneComparator implements Comparator<ZoneId> {

    @Override
    public int compare(ZoneId zoneId1, ZoneId zoneId2) {
        LocalDateTime now = LocalDateTime.now();
        ZoneOffset offset1 = now.atZone(zoneId1).getOffset();
        ZoneOffset offset2 = now.atZone(zoneId2).getOffset();

        return offset1.compareTo(offset2);
    }
}
```

## 5。显示时区

剩下要做的就是通过为每个`OffsetBase enum`值调用`getTimeZoneList()`方法来将上面的片段放在一起，并显示列表:

```java
public class TimezoneDisplayApp {

    public static void main(String... args) {
        TimezoneDisplay display = new TimezoneDisplay();

        System.out.println("Time zones in UTC:");
        List<String> utc = display.getTimeZoneList(
          TimezoneDisplay.OffsetBase.UTC);
        utc.forEach(System.out::println);

        System.out.println("Time zones in GMT:");
        List<String> gmt = display.getTimeZoneList(
          TimezoneDisplay.OffsetBase.GMT);
        gmt.forEach(System.out::println);
    }
}
```

当我们运行上面的代码时，它将打印 UTC 和 GMT 的时区。

下面是输出的一个片段:

```java
Time zones in UTC:
(UTC+14:00) Pacific/Apia
(UTC+14:00) Pacific/Kiritimati
(UTC+14:00) Pacific/Tongatapu
(UTC+14:00) Etc/GMT-14
```

## 6。Java 7 及之前

Java 8 通过使用`Stream`和`Date and Time`API 使这项任务变得更加容易。

然而，如果我们有一个 Java 7 和之前的项目，我们仍然可以通过依赖带有`getAvailableIDs()`方法的`java.util.TimeZone`类来获得相同的结果:

```java
public List<String> getTimeZoneList(OffsetBase base) {
    String[] availableZoneIds = TimeZone.getAvailableIDs();
    List<String> result = new ArrayList<>(availableZoneIds.length);

    for (String zoneId : availableZoneIds) {
        TimeZone curTimeZone = TimeZone.getTimeZone(zoneId);
        String offset = calculateOffset(curTimeZone.getRawOffset());
        result.add(String.format("(%s%s) %s", base, offset, zoneId));
    }
    Collections.sort(result);
    return result;
}
```

与 Java 8 代码的主要区别是偏移量计算。

我们从 **`TimeZone()`的 `getRawOffset()`方法得到的`rawOffset`用毫秒**表示时区的偏移量。

因此，我们需要使用`TimeUnit`类将其转换为小时和分钟:

```java
private String calculateOffset(int rawOffset) {
    if (rawOffset == 0) {
        return "+00:00";
    }
    long hours = TimeUnit.MILLISECONDS.toHours(rawOffset);
    long minutes = TimeUnit.MILLISECONDS.toMinutes(rawOffset);
    minutes = Math.abs(minutes - TimeUnit.HOURS.toMinutes(hours));

    return String.format("%+03d:%02d", hours, Math.abs(minutes));
}
```

## 7。结论

在这个快速教程中，我们已经看到了如何用 UTC 和 GMT 偏移量来编辑所有可用时区的列表。

和往常一样，GitHub 上的[提供了示例的完整源代码。](https://web.archive.org/web/20220524022938/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-datetime-string)