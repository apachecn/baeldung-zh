# 基于 Java 的协同过滤推荐系统

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-collaborative-filtering-recommendations>

## 1。简介

在本教程中，我们将学习 Java 中的 [Slope One](https://web.archive.org/web/20220815033003/https://en.wikipedia.org/wiki/Slope_One) 算法。

我们还将展示协同过滤(CF)问题的示例实现——推荐系统使用的一种机器学习技术**。**

例如，这可以用于预测用户对特定项目的兴趣。

## 2。协同过滤

Slope One 算法是一个基于项目的协同过滤系统。这意味着它完全基于用户项目排名。当我们计算对象之间的相似性时，我们只知道排名的历史，而不知道内容本身。然后，这种相似性被用于预测数据集中不存在的用户-项目对的潜在用户排名。

下面的[图](https://web.archive.org/web/20220815033003/https://commons.wikimedia.org/wiki/File:Collaborative_filtering.gif)显示了获取和计算特定用户评级的完整过程:

[![Collaborative filtering new](img/caf28fddcef363deb39e5e6c5723c0cd.png)](/web/20220815033003/https://www.baeldung.com/wp-content/uploads/2016/12/Collaborative_filtering_new.gif)

首先，用户在系统中对不同的项目进行评分。接下来，算法计算相似度。之后，系统会对用户尚未评分的用户项目评分进行预测。

关于协同过滤主题的更多细节，我们可以参考维基百科的文章。

## 3。斜率一算法

Slope One 被命名为基于评级的最简单形式的非平凡的基于项目的协作过滤。它考虑了来自对同一项目进行评级的所有用户的信息以及来自同一用户对其他项目进行评级的信息来计算相似性矩阵。

在我们的简单示例中，我们将预测商店中商品的用户排名。

让我们从我们的问题和领域的简单 Java 模型开始。

### 3.1。`Java Model`

在我们的模型中，我们有两个主要对象——条目和用户。`Item`类包含项目的名称:

```java
private String itemName;
```

另一方面，`User`类包含用户名:

```java
private String username;
```

最后，我们有一个用于初始化数据的`InputData`类。让我们假设我们将在商店中创建五种不同的产品:

```java
List<Item> items = Arrays.asList(
  new Item("Candy"), 
  new Item("Drink"), 
  new Item("Soda"), 
  new Item("Popcorn"), 
  new Item("Snacks")
);
```

此外，我们将创建三个用户，使用 0.0-1.0 的评分标准对上述内容进行随机评分，其中 0 表示不感兴趣，0.5 表示有点兴趣，1.0 表示完全感兴趣。作为数据初始化的结果，我们将获得一个包含用户项目排名数据的`Map`:

```java
Map<User, HashMap<Item, Double>> data;
```

### 3.2。`Differences and Frequencies Matrices`

根据可用的数据，我们将计算项目之间的关系，以及项目出现的次数。对于每个用户，我们检查他/她对项目的评分:

```java
for (HashMap<Item, Double> user : data.values()) {
    for (Entry<Item, Double> e : user.entrySet()) {
        // ...
    }
}
```

在下一步中，我们检查该项目是否存在于我们的矩阵中。如果这是第一次出现，我们在映射中创建新条目:

```java
if (!diff.containsKey(e.getKey())) {
    diff.put(e.getKey(), new HashMap<Item, Double>());
    freq.put(e.getKey(), new HashMap<Item, Integer>());
} 
```

第一个矩阵用于计算用户评级之间的差异。它的值可能是正的，也可能是负的(因为评级之间的差异可能是负的)，并存储为`Double`。另一方面，频率存储为`Integer`值。

下一步，我们将比较所有项目的评分:

```java
for (Entry<Item, Double> e2 : user.entrySet()) {
    int oldCount = 0;
    if (freq.get(e.getKey()).containsKey(e2.getKey())){
        oldCount = freq.get(e.getKey()).get(e2.getKey()).intValue();
    }

    double oldDiff = 0.0;
    if (diff.get(e.getKey()).containsKey(e2.getKey())){
        oldDiff = diff.get(e.getKey()).get(e2.getKey()).doubleValue();
    }

    double observedDiff = e.getValue() - e2.getValue();
    freq.get(e.getKey()).put(e2.getKey(), oldCount + 1);
    diff.get(e.getKey()).put(e2.getKey(), oldDiff + observedDiff);
}
```

如果之前有人对该项目进行了评级，我们会将频率计数增加 1。此外，我们检查项目评分之间的平均差异，并计算新的`observedDiff`。

**请注意，我们把`oldDiff`和`observedDiff`之和作为一个项目的新值。**

最后，我们计算矩阵内部的相似性得分:

```java
for (Item j : diff.keySet()) {
    for (Item i : diff.get(j).keySet()) {
        double oldValue = diff.get(j).get(i).doubleValue();
        int count = freq.get(j).get(i).intValue();
        diff.get(j).put(i, oldValue / count);
    }
}
```

主要逻辑是将计算出的项目评级的差异除以其出现的次数。在这一步之后，我们可以打印出最终的差异矩阵。

### 3.3。`Predictions`

作为 Slope One 的主要部分，我们将根据现有数据预测所有缺失的评分。为此，我们需要将用户项目评分与上一步中计算的差异矩阵进行比较:

```java
for (Entry<User, HashMap<Item, Double>> e : data.entrySet()) {
    for (Item j : e.getValue().keySet()) {
        for (Item k : diff.keySet()) {
            double predictedValue =
              diff.get(k).get(j).doubleValue() + e.getValue().get(j).doubleValue();
            double finalValue = predictedValue * freq.get(k).get(j).intValue();
            uPred.put(k, uPred.get(k) + finalValue);
            uFreq.put(k, uFreq.get(k) + freq.get(k).get(j).intValue());
        }
    }
    // ...
}
```

之后，我们需要使用下面的代码准备“干净”的预测:

```java
HashMap<Item, Double> clean = new HashMap<Item, Double>();
for (Item j : uPred.keySet()) {
    if (uFreq.get(j) > 0) {
        clean.put(j, uPred.get(j).doubleValue() / uFreq.get(j).intValue());
    }
}
for (Item j : InputData.items) {
    if (e.getValue().containsKey(j)) {
        clean.put(j, e.getValue().get(j));
    } else if (!clean.containsKey(j)) {
        clean.put(j, -1.0);
    }
}
```

对于较大的数据集，要考虑的技巧是只使用具有较大频率值(例如> 1)的条目。请注意，如果预测是不可能的，它的值将等于-1。

**最后，非常重要的一点。**如果我们的算法工作正常，**我们应该会收到用户没有评级的项目的预测，但也会收到他评级的项目的重复评级**。那些重复的评级不应该改变，否则这意味着在你的算法实现中有一个 bug。

### 3.4。`Tips`

影响斜率一算法的主要因素很少。以下是一些如何提高精确度和处理时间的技巧:

*   对于大型数据集，考虑在数据库端获得用户项目评级
*   设定获取评分的时间框架，因为人们的兴趣可能会随着时间的推移而改变，这也将减少处理输入数据所需的时间
*   将大型数据集拆分成较小的数据集——你不需要每天为所有用户计算预测；您可以检查用户是否与预测的项目进行了交互，然后在第二天的处理队列中添加/删除他/她

## 4。结论

在本教程中，我们能够了解斜率一算法。此外，我们介绍了项目推荐系统的协同过滤问题。

本教程的**完整实现**可以在[的 GitHub 项目](https://web.archive.org/web/20220815033003/https://github.com/eugenp/tutorials/tree/master/algorithms-modules/algorithms-miscellaneous-2 "The Full Registration/Authentication Example Project on Github ")中找到。