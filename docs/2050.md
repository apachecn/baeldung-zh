# 在 Java 中求两条线的交点

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-intersection-of-two-lines>

## 1。概述

在这个快速教程中，我们将向**展示如何找到由斜率截距形式的线性函数**定义的两条直线的交点。

## 2。交集的数学公式

平面上的任何直线(垂直线除外)都可以用线性函数来定义:

```
y = mx + b
```

其中`m`是斜率，`b`是 y 轴截距。

对于一条垂直线，`m`将等于无穷大，这就是为什么我们排除它。如果两条直线平行，它们具有相同的斜率，即相同的`m`值。

假设我们有两条线。第一个函数定义第一行:

```
y = m1x + b1
```

第二个函数定义了第二行:

```
y = m2x + b2
```

[![](img/e558b19cd41a063060f11d1018227037.png)](/web/20220523233851/https://www.baeldung.com/wp-content/uploads/2018/09/general-y1-y2.png) 
我们要找到这些线的交点。显然，这个等式对于交点是正确的:

```
y1 = y2
```

让我们代入`y-`变量:

```
m1x + b1 = m2x + b2
```

**从上面的等式我们可以找到`x-`坐标:**

```
x(m1 - m2) = b2 - b1
x = (b2 - b1) / (m1 - m2)
```

**最后可以找到交点的 y 坐标:**

```
y = m1x + b1
```

现在让我们转到实现部分。

## 3。Java 实现

首先，我们有四个输入变量——`m1, b1`用于第一行，而`m2, b2`用于第二行。

其次，我们将把计算出的交点转换成`java.awt.Point`类型的对象。

最后，线可能是平行的，因此让我们将返回值设为`Optional<Point>`:

```
public Optional<Point> calculateIntersectionPoint(
    double m1, 
    double b1, 
    double m2, 
    double b2) {

    if (m1 == m2) {
        return Optional.empty();
    }

    double x = (b2 - b1) / (m1 - m2);
    double y = m1 * x + b1;

    Point point = new Point();
    point.setLocation(x, y);
    return Optional.of(point);
}
```

现在让我们选择一些值，并针对平行线和非平行线测试该方法。

比如，我们把`x`-轴(`y = 0`)作为第一条线，把`y = x – 1` 定义的线作为第二条线。

对于第二条直线，斜率`m`等于`1`表示`45`度， `y`-截距等于`-1` 表示直线在点(0，-1)处与`y`-轴相交。

很明显，第二条线与`x`轴的交点一定是`(1,0`):

[![](img/eee4975bd3f14ee694082dedd350c903.png)](/web/20220523233851/https://www.baeldung.com/wp-content/uploads/2018/09/non-parallel.png)

让我们检查一下。

首先，让我们确保存在一个`Point`，因为这些线不平行，然后检查`x`和`y`的值:

```
@Test
public void givenNotParallelLines_whenCalculatePoint_thenPresent() {
    double m1 = 0;
    double b1 = 0;
    double m2 = 1;
    double b2 = -1;

    Optional<Point> point = service.calculateIntersectionPoint(m1, b1, m2, b2);

    assertTrue(point.isPresent());
    assertEquals(point.get().getX(), 1, 0.001);
    assertEquals(point.get().getY(), 0, 0.001);
}
```

最后，让我们取两条平行线，并确保返回值为空:

[![](img/5e7cb40720c8dd5a1b8e15bc5e982a04.png)](/web/20220523233851/https://www.baeldung.com/wp-content/uploads/2018/09/parallel.png)

```
@Test
public void givenParallelLines_whenCalculatePoint_thenEmpty() {
    double m1 = 1;
    double b1 = 0;
    double m2 = 1;
    double b2 = -1;

    Optional<Point> point = service.calculateIntersectionPoint(m1, b1, m2, b2);

    assertFalse(point.isPresent());
}
```

## 4。结论

在本教程中，我们已经展示了如何计算两条线的交点。

和往常一样，完整的源代码可以在 GitHub 的[上找到。](https://web.archive.org/web/20220523233851/https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lang-math-2)