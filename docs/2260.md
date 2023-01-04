# 用 Java 设计一个遗传算法

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-genetic-algorithm>

## 1。简介

这个系列的目的是**解释遗传算法的概念**。

遗传算法被设计为通过使用与自然界相同的过程来解决问题——它们使用选择、重组和变异的组合来进化问题的解决方案。

让我们首先使用最简单的二进制遗传算法例子来解释那些算法的概念。

## 2。遗传算法如何工作

遗传算法是进化计算的一部分，进化计算是人工智能的一个快速发展的领域。

一个算法从一组被称为**群体**的**解**(由**个体**代表)开始。来自一个群体的解决方案被用来形成一个新群体，因为新群体有可能比旧群体更好。

被选择来形成新解决方案的个体(**后代**)是根据他们的**适应度**来选择的——他们越适合，他们繁衍的机会就越多。

## 3。二进制遗传算法

让我们来看看简单遗传算法的基本过程。

### 3.1。初始化

在初始化步骤中，我们**生成一个随机数`Population`，作为第一个解**。首先，我们需要决定`Population`有多大，以及我们期望的最终解决方案是什么:

```
SimpleGeneticAlgorithm.runAlgorithm(50,
  "1011000100000100010000100000100111001000000100000100000000001111");
```

在上面的例子中，`Population`的大小是 50，正确的解用我们可能随时改变的二进制位串来表示。

下一步，我们将保存我们想要的解决方案，并创建一个随机的`Population`:

```
setSolution(solution);
Population myPop = new Population(populationSize, true);
```

现在我们准备运行程序的主循环。

### 3.2。 **体能检查**

在程序的主循环中，我们将由**通过适应度函数**对每个`Individual`进行评估(简单来说，`Individual`越好，它得到的适应度函数值越高):

```
while (myPop.getFittest().getFitness() < getMaxFitness()) {
    System.out.println(
      "Generation: " + generationCount
      + " Correct genes found: " + myPop.getFittest().getFitness());

    myPop = evolvePopulation(myPop);
    generationCount++;
}
```

让我们先来解释一下**我们是如何得到最适合的`Individual`** :

```
public int getFitness(Individual individual) {
    int fitness = 0;
    for (int i = 0; i < individual.getDefaultGeneLength()
      && i < solution.length; i++) {
        if (individual.getSingleGene(i) == solution[i]) {
            fitness++;
        }
    }
    return fitness;
}
```

正如我们可以观察到的，我们一点一点地比较两个`Individual`对象。如果我们找不到完美的解决方案，我们需要进行下一步，这是对`Population`的改进。

### 3.3。后代

在这一步，我们需要创建一个新的`Population`。首先，我们需要**根据适合度从一个`Population,`中选择**两个父`Individual` 对象。请注意，允许最好的`Individual`从这一代延续到下一代是有益的。这种策略被称为**精英主义:**

```
if (elitism) {
    newPopulation.getIndividuals().add(0, pop.getFittest());
    elitismOffset = 1;
} else {
    elitismOffset = 0;
}
```

为了选出两个最好的`Individual`对象，我们将应用 **[比武选择策略](https://web.archive.org/web/20220122050559/https://en.wikipedia.org/wiki/Tournament_selection)** :

```
private Individual tournamentSelection(Population pop) {
    Population tournament = new Population(tournamentSize, false);
    for (int i = 0; i < tournamentSize; i++) {
        int randomId = (int) (Math.random() * pop.getIndividuals().size());
        tournament.getIndividuals().add(i, pop.getIndividual(randomId));
    }
    Individual fittest = tournament.getFittest();
    return fittest;
}
```

每场锦标赛的获胜者(体能最佳者)将进入下一阶段，即**交叉赛**:

```
private Individual crossover(Individual indiv1, Individual indiv2) {
    Individual newSol = new Individual();
    for (int i = 0; i < newSol.getDefaultGeneLength(); i++) {
        if (Math.random() <= uniformRate) {
            newSol.setSingleGene(i, indiv1.getSingleGene(i));
        } else {
            newSol.setSingleGene(i, indiv2.getSingleGene(i));
        }
    }
    return newSol;
}
```

在交叉中，我们在随机选择的点交换来自每个选择的`Individual`的位。整个过程在以下循环中运行:

```
for (int i = elitismOffset; i < pop.getIndividuals().size(); i++) {
    Individual indiv1 = tournamentSelection(pop);
    Individual indiv2 = tournamentSelection(pop);
    Individual newIndiv = crossover(indiv1, indiv2);
    newPopulation.getIndividuals().add(i, newIndiv);
}
```

如我们所见，在交叉后，我们将新的后代放在新的`Population`中。这一步被称为**验收。**

最后，我们可以执行一个**突变**。突变被用来从一代人到下一代人保持遗传多样性。我们使用了**位反转**类型的变异，其中随机位被简单地反转:

```
private void mutate(Individual indiv) {
    for (int i = 0; i < indiv.getDefaultGeneLength(); i++) {
        if (Math.random() <= mutationRate) {
            byte gene = (byte) Math.round(Math.random());
            indiv.setSingleGene(i, gene);
        }
    }
}
```

在本教程中，所有类型的变异和交叉都得到了很好的描述[。](https://web.archive.org/web/20220122050559/http://www.obitko.com/tutorials/genetic-algorithms/crossover-mutation.php)

然后，我们重复 3.2 和 3.3 小节中的步骤，直到我们达到终止条件，例如最佳解决方案。

## 4。提示和技巧

为了**实现高效的遗传算法**，我们需要调整一组参数。本节将为您提供一些如何从最重要的参数开始的基本建议:

*   **交叉率**–应该很高，大概在 **80%-95%**
*   **突变率**——应该很低，在 **0.5%-1%** 左右。
*   **人口数量**–好的人口数量大约是 **20-30** ，然而，对于某些问题来说，50-100 的人口数量更好
*   **选择**–基础 [**轮盘赌轮盘选择**](/web/20220122050559/https://www.baeldung.com/cs/genetic-algorithms-roulette-selection) 可以配合**精英主义**的概念使用
*   **交叉和变异类型**–这取决于编码和问题

请注意，调整建议通常是对遗传算法的经验研究的结果，它们可能会根据提出的问题而有所不同。

## 5。结论

本教程**介绍了遗传算法的基础**。你可以学习遗传算法**而不需要任何这方面的知识**，只需要基本的计算机编程技能。

本教程中代码片段的完整源代码可以在 GitHub 项目的[中找到。](https://web.archive.org/web/20220122050559/https://github.com/eugenp/tutorials/tree/master/algorithms-genetic)

还请注意，我们使用 [Lombok](https://web.archive.org/web/20220122050559/https://projectlombok.org/) 来生成 getters 和 setters。你可以在你的 IDE [中查看如何正确配置它，本文](/web/20220122050559/https://www.baeldung.com/intro-to-project-lombok)。

有关遗传算法的更多示例，请查看我们系列的所有文章:

*   如何设计一个遗传算法？(这个)
*   [Java 中的旅行推销员问题](/web/20220122050559/https://www.baeldung.com/java-simulated-annealing-for-traveling-salesman)