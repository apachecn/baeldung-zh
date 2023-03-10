# Java 中的旅行推销员问题

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-simulated-annealing-for-traveling-salesman>

## 1。简介

在本教程中，我们将学习模拟退火算法，并展示基于旅行商问题(TSP)的示例实现。

## 2。模拟退火

模拟退火算法是一种解决大搜索空间问题的启发式算法。

灵感和名字来自冶金退火；这是一种涉及加热和控制冷却材料的技术。

一般来说，模拟退火降低了接受更差解决方案的可能性，因为它探索了解决方案空间并降低了系统的温度。下面的[动画](https://web.archive.org/web/20220807183612/https://commons.wikimedia.org/wiki/File:Hill_Climbing_with_Simulated_Annealing.gif)展示了用模拟退火算法寻找最佳解决方案的机制:

[![Hill Climbing with Simulated Annealing](img/0ba1ea419d2c37dcaf4b8f77d6281a78.png)](/web/20220807183612/https://www.baeldung.com/wp-content/uploads/2016/12/Hill_Climbing_with_Simulated_Annealing.gif)

正如我们可以观察到的，该算法在系统的高温下使用更宽的解范围，搜索全局最优。当温度降低时，搜索范围变小，直到找到全局最优。

该算法有几个参数可以使用:

*   迭代次数–模拟的停止条件
*   初始温度——系统的启动能量
*   冷却速率参数–我们降低系统温度的百分比
*   最低温度–可选停止条件
*   模拟时间–可选停止条件

必须仔细选择这些参数的值，因为它们可能对过程的性能有重大影响。

## 3。旅行推销员问题

旅行推销员问题(TSP)是现代世界中最著名的计算机科学优化问题。

简单来说，就是在图中的节点之间寻找最优路径的问题。总行程距离可以是优化标准之一。有关 TSP 的更多详情，请查看[此处](https://web.archive.org/web/20220807183612/https://simple.wikipedia.org/wiki/Travelling_salesman_problem)。

## 4。Java 模型

为了解决 TSP 问题，我们需要两个模型类，即`City`和`Travel`。在第一个示例中，我们将存储图中节点的坐标:

```java
@Data
public class City {

    private int x;
    private int y;

    public City() {
        this.x = (int) (Math.random() * 500);
        this.y = (int) (Math.random() * 500);
    }

    public double distanceToCity(City city) {
        int x = Math.abs(getX() - city.getX());
        int y = Math.abs(getY() - city.getY());
        return Math.sqrt(Math.pow(x, 2) + Math.pow(y, 2));
    }

}
```

类的构造器允许我们创建城市的随机位置。`distanceToCity(..)`逻辑负责计算城市之间的距离。

下面的代码负责对旅行推销员之旅进行建模。让我们从生成旅行中城市的初始顺序开始:

```java
public void generateInitialTravel() {
    if (travel.isEmpty()) {
        new Travel(10);
    }
    Collections.shuffle(travel);
}
```

除了生成初始顺序之外，我们还需要交换旅行顺序中随机的两个城市的方法。我们将使用它在模拟退火算法中搜索更好的解决方案:

```java
public void swapCities() {
    int a = generateRandomIndex();
    int b = generateRandomIndex();
    previousTravel = new ArrayList<>(travel);
    City x = travel.get(a);
    City y = travel.get(b);
    travel.set(a, y);
    travel.set(b, x);
}
```

此外，如果我们的算法不接受新的解决方案，我们需要一种方法来恢复上一步中生成的交换:

```java
public void revertSwap() {
    travel = previousTravel;
}
```

我们要介绍的最后一种方法是总行程距离的计算，它将用作优化标准:

```java
public int getDistance() {
    int distance = 0;
    for (int index = 0; index < travel.size(); index++) {
        City starting = getCity(index);
        City destination;
        if (index + 1 < travel.size()) {
            destination = getCity(index + 1);
        } else {
            destination = getCity(0);
        }
            distance += starting.distanceToCity(destination);
    }
    return distance;
}
```

现在，让我们把重点放在主要部分，模拟退火算法的实现。

## 5。模拟退火实施

在下面的模拟退火实现中，我们将解决 TSP 问题。简单提醒一下，我们的目标是找到穿越所有城市的最短距离。

为了启动流程，我们需要提供三个主要参数，即`startingTemperature`、`numberOfIterations` 和`coolingRate`:

```java
public double simulateAnnealing(double startingTemperature,
  int numberOfIterations, double coolingRate) {
    double t = startingTemperature;
    travel.generateInitialTravel();
    double bestDistance = travel.getDistance();

    Travel currentSolution = travel;
    // ...
}
```

在模拟开始之前，我们生成城市的初始(随机)顺序，并计算旅行的总距离。由于这是第一次计算的距离，我们将它保存在`bestDistance` 变量中，与`currentSolution.`放在一起

在下一步中，我们开始一个主要的模拟循环:

```java
for (int i = 0; i < numberOfIterations; i++) {
    if (t > 0.1) {
        //...
    } else {
        continue;
    }
}
```

循环将持续我们指定的迭代次数。此外，我们添加了一个条件，如果温度将低于或等于 0.1，则停止模拟。这将使我们节省模拟的时间，因为在低温下优化差异几乎不可见。

让我们看看模拟退火算法的主要逻辑:

```java
currentSolution.swapCities();
double currentDistance = currentSolution.getDistance();
if (currentDistance < bestDistance) {
    bestDistance = currentDistance;
} else if (Math.exp((bestDistance - currentDistance) / t) < Math.random()) {
    currentSolution.revertSwap();
}
```

在模拟的每一步中，我们按照旅行顺序随机交换两个城市。

再者，我们计算`currentDistance`。如果新计算出的`currentDistance`小于`bestDistance`，我们将其保存为最佳。

否则，我们检查概率分布的玻尔兹曼函数是否低于从 0-1 范围内随机选取的值。如果是，我们恢复城市的交换。如果没有，我们保持城市的新秩序，因为它可以帮助我们避免局部最小值。

最后，在模拟的每个步骤中，我们通过提供`coolingRate:`来降低温度

```java
t *= coolingRate;
```

在模拟之后，我们返回使用模拟退火找到的最佳解决方案。

**请注意如何选择最佳模拟参数的一些提示:**

*   对于小的解空间，最好降低起始温度并增加冷却速率，因为这将减少模拟时间，而不损失质量
*   对于较大的解空间，请选择较高的起始温度和较小的冷却速率，因为会有更多的局部最小值
*   始终提供足够的时间来模拟系统从高温到低温的过程

在开始主要模拟之前，不要忘记花一些时间对较小的问题实例进行算法调整，因为这将改善最终结果。本文中以[为例展示了模拟退火算法的调整。](https://web.archive.org/web/20220807183612/https://www.researchgate.net/publication/269268529_Simulated_Annealing_algorithm_for_optimization_of_elastic_optical_networks_with_unicast_and_anycast_traffic)

## 6。结论

在这个快速教程中，我们学习了模拟退火算法，并且解决了旅行推销员问题。这很有希望显示这个简单的算法在应用于某些类型的优化问题时是多么的方便。

这篇文章的完整实现可以在 GitHub 上找到[。](https://web.archive.org/web/20220807183612/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-genetic "The Full Registration/Authentication Example Project on Github ")