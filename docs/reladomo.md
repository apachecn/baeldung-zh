# Reladomo 简介

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/reladomo>

## 1。概述

**`[Reladomo (formerly known as Mithra)](https://web.archive.org/web/20220628234451/https://goldmansachs.github.io/reladomo/)`是 Java** 的对象关系映射(ORM)框架，在`Goldman Sachs`开发，目前作为开源项目发布。该框架提供了 ORM 通常需要的特性以及一些额外的特性。

让我们来看看`Reladomo`的一些关键特性:

*   它可以生成 Java 类和 DDL 脚本
*   它由 XML 文件中编写的元数据驱动
*   生成的代码是可扩展的
*   查询语言是面向对象和强类型的
*   该框架提供了对分片的支持(相同的模式，不同的数据集)
*   还包括对测试的支持
*   它提供了一些有用的特性，比如高性能缓存和事务

在接下来的部分，我们将看到设置和一些基本的使用示例。

## 2。`Maven`设置

要开始使用 ORM，我们需要将`[reladomo](https://web.archive.org/web/20220628234451/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22reladomo%22)` 依赖项添加到我们的`pom.xml`文件中:

```java
<dependency>
    <groupId>com.goldmansachs.reladomo</groupId>
    <artifactId>reladomo</artifactId>
    <version>16.5.1</version>
</dependency>
```

