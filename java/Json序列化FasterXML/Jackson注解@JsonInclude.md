# 《@JsonInclude 注解学习笔记》  
**——面向完全初学者的 Jackson 序列化控制指南**

---

## 一、背景知识：什么是 JSON 序列化？

在 Java 开发中，我们经常需要将 Java 对象（比如一个 `User` 类的实例）转换成 **JSON 字符串**，以便通过网络发送给前端、存入数据库或写入文件。这个“对象 → JSON”的过程叫做 **序列化（Serialization）**。

例如：

```java
public class User {
    public String name = "Alice";
    public Integer age = null;
}
```

默认情况下，Jackson 会把上面的对象序列化为：

```json
{"name":"Alice","age":null}
```

但很多时候，我们**不希望输出 `null`、空字符串 `""` 或空列表 `[]`**，因为：
- 前端可能不需要这些无意义的字段；
- 减少数据体积，节省带宽；
- 避免混淆（`null` 和“未提供”含义不同）。

这时候就需要 **控制哪些字段应该出现在最终的 JSON 中** —— 这就是 `@JsonInclude` 的核心作用。

---

## 二、@JsonInclude 是什么？能做什么？

`@JsonInclude` 是 Jackson 提供的一个 **注解（Annotation）**，用于**控制在序列化时，哪些字段/属性会被包含进最终的 JSON 输出中**。

> ✅ 简单说：它决定“**什么值值得被写进 JSON**”。

你可以用它来：
- 忽略 `null` 值；
- 忽略空字符串（`""`）或空集合（如 `List` 为空）；
- 忽略等于“默认值”的字段（比如 `int` 默认是 0）；
- 更精细地控制嵌套结构中的内容（比如只忽略 `Map` 中值为 `null` 的条目）。

---

## 三、基础用法示例

### 1. 忽略所有 `null` 字段

```java
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.databind.ObjectMapper;

@JsonInclude(JsonInclude.Include.NON_NULL)
public class User {
    public String name = "Alice";
    public String email = null;
    public Integer age = 30;
}

// 使用 ObjectMapper 序列化
ObjectMapper mapper = new ObjectMapper();
User user = new User();
String json = mapper.writeValueAsString(user);
System.out.println(json);
```

**输出：**
```json
{"name":"Alice","age":30}
```
→ `email` 字段因为是 `null`，被自动忽略了。

---

### 2. 忽略 `null`、空字符串、空集合等（更严格）

```java
@JsonInclude(JsonInclude.Include.NON_EMPTY)
public class Product {
    public String name = "";          // 空字符串
    public List<String> tags = new ArrayList<>(); // 空列表
    public String description = "Great!";
}
```

**输出：**
```json
{"description":"Great!"}
```
→ `name`（空字符串）和 `tags`（空列表）都被忽略。

---

## 四、@JsonInclude 的参数详解

`@JsonInclude` 主要有两个属性：`value()` 和 `content()`。

### 1. `value()`：控制**字段本身的值**是否被包含

这是最常用的属性，类型为 `JsonInclude.Include` 枚举。可选值如下：

| 枚举值 | 含义 | 被忽略的值示例 |
|--------|------|----------------|
| `ALWAYS` | **总是包含**（默认行为） | 无（全部输出） |
| `NON_NULL` | **非 null 才包含** | `null` |
| `NON_ABSENT` | **非“缺席”才包含** | `Optional.empty()`, `null`（对引用类型） |
| `NON_EMPTY` | **非空才包含** | `null`, `""`, `[]`, `{}`, `Collections.EMPTY_LIST` 等 |
| `NON_DEFAULT` | **非默认值才包含** | 基本类型默认值（如 `int=0`, `boolean=false`），对象引用为 `null` |
| `CUSTOM` | 自定义逻辑（高级用法，需配合 `valueFilter`） | — |
| `USE_DEFAULTS` | 使用全局配置（一般不用） | — |

#### 📌 关键区别说明：

- **`NON_NULL` vs `NON_EMPTY`**：
  - `NON_NULL` 只忽略 `null`；
  - `NON_EMPTY` 还会忽略“逻辑上为空”的值，比如：
    - 字符串 `""`
    - 空数组 `new int[0]`
    - 空集合 `new ArrayList<>()`
    - 空 Map `new HashMap<>()`

- **`NON_DEFAULT` 示例**：
  ```java
  @JsonInclude(JsonInclude.Include.NON_DEFAULT)
  public class Config {
      public int port = 8080;     // 默认值 0，但设为 8080 → 会输出
      public boolean enabled = false; // 默认就是 false → 不输出！
      public String host = null;  // 引用类型默认 null → 不输出
  }
  ```
  如果 `enabled` 是 `true`，则会输出；如果是 `false`（默认值），则忽略。

- **`NON_ABSENT`**：
  主要用于 `Optional<T>`、`AtomicReference` 等“容器类型”。例如：
  ```java
  public Optional<String> nickname = Optional.empty(); // 会被忽略
  public Optional<String> realName = Optional.of("Bob"); // 会输出 "realName":"Bob"
  ```

---

### 2. `content()`：控制**容器内部元素**的包含规则（仅当字段是集合/Map 时有效）

