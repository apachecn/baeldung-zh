# 模型映射器使用指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-modelmapper>

## 1.概观

在之前的教程中，我们已经看到了如何使用 ModelMapper 来[映射列表。](/web/20221208143832/https://www.baeldung.com/java-modelmapper-lists)

在本教程中，我们将展示如何在 ModelMapper 中不同结构的对象之间映射数据。

虽然模型映射器的默认转换在典型情况下工作得很好，但我们将主要关注如何匹配那些不太相似的对象，以便使用默认配置进行处理。

因此，这次我们将着眼于属性映射和配置更改。

## 2.Maven 依赖性

为了开始使用模型映射器[库](https://web.archive.org/web/20221208143832/https://search.maven.org/artifact/org.modelmapper/modelmapper)，我们将把依赖项添加到我们的`pom.xml`:

```java
<dependency>
    <groupId>org.modelmapper</groupId>
    <artifactId>modelmapper</artifactId>
    <version>2.4.4</version>
</dependency>
```

## 3.默认配置

当我们的源和目标对象彼此相似时，ModelMapper 提供了一个插入式解决方案。

让我们分别看看我们的域对象和对应的数据传输对象`Game` 和 `GameDTO`:

```java
public class Game {

    private Long id;
    private String name;
    private Long timestamp;

    private Player creator;
    private List<Player> players = new ArrayList<>();

    private GameSettings settings;

    // constructors, getters and setters
}

public class GameDTO {

    private Long id;
    private String name;

    // constructors, getters and setters
}
```

`GameDTO`只包含两个字段，但是字段类型和名称与源完全匹配。

在这种情况下，模型映射器无需额外配置即可处理转换:

```java
@BeforeEach
public void setup() {
    this.mapper = new ModelMapper();
}

@Test
public void whenMapGameWithExactMatch_thenConvertsToDTO() {
    // when similar source object is provided
    Game game = new Game(1L, "Game 1");
    GameDTO gameDTO = this.mapper.map(game, GameDTO.class);

    // then it maps by default
    assertEquals(game.getId(), gameDTO.getId());
    assertEquals(game.getName(), gameDTO.getName());
}
```

## 4.什么是模型映射器中的属性映射？

在我们的项目中，大多数时候，我们需要定制我们的 d to。当然，这将导致不同的字段、层次以及它们之间不规则的映射。有时，我们还需要一个以上的 DTO 用于单一来源，反之亦然。

因此，**属性映射为我们提供了一种扩展映射逻辑的强大方法。**

让我们通过添加一个新字段`creationTime`来定制我们的`GameDTO`:

```java
public class GameDTO {

    private Long id;
    private String name;
    private Long creationTime;

    // constructors, getters and setters
}
```

我们将把`Game`的`timestamp`字段映射到`GameDTO`的`creationTime`字段。注意**这次源字段名称不同于目的字段名称。**

为了定义属性映射，我们将使用 ModelMapper 的`TypeMap`。

因此，让我们创建一个`TypeMap`对象，并通过它的 *addMapping* 方法添加一个属性映射:

```java
@Test
public void whenMapGameWithBasicPropertyMapping_thenConvertsToDTO() {
    // setup
    TypeMap<Game, GameDTO> propertyMapper = this.mapper.createTypeMap(Game.class, GameDTO.class);
    propertyMapper.addMapping(Game::getTimestamp, GameDTO::setCreationTime);

    // when field names are different
    Game game = new Game(1L, "Game 1");
    game.setTimestamp(Instant.now().getEpochSecond());
    GameDTO gameDTO = this.mapper.map(game, GameDTO.class);

    // then it maps via property mapper
    assertEquals(game.getId(), gameDTO.getId());
    assertEquals(game.getName(), gameDTO.getName());
    assertEquals(game.getTimestamp(), gameDTO.getCreationTime());
}
```

### 4.1.深度映射

还有不同的映射方式。例如， **ModelMapper 可以映射层次结构——不同级别的字段可以深度映射。**

让我们在`GameDTO`中定义一个名为`creator`的`String`字段。

然而，`Game`域上的源`creator`字段不是一个简单的类型，而是一个对象——`Player`:

```java
public class Player {

    private Long id;
    private String name;

    // constructors, getters and setters
}

public class Game {
    // ...

    private Player creator;

    // ...
}

public class GameDTO {
    // ...

    private String creator;

    // ...
}
```

因此，我们不会将整个`Player` 对象的数据，而只是将`name` 字段转移到`GameDTO`。

**为了定义深度映射，我们使用了`TypeMap`的`addMappings`方法，并增加了一个`ExpressionMap`** :

```java
@Test
public void whenMapGameWithDeepMapping_thenConvertsToDTO() {
    // setup
    TypeMap<Game, GameDTO> propertyMapper = this.mapper.createTypeMap(Game.class, GameDTO.class);
    // add deep mapping to flatten source's Player object into a single field in destination
    propertyMapper.addMappings(
      mapper -> mapper.map(src -> src.getCreator().getName(), GameDTO::setCreator)
    );

    // when map between different hierarchies
    Game game = new Game(1L, "Game 1");
    game.setCreator(new Player(1L, "John"));
    GameDTO gameDTO = this.mapper.map(game, GameDTO.class);

    // then
    assertEquals(game.getCreator().getName(), gameDTO.getCreator());
}
```

### 4.2.跳过属性

有时，我们不想在 dto 中公开所有数据。无论是让我们的 dto 更轻便还是隐藏一些合理的数据，这些原因都可能导致我们在转换到 dto 时排除一些字段。

幸运的是， **ModelMapper 通过跳过支持属性排除。**

让我们借助于`skip`方法将`id`字段排除在传输之外:

```java
@Test
public void whenMapGameWithSkipIdProperty_thenConvertsToDTO() {
    // setup
    TypeMap<Game, GameDTO> propertyMapper = this.mapper.createTypeMap(Game.class, GameDTO.class);
    propertyMapper.addMappings(mapper -> mapper.skip(GameDTO::setId));

    // when id is skipped
    Game game = new Game(1L, "Game 1");
    GameDTO gameDTO = this.mapper.map(game, GameDTO.class);

    // then destination id is null
    assertNull(gameDTO.getId());
    assertEquals(game.getName(), gameDTO.getName());
}
```

因此，`GameDTO`的`id`字段被跳过并且不被设置。

### 4.3.`Converter`

ModelMapper 的另一个条款是`Converter`。**我们可以定制特定源到目标映射的转换。**

假设我们在`Game`域中有一个`Player`的集合。让我们把`Player`的计数转移到`GameDTO`。

作为第一步，我们在`GameDTO`中定义一个整数字段`totalPlayers`:

```java
public class GameDTO {
    // ...

    private int totalPlayers;

    // constructors, getters and setters
}
```

我们分别创建了`collectionToSize` `Converter`:

```java
Converter<Collection, Integer> collectionToSize = c -> c.getSource().size();
```

最后，当我们添加我们的`ExpressionMap`时，我们通过`using`方法注册我们的`Converter`:

```java
propertyMapper.addMappings(
  mapper -> mapper.using(collectionToSize).map(Game::getPlayers, GameDTO::setTotalPlayers)
);
```

因此，我们将`Game`的`getPlayers().size()`映射到`GameDTO`的`totalPlayers` 字段:

```java
@Test
public void whenMapGameWithCustomConverter_thenConvertsToDTO() {
    // setup
    TypeMap<Game, GameDTO> propertyMapper = this.mapper.createTypeMap(Game.class, GameDTO.class);
    Converter<Collection, Integer> collectionToSize = c -> c.getSource().size();
    propertyMapper.addMappings(
      mapper -> mapper.using(collectionToSize).map(Game::getPlayers, GameDTO::setTotalPlayers)
    );

    // when collection to size converter is provided
    Game game = new Game();
    game.addPlayer(new Player(1L, "John"));
    game.addPlayer(new Player(2L, "Bob"));
    GameDTO gameDTO = this.mapper.map(game, GameDTO.class);

    // then it maps the size to a custom field
    assertEquals(2, gameDTO.getTotalPlayers());
}
```

### 4.4.`Provider`

在另一个用例中，我们有时需要为目的对象提供一个实例，而不是让 ModalMapper 初始化它。这就是`Provider`派上用场的地方。

相应地， **ModelMapper 的`Provider`是定制目标对象实例化的内置方式。**

我们来做个转换，这次不是`Game`到 DTO，而是`Game`到`Game`。

因此，原则上，我们有一个持久化的`Game`域，我们从它的存储库中获取它。

之后，我们通过合并另一个`Game`对象来更新`Game`实例:

```java
@Test
public void whenUsingProvider_thenMergesGameInstances() {
    // setup
    TypeMap<Game, Game> propertyMapper = this.mapper.createTypeMap(Game.class, Game.class);
    // a provider to fetch a Game instance from a repository
    Provider<Game> gameProvider = p -> this.gameRepository.findById(1L);
    propertyMapper.setProvider(gameProvider);

    // when a state for update is given
    Game update = new Game(1L, "Game Updated!");
    update.setCreator(new Player(1L, "John"));
    Game updatedGame = this.mapper.map(update, Game.class);

    // then it merges the updates over on the provided instance
    assertEquals(1L, updatedGame.getId().longValue());
    assertEquals("Game Updated!", updatedGame.getName());
    assertEquals("John", updatedGame.getCreator().getName());
}
```

### 4.5.条件映射

**ModelMapper 也支持条件映射。**我们可以使用的内置条件方法之一是`Conditions.isNull()`。

让我们跳过`id`字段，以防它是我们的源`Game`对象中的`null`:

```java
@Test
public void whenUsingConditionalIsNull_thenMergesGameInstancesWithoutOverridingId() {
    // setup
    TypeMap<Game, Game> propertyMapper = this.mapper.createTypeMap(Game.class, Game.class);
    propertyMapper.setProvider(p -> this.gameRepository.findById(2L));
    propertyMapper.addMappings(mapper -> mapper.when(Conditions.isNull()).skip(Game::getId, Game::setId));

    // when game has no id
    Game update = new Game(null, "Not Persisted Game!");
    Game updatedGame = this.mapper.map(update, Game.class);

    // then destination game id is not overwritten
    assertEquals(2L, updatedGame.getId().longValue());
    assertEquals("Not Persisted Game!", updatedGame.getName());
}
```

注意，通过结合使用`isNull`条件和`skip`方法，我们保护了目的地`id`不被`null`值覆盖。

此外，**我们还可以定义自定义`Condition` s.**

让我们定义一个条件来检查`Game`的`timestamp`字段是否有值:

```java
Condition<Long, Long> hasTimestamp = ctx -> ctx.getSource() != null && ctx.getSource() > 0;
```

接下来，我们用`when`方法在属性映射器中使用它:

```java
TypeMap<Game, GameDTO> propertyMapper = this.mapper.createTypeMap(Game.class, GameDTO.class);
Condition<Long, Long> hasTimestamp = ctx -> ctx.getSource() != null && ctx.getSource() > 0;
propertyMapper.addMappings(
  mapper -> mapper.when(hasTimestamp).map(Game::getTimestamp, GameDTO::setCreationTime)
);
```

最后，如果`timestamp`的值大于零，模型映射器仅更新`GameDTO`的`creationTime`字段:

```java
@Test
public void whenUsingCustomConditional_thenConvertsDTOSkipsZeroTimestamp() {
    // setup
    TypeMap<Game, GameDTO> propertyMapper = this.mapper.createTypeMap(Game.class, GameDTO.class);
    Condition<Long, Long> hasTimestamp = ctx -> ctx.getSource() != null && ctx.getSource() > 0;
    propertyMapper.addMappings(
      mapper -> mapper.when(hasTimestamp).map(Game::getTimestamp, GameDTO::setCreationTime)
    );

    // when game has zero timestamp
    Game game = new Game(1L, "Game 1");
    game.setTimestamp(0L);
    GameDTO gameDTO = this.mapper.map(game, GameDTO.class);

    // then timestamp field is not mapped
    assertEquals(game.getId(), gameDTO.getId());
    assertEquals(game.getName(), gameDTO.getName());
    assertNotEquals(0L ,gameDTO.getCreationTime());

    // when game has timestamp greater than zero
    game.setTimestamp(Instant.now().getEpochSecond());
    gameDTO = this.mapper.map(game, GameDTO.class);

    // then timestamp field is mapped
    assertEquals(game.getId(), gameDTO.getId());
    assertEquals(game.getName(), gameDTO.getName());
    assertEquals(game.getTimestamp() ,gameDTO.getCreationTime());
}
```

## 5.映射的替代方式

在大多数情况下，属性映射是一个好方法，因为它允许我们做出明确的定义，并清楚地看到映射是如何流动的。

然而，对于一些对象，特别是当它们有不同的属性层次时，**我们可以使用`LOOSE`匹配策略来代替`TypeMap`。**

### 5.1.匹配策略`LOOSE`

为了展示松散匹配的好处，让我们在`GameDTO`中再添加两个属性:

```java
public class GameDTO {
    //...

    private GameMode mode;
    private int maxPlayers;

    // constructors, getters and setters
}
```

注意，`mode`和`maxPlayers`对应于`GameSettings`的属性，它是我们的`Game`源类中的一个内部对象:

```java
public class GameSettings {

    private GameMode mode;
    private int maxPlayers;

    // constructors, getters and setters
}
```

这样，**我们可以执行双向映射**，从`Game`到`GameDTO`以及从**到`Game`的反向映射，而无需定义任何`TypeMap`** :

```java
@Test
public void whenUsingLooseMappingStrategy_thenConvertsToDomainAndDTO() {
    // setup
    this.mapper.getConfiguration().setMatchingStrategy(MatchingStrategies.LOOSE);

    // when dto has flat fields for GameSetting
    GameDTO gameDTO = new GameDTO();
    gameDTO.setMode(GameMode.TURBO);
    gameDTO.setMaxPlayers(8);
    Game game = this.mapper.map(gameDTO, Game.class);

    // then it converts to inner objects without property mapper
    assertEquals(gameDTO.getMode(), game.getSettings().getMode());
    assertEquals(gameDTO.getMaxPlayers(), game.getSettings().getMaxPlayers());

    // when the GameSetting's field names match
    game = new Game();
    game.setSettings(new GameSettings(GameMode.NORMAL, 6));
    gameDTO = this.mapper.map(game, GameDTO.class);

    // then it flattens the fields on dto
    assertEquals(game.getSettings().getMode(), gameDTO.getMode());
    assertEquals(game.getSettings().getMaxPlayers(), gameDTO.getMaxPlayers());
}
```

### 5.2.自动跳过空属性

此外，模型映射器具有一些有用的全局配置。其中之一就是`setSkipNullEnabled`设定。

因此，**如果源属性是`null`，我们可以自动跳过它们，而不用编写任何[条件映射](#5-conditional-mapping)** :

```java
@Test
public void whenConfigurationSkipNullEnabled_thenConvertsToDTO() {
    // setup
    this.mapper.getConfiguration().setSkipNullEnabled(true);
    TypeMap<Game, Game> propertyMap = this.mapper.createTypeMap(Game.class, Game.class);
    propertyMap.setProvider(p -> this.gameRepository.findById(2L));

    // when game has no id
    Game update = new Game(null, "Not Persisted Game!");
    Game updatedGame = this.mapper.map(update, Game.class);

    // then destination game id is not overwritten
    assertEquals(2L, updatedGame.getId().longValue());
    assertEquals("Not Persisted Game!", updatedGame.getName());
}
```

### 5.3.循环引用对象

有时，我们需要处理引用了自身的对象。

通常，这会导致循环依赖，并导致著名的`StackOverflowError`:

```java
org.modelmapper.MappingException: ModelMapper mapping errors:

1) Error mapping com.bealdung.domain.Game to com.bealdung.dto.GameDTO

1 error
	...
Caused by: java.lang.StackOverflowError
	...
```

因此，在这种情况下，另一个配置`setPreferNestedProperties`将帮助我们:

```java
@Test
public void whenConfigurationPreferNestedPropertiesDisabled_thenConvertsCircularReferencedToDTO() {
    // setup
    this.mapper.getConfiguration().setPreferNestedProperties(false);

    // when game has circular reference: Game -> Player -> Game
    Game game = new Game(1L, "Game 1");
    Player player = new Player(1L, "John");
    player.setCurrentGame(game);
    game.setCreator(player);
    GameDTO gameDTO = this.mapper.map(game, GameDTO.class);

    // then it resolves without any exception
    assertEquals(game.getId(), gameDTO.getId());
    assertEquals(game.getName(), gameDTO.getName());
}
```

因此，当我们将`false`传递给`setPreferNestedProperties`时，映射毫无例外地工作。

## 6.结论

在本文中，我们解释了如何使用 ModelMapper 中的属性映射器定制类到类的映射。

我们还看到了一些替代配置的详细示例。

与往常一样，所有示例的源代码都可以在 GitHub 上的[处获得。](https://web.archive.org/web/20221208143832/https://github.com/eugenp/tutorials/tree/master/libraries-6)