# Spring 中的多部分请求处理

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/sprint-boot-multipart-requests>

## 1.介绍

在本教程中，我们将重点介绍在 Spring Boot 中发送多方请求的各种机制。多部分请求包括发送由边界分隔的许多不同类型的数据，作为单个 HTTP 方法调用的一部分。

通常，我们可以发送复杂的 JSON、XML 或 CSV 数据，也可以在这个请求中传输多部分文件。多部分文件的示例包括音频或图像文件。此外，我们可以将带有多部分文件的简单键/值对数据作为多部分请求发送。

现在，让我们来看看发送这些数据的各种方法。

## 2.使用`@ModelAttribute`

让我们考虑一个简单的用例，使用一个表单发送由姓名和文件组成的雇员数据。

首先，我们将创建一个`Employee`抽象来存储表单数据:

```
public class Employee {
    private String name;
    private MultipartFile document;
}
```

然后我们将使用[百里香叶](/web/20221122060810/https://www.baeldung.com/thymeleaf-in-spring-mvc)生成表单:

```
<form action="#" th:action="@{/employee}" th:object="${employee}" method="post" enctype="multipart/form-data">
    <p>name: <input type="text" th:field="*{name}" /></p>
    <p>document:<input type="file" th:field="*{document}" multiple="multiple"/>
    <input type="submit" value="upload" />
    <input type="reset" value="Reset" /></p>
</form>
```

需要注意的重要一点是，我们在视图中将`enctype`声明为`multipart/form-data`。

最后，我们将创建一个接受表单数据的方法，包括多部分文件:

```
@RequestMapping(path = "/employee", method = POST, consumes = { MediaType.MULTIPART_FORM_DATA_VALUE })
public String saveEmployee(@ModelAttribute Employee employee) {
    employeeService.save(employee);
    return "employee/success";
}
```

这里，两个特别重要的细节是:

*   `consumes`属性值设置为`multipart/form-data`
*   **T0 已经将所有的表单数据**抓取到`Employee` POJO 中，包括上传的文件

## 3.使用`@RequestPart`

这个注释**将多部分请求的一部分与方法参数**相关联，这对于将复杂的多属性数据作为有效载荷发送是有用的，例如 JSON 或 XML。

让我们创建一个有两个参数的方法，第一个是类型`Employee`，第二个是`MultipartFile`。此外，我们将用`@RequestPart`来注释这两个论点:

```
@RequestMapping(path = "/requestpart/employee", method = POST, consumes = { MediaType.MULTIPART_FORM_DATA_VALUE })
public ResponseEntity<Object> saveEmployee(@RequestPart Employee employee, @RequestPart MultipartFile document) {
    employee.setDocument(document);
    employeeService.save(employee);
    return ResponseEntity.ok().build();
}
```

现在，为了查看这个注释的运行情况，我们将使用`MockMultipartFile`创建测试:

```
@Test
public void givenEmployeeJsonAndMultipartFile_whenPostWithRequestPart_thenReturnsOK() throws Exception {
    MockMultipartFile employeeJson = new MockMultipartFile("employee", null,
      "application/json", "{\"name\": \"Emp Name\"}".getBytes());

    mockMvc.perform(multipart("/requestpart/employee")
      .file(A_FILE)
      .file(employeeJson))
      .andExpect(status().isOk());
}
```

上面需要注意的重要一点是，我们已经将`Employee`部分的内容类型设置为`application/JSON`。除了多部分文件之外，我们还将这些数据作为 JSON 文件发送。

关于如何测试多部分请求的更多细节可以在[这里](/web/20221122060810/https://www.baeldung.com/spring-multipart-post-request-test#testing-a-multipart-post-request)找到。

## 4.使用`@RequestParam`

发送多部分数据的另一种方式是使用`@RequestParam`。这对于简单的数据尤其有用，这些数据作为键/值对与文件一起发送:

```
@RequestMapping(path = "/requestparam/employee", method = POST, consumes = { MediaType.MULTIPART_FORM_DATA_VALUE })
public ResponseEntity<Object> saveEmployee(@RequestParam String name, @RequestPart MultipartFile document) {
    Employee employee = new Employee(name, document);
    employeeService.save(employee);
    return ResponseEntity.ok().build();
}
```

让我们为这个方法编写测试来演示:

```
@Test
public void givenRequestPartAndRequestParam_whenPost_thenReturns200OK() throws Exception {
    mockMvc.perform(multipart("/requestparam/employee")
      .file(A_FILE)
      .param("name", "testname"))
      .andExpect(status().isOk());
}
```

## 5.结论

在本文中，我们学习了如何在 Spring Boot 中有效地处理多部分请求。

最初，我们使用模型属性发送多部分表单数据。然后我们看了如何使用`@RequestPart`和`@RequestParam`注释分别接收多部分数据。

和往常一样，完整的源代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221122060810/https://github.com/eugenp/tutorials/tree/master/spring-web-modules/spring-mvc-forms-thymeleaf)