当你有一个 `List`、`Map`、数组等字段时，`value()` 控制整个字段是否出现，而 `content()` 控制**里面的元素**是否要过滤。

#### 示例：忽略 Map 中值为 `null` 的条目

```java
@JsonInclude(value = JsonInclude.Include.NON_EMPTY, content = JsonInclude.Include.NON_NULL)
public class Data {
    public Map<String, String> metadata = new HashMap<>();
}

// 初始化
Data d = new Data();
d.metadata.put("version", "1.0");
d.metadata.put("author", null);
d.metadata.put("license", "");
```

**输出：**
```json
{"metadata":{"version":"1.0","license":""}}
```
- 整个 `metadata` 字段是非空的（有 3 个 key），所以字段保留（由 `value = NON_EMPTY` 决定）；
- 但 `author: null` 被移除（由 `content = NON_NULL` 决定）；
- `license: ""` 保留，因为 `content` 只管 `null`，不管空字符串。

> 💡 注意：`content` 只对 **容器类型**（Collection、Map、数组）生效，对普通字段无效。

---

## 五、作用范围与优先级

### 1. 可以加在哪里？

- **类级别（推荐）**：影响该类所有字段。
  ```java
  @JsonInclude(JsonInclude.Include.NON_NULL)
  public class User { ... }
  ```

- **字段/方法级别**：只影响当前字段或 getter 方法。
  ```java
  public class User {
      public String name = "Alice";

      @JsonInclude(JsonInclude.Include.NON_EMPTY)
      public String bio = "";
  }
  ```

- **getter 方法上**（如果你用 getter 控制序列化）：
  ```java
  @JsonInclude(JsonInclude.Include.NON_NULL)
  public String getEmail() { return email; }
  ```

### 2. 与全局 ObjectMapper 配置的关系

你也可以在 `ObjectMapper` 上设置全局规则：

```java
ObjectMapper mapper = new ObjectMapper();
mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
```

**优先级顺序（从高到低）**：
1. **字段/方法上的 `@JsonInclude`**
2. **类上的 `@JsonInclude`**
3. **ObjectMapper 全局配置**
4. **Jackson 默认行为（ALWAYS）**

> ✅ 所以：局部注解 > 全局配置。

### 3. 与 `@JsonIgnore` 的关系

- `@JsonIgnore`：**强制忽略**某个字段，无论值是什么。
- `@JsonInclude`：**根据值的内容决定是否忽略**。

**优先级**：`@JsonIgnore` > `@JsonInclude`  
即：如果一个字段被 `@JsonIgnore` 标记，即使它有值也不会输出。

---

## 六、常见使用场景总结

| 场景 | 推荐配置 |
|------|--------|
| 不想看到 `null` 字段 | `@JsonInclude(NON_NULL)` |
| 前端不需要空字符串/空列表 | `@JsonInclude(NON_EMPTY)` |
| 返回 API 响应时精简数据 | `NON_EMPTY` 或 `NON_NULL` |
| 配置类，只输出非默认项 | `NON_DEFAULT` |
| 处理 `Optional` 类型 | `NON_ABSENT` |
| 过滤 Map 中的 null 值 | `@JsonInclude(value = NON_EMPTY, content = NON_NULL)` |

---

## 七、快速自查清单（未来回顾用）

✅ **问：我想让 JSON 里没有 `null`，怎么写？**  
→ 在类上加：`@JsonInclude(JsonInclude.Include.NON_NULL)`

✅ **问：空字符串 `""` 还是出现了，怎么办？**  
→ 改用 `NON_EMPTY`，它会同时忽略 `null` 和空字符串/空集合。

✅ **问：为什么我设置了 `NON_DEFAULT`，但 `int=0` 还是输出了？**  
→ 检查字段是否真的是基本类型（`int`）还是包装类型（`Integer`）。`NON_DEFAULT` 对 `int` 默认值 `0` 有效，但对 `Integer` 的 `null` 也视为“默认”，会忽略。

✅ **问：能否只对某个字段特殊处理？**  
→ 可以！直接在字段上加 `@JsonInclude(...)`，优先级高于类级别。

✅ **问：全局设置和注解冲突了怎么办？**  
→ 注解优先。放心在关键类上使用注解覆盖全局行为。

---

## 八、附：完整可运行示例

```java
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.databind.ObjectMapper;
import java.util.*;

@JsonInclude(JsonInclude.Include.NON_EMPTY)
public class Demo {
    public String normal = "hello";
    public String emptyStr = "";
    public String nullStr = null;
    public List<String> emptyList = new ArrayList<>();
    public List<String> nonEmptyList = Arrays.asList("a", "b");
    public Map<String, String> map = new HashMap<>();

    public static void main(String[] args) throws Exception {
        Demo d = new Demo();
        d.map.put("valid", "yes");
        d.map.put("nullValue", null);

        ObjectMapper mapper = new ObjectMapper();
        System.out.println(mapper.writeValueAsString(d));
    }
}
```

**输出：**
```json
{"normal":"hello","nonEmptyList":["a","b"],"map":{"valid":"yes"}}
```

---