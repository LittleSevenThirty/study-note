# Spring @Validated 注解完整学习笔记

> **目标读者**：Java 开发初学者，对 Spring、Bean Validation 等概念尚不熟悉  
> **编写时间**：2025年12月  
> **适用版本**：Spring Framework 5.x / 6.x（与 Spring Boot 2.x / 3.x 兼容）

---

## 一、基础部分：@Validated 是什么？为什么用？怎么用？

### 1.1 核心作用

`@Validated` 是 **Spring 框架提供的一个注解**，用于**激活方法级别或类级别的参数校验功能**。它的本质是告诉 Spring：“请在这个地方帮我自动检查传入的数据是否符合预设的规则”。

> 📌 注意：`@Validated` **本身不定义校验规则**，它只是“触发器”。真正的校验规则是由 **Bean Validation 规范**（如 JSR-303、JSR-380）中的注解（如 `@NotNull`、`@Size` 等）定义的。

---

### 1.2 背景知识简要说明（零基础友好）

#### ✅ 什么是 Bean Validation？
- 这是一个 **Java 标准规范**（不是 Spring 特有的），用于对 Java 对象的字段进行**数据合法性校验**。
- 最常见的实现库是 **Hibernate Validator**（即使你不用 Hibernate ORM，也可以单独使用它来做校验）。
- 常见校验注解包括：
  - `@NotNull`：不能为 null
  - `@NotBlank`：字符串非空且非空白（trim 后长度 > 0）
  - `@Size(min=2, max=10)`：长度在 2 到 10 之间
  - `@Email`：必须是合法邮箱格式
  - `@Min(18)`：数值 ≥ 18

#### ✅ Spring 和 Bean Validation 的关系？
- Spring **集成了** Bean Validation。
- 但默认情况下，Spring **只在校验 Controller 的请求参数时自动生效**（通过 `@Valid` 或 `@Validated` + `@RequestBody`）。
- 如果你想在 **Service 层、普通方法参数、配置类等地方也做校验**，就需要显式使用 `@Validated` 并配合 Spring AOP。

---

### 1️⃣3 典型使用场景

| 场景 | 是否需要 `@Validated` | 说明 |
|------|------------------------|------|
| **Controller 中校验 `@RequestBody` 对象** | 可选（可用 `@Valid`） | `@Valid` 是标准注解，`@Validated` 是 Spring 扩展版 |
| **Controller 中校验路径变量/请求参数（如 `@PathVariable`, `@RequestParam`）** | **必须用 `@Validated`** | `@Valid` 不支持这种场景 |
| **Service 方法参数校验** | **必须用 `@Validated`** | 需要开启 Spring AOP 代理 |
| **配置类（`@ConfigurationProperties`）属性校验** | **必须用 `@Validated`** | 用于校验 application.yml 中的配置值 |

---

### 1️⃣4 基本用法示例

#### 示例 1：Controller 中校验请求参数（`@RequestParam`）

```java
@RestController
@Validated // ← 类上加 @Validated，才能校验方法参数
public class UserController {

    // 校验单个参数
    @GetMapping("/user")
    public String getUser(@RequestParam @NotBlank String name,
                          @RequestParam @Min(1) Integer age) {
        return "Name: " + name + ", Age: " + age;
    }
}
```

> ❗如果没有类上的 `@Validated`，`@NotBlank` 和 `@Min` **不会生效**！

---

#### 示例 2：Service 层方法参数校验

```java
@Service
@Validated // ← 必须加在类上！
public class UserService {

    // 校验传入的对象
    public void createUser(@Valid User user) {
        // 业务逻辑
    }

    // 校验单个参数
    public void updateUserEmail(@NotBlank String email, @Positive Long userId) {
        // 业务逻辑
    }
}

// User.java
public class User {
    @NotBlank
    private String username;
    
    @Email
    private String email;
    
    // getters/setters...
}
```

> ⚠️ 注意：Service 层校验依赖 **Spring AOP 代理机制**。这意味着：
> - 方法必须是 **public**
> - 不能在同一个类中调用（如 `this.createUser(...)` 会失效）
> - Spring 容器必须管理该 Bean（即有 `@Service`、`@Component` 等）

---

#### 示例 3：配置类属性校验

```java
@ConfigurationProperties(prefix = "app.user")
@Validated // ← 必须加！
@Component
public class UserConfig {

    @NotBlank
    private String defaultRole;

    @Min(1)
    @Max(100)
    private int maxRetry;

    // getters/setters...
}
```

对应 `application.yml`：
```yaml
app:
  user:
    default-role: admin
    max-retry: 5
```
如果 `max-retry: 200`，启动时会报错！

---

## 二、进阶部分：@Validated 的所有属性详解

`@Validated` 注解定义如下（来自 Spring 源码）：

```java
@Target({ElementType.TYPE, ElementType.METHOD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Validated {
    Class<?>[] value() default {};
}
```

### 2.1 唯一属性：`value`

虽然看起来只有一个属性 `value()`，但它非常强大——用于支持 **分组校验（Validation Groups）**。

#### 🔍 什么是分组校验？
- 同一个对象，在不同场景下需要不同的校验规则。
- 例如：用户注册时需要校验密码，但更新资料时不需要。