我们将使用一个`H2`数据库作为我们的示例，所以我们还要添加 [`h2`](https://web.archive.org/web/20220628234451/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22h2%22%20AND%20g%3A%22com.h2database%22) 依赖项:

```java
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.196</version>
</dependency>
```

**除此之外，我们需要设置插件来生成类和 SQL 文件**，并在执行过程中加载它们。

对于文件生成，我们可以使用通过`maven-antrun-plugin`执行的任务。首先，让我们看看如何定义生成 Java 类的任务:

```java
<plugin>
    <artifactId>maven-antrun-plugin</artifactId>
    <executions>
        <execution>
            <id>generateMithra</id>
            <phase>generate-sources</phase>
            <goals>
                <goal>run</goal>
            </goals>
            <configuration>
                <tasks>
                    <property name="plugin_classpath" 
                      refid="maven.plugin.classpath"/>
                    <taskdef name="gen-reladomo" 
                      classpath="plugin_classpath"
                      classname="com.gs.fw.common.mithra.generator.MithraGenerator"/>
                    <gen-reladomo 
                      xml="${project.basedir}/src/main/resources/reladomo/ReladomoClassList.xml"
                      generateGscListMethod="true"
                      generatedDir="${project.build.directory}/generated-sources/reladomo"
                      nonGeneratedDir="${project.basedir}/src/main/java"/>
                </tasks>
            </configuration>
        </execution>
    </executions>
</plugin> 
```

**`gen-reladomo`任务使用提供的`MithraGenerator`根据`ReladomoClassList.xml`文件中的配置创建 Java 文件。**我们将在后面的小节中仔细看看这个文件包含了什么。

这些任务还有两个定义生成文件位置的属性:

*   `generatedDir`–包含不应修改或版本化的类
*   `nonGeneratedDir`–生成的具体对象类，可以进一步定制和版本化

对应于 Java 对象的数据库表可以手动创建，也可以使用第二个`Ant`任务生成的 DDL 脚本自动创建:

```java
<taskdef 
  name="gen-ddl"
  classname = "com.gs.fw.common.mithra.generator.dbgenerator.MithraDbDefinitionGenerator"
  loaderRef="reladomoGenerator">
    <classpath refid="maven.plugin.classpath"/>
</taskdef>
<gen-ddl 
  xml="${project.basedir}/src/main/resources/reladomo/ReladomoClassList.xml"
  generatedDir="${project.build.directory}/generated-db/sql"
  databaseType="postgres"/>
```

这个任务使用基于前面提到的同一个`ReladomoClassList.xml`文件的`MithraDbDefinitionGenerator`。SQL 脚本将被放在`generated-db/sql`目录中。

为了完成这个插件的定义，我们还必须添加两个用于创建的依赖项:

```java
<plugin>
    <artifactId>maven-antrun-plugin</artifactId>
    <executions>
    //...               
    </executions>
    <dependencies>
        <dependency>
            <groupId>com.goldmansachs.reladomo</groupId>
            <artifactId>reladomogen</artifactId>
            <version>16.5.1</version>
        </dependency>
        <dependency>
            <groupId>com.goldmansachs.reladomo</groupId>
            <artifactId>reladomo-gen-util</artifactId>
            <version>16.5.1</version>
        </dependency>
    </dependencies>
</plugin>
```

最后，使用`build-helper-maven-plugin`，我们可以将生成的文件添加到类路径中:

```java
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>build-helper-maven-plugin</artifactId>
    <executions>
        <execution>
            <id>add-source</id>
            <phase>generate-sources</phase>
            <goals>
                <goal>add-source</goal>
            </goals>
            <configuration>
                <sources>
                    <source>${project.build.directory}/generated-sources/reladomo</source>
                </sources>
            </configuration>
        </execution>
        <execution>
            <id>add-resource</id>
            <phase>generate-resources</phase>
            <goals>
                <goal>add-resource</goal>
            </goals>
            <configuration>
                <resources>
                    <resource>
                        <directory>${project.build.directory}/generated-db/</directory>
                    </resource>
                </resources>
            </configuration>
        </execution>
    </executions>
</plugin>
```

添加 DDL 脚本是可选的。在我们的示例中，我们将使用内存中的数据库，因此我们希望执行脚本来创建表。

## 3。XML 配置

可以在几个 XML 文件中定义`Reladomo`框架的元数据。

### 3.1。对象 XML 文件

我们想要创建的每个实体都需要在它的 XML 文件中定义。

让我们用两个实体创建一个简单的例子:部门和雇员。下面是我们的领域模型的可视化表示:

[![tables](img/67607aebc224f557f118979efb5024cd.png)](/web/20220628234451/https://www.baeldung.com/wp-content/uploads/2017/09/tables.png)

让我们定义第一个`Department.xml`文件:

```java
<MithraObject objectType="transactional">
    <PackageName>com.baeldung.reladomo</PackageName>
    <ClassName>Department</ClassName>
    <DefaultTable>departments</DefaultTable>

    <Attribute name="id" javaType="long" 
      columnName="department_id" primaryKey="true"/>
    <Attribute name="name" javaType="String" 
      columnName="name" maxLength="50" truncate="true"/>
    <Relationship name="employees" relatedObject="Employee" 
      cardinality="one-to-many" 
      reverseRelationshipName="department" 
      relatedIsDependent="true">
         Employee.departmentId = this.id
    </Relationship>
</MithraObject>
```

我们可以看到在上面的**中，实体被定义在一个名为`MithraObject`** 的根元素中。然后，我们指定了相应数据库表的包、类和名称。

类型的每个属性都是使用一个`Attribute`元素定义的，我们可以为它声明名称、Java 类型和列名。

**我们可以用`Relationship`标签来描述对象之间的关系。**在我们的例子中，我们已经定义了`Department`和`Employee`对象之间的`one-to-many`关系，基于表达式:

```java
Employee.departmentId = this.id
```

**`reverseRelationshipName`属性可用于使关系双向化，而无需定义两次。**

属性允许我们级联操作。

接下来，让我们以类似的方式创建`Employee.xml`文件:

```java
<MithraObject objectType="transactional">
    <PackageName>com.baeldung.reladomo</PackageName>
    <ClassName>Employee</ClassName>
    <DefaultTable>employees</DefaultTable>

    <Attribute name="id" javaType="long" 
      columnName="employee_id" primaryKey="true"/>
    <Attribute name="name" javaType="String" 
      columnName="name" maxLength="50" truncate="true"/>
    <Attribute name="departmentId" javaType="long" 
      columnName="department_id"/>
</MithraObject>
```

### 3.2。`ReladomoClassList.xml`文件

需要被告知它应该生成的对象。

在`Maven`部分中，我们将`ReladomoClassList.xml`文件定义为生成任务的源，所以现在应该创建文件了:

```java
<Mithra>
    <MithraObjectResource name="Department"/>
    <MithraObjectResource name="Employee"/>
</Mithra>
```

这是一个简单的文件，包含一个实体列表，将根据 XML 配置为这些实体生成类。

## 4。生成的类

现在，我们已经拥有了通过使用命令`mvn clean install`构建`Maven`应用程序来开始代码生成所需的所有元素。

具体的类将在指定包的 `src/main/java`文件夹中生成:

[![classes](img/a86599c925c119d1fc1e9d25bb0badbe.png)](/web/20220628234451/https://www.baeldung.com/wp-content/uploads/2017/08/classes.png)

这些是简单的类，我们可以在其中添加自定义代码。例如，`Department`类只包含一个不应该被移除的构造函数:

```java
public class Department extends DepartmentAbstract {
    public Department() {
        super();
        // You must not modify this constructor. Mithra calls this internally.
        // You can call this constructor. You can also add new constructors.
    }
}
```

如果我们想给这个类添加一个自定义构造函数，它也需要调用父构造函数:

```java
public Department(long id, String name){
    super();
    this.setId(id);
    this.setName(name);
}
```

这些类基于`generated-sources/reladomo`文件夹中的抽象和实用类:

[![gen classes](img/984a1bb6405febdd9585d47599a1e2ee.png)](/web/20220628234451/https://www.baeldung.com/wp-content/uploads/2017/08/gen-classes.png)

该文件夹中的主要类类型有:

*   `DepartmentAbstract`和`EmployeeAbstract`类——包含使用定义的实体的方法
*   `DepartmentListAbstract`和`EmployeeListAbstract`——包含使用部门和员工列表的方法
*   `DepartmentFinder`和`EmployeeFinder`——这些提供了查询实体的方法
*   其他实用程序类别

通过生成这些类，对我们的实体执行 CRUD 操作所需的大部分代码已经为我们创建好了。

## 5。Reladomo 应用程序

为了对数据库执行操作，我们需要一个允许我们获得数据库连接的连接管理器类。

### 5.1。连接管理器

当使用单个数据库时，我们可以实现`SourcelessConnectionManager`接口:

```java
public class ReladomoConnectionManager implements SourcelessConnectionManager {

    private static ReladomoConnectionManager instance;
    private XAConnectionManager xaConnectionManager;

    public static synchronized ReladomoConnectionManager getInstance() {
        if (instance == null) {
            instance = new ReladomoConnectionManager();
        }
        return instance;
    }

    private ReladomoConnectionManager() {
        this.createConnectionManager();
    }
    //...
}
```

我们的`ReladomoConnectionManager`类实现了单例模式，并且基于一个`XAConnectionManager`，它是一个事务连接管理器的实用类。

让我们仔细看看`createConnectionManager()`方法:

```java
private XAConnectionManager createConnectionManager() {
    xaConnectionManager = new XAConnectionManager();
    xaConnectionManager.setDriverClassName("org.h2.Driver");
    xaConnectionManager.setJdbcConnectionString("jdbc:h2:mem:myDb");
    xaConnectionManager.setJdbcUser("sa");
    xaConnectionManager.setJdbcPassword("");
    xaConnectionManager.setPoolName("My Connection Pool");
    xaConnectionManager.setInitialSize(1);
    xaConnectionManager.setPoolSize(10);
    xaConnectionManager.initialisePool();
    return xaConnectionManager;
}
```

在这个方法中，我们已经设置了创建到一个`H2`内存数据库的连接所必需的属性。

此外，我们需要从`SourcelessConnectionManager`接口实现几个方法:

```java
@Override
public Connection getConnection() {
    return xaConnectionManager.getConnection();
}

@Override
public DatabaseType getDatabaseType() {
    return H2DatabaseType.getInstance();
}

@Override
public TimeZone getDatabaseTimeZone() {
    return TimeZone.getDefault();
}

@Override
public String getDatabaseIdentifier() {
    return "myDb";
}

@Override 
public BulkLoader createBulkLoader() throws BulkLoaderException { 
    return null; 
}
```

最后，让我们添加一个定制方法来执行生成的 DDL 脚本，这些脚本创建了我们的数据库表:

```java
public void createTables() throws Exception {
    Path ddlPath = Paths.get(ClassLoader.getSystemResource("sql").toURI());
    try (
      Connection conn = xaConnectionManager.getConnection();
      Stream<Path> list = Files.list(ddlPath)) {

        list.forEach(path -> {
            try {
                RunScript.execute(conn, Files.newBufferedReader(path));
            } 
            catch (SQLException | IOException exc){
                exc.printStackTrace();
            }
        });
    }
}
```

当然，这对于生产应用程序是不必要的，在生产应用程序中，不会在每次执行时都重新创建表。

### 5.2。正在初始化`Reladomo`

`Reladomo`初始化过程使用一个配置文件，该文件指定了连接管理器类和使用的对象类型。让我们定义一个`ReladomoRuntimeConfig.xml`文件:

```java
<MithraRuntime>
    <ConnectionManager 
      className="com.baeldung.reladomo.ReladomoConnectionManager ">
    <MithraObjectConfiguration 
      className="com.baeldung.reladomo.Department" cacheType="partial"/>
    <MithraObjectConfiguration 
      className="com.baeldung.reladomo.Employee " cacheType="partial"/>
    </ConnectionManager>
</MithraRuntime>
```

接下来，我们可以创建一个主类，首先调用`createTables()`方法，然后**使用`MithraManager`类加载配置并初始化`Reladomo`** :

```java
public class ReladomoApplication {
    public static void main(String[] args) {
        try {
            ReladomoConnectionManager.getInstance().createTables();
        } catch (Exception e1) {
            e1.printStackTrace();
        }
        MithraManager mithraManager = MithraManagerProvider.getMithraManager();
        mithraManager.setTransactionTimeout(120);

        try (InputStream is = ReladomoApplication.class.getClassLoader()
          .getResourceAsStream("ReladomoRuntimeConfig.xml")) {
            MithraManagerProvider.getMithraManager()
              .readConfiguration(is);

            //execute operations
        }
        catch (IOException exc){
            exc.printStackTrace();
        }     
    }
}
```

### 5.3。执行积垢操作

现在让我们使用`Reladomo`生成的类在我们的实体上执行一些操作。

首先，让我们创建两个`Department`和`Employee`对象，然后使用`cascadeInsert()`方法保存它们:

```java
Department department = new Department(1, "IT");
Employee employee = new Employee(1, "John");
department.getEmployees().add(employee);
department.cascadeInsert();
```

每个对象也可以通过调用 `insert()`方法单独保存。在我们的例子中，可以使用`cascadeInsert()`,因为我们已经将`relatedIsDependent=true`属性添加到了关系定义中。

**要查询对象，我们可以使用生成的`Finder`类:**

```java
Department depFound = DepartmentFinder
  .findByPrimaryKey(1);
Employee empFound = EmployeeFinder
  .findOne(EmployeeFinder.name().eq("John"));
```

以这种方式获得的对象是“活”对象，这意味着使用 setters 对它们进行的任何更改都会立即反映在数据库中:

```java
empFound.setName("Steven");
```

为了避免这种行为，我们可以获得分离的对象:

```java
Department depDetached = DepartmentFinder
  .findByPrimaryKey(1).getDetachedCopy();
```

要删除对象，我们可以使用`delete()`方法:

```java
empFound.delete();
```

### 5.4。交易管理

如果我们希望一组操作作为一个单元执行或不作为一个单元执行，我们可以将它们包装在一个事务中:

```java
mithraManager.executeTransactionalCommand(tx -> {
    Department dep = new Department(2, "HR");
    Employee emp = new Employee(2, "Jim");
    dep.getEmployees().add(emp);
    dep.cascadeInsert();
    return null;
});
```

## 6。`Reladomo`测试支持

在上面的章节中，我们在 Java 主类中编写了我们的例子。

如果我们想为我们的应用程序编写测试，一种方法是简单地在测试类中编写相同的代码。

然而，**为了更好的测试支持，Reladomo 还提供了`MithraTestResource`类。**这允许我们使用不同的配置和内存数据库进行测试。

首先，我们需要添加额外的`[reladomo-test-util](https://web.archive.org/web/20220628234451/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22reladomo-test-util%22)`依赖项，以及`[junit](https://web.archive.org/web/20220628234451/https://search.maven.org/classic/#search%7Cga%7C1%7Ca%3A%22junit%22%20AND%20g%3A%22junit%22)`依赖项:

```java
<dependency>
    <groupId>com.goldmansachs.reladomo</groupId>
    <artifactId>reladomo-test-util</artifactId>
    <version>16.5.1</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>
```

接下来，我们必须创建一个使用`ConnectionManagerForTests`类的`ReladomoTestConfig.xml`文件:

```java
<MithraRuntime>
    <ConnectionManager 
      className="com.gs.fw.common.mithra.test.ConnectionManagerForTests">
        <Property name="resourceName" value="testDb"/>
        <MithraObjectConfiguration 
          className="com.baeldung.reladomo.Department" cacheType="partial"/>
        <MithraObjectConfiguration 
          className="com.baeldung.reladomo.Employee " cacheType="partial"/>
    </ConnectionManager>
 </MithraRuntime>
```

这个连接管理器配置一个只用于测试的内存中的`H2`数据库。

**`MithraTestResource`类的一个方便的特性是我们可以提供带有测试数据**的文本文件，格式如下:

```java
class com.baeldung.reladomo.Department
id, name
1, "Marketing"

class com.baeldung.reladomo.Employee
id, name
1, "Paul"
```

让我们创建一个`JUnit`测试类并在一个`@Before`方法中设置我们的`MithraTestResource`实例:

```java
public class ReladomoTest {
    private MithraTestResource mithraTestResource;

    @Before
    public void setUp() throws Exception {
        this.mithraTestResource 
          = new MithraTestResource("reladomo/ReladomoTestConfig.xml");

        ConnectionManagerForTests connectionManager
          = ConnectionManagerForTests.getInstanceForDbName("testDb");
        this.mithraTestResource.createSingleDatabase(connectionManager);
        mithraTestResource.addTestDataToDatabase("reladomo/test-data.txt", 
          connectionManager);

        this.mithraTestResource.setUp();
    }
}
```

然后我们可以编写一个简单的`@Test`方法来验证我们的测试数据是否被加载:

```java
@Test
public void whenGetTestData_thenOk() {
    Employee employee = EmployeeFinder.findByPrimaryKey(1);
    assertEquals(employee.getName(), "Paul");
}
```

测试运行后，需要清除测试数据库:

```java
@After
public void tearDown() throws Exception {
    this.mithraTestResource.tearDown();
}
```

## 7。结论

在本文中，我们介绍了`Reladomo` ORM 框架的主要特性，以及常见用法的设置和示例。

示例的源代码可以在 GitHub 的[中找到。](https://web.archive.org/web/20220628234451/https://github.com/eugenp/tutorials/tree/master/libraries-data-db)