# OData 协议指南

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/odata>

## 1.介绍

在本教程中，我们将探索 **[OData](https://web.archive.org/web/20220630135825/https://www.odata.org/) ，这是一种允许使用 RESTFul API 轻松访问数据集的标准协议。**

## 2.什么是`OData`？

OData 是使用 RESTful API 访问数据的 OASIS 和 ISO/IEC 标准。因此，它允许消费者使用标准 HTTP 调用来发现和浏览数据集。

**例如，我们可以通过一个简单的`curl `一行程序**访问[公开可用的 OData 服务](https://web.archive.org/web/20220630135825/https://www.odata.org/odata-services/):

```java
curl -s https://services.odata.org/V2/Northwind/Northwind.svc/Regions
<?xml version="1.0" encoding="utf-8" standalone="yes"?>
<feed xml:base="https://services.odata.org/V2/Northwind/Northwind.svc/" 
  xmlns:d="http://schemas.microsoft.com/ado/2007/08/dataservices" 
  xmlns:m="http://schemas.microsoft.com/ado/2007/08/dataservices/metadata" 
  >
    <title type="text">Regions</title>
    <id>https://services.odata.org/V2/Northwind/Northwind.svc/Regions</id>
... rest of xml response omitted
```

在撰写本文时，OData 协议处于第四个版本——更准确地说是 4.01。OData V4 在 2014 年达到了 OASIS 标准级别，但它的历史更长。我们可以追溯到微软的 Astoria 项目，该项目于 2007 年更名为 ADO.Net 数据服务公司。宣布这个项目的[原始博客条目](https://web.archive.org/web/20220630135825/https://devblogs.microsoft.com/odata/welcome/)仍然可以在微软的 OData 博客上找到。

与 JDBC 或 ODBC 等标准 API 相比，使用基于标准的协议来访问数据集会带来一些好处。作为最终用户级别的消费者，我们可以使用 Excel 等流行工具从任何兼容提供商处检索数据。大量可用的 REST 客户端库也有助于编程。

作为提供者，采用 OData 也有好处:一旦我们创建了一个兼容的服务，我们就可以专注于提供有价值的数据集，最终用户可以使用他们选择的工具来消费这些数据集。因为它是基于 HTTP 的协议，所以我们还可以利用安全机制、监控和日志记录等方面。

这些特征使得 OData 成为政府机构在实现公共数据服务时的热门选择，我们可以通过查看这个目录来检查。

## 3.OData 概念

OData 协议的核心是实体数据模型的概念——简称 EDM。EDM 描述了 OData 提供者通过包含许多元实体的元数据文档公开的数据:

*   实体类型及其属性(如`Person`、`Customer`、`Order`等)和关键字
*   实体之间的关系
*   用于描述嵌入到实体中的结构化类型的复杂类型(比如，作为`Customer`类型一部分的地址类型)
*   实体集，集合给定类型的实体

规范要求这个元数据文档必须在用于访问服务的根 URL 的标准位置`$metadata`可用。例如，如果我们在`http://example.org/odata.svc/`有一个可用的 OData 服务，那么它的元数据文档将在`http://example.org/odata.svc/$metadata`可用。

返回的文档包含一组描述该服务器支持的模式的 XML:

```java
<?xml version="1.0"?>
<edmx:Edmx 
  xmlns:edmx="http://schemas.microsoft.com/ado/2007/06/edmx" 
  Version="1.0">
    <edmx:DataServices 
      xmlns:m="http://schemas.microsoft.com/ado/2007/08/dataservices/metadata" 
      m:DataServiceVersion="1.0">
    ... schema elements omitted
    </edmx:DataServices>
</edmx:Edmx>
```

让我们把这份文件分成几个主要部分。

顶层元素`<edmx:Edmx> `只能有一个子元素，`<edmx:DataServices>`元素`.` **这里需要注意的重要一点是名称空间 URI，因为它允许我们识别服务器使用哪个 OData 版本。在这种情况下，名称空间表明我们有一个 OData V2 服务器，它使用微软的标识符。**

一个`DataServices`元素可以有一个或多个`Schema `元素，每个元素描述一个可用的数据集。由于对`Schema`中可用元素的完整描述超出了本文的范围，所以我们将把重点放在最重要的元素上:`EntityTypes, Associations,` 和`EntitySets`。

### 3.1.`EntityType`元素

该元素定义给定实体的可用属性，包括其主键。它还可能包含关于与其他模式类型的关系的信息，通过看一个例子——a`CarMaker –` ,我们将能够看到它与其他 ORM 技术中的描述没有太大的不同，比如 JPA:

```java
<EntityType Name="CarMaker">
    <Key>
        <PropertyRef Name="Id"/>
    </Key>
    <Property Name="Id" Type="Edm.Int64" 
      Nullable="false"/>
    <Property Name="Name" Type="Edm.String" 
      Nullable="true" 
      MaxLength="255"/>
    <NavigationProperty Name="CarModelDetails" 
      Relationship="default.CarModel_CarMaker_Many_One0" 
      FromRole="CarMaker" 
      ToRole="CarModel"/>
</EntityType>
```

这里，我们的`CarMaker`只有两个属性——`Id `和`Name`——以及与另一个`EntityType`的关联。`Key s`子元素将实体的主键定义为其`Id `属性，每个`Property`元素包含关于实体属性的数据，如名称、类型或可空性。

`NavigationProperty`是一种特殊的属性，它描述了一个相关实体的“访问点”。

### 3.2.`Association `元素

一个`Association `元素描述了两个实体之间的关联，包括两端的多重性和可选的参照完整性约束:

```java
<Association Name="CarModel_CarMaker_Many_One0">
    <End Type="default.CarModel" Multiplicity="*" Role="CarModel"/>
    <End Type="default.CarMaker" Multiplicity="1" Role="CarMaker"/>
    <ReferentialConstraint>
        <Principal Role="CarMaker">
            <PropertyRef Name="Id"/>
        </Principal>
        <Dependent Role="CarModel">
            <PropertyRef Name="Maker"/>
        </Dependent>
    </ReferentialConstraint>
</Association>
```

这里，`Association` 元素定义了一个`CarModel`和`CarMaker`实体之间的一对多关系，其中前者作为依赖方。

### 3.3.`EntitySet` 元素

我们将探索的最后一个模式概念是`EntitySet` 元素，它表示给定类型的实体集合。虽然很容易将它们比作一张表——在许多情况下，它们就是表——但更好的类比是视图。原因是，对于同一个`EntityType` ，我们可以有多个`EntitySet`元素，每个元素代表可用数据的不同子集。

作为顶级模式元素的 `EntityContainer`元素将所有可用的`EntitySet`分组:

```java
<EntityContainer Name="defaultContainer" 
  m:IsDefaultEntityContainer="true">
    <EntitySet Name="CarModels" 
      EntityType="default.CarModel"/>
    <EntitySet Name="CarMakers" 
      EntityType="default.CarMaker"/>
</EntityContainer>
```

在我们的简单例子中，我们只有两个`EntitySet`，但是我们也可以添加额外的视图，比如`ForeignCarMakers` 或者 `HistoricCarMakers`。

## 4.OData URLs 和方法

为了访问 OData 服务公开的数据，我们使用常规的 HTTP 动词:

*   GET 返回一个或多个实体
*   POST 将新实体添加到现有的`Entity Set`
*   PUT 替换给定的实体
*   补丁替换给定实体的特定属性
*   删除删除给定的实体

所有这些操作都需要一个资源路径来执行。资源路径可以定义实体集、实体或者甚至实体中的属性。

让我们来看一个用于访问我们之前的 OData 服务的示例 URL:

```java
http://example.org/odata/CarMakers 
```

这个 URL 的第一部分，从协议开始到`odata/`路径段，被称为`service root URL`，对于这个服务的所有资源路径都是一样的。**由于服务根总是相同的，我们将在下面的 URL 示例中用省略号(“…”**来代替它。

在这种情况下，`CarMakers`指的是服务元数据中声明的`EntitySets`之一。我们可以使用常规浏览器访问这个 URL，然后它应该返回一个包含所有现有的这种类型的实体的文档:

```java
<?xml version="1.0" encoding="utf-8"?>
<feed  
  xmlns:m="http://schemas.microsoft.com/ado/2007/08/dataservices/metadata" 
  xmlns:d="http://schemas.microsoft.com/ado/2007/08/dataservices" 
  xml:base="http://localhost:8080/odata/">
    <id>http://localhost:8080/odata/CarMakers</id>
    <title type="text">CarMakers</title>
    <updated>2019-04-06T17:51:33.588-03:00</updated>
    <author>
        <name/>
    </author>
    <link href="CarMakers" rel="self" title="CarMakers"/>
    <entry>
      <id>http://localhost:8080/odata/CarMakers(1L)</id>
      <title type="text">CarMakers</title>
      <updated>2019-04-06T17:51:33.589-03:00</updated>
      <category term="default.CarMaker" 
        scheme="http://schemas.microsoft.com/ado/2007/08/dataservices/scheme"/>
      <link href="CarMakers(1L)" rel="edit" title="CarMaker"/>
      <link href="CarMakers(1L)/CarModelDetails" 
        rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/CarModelDetails" 
        title="CarModelDetails" 
        type="application/atom+xml;type=feed"/>
        <content type="application/xml">
            <m:properties>
                <d:Id>1</d:Id>
                <d:Name>Special Motors</d:Name>
            </m:properties>
        </content>
    </entry>  
  ... other entries omitted
</feed>
```

对于每个`CarMaker`实例，返回的文档包含一个`entry`元素。

让我们更仔细地看看我们有哪些可用的信息:

*   `id`:该特定实体的链接
*   `title/author/updated`:关于该条目的元数据
*   `link`元素:用于指向用于编辑实体的资源(`rel=”edit”`)或指向相关实体的链接。在这种情况下，我们有一个链接将我们带到与这个特定的`CarMaker`相关联的一组`CarModel`实体。
*   `content`:实体`CarModel`的属性值

这里需要注意的重要一点是使用键-值对来标识实体集内的特定实体。在我们的例子中，键是数字的，所以像`CarMaker(1L)`这样的资源路径引用主键值等于 1 的实体——这里的“`L`”只是表示一个`long`值，可以省略。

## 5.查询选项

我们可以将查询选项传递给资源 URL，以便修改返回数据的许多方面，例如限制返回集的大小或顺序。OData 规范定义了一组丰富的选项，但这里我们将重点关注最常见的选项。

一般来说，查询选项可以相互组合，从而允许客户端轻松实现常见的功能，如分页、过滤和排序结果列表。

### 5.1.`$top`和`$skip`

我们可以使用`$top`和`$skip`查询选项:在大型数据集中进行**导航**

```java
.../CarMakers?$top=10&$skip=10 
```

`$top`告诉服务我们只需要`CarMakers`实体集的前 10 条记录。在 `$top,` 之前应用的`$skip,`告诉服务器跳过前 10 条记录。

知道给定的`Entity Set` 的大小通常很有用，为此，我们可以使用`$count`子资源:

```java
.../CarMakers/$count 
```

该资源生成一个包含相应集合大小的`text/plain`文档。在这里，我们必须注意一个提供者支持的具体 OData 版本。虽然 OData V2 支持将`$count`作为集合中的子资源，但 V4 允许将其用作查询参数。在本例中，`$count`是一个布尔值，所以我们需要相应地更改 URL:

```java
.../CarMakers?$count=true 
```

### 5.2.`$filter`

我们使用`$filter`查询选项**将从给定的`Entity Set`** 返回的实体限制为符合给定标准的实体。`$filter`的值是一个逻辑表达式，支持基本运算符、分组和许多有用的函数。例如，让我们构建一个查询，它返回所有的`CarMaker`实例，其中它的`Name `属性以字母‘B’开始:

```java
.../CarMakers?$filter=startswith(Name,'B') 
```

现在，让我们结合一些逻辑运算符来搜索特定的`Year`和`Maker`的`CarModels `:

```java
.../CarModels?$filter=Year eq 2008 and CarMakerDetails/Name eq 'BWM' 
```

这里，我们使用了等式运算符`eq`来指定属性的值。我们还可以看到如何在表达式中使用相关实体的属性。

### 5.3.`$expand`

默认情况下，OData 查询不会返回相关实体的数据，这通常是可以的。我们可以使用`$expand`查询选项来请求来自给定相关实体的数据包含在主内容中。

使用我们的示例域，让我们构建一个 URL，它从给定的模型`and`的制造者返回数据，从而避免额外的到服务器的往返行程:

```java
.../CarModels(1L)?$expand=CarMakerDetails 
```

返回的文档现在包括作为相关实体一部分的`CarMaker`数据:

```java
<?xml version="1.0" encoding="utf-8"?>
<entry  
  xmlns:m="http://schemas.microsoft.com/ado/2007/08/dataservices/metadata" 
  xmlns:d="http://schemas.microsoft.com/ado/2007/08/dataservices" 
  xml:base="http://localhost:8080/odata/">
    <id>http://example.org/odata/CarModels(1L)</id>
    <title type="text">CarModels</title>
    <updated>2019-04-07T11:33:38.467-03:00</updated>
    <category term="default.CarModel" 
      scheme="http://schemas.microsoft.com/ado/2007/08/dataservices/scheme"/>
    <link href="CarModels(1L)" rel="edit" title="CarModel"/>
    <link href="CarModels(1L)/CarMakerDetails" 
      rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/CarMakerDetails" 
      title="CarMakerDetails" 
      type="application/atom+xml;type=entry">
        <m:inline>
            <entry xml:base="http://localhost:8080/odata/">
                <id>http://example.org/odata/CarMakers(1L)</id>
                <title type="text">CarMakers</title>
                <updated>2019-04-07T11:33:38.492-03:00</updated>
                <category term="default.CarMaker" 
                  scheme="http://schemas.microsoft.com/ado/2007/08/dataservices/scheme"/>
                <link href="CarMakers(1L)" rel="edit" title="CarMaker"/>
                <link href="CarMakers(1L)/CarModelDetails" 
                  rel="http://schemas.microsoft.com/ado/2007/08/dataservices/related/CarModelDetails" 
                  title="CarModelDetails" 
                  type="application/atom+xml;type=feed"/>
                <content type="application/xml">
                    <m:properties>
                        <d:Id>1</d:Id>
                        <d:Name>Special Motors</d:Name>
                    </m:properties>
                </content>
            </entry>
        </m:inline>
    </link>
    <content type="application/xml">
        <m:properties>
            <d:Id>1</d:Id>
            <d:Maker>1</d:Maker>
            <d:Name>Muze</d:Name>
            <d:Sku>SM001</d:Sku>
            <d:Year>2018</d:Year>
        </m:properties>
    </content>
</entry>
```

### 5.4.`$select`

我们使用$select 查询选项通知 OData 服务，它应该只返回给定属性的值。这在我们的实体有大量属性，但我们只对其中的一部分感兴趣的场景中很有用。

让我们在一个只返回`Name`和`Sku `属性的查询中使用这个选项:

```java
.../CarModels(1L)?$select=Name,Sku 
```

生成的文档现在只具有请求的属性:

```java
... xml omitted
    <content type="application/xml">
        <m:properties>
            <d:Name>Muze</d:Name>
            <d:Sku>SM001</d:Sku>
        </m:properties>
    </content>
... xml omitted
```

我们还可以看到，甚至相关的实体也被省略了。为了包含它们，我们需要在`$select`选项中包含关系的名称。

### 5.5.`$orderBy`

`$orderBy`选项的工作方式与它的 SQL 对应物非常相似。我们用它来指定**我们希望服务器返回一组给定实体的顺序。**在其更简单的形式中，它的值只是所选实体的属性名称列表，可选地通知订单方向:

```java
.../CarModels?$orderBy=Name asc,Sku desc 
```

这个查询将产生一个按名称和 SKU 排序的列表，分别按升序和降序排列。

这里一个重要的细节是给定属性的方向部分的使用情况:虽然规范要求服务器必须支持关键字`asc` 和`desc`的任何大写和小写字母组合，但是**也要求客户端只能使用小写的** `.`

### 5.6.`$format`

这个选项定义了服务器应该使用的数据表示格式，它优先于任何 HTTP 内容协商头，比如`Accept`。它的值必须是完整的 MIME 类型或特定于格式的缩写形式。

例如，**我们可以用`json `作为`application/json`的缩写:**

```java
.../CarModels?$format=json 
```

这个 URL 指示我们的服务使用 JSON 格式返回数据，而不是像我们之前看到的那样使用 XML。当该选项不存在时，服务器将使用`Accept` 头的值(如果存在)。当两者都不可用时，服务器可以自由选择任何一种表示——通常是 XML 或 JSON。

具体到 JSON，它基本上是无模式的。然而，OData 4.01 也为元数据端点定义了一个 JSON 模式。这意味着我们现在可以编写完全摆脱 XML 处理的客户机，如果它们选择这样做的话。

## 6.结论

在对 OData 的简要介绍中，我们已经介绍了它的基本语义以及如何执行简单的数据集导航。我们的后续文章将从我们离开的地方继续，并直接进入 Olingo 图书馆。然后我们将看到如何使用这个库实现示例服务。

代码示例一如既往地可以在 GitHub 上找到。