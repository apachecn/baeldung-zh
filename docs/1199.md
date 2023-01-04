# 组合 JPA 和/或标准谓词

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-and-or-criteria-predicates>

## 1。概述

当查询数据库中的记录时，JPA Criteria API 可以很容易地用于添加多个 AND/OR 条件。在本教程中，我们将探索一个结合了多个 AND/OR 谓词的 JPA 标准查询的快速示例。

如果您还不熟悉谓词，我们建议您先阅读一下[基本 JPA 标准查询](/web/20221208143841/https://www.baeldung.com/spring-data-criteria-queries)。

## 2。示例应用程序

对于我们的例子，我们将考虑一个`Item`实体的清单，每个实体都有一个`id,` `name`、`color`和`grade`:

```
@Entity
public class Item {

    @Id
    private Long id;
    private String color;
    private String grade;
    private String name;

    // standard getters and setters
}
```

## 3。使用一个`AND`谓词组合两个`OR`谓词

让我们考虑一个场景，在这个场景中，我们需要找到同时具备以下两个条件的项目:

1.  红色或蓝色
    和
2.  a 级或 B 级

我们可以使用 JPA Criteria API 的`and()`和`or()`复合谓词轻松地完成这个**。**

首先，我们将设置我们的查询:

```
CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
CriteriaQuery<Item> criteriaQuery = criteriaBuilder.createQuery(Item.class);
Root<Item> itemRoot = criteriaQuery.from(Item.class);
```

现在我们需要构建一个`Predicate`来查找蓝色或红色的项目:

```
Predicate predicateForBlueColor
  = criteriaBuilder.equal(itemRoot.get("color"), "blue");
Predicate predicateForRedColor
  = criteriaBuilder.equal(itemRoot.get("color"), "red");
Predicate predicateForColor
  = criteriaBuilder.or(predicateForBlueColor, predicateForRedColor);
```

接下来，我们将构建一个`Predicate`来查找等级 A 或 B 的项目:

```
Predicate predicateForGradeA
  = criteriaBuilder.equal(itemRoot.get("grade"), "A");
Predicate predicateForGradeB
  = criteriaBuilder.equal(itemRoot.get("grade"), "B");
Predicate predicateForGrade
  = criteriaBuilder.or(predicateForGradeA, predicateForGradeB);
```

最后，我们将定义这两个变量的和`Predicate`，应用`where()`方法，并执行我们的查询:

```
Predicate finalPredicate
  = criteriaBuilder.and(predicateForColor, predicateForGrade);
criteriaQuery.where(finalPredicate);
List<Item> items = entityManager.createQuery(criteriaQuery).getResultList();
```

## 4。使用一个`OR`谓词组合两个`AND`谓词

相反，让我们考虑一个场景，我们需要查找具有以下任一特征的项目:

1.  红色和 D 级
    或
2.  蓝色和 B 级

逻辑非常相似，但是这里我们首先创建两个 AND `Predicate`，然后使用 OR `Predicate`将它们组合起来:

```
CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
CriteriaQuery<Item> criteriaQuery = criteriaBuilder.createQuery(Item.class);
Root<Item> itemRoot = criteriaQuery.from(Item.class);

Predicate predicateForBlueColor
  = criteriaBuilder.equal(itemRoot.get("color"), "red");
Predicate predicateForGradeA
  = criteriaBuilder.equal(itemRoot.get("grade"), "D");
Predicate predicateForBlueColorAndGradeA
  = criteriaBuilder.and(predicateForBlueColor, predicateForGradeA);

Predicate predicateForRedColor
  = criteriaBuilder.equal(itemRoot.get("color"), "blue");
Predicate predicateForGradeB
  = criteriaBuilder.equal(itemRoot.get("grade"), "B");
Predicate predicateForRedColorAndGradeB
  = criteriaBuilder.and(predicateForRedColor, predicateForGradeB);

Predicate finalPredicate
  = criteriaBuilder
  .or(predicateForBlueColorAndGradeA, predicateForRedColorAndGradeB);
criteriaQuery.where(finalPredicate);
List<Item> items = entityManager.createQuery(criteriaQuery).getResultList();
```

## 5。结论

在本文中，我们使用 JPA Criteria API 来实现需要组合 AND/OR 谓词的用例。

像往常一样，本文的完整源代码可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221208143841/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa-2)