# 在 Hibernate 中映射 LOB 数据

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/hibernate-lob>

## 1。概述

LOB 或大型对象是指用于存储大型对象的可变长度数据类型。

该数据类型有两种变体:

*   CLOB–`Character Large Object`将存储大型文本数据
*   BLOB–`Binary Large Object`用于存储二进制数据，如图像、音频或视频

在本教程中，我们将展示如何利用 Hibernate ORM 来持久化大型对象。

## 2。设置

例如，我们将使用 Hibernate 5 和 H2 数据库。因此，我们必须在我们的`pom.xml:`中将它们声明为依赖项

```java
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.4.12.Final</version>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.196</version>
</dependency>
```

依赖关系的最新版本在 [Maven 中央存储库](https://web.archive.org/web/20220926194322/https://search.maven.org/classic/#search%7Cga%7C1%7C(g%3A%22com.h2database%22%20OR%20g%3A%22org.hibernate%22)%20AND%20(a%3A%22h2%22%20OR%20a%3A%22hibernate-core%22))中。

要更深入地了解配置 Hibernate，请参考我们的一篇介绍性文章。

## 3。LOB 数据模型

我们的模型`“User”`有 id、name 和 photo 作为属性。我们将在`User`的 photo 属性中存储一个图像，并将它映射到一个 BLOB:

```java
@Entity
@Table(name="user")
public class User {

    @Id
    private String id;

    @Column(name = "name", columnDefinition="VARCHAR(128)")
    private String name;

    @Lob
    @Column(name = "photo", columnDefinition="BLOB")
    private byte[] photo;

    // ...
}
```

`@Lob`注释指定数据库应该将属性存储为`Large Object`。`@Column`注释中的`columnDefinition`定义了属性的列类型。

因为我们要保存`byte array`，所以我们使用`BLOB.`

## 4。用途

### 4.1.启动休眠会话

```java
session = HibernateSessionUtil
  .getSessionFactory("hibernate.properties")
  .openSession();
```

使用 helper 类，我们将使用`hibernate.properties`文件中提供的数据库信息构建`Hibernate Session`。

### 4.2.正在创建用户实例

让我们假设用户将照片上传为图像文件:

```java
User user = new User();

InputStream inputStream = this.getClass()
  .getClassLoader()
  .getResourceAsStream("profile.png");

if(inputStream == null) {
    fail("Unable to get resources");
}
user.setId("1");
user.setName("User");
user.setPhoto(IOUtils.toByteArray(inputStream)); 
```

我们借助 [`Apache Commons IO`](/web/20220926194322/https://www.baeldung.com/apache-commons-io) 库将图像文件转换成字节数组，最后，我们将字节数组赋为新创建的`User`对象的一部分。

### 4.3.持久化大对象

通过使用`Session`存储`User`,`Hibernate`将把对象转换成数据库记录:

```java
session.persist(user); 
```

因为在类`User`上声明了`@Lob`注释，`Hibernate`知道它应该将`“photo”`属性存储为`BLOB`数据类型。

### 4.4.数据有效性

我们将从数据库中检索数据，并使用`Hibernate`将其映射回`Java`对象，以便与插入的数据进行比较。

由于我们知道插入的`User` ' `s id`，我们将使用它从数据库中检索数据:

```java
User result = session.find(User.class, "1"); 
```

让我们将查询结果与输入`User`的数据进行比较:

```java
assertNotNull(
  "Query result is null", 
  result);

assertEquals(
  "User's name is invalid", 
  user.getName(), result.getName() );

assertTrue(
  "User's photo is corrupted", 
  Arrays.equals(user.getPhoto(), result.getPhoto()) ); 
```

`Hibernate`将使用注释上相同的映射信息将数据库中的数据映射到`Java`对象。

因此，检索到的`User`对象将具有与插入数据相同的信息。

## 5。结论

`LOB`是用于存储大型对象数据的数据类型。`LOB`有两种，分别叫做`BLOB`和`CLOB`。`BLOB`用于存储二进制数据，而`CLOB`用于存储文本数据。

使用`Hibernate`，我们已经演示了如何非常容易地将数据映射到`Java`对象，只要我们在数据库中定义正确的数据模型和适当的表结构。

一如既往，这篇文章的代码可以在 GitHub 上找到。