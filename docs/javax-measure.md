# javax.measure 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/javax-measure>

## 1。概述

在本文中，我们将介绍度量单位 API——它为**提供了一种在 Java** 中表示度量和单位的统一方式。

在处理包含物理量的程序时，我们需要消除所用单位的不确定性。我们必须管理好数字和它的单位，以防止计算中的错误。

`JSR-363`(原为`JSR-275`或`javax.measure`库)帮助我们节省开发时间，同时，使代码更易读。

## 2。Maven 依赖关系

让我们简单地从 Maven 依赖项开始引入库:

```java
<dependency>
    <groupId>javax.measure</groupId>
    <artifactId>unit-api</artifactId>
    <version>1.0</version>
</dependency> 
```

最新版本可以在 [Maven Central](https://web.archive.org/web/20220627082830/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22unit-api%22) 上找到。

`unit-api`项目包含一组定义如何处理数量和单位的界面。对于示例，我们将使用`JSR-363`的参考实现，即`[unit-ri](https://web.archive.org/web/20220627082830/https://search.maven.org/classic/#search%7Cga%7C1%7Cunit-ri)`:

```java
<dependency>
    <groupId>tec.units</groupId>
    <artifactId>unit-ri</artifactId>
    <version>1.0.3</version>
</dependency>
```

## 3。探索 API

让我们来看一个例子，我们想把水储存在一个水箱里。

遗留实现将如下所示:

```java
public class WaterTank {
    public void setWaterQuantity(double quantity);
}
```

正如我们所看到的，上面的代码没有提到水量的单位，并且由于`double`类型的存在，不适合精确的计算。

如果开发人员错误地传递了与我们期望的度量单位不同的值，这可能会导致严重的计算错误。这种错误很难检测和解决。

**`JSR-363`API 为我们提供了`Quantity`和`Unit`接口** `,` ，它们解决了这种混乱，并且将这类错误排除在我们程序的范围之外。

### 3.1。简单的例子

现在，让我们探索一下，看看这在我们的例子中是如何有用的。

如前所述， *JSR-363* 包含**`Quantity`接口，它代表一个数量属性**，如体积或面积。该库提供了许多子接口，对最常用的可量化属性进行建模。一些例子有:`Volume`、`Length`、`ElectricCharge`、`Energy`、`Temperature`。

我们可以定义 *Quantity < Volume >* 对象，在我们的例子中它应该存储水量:

```java
public class WaterTank {
    public void setCapacityMeasure(Quantity<Volume> capacityMeasure);
}
```

除了`Quantity`接口，**我们还可以使用`Unit`接口来标识属性**的度量单位。常用单位的定义可以在`unit-ri`库中找到，例如:`KELVIN`、`METRE`、`NEWTON`、`CELSIUS`。

类型为`Quantity<Q extends Quantity<Q>>`的对象有检索单位和值的方法:`getUnit()`和`getValue()`。

让我们看一个设置水量值的示例:

```java
@Test
public void givenQuantity_whenGetUnitAndConvertValue_thenSuccess() {
    WaterTank waterTank = new WaterTank();
    waterTank.setCapacityMeasure(Quantities.getQuantity(9.2, LITRE));
    assertEquals(LITRE, waterTank.getCapacityMeasure().getUnit());

    Quantity<Volume> waterCapacity = waterTank.getCapacityMeasure();
    double volumeInLitre = waterCapacity.getValue().doubleValue();
    assertEquals(9.2, volumeInLitre, 0.0f);
}
```

我们也可以将 `LITRE`中的这个`Volume` 快速转换成任何其他单位:

```java
double volumeInMilliLitre = waterCapacity
  .to(MetricPrefix.MILLI(LITRE)).getValue().doubleValue();
assertEquals(9200.0, volumeInMilliLitre, 0.0f);
```

但是，当我们试图将水量转换成另一个单位时——这不是类型`Volume`,我们得到一个编译错误:

```java
// compilation error
waterCapacity.to(MetricPrefix.MILLI(KILOGRAM));
```

### 3.2。类参数化

为了保持维度的一致性，框架自然会利用泛型。

类和接口通过它们的数量类型参数化，这使得在编译时检查我们的单元成为可能。编译器将根据它可以识别的内容给出错误或警告:

```java
Unit<Length> Kilometer = MetricPrefix.KILO(METRE);
Unit<Length> Centimeter = MetricPrefix.CENTI(LITRE); // compilation error
```

总是有可能使用`asType()`方法绕过类型检查:

```java
Unit<Length> inch = CENTI(METER).times(2.54).asType(Length.class);
```

如果不确定数量的类型，我们也可以使用通配符:

```java
Unit<?> kelvinPerSec = KELVIN.divide(SECOND);
```

## 4。单位转换

`Unit` s 可以从`SystemOfUnits`中检索。规范的参考实现包含接口的`Units`实现，它提供了一组静态常量，代表最常用的单位。

此外，我们还可以创建一个全新的定制单元，或者通过对现有单元应用代数运算来创建一个单元。

使用标准单位的好处是我们不会陷入转换陷阱。

我们也可以使用前缀，或者来自`MetricPrefix`类的乘数，比如`KILO(Unit<Q> unit)`和`CENTI(Unit<Q> unit)`，它们分别相当于乘和除 10 的幂。

例如，我们可以将“千米”和“厘米”定义为:

```java
Unit<Length> Kilometer = MetricPrefix.KILO(METRE);
Unit<Length> Centimeter = MetricPrefix.CENTI(METRE);
```

这些可以用在我们想要的单元不能直接得到的时候。

### 4.1。定制单位

在任何情况下，如果一个单位在单位制中不存在，我们可以用新的符号创造新的单位:

*   `AlternateUnit –` 量纲相同但符号和性质不同的新单位
*   作为其他单位理性力量的产物而产生的新单位

让我们使用这些类创建一些定制单元。`AlternateUnit`为压力的一个例子:

```java
@Test
public void givenUnit_whenAlternateUnit_ThenGetAlternateUnit() {
    Unit<Pressure> PASCAL = NEWTON.divide(METRE.pow(2))
      .alternate("Pa").asType(Pressure.class);
    assertTrue(SimpleUnitFormat.getInstance().parse("Pa")
      .equals(PASCAL));
}
```

同样，`ProductUnit` 及其转换的例子:

```java
@Test
public void givenUnit_whenProduct_ThenGetProductUnit() {
    Unit<Area> squareMetre = METRE.multiply(METRE).asType(Area.class);
    Quantity<Length> line = Quantities.getQuantity(2, METRE);
    assertEquals(line.multiply(line).getUnit(), squareMetre);
}
```

这里，我们通过将`METRE`与自身相乘创建了一个`squareMetre`复合单元。

接下来，对于单元的类型，框架还提供了一个`UnitConverter`类，帮助我们将一个单元转换成另一个单元，或者创建一个新的派生单元`TransformedUnit`。

让我们看一个将双精度值的单位从米转换为千米的示例:

```java
@Test
public void givenMeters_whenConvertToKilometer_ThenConverted() {
    double distanceInMeters = 50.0;
    UnitConverter metreToKilometre = METRE.getConverterTo(MetricPrefix.KILO(METRE));
    double distanceInKilometers = metreToKilometre.convert(distanceInMeters );
    assertEquals(0.05, distanceInKilometers, 0.00f);
}
```

为了促进数量与其单位的明确的电子通信，库提供了`UnitFormat`接口*、*，该接口将系统范围的标签与`Units`相关联。

让我们使用`SimpleUnitFormat`实现来检查一些系统单元的标签:

```java
@Test
public void givenSymbol_WhenCompareToSystemUnit_ThenSuccess() {
    assertTrue(SimpleUnitFormat.getInstance().parse("kW")
      .equals(MetricPrefix.KILO(WATT)));
    assertTrue(SimpleUnitFormat.getInstance().parse("ms")
      .equals(SECOND.divide(1000)));
}
```

## 5。使用数量执行操作

`Quantity`接口包含最常见的数学运算方法:`add()`、`subtract()`、`multiply()`、`divide()`。使用这些，我们可以在`Quantity`对象之间执行操作:

```java
@Test
public void givenUnits_WhenAdd_ThenSuccess() {
    Quantity<Length> total = Quantities.getQuantity(2, METRE)
      .add(Quantities.getQuantity(3, METRE));
    assertEquals(total.getValue().intValue(), 5);
}
```

这些方法还验证它们正在操作的对象的`Units`。例如，试图将米乘以升会导致编译错误:

```java
// compilation error
Quantity<Length> total = Quantities.getQuantity(2, METRE)
  .add(Quantities.getQuantity(3, LITRE));
```

另一方面，以具有相同尺寸的单位表示的两个对象可以相加:

```java
Quantity<Length> totalKm = Quantities.getQuantity(2, METRE)
  .add(Quantities.getQuantity(3, MetricPrefix.KILO(METRE)));
assertEquals(totalKm.getValue().intValue(), 3002);
```

在本例中，米和千米单位都对应于`Length`维度，因此可以添加它们。结果以第一个对象的单位表示。

## 6。结论

在本文中，我们看到*度量单位 API* 给了我们一个方便的度量模型。此外，除了`Quantity`和`Unit`的用法，我们还看到了以多种方式将一个单位转换成另一个单位是多么方便。

要了解更多信息，您可以随时点击查看[项目。](https://web.archive.org/web/20220627082830/https://github.com/unitsofmeasurement)

和往常一样，完整的代码可以在 GitHub 上获得[。](https://web.archive.org/web/20220627082830/https://github.com/eugenp/tutorials/tree/master/libraries-data-2)