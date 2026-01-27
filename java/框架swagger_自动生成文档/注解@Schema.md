# **注解@Schema**
## 一句话记住 @Schema

> **`@Schema` 是你在 Java 代码中“写给 Swagger 看的注释”，用于精准控制 API 文档中数据模型的 


## 二、@Schema 注解是什么？

- **全限定名**：`io.swagger.v3.oas.annotations.media.Schema`
- **作用**：用于描述 **数据模型**（如 POJO 类、字段、方法参数、返回值等）在 OpenAPI 文档中的表现形式。
- **本质**：它是 OpenAPI 规范中 **Schema Object** 的 Java 注解映射。

> ✅ 简单说：`@Schema` 告诉 Swagger：“这个类/字段在 API 文档里应该怎么展示”。

---

## 三、核心用途与典型场景

### 1. 描述类（Model）的整体信息
```java
@Schema(description = "用户信息模型", name = "User")
public class User {
    // ...
}
```
- 生成文档时，该类会被标注为 “用户信息模型”，并在模型列表中显示为 `User`。

### 2. 描述字段的含义、格式、约束等
```java
public class User {
    @Schema(description = "用户唯一ID", example = "1001", requiredMode = RequiredMode.REQUIRED)
    private Long id;

    @Schema(description = "用户名", maxLength = 50, nullable = false)
    private String username;

    @Schema(description = "邮箱地址", format = "email", pattern = "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$")
    private String email;
}
```
- `example`：提供示例值，方便前端/测试理解；
- `requiredMode`：标记是否必填（对应 OpenAPI 的 `required` 字段）；
- `maxLength` / `pattern`：定义字符串校验规则；
- `format = "email"`：提示该字段是邮箱格式（Swagger UI 可能高亮或校验）；
- `nullable = false`：表示该字段不允许为 null（注意：与 `@NotNull` 不同，这是文档层面的说明）。

### 3. 控制字段是否在文档中显示
```java
@Schema(hidden = true)
private String internalToken; // 内部字段，不暴露给 API 文档
```

### 4. 用于方法参数或返回值（较少见，通常用 `@ApiResponse` 配合）
```java
@GetMapping("/user")
@Operation(summary = "获取用户")
@ApiResponse(responseCode = "200", description = "成功返回用户", 
             content = @Content(schema = @Schema(implementation = User.class)))
public User getUser() { ... }
```

---

## 四、与其他注解的关系

| 注解 | 所属包 | 用途 |
|------|--------|------|
| `@Schema` | `io.swagger.v3.oas.annotations.media` | 描述数据模型（类、字段） |
| `@Operation` | `io.swagger.v3.oas.annotations` | 描述一个 API 操作（即一个 HTTP 方法） |
| `@ApiResponse` | `io.swagger.v3.oas.annotations.responses` | 描述某个响应状态码的返回结构 |
| `@Parameter` | `io.swagger.v3.oas.annotations.parameters` | 描述路径/查询/请求体参数 |

> ⚠️ 注意：不要与旧版 Swagger 2 的 `io.swagger.annotations.ApiModel` / `@ApiModelProperty` 混淆。`@Schema` 属于 **OpenAPI 3.0+**（Swagger 3）。

---

## 五、最佳实践建议

1. **优先使用 `@Schema` 而非旧版注解**（如项目使用 Springdoc OpenAPI 1.6+）；
2. **为关键字段添加 `description` 和 `example`**，极大提升文档可读性；
3. **避免过度注解**：如果字段名已足够清晰（如 `username`），可省略描述；
4. **结合 JSR-380 校验注解**（如 `@NotBlank`, `@Email`）使用，但注意：
   - 校验注解用于运行时验证；
   - `@Schema` 仅用于文档生成，两者互补。

---