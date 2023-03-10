# 在不相关的实体之间构造 JPA 查询

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-query-unrelated-entities>

## 1.概观

在本教程中，我们将看到如何在不相关的实体之间构造 JPA 查询。

## 2。Maven 依赖关系

让我们从向我们的`pom.xml`添加必要的依赖项开始。

首先，我们需要为 [Java 持久性 API](https://web.archive.org/web/20221128035216/https://search.maven.org/artifact/javax.persistence/javax.persistence-api) 添加一个依赖项:

```java
<dependency>
   <groupId>javax.persistence</groupId>
   <artifactId>javax.persistence-api</artifactId>
   <version>2.2</version>
</dependency> 
```

然后，我们为实现 Java 持久性 API 的 [Hibernate ORM](https://web.archive.org/web/20221128035216/https://search.maven.org/artifact/org.hibernate/hibernate-core) 添加一个依赖项:

```java
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.4.14.Final</version>
</dependency>
```

最后，我们添加一些 [QueryDSL](/web/20221128035216/https://www.baeldung.com/querydsl-with-jpa-tutorial) 依赖项；即，`[querydsl-apt](https://web.archive.org/web/20221128035216/https://search.maven.org/artifact/com.querydsl/querydsl-apt)`[`querydsl-jpa`](https://web.archive.org/web/20221128035216/https://search.maven.org/artifact/com.querydsl/querydsl-jpa):

```java
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-apt</artifactId>
    <version>4.3.1</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-jpa</artifactId>
    <version>4.3.1</version>
</dependency> 
```

## 3.领域模型

我们示例的领域是鸡尾酒吧。这里我们在数据库中有两个表:

*   存储我们酒吧出售的鸡尾酒及其价格的`menu`表，以及
*   `recipes`表存储制作鸡尾酒的说明

[![one to one](img/3835179cca4c95823a61b8dfa80aabc9.png)](/web/20221128035216/https://www.baeldung.com/wp-content/uploads/2020/04/one-to-one.png)

这两个表没有严格的联系。鸡尾酒可以出现在我们的菜单上，而不需要保留配方说明。此外，我们可能有尚未出售的鸡尾酒配方。

在我们的示例中，我们将找到菜单上所有我们有可用配方的鸡尾酒。

## 4.JPA 实体

我们可以很容易地创建两个 JPA 实体来表示我们的表:

```java
@Entity
@Table(name = "menu")
public class Cocktail {
    @Id
    @Column(name = "cocktail_name")
    private String name;

    @Column
    private double price;

    // getters & setters
} 
```

```java
@Entity
@Table(name="recipes")
public class Recipe {
    @Id
    @Column(name = "cocktail")
    private String cocktail;

    @Column
    private String instructions;

    // getters & setters
}
```

在*菜单*和`recipes`表**之间，存在一种潜在的一对一关系，而没有显式的外键约束**。例如，如果我们有一个*菜单*记录，其`cocktail_name`列的值为“Mojito ”,还有一个`recipes`记录，其`cocktail` 列的值为“Mojito ”,那么`menu`记录与这个`recipes`记录相关联。

为了在我们的`Cocktail`实体中表示这种关系，我们添加了用各种注释标注的`recipe`字段:

```java
@Entity
@Table(name = "menu")
public class Cocktail {
    // ...

    @OneToOne
    @NotFound(action = NotFoundAction.IGNORE)
    @JoinColumn(name = "cocktail_name", 
       referencedColumnName = "cocktail", 
       insertable = false, updatable = false, 
       foreignKey = @javax.persistence
         .ForeignKey(value = ConstraintMode.NO_CONSTRAINT))
    private Recipe recipe;

    // ...
}
```

第一个注释是`[@OneToOne](/web/20221128035216/https://www.baeldung.com/jpa-one-to-one),`，它声明了与`Recipe`实体的基本一对一关系。

接下来，我们用`@NotFound(action = NotFoundAction.IGNORE)` Hibernate 注释来注释`recipe`字段。这告诉我们的 ORM 不要在`menu`表中不存在`cocktail`的`recipe`时抛出异常。

**将`Cocktail`与其关联的`Recipe` 关联起来的标注是 [`@JoinColumn`](/web/20221128035216/https://www.baeldung.com/jpa-join-column) 。通过使用这个注释，我们定义了两个实体之间的伪外键关系。**

最后，通过将`foreignKey`属性设置为`@javax.persistence.ForeignKey(value = ConstraintMode.NO_CONSTRAINT),` ，我们指示 JPA 提供者不要生成外键约束。

## 5.JPA 和 QueryDSL 查询

因为我们对检索与`Recipe,` 相关联的`Cocktail`实体感兴趣，所以我们可以通过将`Cocktail`实体与其相关联的`Recipe`实体相结合来查询该实体。

我们可以构造查询的一种方法是使用 [JPQL](https://web.archive.org/web/20221128035216/https://docs.oracle.com/html/E13946_04/ejb3_langref.html) :

```java
entityManager.createQuery("select c from Cocktail c join c.recipe")
```

或者使用 QueryDSL 框架:

```java
new JPAQuery<Cocktail>(entityManager)
  .from(QCocktail.cocktail)
  .join(QCocktail.cocktail.recipe)
```

获得期望结果的另一种方法是将`Cocktail`与`Recipe` 实体连接起来，并使用`on`子句直接定义查询中的底层关系。

我们可以使用 JPQL 做到这一点:

```java
entityManager.createQuery("select c from Cocktail c join Recipe r on c.name = r.cocktail")
```

或者使用 QueryDSL 框架:

```java
new JPAQuery(entityManager)
  .from(QCocktail.cocktail)
  .join(QRecipe.recipe)
  .on(QCocktail.cocktail.name.eq(QRecipe.recipe.cocktail))
```

## 6.一对一连接单元测试

让我们开始创建一个单元测试来测试上面的查询。在我们的测试用例运行之前，我们必须插入一些数据到我们的数据库表中。

```java
public class UnrelatedEntitiesUnitTest {
    // ...

    @BeforeAll
    public static void setup() {
        // ...

        mojito = new Cocktail();
        mojito.setName("Mojito");
        mojito.setPrice(12.12);
        ginTonic = new Cocktail();
        ginTonic.setName("Gin tonic");
        ginTonic.setPrice(10.50);
        Recipe mojitoRecipe = new Recipe(); 
        mojitoRecipe.setCocktail(mojito.getName()); 
        mojitoRecipe.setInstructions("Some instructions for making a mojito cocktail!");
        entityManager.persist(mojito);
        entityManager.persist(ginTonic);
        entityManager.persist(mojitoRecipe);

        // ...
    }

    // ... 
}
```

在`setup` 方法中，我们保存了两个`Cocktail`实体,`mojito` 和`ginTonic.` ,然后，我们添加了一个`recipe`,用于演示如何制作“莫吉托”`cocktail`。

现在，我们可以测试前一部分的查询结果。我们知道只有`mojito` 鸡尾酒有关联的`Recipe` 实体，所以我们希望各种查询只返回`mojito` 鸡尾酒:

```java
public class UnrelatedEntitiesUnitTest {
    // ...

    @Test
    public void givenCocktailsWithRecipe_whenQuerying_thenTheExpectedCocktailsReturned() {
        // JPA
        Cocktail cocktail = entityManager.createQuery("select c " +
          "from Cocktail c join c.recipe", Cocktail.class)
          .getSingleResult();
        verifyResult(mojito, cocktail);

        cocktail = entityManager.createQuery("select c " +
          "from Cocktail c join Recipe r " +
          "on c.name = r.cocktail", Cocktail.class).getSingleResult();
        verifyResult(mojito, cocktail);

        // QueryDSL
        cocktail = new JPAQuery<Cocktail>(entityManager).from(QCocktail.cocktail)
          .join(QCocktail.cocktail.recipe)
          .fetchOne();
        verifyResult(mojito, cocktail);

        cocktail = new JPAQuery<Cocktail>(entityManager).from(QCocktail.cocktail)
          .join(QRecipe.recipe)
          .on(QCocktail.cocktail.name.eq(QRecipe.recipe.cocktail))
          .fetchOne();
        verifyResult(mojito, cocktail);
    }

    private void verifyResult(Cocktail expectedCocktail, Cocktail queryResult) {
        assertNotNull(queryResult);
        assertEquals(expectedCocktail, queryResult);
    }

    // ...
} 
```

`verifyResult`方法帮助我们验证查询返回的结果是否等于预期的结果。

## 7.一对多基础关系

让我们改变我们的例子的领域，以显示我们如何能够用一对多的底层关系来**连接两个实体。**

[![one to many](img/09c7f89895901f2481c61e7d90913365.png)](/web/20221128035216/https://www.baeldung.com/wp-content/uploads/2020/04/one-to-many.png) 
我们用`multiple_recipes`表代替了`recipes`表，在这里我们可以为同一个`cocktail`存储任意多的`recipes`。

```java
@Entity
@Table(name = "multiple_recipes")
public class MultipleRecipe {
    @Id
    @Column(name = "id")
    private Long id;

    @Column(name = "cocktail")
    private String cocktail;

    @Column(name = "instructions")
    private String instructions;

    // getters & setters
}
```

现在，**`Cocktail`实体通过一对多的底层关系**与`MultipleRecipe`实体相关联:

```java
@Entity
@Table(name = "cocktails")
public class Cocktail {    
    // ...

    @OneToMany
    @NotFound(action = NotFoundAction.IGNORE)
    @JoinColumn(
       name = "cocktail", 
       referencedColumnName = "cocktail_name", 
       insertable = false, 
       updatable = false, 
       foreignKey = @javax.persistence
         .ForeignKey(value = ConstraintMode.NO_CONSTRAINT))
    private List<MultipleRecipe> recipeList;

    // getters & setters
} 
```

为了找到并获得至少有一个可用的`MultipleRecipe,` 的`Cocktail`实体，我们可以通过将`Cocktail`实体与其关联的`MultipleRecipe`实体连接起来来查询它。

我们可以使用 JPQL 做到这一点:

```java
entityManager.createQuery("select c from Cocktail c join c.recipeList");
```

或者使用 QueryDSL 框架:

```java
new JPAQuery(entityManager).from(QCocktail.cocktail)
  .join(QCocktail.cocktail.recipeList);
```

还有一个选项是不使用定义了`Cocktail`和`MultipleRecipe` 实体`.` 之间一对多关系的`recipeList` 字段，相反，我们可以为这两个实体编写一个连接查询，并通过使用 JPQL“on”子句来确定它们的底层关系:

```java
entityManager.createQuery("select c "
  + "from Cocktail c join MultipleRecipe mr "
  + "on mr.cocktail = c.name");
```

最后，我们可以通过使用 QueryDSL 框架来构建相同的查询:

```java
new JPAQuery(entityManager).from(QCocktail.cocktail)
  .join(QMultipleRecipe.multipleRecipe)
  .on(QCocktail.cocktail.name.eq(QMultipleRecipe.multipleRecipe.cocktail)); 
```

## 8.一对多连接单元测试

这里，我们将添加一个新的测试用例来测试前面的查询。在这样做之前，我们必须在我们的`setup` 方法中持久化一些`MultipleRecipe` 实例:

```java
public class UnrelatedEntitiesUnitTest {    
    // ...

    @BeforeAll
    public static void setup() {
        // ...

        MultipleRecipe firstMojitoRecipe = new MultipleRecipe();
        firstMojitoRecipe.setId(1L);
        firstMojitoRecipe.setCocktail(mojito.getName());
        firstMojitoRecipe.setInstructions("The first recipe of making a mojito!");
        entityManager.persist(firstMojitoRecipe);
        MultipleRecipe secondMojitoRecipe = new MultipleRecipe();
        secondMojitoRecipe.setId(2L);
        secondMojitoRecipe.setCocktail(mojito.getName());
        secondMojitoRecipe.setInstructions("The second recipe of making a mojito!"); 
        entityManager.persist(secondMojitoRecipe);

        // ...
    }

    // ... 
} 
```

然后我们可以开发一个测试用例，在这里我们验证当我们在上一节中展示的查询被执行时，它们返回与至少一个`MultipleRecipe`实例相关联的`Cocktail `实体:

```java
public class UnrelatedEntitiesUnitTest {
    // ...

    @Test
    public void givenCocktailsWithMultipleRecipes_whenQuerying_thenTheExpectedCocktailsReturned() {
        // JPQL
        Cocktail cocktail = entityManager.createQuery("select c "
          + "from Cocktail c join c.recipeList", Cocktail.class)
          .getSingleResult();
        verifyResult(mojito, cocktail);

        cocktail = entityManager.createQuery("select c "
          + "from Cocktail c join MultipleRecipe mr "
          + "on mr.cocktail = c.name", Cocktail.class)
          .getSingleResult();
        verifyResult(mojito, cocktail);

        // QueryDSL
        cocktail = new JPAQuery<Cocktail>(entityManager).from(QCocktail.cocktail)
          .join(QCocktail.cocktail.recipeList)
          .fetchOne();
        verifyResult(mojito, cocktail);

        cocktail = new JPAQuery<Cocktail>(entityManager).from(QCocktail.cocktail)
          .join(QMultipleRecipe.multipleRecipe)
          .on(QCocktail.cocktail.name.eq(QMultipleRecipe.multipleRecipe.cocktail))
          .fetchOne();
        verifyResult(mojito, cocktail);
    }

    // ...

}
```

## 9.多对多基础关系

在这一部分，我们选择根据基本成分对菜单中的鸡尾酒进行分类。例如，莫吉托鸡尾酒的基本成分是朗姆酒，所以朗姆酒是我们菜单上的一个鸡尾酒类别。

为了在我们的领域中描述上述内容，我们将`category`字段添加到`Cocktail` 实体中:

```java
@Entity
@Table(name = "menu")
public class Cocktail {
    // ...

    @Column(name = "category")
    private String category;

     // ...
} 
```

此外，我们可以将`base_ingredient`列添加到`multiple_recipes`表中，以便能够搜索基于特定饮料的食谱。

```java
@Entity
@Table(name = "multiple_recipes")
public class MultipleRecipe {
    // ...

    @Column(name = "base_ingredient")
    private String baseIngredient;

    // ...
}
```

完成上述操作后，下面是我们的数据库模式:

[![many to many](img/6f6eb8d33cc91e97f102b84d7e95f591.png)](/web/20221128035216/https://www.baeldung.com/wp-content/uploads/2020/04/many_to_many.png)

现在，**在`Cocktail`和`MultipleRecipe` 实体**之间有一个多对多的底层关系。许多`MultipleRecipe `实体可以与许多`Cocktail `实体相关联，它们的`category`值等于`MultipleRecipe`实体的`baseIngredient` 值。

为了找到并获得`MultipleRecipe`实体，它们的`baseIngredient `作为一个类别存在于`Cocktail `实体中，我们可以使用 JPQL 连接这两个实体:

```java
entityManager.createQuery("select distinct r " 
  + "from MultipleRecipe r " 
  + "join Cocktail c " 
  + "on r.baseIngredient = c.category", MultipleRecipe.class)
```

或者使用 QueryDSL:

```java
QCocktail cocktail = QCocktail.cocktail; 
QMultipleRecipe multipleRecipe = QMultipleRecipe.multipleRecipe; 
new JPAQuery(entityManager).from(multipleRecipe)
  .join(cocktail)
  .on(multipleRecipe.baseIngredient.eq(cocktail.category))
  .fetch(); 
```

## 10.多对多连接单元测试

在继续我们的测试用例之前，我们必须设置我们的`Cocktail`实体的`category`和我们的`MultipleRecipe`实体的`baseIngredient`:

```java
public class UnrelatedEntitiesUnitTest {
    // ...

    @BeforeAll
    public static void setup() {
        // ...

        mojito.setCategory("Rum");
        ginTonic.setCategory("Gin");
        firstMojitoRecipe.setBaseIngredient(mojito.getCategory());
        secondMojitoRecipe.setBaseIngredient(mojito.getCategory());

        // ...
    }

    // ... 
}
```

然后，我们可以验证当我们之前显示的查询被执行时，它们返回预期的结果:

```java
public class UnrelatedEntitiesUnitTest {
    // ...

    @Test
    public void givenMultipleRecipesWithCocktails_whenQuerying_thenTheExpectedMultipleRecipesReturned() {
        Consumer<List<MultipleRecipe>> verifyResult = recipes -> {
            assertEquals(2, recipes.size());
            recipes.forEach(r -> assertEquals(mojito.getName(), r.getCocktail()));
        };

        // JPQL
        List<MultipleRecipe> recipes = entityManager.createQuery("select distinct r "
          + "from MultipleRecipe r "
          + "join Cocktail c " 
          + "on r.baseIngredient = c.category",
          MultipleRecipe.class).getResultList();
        verifyResult.accept(recipes);

        // QueryDSL
        QCocktail cocktail = QCocktail.cocktail;
        QMultipleRecipe multipleRecipe = QMultipleRecipe.multipleRecipe;
        recipes = new JPAQuery<MultipleRecipe>(entityManager).from(multipleRecipe)
          .join(cocktail)
          .on(multipleRecipe.baseIngredient.eq(cocktail.category))
          .fetch();
        verifyResult.accept(recipes);
    }

    // ...
} 
```

## 11.结论

在本教程中，我们介绍了通过使用 JPQL 或 QueryDSL 框架在不相关的实体之间构建 JPA 查询的各种方法。

和往常一样，代码可以在 GitHub 的[上获得。](https://web.archive.org/web/20221128035216/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa-2)