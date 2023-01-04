# JPA 存储过程指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/jpa-stored-procedures>

## 1。简介

在这个快速教程中，我们将探索 Java 持久性 API (JPA)中存储过程的使用。

## 2。项目设置

### 2.1。Maven 设置

我们首先需要在我们的`pom.xml` **`:`** 中定义以下依赖关系

*   `javax.javaee-api`–因为它包括 JPA API
*   JPA API 实现——在这个例子中，我们将使用`**Hibernate**`,但是 **EclipseLink** 也是一个不错的选择
*   一个 MySQL 数据库

```java
<properties>
    <jee.version>7.0</jee.version>
    <mysql.version>11.2.0.4</mysql.version>
    <hibernate.version>5.1.38</hibernate.version>
</properties>
<dependencies>
    <dependency>
        <groupId>javax</groupId>
        <artifactId>javaee-api</artifactId>
        <version>${jee.version}</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-core</artifactId>
        <version>${hibernate.version}</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql.version}</version>
    </dependency>
</dependencies>
```

### 2.2。持久性单元定义

第二步是创建`src/main/resources/META-INF/persistence.xml`文件——它包含持久性单元定义:

```java
<?xml version="1.0" encoding="UTF-8"?>
<persistence 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/persistence
    http://java.sun.com/xml/ns/persistence/persistence_2_1.xsd"
    version="2.1">

    <persistence-unit name="jpa-db">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <class>com.baeldung.jpa.model.Car</class>
        <properties>
            <property name="javax.persistence.jdbc.driver" 
              value="com.mysql.jdbc.Driver" />
            <property name="javax.persistence.jdbc.url" 
              value="jdbc:mysql://127.0.0.1:3306/baeldung" />
            <property name="javax.persistence.jdbc.user" 
              value="baeldung" />
            <property name="javax.persistence.jdbc.password" 
              value="YourPassword" />
            <property name="hibernate.dialect" 
              value="org.hibernate.dialect.MySQLDialect" />
            <property name="hibernate.show_sql" value="true" />
        </properties>
    </persistence-unit>

</persistence>
```

如果引用 JNDI 数据源(JEE 环境)，则不需要定义所有 Hibernate 属性:

```java
<jta-data-source>java:jboss/datasources/JpaStoredProcedure</jta-data-source>
```

### 2.3。表格创建脚本

现在让我们创建一个`Table ( CAR )`——有三个属性:`ID, MODEL` 和 `YEAR` :

```java
CREATE TABLE `car` (
  `ID` int(10) NOT NULL AUTO_INCREMENT,
  `MODEL` varchar(50) NOT NULL,
  `YEAR` int(4) NOT NULL,
  PRIMARY KEY (`ID`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;
```

当然，前提是数据库模式和权限已经存在。

### 2.4。在数据库上创建存储过程

跳转到 java 代码之前的最后一步是在 MySQL 数据库中创建存储过程:

```java
DELIMITER $
CREATE DEFINER=`root`@`localhost` PROCEDURE `FIND_CAR_BY_YEAR`(in p_year int)
begin
SELECT ID, MODEL, YEAR
    FROM CAR
    WHERE YEAR = p_year;
end
$
DELIMITER ;
```

## 3。JPA 存储过程

我们现在准备使用 JPA 与数据库通信，并执行我们定义的存储过程。

一旦我们这样做了，我们也将能够迭代结果作为`Car.`的`List`

### 3.1。`Car` 实体

实体管理器将把它映射到数据库表中。

注意，我们还通过使用 **`@NamedStoredProcedureQueries`** 注释直接在实体上定义了存储过程:

```java
@Entity
@Table(name = "CAR")
@NamedStoredProcedureQueries({
  @NamedStoredProcedureQuery(
    name = "findByYearProcedure", 
    procedureName = "FIND_CAR_BY_YEAR", 
    resultClasses = { Car.class }, 
    parameters = { 
        @StoredProcedureParameter(
          name = "p_year", 
          type = Integer.class, 
          mode = ParameterMode.IN) }) 
})
public class Car {

    private long id;
    private String model;
    private Integer year;

    public Car(String model, Integer year) {
        this.model = model;
        this.year = year;
    }

    public Car() {
    }

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "ID", unique = true, nullable = false, scale = 0)
    public long getId() {
        return id;
    }

    @Column(name = "MODEL")
    public String getModel() {
        return model;
    }

    @Column(name = "YEAR")
    public Integer getYear() {
        return year;
    }

    // standard setter methods
}
```

### 3.2。访问数据库

现在，定义好了一切，让我们编写几个实际使用 JPA 执行过程的测试。

我们将检索给定`year` :中的所有`Cars`

```java
public class StoredProcedureTest {

    private static EntityManagerFactory factory = null;
    private static EntityManager entityManager = null;

    @BeforeClass
    public static void init() {
        factory = Persistence.createEntityManagerFactory("jpa-db");
        entityManager = factory.createEntityManager();
    }

    @Test
    public void findCarsByYearWithNamedStored() {
        StoredProcedureQuery findByYearProcedure = 
          entityManager.createNamedStoredProcedureQuery("findByYearProcedure");

        StoredProcedureQuery storedProcedure = 
          findByYearProcedure.setParameter("p_year", 2015);

        storedProcedure.getResultList()
          .forEach(c -> Assert.assertEquals(new Integer(2015), ((Car) c).getYear())); 
    }

    @Test
    public void findCarsByYearNoNamedStored() {
        StoredProcedureQuery storedProcedure = 
          entityManager
            .createStoredProcedureQuery("FIND_CAR_BY_YEAR",Car.class)
            .registerStoredProcedureParameter(1, Integer.class, ParameterMode.IN)
            .setParameter(1, 2015);

        storedProcedure.getResultList()
          .forEach(c -> Assert.assertEquals(new Integer(2015), ((Car) c).getYear()));
    }

} 
```

注意，在第二个测试中，**我们不再使用我们在实体**上定义的存储过程。相反，我们从头开始定义程序。

当您需要使用存储过程，但没有修改实体并重新编译它们的选项时，这可能非常有用。

## 4。结论

在本教程中，我们讨论了在 Java 持久性 API 中使用存储过程。

本文中使用的示例在 GitHub 中作为[示例项目提供。](https://web.archive.org/web/20220709111333/https://github.com/eugenp/tutorials/tree/master/persistence-modules/java-jpa)