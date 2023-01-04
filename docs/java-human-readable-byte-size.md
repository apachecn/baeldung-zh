# 用 Java 将字节大小转换成人类可读的格式

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-human-readable-byte-size>

## 1.概观

当我们在 Java 中得到一个文件的大小时，通常，我们会得到字节值。然而，一旦一个文件足够大，例如 123456789 字节，看到以字节表示的长度对我们试图理解文件有多大就成了一个挑战。

在本教程中，我们将探索如何用 Java 将文件大小从字节转换成人类可读的格式。

## 2.问题简介

正如我们前面谈到的，当一个文件的字节数很大时，对人类来说不容易理解。因此，当我们向人类呈现大量数据时，我们通常使用适当的 SI 前缀，如 KB、MB、GB 等，以使大量数据可读。比如“270GB”就比“282341192 字节”容易理解得多。

然而，当我们通过标准的 Java API 获取文件大小时，通常是以字节为单位的。因此，为了具有人类可读的格式，我们需要将值从字节单位动态转换为相应的二进制前缀，例如，将“282341192 字节”转换为“207MiB”，或者将“2048 字节”转换为“2KiB”。

值得一提的是，单位前缀有两种变体:

*   [二进制前缀](https://web.archive.org/web/20221208143917/https://en.wikipedia.org/wiki/Binary_prefix)——它们是 1024 的幂；例如，1GiB = 1024 MiB，1GiB = 1024 MiB，依此类推
*   SI ( [国际单位制](https://web.archive.org/web/20221208143917/https://en.wikipedia.org/wiki/International_System_of_Units))前缀——它们是 1000 的幂；例如，1MB = 1000 KB，1GB = 1000 MB，以此类推。

我们的教程将关注二进制前缀和 SI 前缀。

## 3.解决问题

我们可能已经意识到**解决问题的关键是动态地找到合适的单元。**

例如，如果输入小于 1024，比如说 200，那么我们需要以字节为单位拥有“200 个字节”。然而，当输入大于 1024 但小于 1024 * 1024 时，例如 4096，我们应该使用 KiB 单位，所以我们有“4 KiB”。

但是，让我们一步一步地解决问题。在我们深入单元确定逻辑之前，让我们首先定义所有需要的单元及其边界。

### 3.1.定义所需单位

正如我们所知，**一个单位乘以 1024 将转移到下一级单位**。因此，我们可以创建常数，用它们的基值表示所有必需的单位:

```java
private static long BYTE = 1L;
private static long KiB = BYTE << 10;
private static long MiB = KiB << 10;
private static long GiB = MiB << 10;
private static long TiB = GiB << 10;
private static long PiB = TiB << 10;
private static long EiB = PiB << 10; 
```

如上面的代码所示，我们使用了二进制的[左移运算符](/web/20221208143917/https://www.baeldung.com/java-bitwise-operators#1-signed-left-shift-ltlt) ( < <)来计算基值。在这里，**、`x << 10`、`x * 1024`都一样，因为 1024 是 2 的 10 次方**。

对于 SI 前缀**，一个单元乘以 1000 将过渡到下一级的单元**。因此，我们可以创建常数，用它们的基值表示所有必需的单位:

```java
private static long KB = BYTE * 1000;
private static long MB = KB * 1000;
private static long GB = MB * 1000;
private static long TB = GB * 1000;
private static long PB = TB * 1000;
private static long EB = PB * 1000;
```

### 3.1.定义数字格式

假设我们已经确定了正确的单位，并希望将文件大小表示为两位小数，我们可以创建一个方法来输出结果:

```java
private static DecimalFormat DEC_FORMAT = new DecimalFormat("#.##");

private static String formatSize(long size, long divider, String unitName) {
    return DEC_FORMAT.format((double) size / divider) + " " + unitName;
} 
```

接下来，让我们快速了解该方法的作用。正如我们在上面的代码中看到的，首先，我们定义了[数字格式](/web/20221208143917/https://www.baeldung.com/java-number-formatting#3-formatting-numbers-with-two-zeros-after-the-decimal) `DEC_FORMAT.`

`divider`参数是所选单位的基值，而`String`参数`unitName`是单位的名称。例如，如果我们选择了 KiB 作为合适的单位，`divider=1024`和`unitName = “KiB”.`

此方法集中了除法计算和数字格式转换。

现在，是时候进入解决方案的核心部分了:找到合适的单元。

### 3.2.确定单位

我们先来看看单位确定方法的实现:

```java
public static String toHumanReadableBinaryPrefixes(long size) {
    if (size < 0)
        throw new IllegalArgumentException("Invalid file size: " + size);
    if (size >= EiB) return formatSize(size, EiB, "EiB");
    if (size >= PiB) return formatSize(size, PiB, "PiB");
    if (size >= TiB) return formatSize(size, TiB, "TiB");
    if (size >= GiB) return formatSize(size, GiB, "GiB");
    if (size >= MiB) return formatSize(size, MiB, "MiB");
    if (size >= KiB) return formatSize(size, KiB, "KiB");
    return formatSize(size, BYTE, "Bytes");
} 
```

```java
public static String toHumanReadableSIPrefixes(long size) {
    if (size < 0)
        throw new IllegalArgumentException("Invalid file size: " + size);
    if (size >= EB) return formatSize(size, EB, "EB");
    if (size >= PB) return formatSize(size, PB, "PB");
    if (size >= TB) return formatSize(size, TB, "TB");
    if (size >= GB) return formatSize(size, GB, "GB");
    if (size >= MB) return formatSize(size, MB, "MB");
    if (size >= KB) return formatSize(size, KB, "KB");
    return formatSize(size, BYTE, "Bytes");
} 
```

现在，让我们浏览一下这个方法，了解它是如何工作的。

首先，我们要确保输入是一个正数。

**然后，我们按照从高(EB)到低(Byte)的方向检查单位。**一旦我们发现输入`size`大于或等于当前单位的基值，当前单位将是正确的。

一旦我们找到了正确的单元，我们就可以调用之前创建的`formatSize`方法来获得作为`String`的最终结果。

### 3.3.测试解决方案

现在，让我们编写一个单元测试方法来验证我们的解决方案是否如预期的那样工作。为了简化测试方法，让我们用[初始化一个`Map`](/web/20221208143917/https://www.baeldung.com/java-initialize-hashmap) `<Long, String>` 保存输入和相应的预期结果:

```java
private static Map<Long, String> DATA_MAP_BINARY_PREFIXES = new HashMap<Long, String>() {{
    put(0L, "0 Bytes");
    put(1023L, "1023 Bytes");
    put(1024L, "1 KiB");
    put(12_345L, "12.06 KiB");
    put(10_123_456L, "9.65 MiB");
    put(10_123_456_798L, "9.43 GiB");
    put(1_777_777_777_777_777_777L, "1.54 EiB");
}}; 
```

```java
private final static Map<Long, String> DATA_MAP_SI_PREFIXES = new HashMap<Long, String>() {{
    put(0L, "0 Bytes");
    put(999L, "999 Bytes");
    put(1000L, "1 KB");
    put(12_345L, "12.35 KB");
    put(10_123_456L, "10.12 MB");
    put(10_123_456_798L, "10.12 GB");
    put(1_777_777_777_777_777_777L, "1.78 EB");
}};
```

接下来，让[遍历`Map`](/web/20221208143917/https://www.baeldung.com/java-iterate-map) `DATA_MAP`，将每个键值作为输入，验证是否能得到预期的结果:

```java
DATA_MAP.forEach((in, expected) -> Assert.assertEquals(expected, FileSizeFormatUtil.toHumanReadable(in)));
```

当我们执行单元测试时，它通过了。

## 4.使用枚举和循环改进解决方案

到目前为止，我们已经解决了这个问题。解决方案非常简单。在`toHumanReadable`方法中，我们已经编写了多个`if`语句来确定单位。

如果我们仔细考虑这个解决方案，有几点可能容易出错:

*   这些`if`语句的顺序必须固定，因为它们在方法中。
*   在每个`if`语句中，我们将单位常数和相应的名称硬编码为一个`String`对象。

接下来，我们来看看如何改进解决方案。

### 4.1.创建`SizeUnit enum`

实际上，我们可以将单位常数转换成一个 [`enum`](/web/20221208143917/https://www.baeldung.com/a-guide-to-java-enums) ,这样我们就不必在方法中硬编码名称了:

```java
enum SizeUnitBinaryPrefixes {
    Bytes(1L),
    KiB(Bytes.unitBase << 10),
    MiB(KiB.unitBase << 10),
    GiB(MiB.unitBase << 10),
    TiB(GiB.unitBase << 10),
    PiB(TiB.unitBase << 10),
    EiB(PiB.unitBase << 10);

    private final Long unitBase;

    public static List<SizeUnitBinaryPrefixes> unitsInDescending() {
        List<SizeUnitBinaryPrefixes> list = Arrays.asList(values());
        Collections.reverse(list);
        return list;
    }
   //getter and constructor are omitted
} 
```

```java
enum SizeUnitSIPrefixes {
    Bytes(1L),
    KB(Bytes.unitBase * 1000),
    MB(KB.unitBase * 1000),
    GB(MB.unitBase * 1000),
    TB(GB.unitBase * 1000),
    PB(TB.unitBase * 1000),
    EB(PB.unitBase * 1000);

    private final Long unitBase;

    public static List<SizeUnitSIPrefixes> unitsInDescending() {
        List<SizeUnitSIPrefixes> list = Arrays.asList(values());
        Collections.reverse(list);
        return list;
     }
    //getter and constructor are omitted
}
```

正如上面的`enum SizeUnit`所示，一个`SizeUnit`实例包含了`unitBase`和`name`。

此外，因为我们希望以后按“降序”检查单元，所以我们创建了一个助手方法，`unitsInDescending,`来按要求的顺序返回所有单元。

**有了这个`enum`，我们就不用手动给名字打码了。**

接下来，让我们看看是否可以对这组`if`语句进行一些改进。

### 4.2.使用循环来确定单位

由于我们的`SizeUnit enum` 可以按降序提供`List`中的所有单元，我们可以用一个`for`循环来替换`if`语句集:

```java
public static String toHumanReadableWithEnum(long size) {
    List<SizeUnit> units = SizeUnit.unitsInDescending();
    if (size < 0) {
        throw new IllegalArgumentException("Invalid file size: " + size);
    }
    String result = null;
    for (SizeUnit unit : units) {
        if (size >= unit.getUnitBase()) {
            result = formatSize(size, unit.getUnitBase(), unit.name());
            break;
        }
    }
    return result == null ? formatSize(size, SizeUnit.Bytes.getUnitBase(), SizeUnit.Bytes.name()) : result;
} 
```

如上面的代码所示，该方法遵循与第一个解决方案相同的逻辑。另外，**它避免了那些单元常量，多个`if`语句，以及硬编码的单元名。**

为了确保它按预期工作，让我们测试我们的解决方案:

```java
DATA_MAP.forEach((in, expected) -> Assert.assertEquals(expected, FileSizeFormatUtil.toHumanReadableWithEnum(in)));
```

当我们执行它时，测试就通过了。

## 5.使用`Long.numberOfLeadingZeros`方法

我们已经解决了这个问题，通过一个接一个地检查单元，并选择第一个满足我们条件的单元。

或者，我们可以使用 Java 标准 API 中的 [`Long.numberOfLeadingZeros`](https://web.archive.org/web/20221208143917/https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Long.html#numberOfLeadingZeros(long)) 方法来确定给定的大小值属于哪个单位。

接下来，让我们仔细看看这个有趣的方法。

### 5.1.`Long.numberOfLeadingZeros`方法介绍

**`Long.numberOfLeadingZeros`方法返回给定`Long`值的二进制表示中最左边一位之前的零位数。**

由于 **Java 的`Long`类型是 64 位整数，`Long.numberOfLeadingZeros(0L) = 64`。**举几个例子可以帮助我们快速理解这种方法:

```java
1L  = 00... (63 zeros in total) ..            0001 -> Long.numberOfLeadingZeros(1L) = 63
1024L = 00... (53 zeros in total) .. 0100 0000 0000 -> Long.numberOfLeadingZeros(1024L) = 53
```

现在，我们已经理解了`Long.numberOfLeadingZeros`方法。但是为什么它能帮助我们确定单位呢？

让我们弄清楚。

### 5.2.解决问题的想法

我们知道单位之间的因子是 1024，也就是 2 的 10 次方(`2^10`)。因此，**如果我们计算每个单元基值的前导零个数，那么两个相邻单元的差值总是 10** :

```java
Index  Unit	numberOfLeadingZeros(unit.baseValue)
----------------------------------------------------
0      Byte	63
1      KiB  	53
2      MiB  	43
3      GiB  	33
4      TiB  	23
5      PiB  	13
6      EiB       3 
```

此外，**我们可以计算输入值的前导零的数量，并查看结果落在哪个单位的范围内，以找到合适的单位**。

接下来，我们来看一个例子，如何确定尺寸 4096 的单位并计算单位基准值:

```java
if 4096 < 1024 (Byte's base value)  -> Byte 
else:
    numberOfLeadingZeros(4096) = 51
    unitIdx = (numberOfLeadingZeros(1) - 51) / 10 = (63 - 51) / 10 = 1
    unitIdx = 1  -> KB (Found the unit)
    unitBase = 1 << (unitIdx * 10) = 1 << 10 = 1024
```

接下来，让我们将这个逻辑实现为一个方法。

### 5.3.实现这个想法

让我们创建一个方法来实现我们刚才讨论的想法:

```java
public static String toHumanReadableByNumOfLeadingZeros(long size) {
    if (size < 0) {
        throw new IllegalArgumentException("Invalid file size: " + size);
    }
    if (size < 1024) return size + " Bytes";
    int unitIdx = (63 - Long.numberOfLeadingZeros(size)) / 10;
    return formatSize(size, 1L << (unitIdx * 10), " KMGTPE".charAt(unitIdx) + "iB");
} 
```

正如我们所看到的，上面的方法非常简洁。它不需要单位常数或者一个`enum`。相反，**我们创建了一个`String`包含单元:`” KMGTPE”`。然后，我们使用计算出的`unitIdx`来选择正确的单元字母，并添加“iB”来构建完整的单元名称。**

值得一提的是，我们在`String` `” KMGTPE”`中故意将第一个字符留空。这是因为单位“`Byte`”没有按照模式“`*B`”，我们单独处理:`if (size < 1024) return size + ” Bytes”;`

同样，让我们编写一个测试方法来确保它按预期工作:

```java
DATA_MAP.forEach((in, expected) -> Assert.assertEquals(expected, FileSizeFormatUtil.toHumanReadableByNumOfLeadingZeros(in)));
```

## 6.使用 Apache Commons IO

到目前为止，我们已经实现了两种不同的方法来将文件大小值转换为人类可读的格式。

实际上，一些外部库已经提供了解决问题的方法: [Apache Commons-IO](/web/20221208143917/https://www.baeldung.com/apache-commons-io) 。

Apache Commons-IO 的`[FileUtils](/web/20221208143917/https://www.baeldung.com/apache-commons-io#1-fileutils)`允许我们通过`byteCountToDisplaySize`方法将字节大小转换为人类可读的格式。

然而，**这个方法会自动将小数部分向上舍入**。

最后，让我们用输入数据测试一下`byteCountToDisplaySize`方法，看看它会打印出什么:

```java
DATA_MAP.forEach((in, expected) -> System.out.println(in + " bytes -> " + FileUtils.byteCountToDisplaySize(in)));
```

测试输出:

```java
0 bytes -> 0 bytes
1024 bytes -> 1 KB
1777777777777777777 bytes -> 1 EB
12345 bytes -> 12 KB
10123456 bytes -> 9 MB
10123456798 bytes -> 9 GB
1023 bytes -> 1023 bytes
```

## 7.结论

在本文中，我们讨论了将文件大小(以字节为单位)转换为人类可读格式的不同方法。

和往常一样，本文中的代码可以从 GitHub 上的[处获得。](https://web.archive.org/web/20221208143917/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-numbers-4)