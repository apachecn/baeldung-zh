# 使用 Swagger Codegen 进行自定义验证

> 原文：<https://web.archive.org/web/20220930061024/https://www.baeldung.com/java-swagger-custom-validation>

## 1.概观

当我们想用 Swagger 生成验证时，我们通常使用[基本规范](https://web.archive.org/web/20221030081812/https://swagger.io/docs/specification/describing-parameters/)。然而，我们可能需要添加 Spring [定制验证注释](/web/20221030081812/https://www.baeldung.com/spring-mvc-custom-validator)。

本教程将介绍如何使用这些验证来生成模型和 REST APIs，同时重点关注 OpenAPI 服务器生成器，而不是约束验证器。

## 2.设置

对于设置，我们将使用之前的 Baeldung 教程[从 OpenAPI 3.0.0 定义](/web/20221030081812/https://www.baeldung.com/java-openapi-generator-server)生成服务器。接下来，我们将添加一些[定制验证注释](/web/20221030081812/https://www.baeldung.com/spring-mvc-custom-validator)以及所有需要的依赖项。

## 3.PetStore API OpenAPI 定义

假设我们有 PetStore API OpenAPI 定义，我们需要为 REST API 和所描述的模型添加自定义验证。

### 3.1.API 模型的自定义验证

为了创建宠物，我们需要让 Swagger 使用我们的自定义验证注释来测试宠物的名字是否大写。因此，为了这个教程，我们就称它为`Capitalized`。

因此，请遵守以下示例中的`x-constraints`规范。这足以让 Swagger 知道我们需要生成另一种不同于已知类型的注释:

```java
openapi: 3.0.1
info:
  version: "1.0"
  title: PetStore
paths:
  /pets:
    post:
      #.. post described here
components:
  schemas:
    Pet:
      type: object
      required:
        - id
        - name
      properties:
        id:
          type: integer
          format: int64
        name:
          type: string
          x-constraints: "Capitalized(required = true)"
        tag:
          type: string
```

### 3.2.REST API 端点的定制验证

如上所述，我们将描述一个端点，以同样的方式按名称查找所有宠物。为了演示我们的目的，让我们假设我们的系统是区分大小写的，因此我们将为输入参数`name` 再次添加相同的`x-constraints` 验证:

```java
/pets:
    # post defined here
    get: 
      tags: 
        - pet 
      summary: Finds Pets by name 
      description: 'Find pets by name' 
      operationId: findPetsByTags 
      parameters: 
        - <em>name: name</em> 
          in: query 
          schema:
            type: string 
          description: Tags to filter by 
          required: true 
          x-constraints: "Capitalized(required = true)" 
      responses: 
        '200':
          description: default response
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Pet'
        '400': 
          description: Invalid tag value
```

## 4.创建`Capitalized` 注释

为了实施自定义验证，我们需要创建一个注释来保证功能性。

首先，我们制作注释接口—`@Capitalized`:

```java
@Documented
@Constraint(validatedBy = {Capitalized.class})
@Target({ElementType.PARAMETER, ElementType.FIELD, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface Capitalized{
    String message() default "Name should be capitalized.";
    boolean required() default true;
    // default annotation methods
}
```

注意，我们制作`required` 方法是为了演示——我们将在后面解释。

接下来，我们添加上面的`@Constraint` 注释中提到的`CapitalizedValidator`:

```java
public class CapitalizedValidator implements ConstraintValidator<Capitalized, String> {

    @Override
    public boolean isValid(String nameField, ConstraintValidatorContext context) {
        // validation code here
    }
}
```

## 5.生成验证注释

### 5.1.指定 Mustache 模板目录

为了生成带有`@Capitalized`验证注释的模型，我们需要特定的 mustache 模板告诉 Swagger 在模型中生成它。

因此，在 OpenAPI 生成器插件中，在`<configuration>[..]</configuration` 标签内，我们需要添加一个模板目录:

```java
<plugin>  
  //... 
  <executions>
    <execution>
      <configuration
        //...
        <templateDirectory>
          ${project.basedir}/src/main/resources/openapi/templates
        </templateDirectory>
        //...
      </configuration>
    </execution>
  </executions>        
  //...
</plugin> 
```

### 5.2.添加 Mustache Bean 验证配置

在本章中，我们将配置 Mustache 模板来生成验证规范。为了添加更多的细节，我们将修改`[beanValidationCore.mustache](https://web.archive.org/web/20221030081812/https://raw.githubusercontent.com/swagger-api/swagger-codegen/master/modules/swagger-codegen/src/main/resources/Java/beanValidationCore.mustache)`、 `[model.mustache](https://web.archive.org/web/20221030081812/https://github.com/swagger-api/swagger-codegen/blob/master/modules/swagger-codegen/src/main/resources/Java/model.mustache)`和`[api.muctache](https://web.archive.org/web/20221030081812/https://github.com/swagger-api/swagger-codegen/blob/master/modules/swagger-codegen/src/main/resources/Java/api.mustache)`文件来成功地生成代码。

首先，`swagger-codegen `模块的`[beanValidationCore.mustache](https://web.archive.org/web/20221030081812/https://raw.githubusercontent.com/swagger-api/swagger-codegen/master/modules/swagger-codegen/src/main/resources/Java/beanValidationCore.mustache)`需要修改，增加一个厂商扩展规范:

```java
{{{ vendorExtensions.x-constraints }}}
```

其次，如果我们有一个带有类似于`@Capitalized(required = “true”)`的内部属性的注释，那么需要在`[beanValidationCore.mustache](https://web.archive.org/web/20221030081812/https://raw.githubusercontent.com/swagger-api/swagger-codegen/master/modules/swagger-codegen/src/main/resources/Java/beanValidationCore.mustache)`文件的第二行指定一个特定的模式:

```java
{{#required}}@Capitalized(required="{{{pattern}}}") {{/required}}
```

第三，我们需要修改`[model.mustache](https://web.archive.org/web/20221030081812/https://github.com/swagger-api/swagger-codegen/blob/master/modules/swagger-codegen/src/main/resources/Java/model.mustache)`规范来包含必要的导入。例如，我们将导入 `@Capitalized`注释和`Capitalized` `.` ，这些导入应该被插入到 [`model.mustache` :](https://web.archive.org/web/20221030081812/https://github.com/swagger-api/swagger-codegen/blob/master/modules/swagger-codegen/src/main/resources/Java/model.mustache) 的`package`标签之后

```java
{{#imports}}import {{import}}; {{/imports}} import 
com.baeldung.openapi.petstore.validator.CapitalizedValidator; 
import com.baeldung.openapi.petstore.validator.Capitalized;
```

最后，为了在 API 中生成注释，我们需要为`[api.mustache](https://web.archive.org/web/20221030081812/https://github.com/swagger-api/swagger-codegen/blob/master/modules/swagger-codegen/src/main/resources/Java/api.mustache)`文件中的`@Capitalized` 注释添加导入。

```java
{{#imports}}import {{import}}; {{/imports}} import 
com.baeldung.openapi.petstore.validator.Capitalized;
```

另外，`[api.mustache](https://web.archive.org/web/20221030081812/https://github.com/swagger-api/swagger-codegen/blob/master/modules/swagger-codegen/src/main/resources/Java/api.mustache)`依赖于`[cookieParams.mustache](https://web.archive.org/web/20221030081812/https://github.com/OpenAPITools/openapi-generator/blob/master/modules/openapi-generator/src/main/resources/JavaSpring/cookieParams.mustache)`文件。因此，我们需要将它添加到`openapi/templates`目录中。

## 6.生成来源

最后，我们可以使用生成的代码。我们至少需要运行。这将生成模型:

```java
public class Pet {
    @JsonProperty("id")
    private Long id = null;

    @JsonProperty("name")
    private String name = null;
    // other parameters
    @Schema(required = true, description = "")
    @<code class="language-java">Capitalized public String getName() { return name; } // default getters and setter }
```

它还会生成一个 API:

```java
default ResponseEntity<List<Pet>> findPetsByTags(
    @Capitalized(required = true)
    @ApiParam(value = "Tags to filter by") 
    @Valid @RequestParam(value = "name", required = false) String name) {

    // default generated code here 
    return new ResponseEntity<>(HttpStatus.NOT_IMPLEMENTED);
}
```

## 7.使用`curl`进行测试

启动应用程序后，我们将运行一些`curl`命令来测试它。

此外，请注意，违反约束确实会抛出一个`ConstraintViolationException.` 异常，需要通过 [`@ControllerAdvice`](/web/20221030081812/https://www.baeldung.com/exception-handling-for-rest-with-spring) 进行适当处理，以返回一个 400 错误请求状态。

### 7.1.测试`Pet`模型验证

这个`Pet`型号有个小写的`name`。因此，应用程序应该返回 400 错误请求:

```java
curl -X 'POST' \
  'http://localhost:8080/pet' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "id": 1,
  "name": "rockie"
}'
```

### 7.2.测试 Find `Pet` API

与上面的方式相同，因为`name `是小写的，所以应用程序也应该返回一个 400 错误请求:

```java
curl -I http://localhost:8080/pets/name="rockie"
```

## 8.结论

在本教程中，我们看到了在实现 REST API 服务器时，如何使用 Spring 生成自定义约束验证器。

和往常一样，代码可以在 GitHub 上找到[。](https://web.archive.org/web/20221030081812/https://github.com/eugenp/tutorials/tree/master/spring-swagger-codegen/custom-validations-opeanpi-codegen)