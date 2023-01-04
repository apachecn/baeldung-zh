# EntityManager#getReference()快速指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-entity-manager-get-reference>

## 1.介绍

从第一个版本开始，`EntityManager`类的`getReference()`方法就是 JPA 规范的一部分。但是，这种方法让一些开发人员感到困惑，因为它的行为会因底层的持久性提供者而异。

在本教程中，我们将解释如何在 [`Hibernate EntityManager`](/web/20220628082429/https://www.baeldung.com/hibernate-entitymanager) 中使用`getReference()`方法。

## 2.`EntityManager`获取操作

首先，我们将看看如何通过实体的主键来获取实体。无需编写任何查询，`EntityManager`为我们提供了两个基本方法来实现这一点。

### 2.1.`find()`

`find()`是获取实体的最常见方法:

```java
Game game = entityManager.find(Game.class, 1L); 
```

当我们请求时，这个方法初始化实体。

### 2.2.`getReference()`

类似于`find()`方法，`getReference()`也是另一种检索实体的方法:

```java
Game game = entityManager.getReference(Game.class, 1L); 
```

然而，返回的对象是一个实体代理**，它只初始化了主键字段**。其他字段保持未设置，除非我们主动请求它们。

接下来，我们来看看这两种方法在各种场景下的表现。

## 3.一个示例用例

为了演示`EntityManager`获取操作，我们将创建两个模型，`Game`和`Player,`作为我们的领域，许多玩家可以参与同一个游戏。

### 3.1.领域模型

首先，让我们定义一个名为`Game:`的实体

```java
@Entity
public class Game {

    @Id
    private Long id;

    private String name;

    // standard constructors, getters, setters

} 
```

接下来，我们定义我们的`Player`实体:

```java
@Entity
public class Player {

    @Id
    private Long id;

    private String name;

    // standard constructors, getters, setters

} 
```

### 3.2.配置关系

我们需要**配置一个从`Player`到`Game`到**的`@ManyToOne`关系。因此，让我们为我们的`Player`实体添加一个`game`属性:

```java
@ManyToOne
private Game game; 
```

## 4.测试案例

在我们开始编写我们的测试方法之前，单独定义我们的测试数据是一个很好的实践:

```java
entityManager.getTransaction().begin();

entityManager.persist(new Game(1L, "Game 1"));
entityManager.persist(new Game(2L, "Game 2"));
entityManager.persist(new Player(1L,"Player 1"));
entityManager.persist(new Player(2L, "Player 2"));
entityManager.persist(new Player(3L, "Player 3"));

entityManager.getTransaction().commit(); 
```

此外，为了检查底层 SQL 查询，我们应该**在我们的`persistence.xml`中配置 Hibernate 的`hibernate.show_sql`属性**:

```java
<property name="hibernate.show_sql" value="true"/> 
```

### 4.1.更新实体字段

首先，我们将检查使用`find()`方法更新实体的最常见方式。

因此，让我们先编写一个测试方法来获取`Game`实体，然后简单地更新它的`name`字段:

```java
Game game1 = entityManager.find(Game.class, 1L);
game1.setName("Game Updated 1");

entityManager.persist(game1); 
```

运行测试方法向我们展示了执行的 SQL 查询:

```java
Hibernate: select game0_.id as id1_0_0_, game0_.name as name2_0_0_ from Game game0_ where game0_.id=?
Hibernate: update Game set name=? where id=? 
```

正如我们注意到的，**`SELECT`查询在这种情况下**看起来是不必要的。由于在更新操作之前我们不需要读取`Game`实体的任何字段，我们想知道是否有某种方法可以只执行`UPDATE`查询。

因此，让我们看看`getReference()`方法在相同场景中的表现:

```java
Game game1 = entityManager.getReference(Game.class, 1L);
game1.setName("Game Updated 2");

entityManager.persist(game1); 
```

令人惊讶的是，**运行测试方法的结果仍然相同，我们看到`SELECT`查询仍然是**。

正如我们所见，当我们使用`getReference()`更新实体字段时，Hibernate 确实执行了一个`SELECT`查询。

因此，如果我们执行实体代理字段的任何 setter，使用`getReference()`方法的**不会避免额外的`SELECT`查询。**

### 4.2.删除实体

当我们执行删除操作时，也会发生类似的情况。

让我们定义另外两个测试方法来删除一个`Player`实体:

```java
Player player2 = entityManager.find(Player.class, 2L);
entityManager.remove(player2); 
```

```java
Player player3 = entityManager.getReference(Player.class, 3L);
entityManager.remove(player3); 
```

运行这些测试方法向我们展示了相同的查询:

```java
Hibernate: select
    player0_.id as id1_1_0_,
    player0_.game_id as game_id3_1_0_,
    player0_.name as name2_1_0_,
    game1_.id as id1_0_1_,
    game1_.name as name2_0_1_ from Player player0_
    left outer join Game game1_ on player0_.game_id=game1_.id
    where player0_.id=?
Hibernate: delete from Player where id=? 
```

同样，对于删除操作，结果是相似的。**即使我们不读取`Player`实体的任何字段，Hibernate 也会执行一个额外的`SELECT`查询。**

因此，**当我们删除一个现有实体时，选择`getReference()`还是`find()`方法没有区别。**

在这一点上，我们想知道,`getReference()`会有什么不同吗？让我们转到实体关系，找出答案。

### 4.3.更新实体关系

当我们需要保存实体之间的关系时，另一个常见的用例出现了。

让我们添加另一个方法，通过简单地更新`Player`的`game`属性来演示`Player`参与到`Game`中:

```java
Game game1 = entityManager.find(Game.class, 1L);

Player player1 = entityManager.find(Player.class, 1L);
player1.setGame(game1);

entityManager.persist(player1); 
```

再次运行测试给我们一个类似的结果，当使用`find()`方法时**我们仍然可以看到`SELECT`查询:**

```java
Hibernate: select game0_.id as id1_0_0_, game0_.name as name2_0_0_ from Game game0_ where game0_.id=?
Hibernate: select
    player0_.id as id1_1_0_,
    player0_.game_id as game_id3_1_0_,
    player0_.name as name2_1_0_,
    game1_.id as id1_0_1_,
    game1_.name as name2_0_1_ from Player player0_
    left outer join Game game1_ on player0_.game_id=game1_.id
    where player0_.id=?
Hibernate: update Player set game_id=?, name=? where id=? 
```

现在，让我们为**定义另一个测试，看看`getReference()`方法在这种情况下如何工作**:

```java
Game game2 = entityManager.getReference(Game.class, 2L);

Player player1 = entityManager.find(Player.class, 1L);
player1.setGame(game2);

entityManager.persist(player1); 
```

希望运行测试能给我们预期的行为:

```java
Hibernate: select
    player0_.id as id1_1_0_,
    player0_.game_id as game_id3_1_0_,
    player0_.name as name2_1_0_,
    game1_.id as id1_0_1_,
    game1_.name as name2_0_1_ from Player player0_
    left outer join Game game1_ on player0_.game_id=game1_.id
    where player0_.id=?
Hibernate: update Player set game_id=?, name=? where id=? 
```

我们看到，当我们这次使用`getReference()`时， **Hibernate 并没有对`Game`实体执行`SELECT`查询。**

所以，在这种情况下选择`getReference()`看起来是一个好的做法。这是因为代理`Game`实体足以从`Player`实体创建关系，而`Game`实体不需要初始化。

因此，当我们更新实体关系时，**使用`getReference()`可以消除不必要的数据库往返。**

## 5.休眠一级缓存

有时令人困惑的是，在某些情况下，**方法`find()`和`getReference()`可能都不执行任何`SELECT`查询。**

让我们设想一种情况，在我们操作之前，我们的实体已经被加载到持久化上下文中:

```java
entityManager.getTransaction().begin();
entityManager.persist(new Game(1L, "Game 1"));
entityManager.persist(new Player(1L, "Player 1"));
entityManager.getTransaction().commit();

entityManager.getTransaction().begin();
Game game1 = entityManager.getReference(Game.class, 1L);

Player player1 = entityManager.find(Player.class, 1L);
player1.setGame(game1);

entityManager.persist(player1);
entityManager.getTransaction().commit(); 
```

运行测试表明，只有更新查询被执行:

```java
Hibernate: update Player set game_id=?, name=? where id=? 
```

在这种情况下，我们应该注意到，**我们看不到任何`SELECT`查询，无论我们使用`find()`还是`getReference()`T5。这是因为我们的实体缓存在 Hibernate 的一级缓存中`.`**

因此，**当我们的实体存储在 Hibernate 的一级缓存中时，`find()`和`getReference()`方法的行为是相同的，不会命中我们的数据库**。

## 6.不同的 JPA 实现

最后提醒一下，我们应该知道`getReference()`方法的行为取决于底层的持久性提供者。

根据 [JPA 2 规范](https://web.archive.org/web/20220628082429/https://github.com/eclipse-ee4j/jpa-api/blob/ffe76306a2846df4849612af20ef03ecc6ff4843/api/src/main/java/javax/persistence/EntityManager.java#L258)，持久性提供者被允许在调用`getReference()`方法时抛出`EntityNotFoundException`。因此，对于其他持久性提供者来说可能会有所不同，当我们使用`getReference()`时可能会遇到`EntityNotFoundException`。

尽管如此，默认情况下， **Hibernate 并不遵循`getReference()`的规范，以在可能的情况下保存数据库往返**。因此，当我们检索实体代理时，即使它们不存在于数据库中，它也不会抛出异常。

或者， **Hibernate 提供了一个配置属性，为那些想要遵循 JPA 规范的人提供了一种固执己见的方式。**

在这种情况下，我们可以考虑将`hibernate.jpa.compliance.proxy`属性设置为`true`:

```java
<property name="hibernate.jpa.compliance.proxy" value="true"/> 
```

有了这个设置，Hibernate 在任何情况下都会初始化实体代理，这意味着即使我们使用了`getReference()`，它也会执行一个`SELECT`查询。

## 7.结论

在本教程中，我们探索了一些可以受益于引用代理对象的用例，并学习了如何在 Hibernate 中使用`EntityManager`的 `getReference()`方法。

像往常一样，本教程的所有代码样本和更多测试用例都可以在 GitHub 上获得[。](https://web.archive.org/web/20220628082429/https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate-jpa)