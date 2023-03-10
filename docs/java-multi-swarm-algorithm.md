# Java 中的多种群优化算法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-multi-swarm-algorithm>

## 1。简介

在这篇文章中，我们将看看多种群优化算法。与同类的其他算法一样，它的目的是通过最大化或最小化特定的函数(称为适应度函数)来找到问题的最佳解决方案。

先说一些理论。

## 2。多种群优化如何工作

多群算法是[群](https://web.archive.org/web/20220625234719/https://en.wikipedia.org/wiki/Particle_swarm_optimization)算法的变体。顾名思义，**群算法通过模拟一组物体在可能解的空间中的运动来解决一个问题。**在多群版本中，有多个群而不是只有一个。

蜂群的基本组成部分叫做粒子。粒子由它的实际位置定义，这也是我们问题的一个可能的解决方案，还有它的速度，用来计算下一个位置。

粒子的速度不断变化，倾向于在所有粒子群中找到的最佳位置，具有一定程度的随机性，以增加覆盖的空间量。

这最终将大多数粒子引向一组有限的点，这些点是适应度函数中的局部最小值或最大值，这取决于我们是试图最小化还是最大化它。

虽然找到的点总是函数的局部最小值或最大值，但它不一定是全局的，因为不能保证算法已经完全探索了解的空间。

出于这个原因，多种群被称为是一种[元启发式](/web/20220625234719/https://www.baeldung.com/cs/nature-inspired-algorithms)–**它找到的解决方案是最好的，但它们可能不是绝对最好的。**

## 3。实施

现在我们知道了什么是多群以及它是如何工作的，让我们看看如何实现它。

对于我们的例子，我们将尝试解决 StackExchange 上发布的这个现实生活中的优化问题:

> 在英雄联盟中，玩家防御物理伤害时的有效生命值由`E=H(100+A)/100`给出，其中`H`是生命值， `A`是护甲。
> 
> 生命值每单位 2.5 金，护甲每单位 18 金。你有 3600 金，你需要优化你的生命值和护甲的有效性`E`来尽可能长时间的抵抗敌方队伍的攻击。每种你应该买多少？

### 3.1。粒子

我们从建模我们的基本构造开始，一个粒子。粒子的状态包括其当前位置，即解决问题的一对生命值和护甲值，粒子在两个轴上的速度以及粒子适应度得分。

我们还将存储我们找到的最佳位置和适应性分数，因为我们需要它们来更新粒子速度:

```java
public class Particle {
    private long[] position;
    private long[] speed;
    private double fitness;
    private long[] bestPosition;	
    private double bestFitness = Double.NEGATIVE_INFINITY;

    // constructors and other methods
}
```

我们选择使用`long` 数组来表示速度和位置，因为我们可以从问题陈述中推断出我们不能购买护甲或生命值的分数，因此解决方案必须在整数域中。

我们不想使用`int` ,因为那会在计算过程中导致溢出问题。

### 3.2。蜂群

接下来，让我们把蜂群定义为粒子的集合。同样，我们还将存储历史最佳位置和分数，以供以后计算。

群体还需要通过给每个粒子分配一个随机的初始位置和速度来处理粒子的初始化。

我们可以粗略地估计出解的一个边界，所以我们把这个极限加到随机数生成器上。

这将减少运行算法所需的计算能力和时间:

```java
public class Swarm {
    private Particle[] particles;
    private long[] bestPosition;
    private double bestFitness = Double.NEGATIVE_INFINITY;

    public Swarm(int numParticles) {
        particles = new Particle[numParticles];
        for (int i = 0; i < numParticles; i++) {
            long[] initialParticlePosition = { 
              random.nextInt(Constants.PARTICLE_UPPER_BOUND),
              random.nextInt(Constants.PARTICLE_UPPER_BOUND) 
            };
            long[] initialParticleSpeed = { 
              random.nextInt(Constants.PARTICLE_UPPER_BOUND),
              random.nextInt(Constants.PARTICLE_UPPER_BOUND) 
            };
            particles[i] = new Particle(
              initialParticlePosition, initialParticleSpeed);
        }
    }

    // methods omitted
}
```

### 3.3。多温

最后，让我们通过创建一个 Multiswarm 类来结束我们的模型。

类似于蜂群，我们将跟踪蜂群的集合以及在所有蜂群中找到的最佳粒子位置和适应度。

我们还将存储一个对 fitness 函数的引用供以后使用:

```java
public class Multiswarm {
    private Swarm[] swarms;
    private long[] bestPosition;
    private double bestFitness = Double.NEGATIVE_INFINITY;
    private FitnessFunction fitnessFunction;

    public Multiswarm(
      int numSwarms, int particlesPerSwarm, FitnessFunction fitnessFunction) {
        this.fitnessFunction = fitnessFunction;
        this.swarms = new Swarm[numSwarms];
        for (int i = 0; i < numSwarms; i++) {
            swarms[i] = new Swarm(particlesPerSwarm);
        }
    }

    // methods omitted
}
```

### 3.4。健身功能

现在让我们实现健身功能。

为了将算法逻辑从这个特定的问题中分离出来，我们将引入一个具有单一方法的接口。

此方法将粒子位置作为参数，并返回一个值来指示它有多好:

```java
public interface FitnessFunction {
    public double getFitness(long[] particlePosition);
}
```

假设发现的结果根据问题约束是有效的，测量适合度只是返回我们想要最大化的计算的有效健康的问题。

对于我们的问题，我们有以下特定的验证约束:

*   解决方案只能是正整数
*   解决方案必须是可行的，并提供一定数量的黄金

当违反其中一个约束时，我们返回一个负数，告诉我们离有效性边界有多远。

这要么是在前一种情况下发现的数量，要么是在后一种情况下无法获得的黄金数量:

```java
public class LolFitnessFunction implements FitnessFunction {

    @Override
    public double getFitness(long[] particlePosition) {
        long health = particlePosition[0];
        long armor = particlePosition[1];

        if (health < 0 && armor < 0) {
            return -(health * armor);
        } else if (health < 0) {
            return health;
        } else if (armor < 0) {
            return armor;
        }

        double cost = (health * 2.5) + (armor * 18);
        if (cost > 3600) {
            return 3600 - cost;
        } else {
            long fitness = (health * (100 + armor)) / 100;
            return fitness;
        }
    }
}
```

### 3.5。主循环

主程序将在所有粒子群中的所有粒子之间迭代，并执行以下操作:

*   计算粒子适应度
*   如果找到了新的最佳位置，则更新粒子、群体和多重扫描历史
*   通过将当前速度添加到每个维度来计算新的粒子位置
*   计算新的粒子速度

目前，我们将速度更新留到下一节，通过创建一个专用的方法:

```java
public void mainLoop() {
    for (Swarm swarm : swarms) {
        for (Particle particle : swarm.getParticles()) {
            long[] particleOldPosition = particle.getPosition().clone();
            particle.setFitness(fitnessFunction.getFitness(particleOldPosition));

            if (particle.getFitness() > particle.getBestFitness()) {
                particle.setBestFitness(particle.getFitness());				
                particle.setBestPosition(particleOldPosition);
                if (particle.getFitness() > swarm.getBestFitness()) {						
                    swarm.setBestFitness(particle.getFitness());
                    swarm.setBestPosition(particleOldPosition);
                    if (swarm.getBestFitness() > bestFitness) {
                        bestFitness = swarm.getBestFitness();
                        bestPosition = swarm.getBestPosition().clone();
                    }
                }
            }

            long[] position = particle.getPosition();
            long[] speed = particle.getSpeed();
            position[0] += speed[0];
            position[1] += speed[1];
            speed[0] = getNewParticleSpeedForIndex(particle, swarm, 0);
            speed[1] = getNewParticleSpeedForIndex(particle, swarm, 1);
        }
    }
}
```

### 3.6。速度更新

对于粒子来说，改变速度是必要的，因为这是它探索不同可能解决方案的方式。

粒子的速度将需要使粒子朝着它自己、它的群和所有的群找到的最佳位置移动，给这些中的每一个分配一定的权重。我们将这些权重分别称为**、认知权重、社交权重和全局权重**。

为了增加一些变化，我们将这些权重乘以 0 到 1 之间的一个随机数。**我们还将在公式**中增加一个惯性因子，以激励粒子不要减速太多:

```java
private int getNewParticleSpeedForIndex(
  Particle particle, Swarm swarm, int index) {

    return (int) ((Constants.INERTIA_FACTOR * particle.getSpeed()[index])
      + (randomizePercentage(Constants.COGNITIVE_WEIGHT)
      * (particle.getBestPosition()[index] - particle.getPosition()[index]))
      + (randomizePercentage(Constants.SOCIAL_WEIGHT) 
      * (swarm.getBestPosition()[index] - particle.getPosition()[index]))
      + (randomizePercentage(Constants.GLOBAL_WEIGHT) 
      * (bestPosition[index] - particle.getPosition()[index])));
}
```

惯性、认知、社会和全局权重的可接受值分别为 0.729、1.49445、1.49445 和 0.3645。

## 4。结论

在本教程中，我们介绍了群算法的理论和实现。我们也看到了如何根据具体问题设计一个适应度函数。

如果你想了解更多关于这个话题的内容，看看这本书和[这篇文章](https://web.archive.org/web/20220625234719/https://msdn.microsoft.com/en-us/magazine/dn385711.aspx)，它们也是这篇文章的信息来源。

和往常一样，这个例子的所有代码都可以在 GitHub 项目中找到[。](https://web.archive.org/web/20220625234719/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-4)