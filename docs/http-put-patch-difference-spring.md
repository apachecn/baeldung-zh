# REST API 中的 HTTP PUT 与 HTTP PATCH

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/http-put-patch-difference-spring>

## 1。概述

在这个快速教程中，我们将看看 [HTTP PUT 和 PATCH 动词](/web/20220627185008/https://www.baeldung.com/cs/http-put-vs-patch)之间的区别，以及这两种操作的语义。

我们将使用 Spring 来实现支持这两种类型操作的两个 REST 端点，以便更好地理解它们的区别和正确的使用方法。

## 2。什么时候用 Put，什么时候打补丁？

让我们从简单和稍微简单的陈述开始。

当客户需要完全替换现有资源时，他们可以使用 PUT。当他们进行部分更新时，他们可以使用 HTTP 补丁。

例如，当更新资源的单个字段时，发送完整的资源表示可能很麻烦，并且会使用大量不必要的带宽。在这种情况下，补丁的语义更有意义。

这里要考虑的另一个重要方面是**幂等性。PUT 是幂等的；补丁可以是幂等的，但不是必须的。**因此，根据我们正在实现的操作的语义，我们也可以基于这个特征选择一个或另一个。

## 3。实施上传和补丁逻辑

假设我们想要实现 REST API 来更新带有多个字段的`HeavyResource` :

```java
public class HeavyResource {
    private Integer id;
    private String name;
    private String address;
    // ...
```

首先，我们需要使用 PUT 创建处理资源完全更新的端点:

```java
@PutMapping("/heavyresource/{id}")
public ResponseEntity<?> saveResource(@RequestBody HeavyResource heavyResource,
  @PathVariable("id") String id) {
    heavyResourceRepository.save(heavyResource, id);
    return ResponseEntity.ok("resource saved");
}
```

这是更新资源的标准端点。

现在让我们假设地址字段经常被客户端更新。在这种情况下，**我们不希望发送包含所有字段**的整个`HeavyResource` 对象，但是我们希望能够通过补丁方法只更新`address`字段。

我们可以创建一个`HeavyResourceAddressOnly` DTO 来表示地址字段的部分更新:

```java
public class HeavyResourceAddressOnly {
    private Integer id;
    private String address;

    // ...
}
```

接下来，我们可以利用修补方法发送部分更新:

```java
@PatchMapping("/heavyresource/{id}")
public ResponseEntity<?> partialUpdateName(
  @RequestBody HeavyResourceAddressOnly partialUpdate, @PathVariable("id") String id) {

    heavyResourceRepository.save(partialUpdate, id);
    return ResponseEntity.ok("resource address updated");
}
```

有了这个更细粒度的 DTO，我们可以只发送我们需要更新的字段，而没有发送整个`HeavyResource`的开销。

如果我们有大量这样的部分更新操作，我们也可以跳过为每个 out 创建自定义 DTO，而只使用地图:

```java
@RequestMapping(value = "/heavyresource/{id}", method = RequestMethod.PATCH, consumes = MediaType.APPLICATION_JSON_VALUE)
public ResponseEntity<?> partialUpdateGeneric(
  @RequestBody Map<String, Object> updates,
  @PathVariable("id") String id) {

    heavyResourceRepository.save(updates, id);
    return ResponseEntity.ok("resource updated");
}
```

这个解决方案将在实现 API 方面给予我们更多的灵活性，但是我们也确实失去了一些东西，比如验证。

## 4。测试上传和补丁

最后，让我们为这两种 HTTP 方法编写测试。

首先，我们想通过 PUT 方法测试完整资源的更新:

```java
mockMvc.perform(put("/heavyresource/1")
  .contentType(MediaType.APPLICATION_JSON_VALUE)
  .content(objectMapper.writeValueAsString(
    new HeavyResource(1, "Tom", "Jackson", 12, "heaven street")))
  ).andExpect(status().isOk());
```

部分更新的执行是通过使用修补方法实现的:

```java
mockMvc.perform(patch("/heavyrecource/1")
  .contentType(MediaType.APPLICATION_JSON_VALUE)
  .content(objectMapper.writeValueAsString(
    new HeavyResourceAddressOnly(1, "5th avenue")))
  ).andExpect(status().isOk());
```

我们还可以为更通用的方法编写一个测试:

```java
HashMap<String, Object> updates = new HashMap<>();
updates.put("address", "5th avenue");

mockMvc.perform(patch("/heavyresource/1")
    .contentType(MediaType.APPLICATION_JSON_VALUE)
    .content(objectMapper.writeValueAsString(updates))
  ).andExpect(status().isOk()); 
```

## 5。处理具有`Null`值的部分请求

当我们编写一个补丁方法的实现时，我们需要指定一个契约，说明当我们将`null`作为`HeavyResourceAddressOnly`中`address` 字段的值时，如何处理案例。

假设客户端发送以下请求:

```java
{
   "id" : 1,
   "address" : null
}
```

然后，我们可以将`address`字段的值设置为`null`，或者通过将其视为无变化来忽略这样的请求。

我们应该选择一种处理`null` 的策略，并在每个补丁方法实现中坚持使用它。

## 6。结论

在这篇简短的文章中，我们重点了解了 HTTP PATCH 和 PUT 方法之间的区别。

我们实现了一个简单的 Spring REST 控制器，通过 PUT 方法更新资源，并使用 PATCH 进行部分更新。

所有这些例子和代码片段的实现都可以在 [GitHub 项目](https://web.archive.org/web/20220627185008/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-resttemplate)中找到。这是一个 Maven 项目，因此应该很容易导入和运行。