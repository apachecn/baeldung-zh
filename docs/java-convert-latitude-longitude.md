# 将纬度和经度转换为 Java 中的 2D 点

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-convert-latitude-longitude>

## 1。概述

当实现使用地图的应用程序时，我们通常会遇到坐标转换的问题。很多时候，我们需要**将经纬度转换成一个 2D 点来显示**。幸运的是，为了解决这个问题，我们可以利用墨卡托投影的公式。

在本教程中，我们将涵盖墨卡托投影，并学习如何实现它的两个变种。

## 2。墨卡托投影

墨卡托投影是由佛兰德制图家杰拉杜斯·麦卡托在 1569 年引入的一种地图投影。地图投影将地球上的经纬度坐标转换为平面上的一点。换句话说，**它将地球表面的一个点转化为平面地图上的一个点**。

有两种实现墨卡托投影的方法。伪墨卡托投影将地球视为球体。真实墨卡托投影将地球建模为椭球体。我们将实现这两个版本。

让我们从两种墨卡托投影实现的基类开始:

```java
abstract class Mercator {
    final static double RADIUS_MAJOR = 6378137.0;
    final static double RADIUS_MINOR = 6356752.3142;

    abstract double yAxisProjection(double input);
    abstract double xAxisProjection(double input);
}
```

这个类还提供了以米为单位的地球的长径和短径。众所周知，地球不完全是一个球体。因此，我们需要两个半径。首先，**大半径是从地球中心到赤道**的距离。其次，**小半径是从地球中心到南北极的距离**。

### 2.1。球形墨卡托投影

伪投影模型将地球视为球体。与地球被投影成更精确形状的椭圆投影相反。这种方法允许我们从**快速估算**到更精确但计算量更大的椭圆投影。因此，在这个投影中距离的直接**测量将是近似的。**

此外，地图上形状的比例会发生微小的变化。由于纬度和地图上物体形状的比例，比如国家、湖泊、河流等等。难道**不是被精确保存下来的**。

这也被称为[网络墨卡托](https://web.archive.org/web/20220626204446/https://en.wikipedia.org/wiki/Web_Mercator_projection)投影——通常用于包括谷歌地图在内的网络应用。

让我们实现这种方法:

```java
public class SphericalMercator extends Mercator {

    @Override
    double xAxisProjection(double input) {
        return Math.toRadians(input) * RADIUS_MAJOR;
    }

    @Override
    double yAxisProjection(double input) {
        return Math.log(Math.tan(Math.PI / 4 + Math.toRadians(input) / 2)) * RADIUS_MAJOR;
    }
}
```

这种方法首先要注意的是，这种方法用一个常数而不是两个常数**来表示地球的**半径**。其次，我们可以看到，我们实现了两个函数，用于转换到 **x 轴投影**和 **y 轴投影**。在上面的类中，我们使用了 java 提供的`Math`库来帮助我们简化代码。**

让我们测试一个简单的转换:

```java
Assert.assertEquals(2449028.7974520186, sphericalMercator.xAxisProjection(22));
Assert.assertEquals(5465442.183322753, sphericalMercator.yAxisProjection(44));
```

值得注意的是，这种投影会将点映射到(-20037508.34，-23810769.32，20037508.34，23810769.32)的一个包围盒(左、下、右、上)。

### 2 **.2。椭圆墨卡托投影**

真实投影将地球建模为椭球体。这个投影给出了地球上任何地方物体的****。当然，**尊重地图上的物体，但是** **不是 100%准确**。然而，这种方法不是最常用的，因为它计算复杂。****

 ****让我们实现这种方法:

```java
class EllipticalMercator extends Mercator {
    @Override
    double yAxisProjection(double input) {

        input = Math.min(Math.max(input, -89.5), 89.5);
        double earthDimensionalRateNormalized = 1.0 - Math.pow(RADIUS_MINOR / RADIUS_MAJOR, 2);

        double inputOnEarthProj = Math.sqrt(earthDimensionalRateNormalized) * 
          Math.sin( Math.toRadians(input));

        inputOnEarthProj = Math.pow(((1.0 - inputOnEarthProj) / (1.0+inputOnEarthProj)), 
          0.5 * Math.sqrt(earthDimensionalRateNormalized));

        double inputOnEarthProjNormalized = 
          Math.tan(0.5 * ((Math.PI * 0.5) - Math.toRadians(input))) / inputOnEarthProj;

        return (-1) * RADIUS_MAJOR * Math.log(inputOnEarthProjNormalized);
    }

    @Override
    double xAxisProjection(double input) {
        return RADIUS_MAJOR * Math.toRadians(input);
    }
}
```

上面我们可以看到这种方法在 y 轴上的投影是多么复杂。这是因为它应该考虑到非圆形的地球形状。虽然真实的墨卡托方法看起来很复杂，但它比球面方法更准确，因为它使用半径来表示地球的一个小半径和一个大半径。

让我们测试一个简单的转换:

```java
Assert.assertEquals(2449028.7974520186, ellipticalMercator.xAxisProjection(22));
Assert.assertEquals(5435749.887511954, ellipticalMercator.yAxisProjection(44));
```

该投影会将点映射到(-20037508.34，-34619289.37，20037508.34，34619289.37)的边界框中。

## 3 **。结论**

如果我们需要将纬度和经度坐标转换到 2D 表面，我们可以使用墨卡托投影。根据我们实现所需的精度，我们可以使用球形或椭圆形方法。

和往常一样，我们可以在 GitHub 上找到这篇文章的代码[。](https://web.archive.org/web/20220626204446/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-math-2)****