#### ✅ 使用步骤：

**Step 1：定义分组接口（标记接口）**

```java
public interface CreateGroup {}
public interface UpdateGroup {}
```

**Step 2：在实体类字段上指定分组**

```java
public class User {
    @NotNull(groups = UpdateGroup.class)
    private Long id; // 更新时必须有 ID

    @NotBlank(groups = {CreateGroup.class, UpdateGroup.class})
    private String username;

    @NotBlank(groups = CreateGroup.class) // 只在创建时校验密码
    private String password;
}
```

**Step 3：在方法上使用 `@Validated` 指定分组**

```java
@Service
@Validated
public class UserService {

    // 创建用户 → 使用 CreateGroup
    public void createUser(@Validated(CreateGroup.class) User user) {
        // ...
    }

    // 更新用户 → 使用 UpdateGroup
    public void updateUser(@Validated(UpdateGroup.class) User user) {
        // ...
    }
}
```

> 💡 `@Validated` 的 `value` 属性就是用来传入这些分组接口的！
> - `@Validated(CreateGroup.class)` 等价于 `@Validated(value = CreateGroup.class)`
> - 可以传多个：`@Validated({CreateGroup.class, AdminGroup.class})`

#### 🔄 默认分组（Default Group）
- 如果字段没有指定 `groups`，则属于 **默认分组 `javax.validation.groups.Default`**。
- 当你使用 `@Valid` 或 `@Validated` **不带 value** 时，只校验默认分组的字段。
- 如果你指定了分组（如 `@Validated(CreateGroup.class)`），**默认分组不会被校验**！
  - 解决方案：把 `Default.class` 也加进去：
    ```java
    @Validated({CreateGroup.class, Default.class})
    ```

---

### 2.2 @Validated vs @Valid 的区别（重要！）

| 特性 | `@Valid`（标准 JSR） | `@Validated`（Spring 扩展） |
|------|----------------------|----------------------------|
| 来源 | Bean Validation 规范 | Spring Framework |
| 支持分组 | ✅ | ✅ |
| 可用于方法参数（如 `@RequestParam`） | ❌ | ✅ |
| 可用于类级别激活校验 | ❌ | ✅（必须加在类上） |
| 支持嵌套校验（级联） | ✅（用 `@Valid`） | ✅（需配合 `@Valid`） |
| Spring AOP 集成 | 有限 | 完整支持 |

> ✅ **最佳实践**：
> - Controller 中校验 `@RequestBody`：两者皆可，推荐 `@Valid`
> - 校验简单参数（`@RequestParam`/`@PathVariable`）或 Service 层：**必须用 `@Validated`**
> - 配置类：**必须用 `@Validated`**

---

### 2.3 嵌套对象校验（级联校验）

```java
public class Order {
    @Valid // ← 关键！告诉校验器要深入校验 address
    private Address address;
}

public class Address {
    @NotBlank
    private String street;
}
```

在 Service 中：

```java
@Service
@Validated
public class OrderService {
    public void placeOrder(@Valid Order order) { } // 自动校验 address.street
}
```

> 🔑 注意：嵌套校验靠的是 `@Valid`（标准注解），不是 `@Validated`！

---

## 三、常见问题与注意事项

### ❓ 为什么加了 `@Validated` 还是不生效？

1. **没加在类上**：`@Validated` 必须加在 **类级别** 才能激活方法参数校验（尤其是 Controller 和 Service）。
2. **方法不是 public**：AOP 代理要求方法是 public。
3. **自己调用自己**：如 `this.method()`，绕过了代理。
4. **缺少校验实现库**：确保项目中有 `hibernate-validator`（Spring Boot Web 默认包含）。

### ❓ 如何自定义校验规则？

可以创建自定义约束注解（如 `@Phone`），但这属于 Bean Validation 范畴，不在 `@Validated` 范围内。

### ❓ 校验失败时抛什么异常？

- Controller 层：`MethodArgumentNotValidException` 或 `ConstraintViolationException`
- Service 层：`ConstraintViolationException`
- 可通过 `@ControllerAdvice` 全局处理。

---

## 四、总结速查表

| 问题 | 答案 |
|------|------|
| `@Validated` 作用？ | 激活 Spring 中的方法/参数级数据校验 |
| 必须和谁一起用？ | Bean Validation 注解（如 `@NotNull`） |
| 加在哪儿？ | **类上**（Controller / Service / ConfigurationProperties） |
| 能校验 `@RequestParam` 吗？ | ✅ 可以（`@Valid` 不行） |
| 支持分组吗？ | ✅ 用 `value = Group.class` |
| 和 `@Valid` 区别？ | `@Validated` 是 Spring 增强版，支持更多场景 |
| 嵌套对象怎么校验？ | 字段上加 `@Valid`，方法参数用 `@Valid` 或 `@Validated` |

---

> ✅ **记住一句话**：  
> **“想在 Spring 里校验方法参数（尤其是简单类型或 Service 层），就给类加上 `@Validated`！”**

--- 

**附：Maven 依赖（通常无需手动添加）**

```xml
<!-- Spring Boot Web 已包含 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```