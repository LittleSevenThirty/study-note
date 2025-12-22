# **@TableField讲解**

# 📘 MyBatis-Plus `@TableField` 注解详解笔记

> 全限定类名：`com.baomidou.mybatisplus.annotation.TableField`

---

## 一、基础讲解：它是什么？为什么用？

### 1.1 核心用途

`@TableField` 是 **MyBatis-Plus（简称 MP）** 提供的一个注解，用于**描述 Java 实体类中的字段（field）与数据库表中列（column）之间的映射关系**。  
当你使用 MP 进行数据库操作时，MP 会自动根据实体类生成 SQL 语句（如 INSERT、SELECT、UPDATE 等）。但默认情况下，MP 假设：

- Java 字段名（驼峰命名，如 `userName`）
- 对应数据库列名（下划线命名，如 `user_name`）

这种“自动转换”在大多数情况下有效，但**当实际情况不符合这个规则时，就需要 `@TableField` 来干预**。

### 1️⃣ 主要解决以下问题：

| 场景 | 说明 | 是否需要 `@TableField` |
|------|------|------------------|
| **字段名 ≠ 列名** | 比如 Java 字段叫 `email`，但数据库列叫 `mail_addr` | ✅ 需要 |
| **字段不是数据库列** | 比如实体类中有 `fullName`（由 `firstName + lastName` 拼接），但数据库没有这列 | ✅ 需要标记为“不存在” |
| **自动填充字段** | 比如创建时间 `createTime`，希望在插入时自动填入当前时间 | ✅ 需要配置 `fill` |
| **逻辑删除字段** | 比如 `deleted` 字段，值为 0 表示未删，1 表示已删（不真正删除记录） | ✅ 需配合 `@TableLogic`，但有时也用 `@TableField` 辅助 |
| **控制字段是否参与查询/更新** | 比如密码字段 `password`，查询时不返回，或更新时需特殊处理 | ✅ 可通过 `select = false` 或 `update = "%s+1"` 控制 |

---

### 1.2 典型使用示例

#### 示例 1：字段名与列名不一致
```java
public class User {
    private Long id;

    @TableField("mail_addr") // 数据库列是 mail_addr，Java 字段是 email
    private String email;
}
```
> 查询时，MP 会生成：`SELECT id, mail_addr AS email FROM user`

---

#### 示例 2：忽略非表字段（exist = false）
```java
public class User {
    private String username;

    @TableField(exist = false) // 数据库没有 fullName 列
    private String fullName; // 仅用于程序内部计算
}
```
> MP 在生成 SQL 时会**完全忽略 `fullName` 字段**，不会尝试映射或操作它。

---

#### 示例 3：自动填充创建/更新时间
```java
public class User {
    @TableField(fill = FieldFill.INSERT) // 插入时自动填充
    private LocalDateTime createTime;

    @TableField(fill = FieldFill.INSERT_UPDATE) // 插入和更新时都填充
    private LocalDateTime updateTime;
}
```
> 配合 MP 的 **MetaObjectHandler** 实现类，可在插入/更新时自动设置时间。

---

#### 示例 4：查询时不返回敏感字段（如密码）
```java
public class User {
    @TableField(select = false) // 查询时不会 SELECT password
    private String password;
}
```
> 执行 `userMapper.selectById(1)` 时，SQL 不会包含 `password` 字段。

---

#### 示例 5：自定义更新逻辑（如递增）
```java
public class Product {
    @TableField(update = "%s+1") // 更新时字段值 +1
    private Integer viewCount;
}
```
> 调用 `updateById()` 时，会生成：`UPDATE product SET view_count = view_count + 1 WHERE id = ?`

---

## 二、参数详解：所有属性说明（零前提假设）

> 默认情况下，**不加 `@TableField` 的字段会被 MP 自动处理为“普通字段”**（即按驼峰转下划线映射，参与所有 CRUD）。

以下是 `@TableField` 的全部公开属性（截至 MyBatis-Plus 3.5.x）：

---

### 🔹 `value`（最常用）
- **作用**：指定数据库列名。
- **类型**：`String`
- **默认值**：空字符串 `""`
- **说明**：
  - 如果为空，则 MP 使用**字段名转下划线**作为列名（如 `userName` → `user_name`）。
  - 如果不为空，则**直接使用该值作为列名**。
- **示例**：
  ```java
  @TableField("user_email")
  private String email;
  ```

---

### 🔹 `exist`
- **作用**：标记该字段**是否存在于数据库表中**。
- **类型**：`boolean`
- **默认值**：`true`
- **说明**：
  - `true`：该字段对应数据库列（默认行为）。
  - `false`：该字段**只是 Java 对象的属性，数据库没有此列**，MP 会忽略它。
- **典型场景**：DTO 字段、计算字段、临时缓存字段等。
- **示例**：
  ```java
  @TableField(exist = false)
  private String tempToken;
  ```

---

### 🔹 `fill`
- **作用**：指定**自动填充策略**（何时自动赋值）。
- **类型**：`FieldFill` 枚举
- **默认值**：`FieldFill.DEFAULT`
- **可选值**：
  - `DEFAULT`：不自动填充（默认）
  - `INSERT`：仅插入时填充
  - `UPDATE`：仅更新时填充
  - `INSERT_UPDATE`：插入和更新时都填充
- **如何生效？**
  - 需要**自定义一个类实现 `MetaObjectHandler` 接口**，并在其中编写填充逻辑。
  - MP 会在执行 INSERT/UPDATE 前调用你的填充方法。
- **示例**：
  ```java
  @TableField(fill = FieldFill.INSERT)
  private LocalDateTime createTime;
  ```
  ```java
  // 自定义填充器（需注册为 Spring Bean）
  @Component
  public class MyMetaObjectHandler implements MetaObjectHandler {
      @Override
      public void insertFill(MetaObject metaObject) {
          this.strictInsertFill(metaObject, "createTime", LocalDateTime::now, LocalDateTime.class);
      }
  }
  ```

---

### 🔹 `select`
- **作用**：控制该字段**是否参与 SELECT 查询**。
- **类型**：`boolean`
- **默认值**：`true`
- **说明**：
  - `false`：MP 在生成 `SELECT` 语句时**不会包含该字段**。
  - 常用于**敏感字段**（如密码、密钥）避免泄露。
- **注意**：即使 `select = false`，你仍可通过手动写 SQL 查询该字段。
- **示例**：
  ```java
  @TableField(select = false)
  private String password;
  ```

---

### 🔹 `update`
- **作用**：自定义该字段在 UPDATE 语句中的**赋值表达式**。
- **类型**：`String`
- **默认值**：空字符串 `""`
- **说明**：
  - 如果为空，MP 使用 `#{字段名}` 正常赋值。
  - 如果不为空，MP 会将 `%s` 替换为列名，生成表达式。
- **典型场景**：字段递增、拼接字符串等。
- **示例**：
  ```java
  @TableField(update = "%s+1")
  private Integer loginCount;
  ```
  > 生成 SQL：`UPDATE user SET login_count = login_count + 1 WHERE ...`

  ```java
  @TableField(update = "CONCAT(%s,'_updated')")
  private String status;
  ```
  > 生成：`UPDATE user SET status = CONCAT(status, '_updated') WHERE ...`

---

### 🔹 `condition`
- **作用**：自定义该字段在 WHERE 条件中的**比较方式**。
- **类型**：`String`
- **默认值**：空字符串 `""`
- **说明**：
  - 默认使用 `=` 进行比较（如 `WHERE name = ?`）。
  - 可指定其他条件模板，如 `LIKE`、`>` 等。
  - 模板中 `%s` 会被替换为列名，`%p` 会被替换为参数占位符（如 `?`）。
- **示例**：
  ```java
  @TableField(condition = "%s LIKE CONCAT('%%', %p, '%%')")
  private String keyword;
  ```
  > 当你用 `QueryWrapper.eq("keyword", "abc")` 时，实际生成：  
  > `WHERE keyword LIKE CONCAT('%', ?, '%')` → 实现模糊查询。

> ⚠️ 注意：此功能较少使用，通常直接在 `QueryWrapper` 中写 `.like()` 更直观。

---

### 🔹 `jdbcType`
- **作用**：显式指定 JDBC 类型（如 VARCHAR、TIMESTAMP 等）。
- **类型**：`JdbcType`（MyBatis 枚举）
- **默认值**：`JdbcType.UNDEFINED`
- **说明**：
  - 一般不需要设置，MP 和 MyBatis 能自动推断。
  - 在**某些数据库（如 Oracle）或特殊字段（如 CLOB/BLOB）** 时可能需要显式指定。
- **示例**：
  ```java
  @TableField(jdbcType = JdbcType.VARCHAR)
  private String remark;
  ```

---

### 🔹 `typeHandler`
- **作用**：指定**类型处理器**（TypeHandler），用于 Java 类型 ↔ 数据库类型的转换。
- **类型**：`Class<? extends TypeHandler>`
- **默认值**：`UnknownTypeHandler.class`
- **说明**：
  - 适用于**复杂类型转换**，比如将 JSON 字符串 ↔ Java 对象。
  - 需要你自定义 `TypeHandler` 实现类。
- **示例**：
  ```java
  @TableField(typeHandler = JacksonTypeHandler.class)
  private Map<String, Object> config;
  ```
  > 插入时，`config` 对象会被 `JacksonTypeHandler` 转为 JSON 字符串存入数据库；查询时反向转换。

---

### 🔹 `el`（已废弃，不推荐使用）
- **历史用途**：早期用于 SpEL 表达式（Spring Expression Language）。
- **现状**：**从 MP 3.4+ 起已废弃**，请勿使用。

---

## 三、补充说明：你需要知道的背景知识（极简版）

### ❓ 什么是“字段映射”？
- ORM（对象关系映射）框架（如 MyBatis-Plus）会把 Java 对象（如 `User`）和数据库表（如 `user`）对应起来。
- 默认规则：`userName`（Java） ↔ `user_name`（DB）。
- 当规则不适用时，用 `@TableField` 告诉框架“这个字段对应哪一列”。

### ❓ 为什么要标记“非表字段”？
- 如果实体类有字段但数据库没有，MP 默认会尝试操作它，导致 SQL 报错（如 “Unknown column 'fullName'”）。
- 用 `exist = false` 告诉 MP：“别管这个字段，它不属于表”。

### ❓ 自动填充怎么触发？
1. 在字段上加 `@TableField(fill = ...)`
2. 编写一个类实现 `MetaObjectHandler`
3. 在该类中重写 `insertFill()` / `updateFill()`
4. 将该类注册为 Spring Bean（加 `@Component`）
5. MP 在执行 INSERT/UPDATE 前会自动调用你的填充方法

---

## 四、总结速查表

| 属性 | 用途 | 常用值 | 备注 |
|------|------|--------|------|
| `value` | 指定列名 | `"real_column"` | 最常用 |
| `exist` | 是否是表字段 | `false` | 忽略非 DB 字段 |
| `fill` | 自动填充时机 | `INSERT`, `INSERT_UPDATE` | 需配合 `MetaObjectHandler` |
| `select` | 是否参与查询 | `false` | 隐藏敏感字段 |
| `update` | 自定义更新表达式 | `"%s+1"` | 实现字段递增等 |
| `condition` | 自定义 WHERE 条件 | `"%s LIKE ..."` | 较少用 |
| `jdbcType` | 指定 JDBC 类型 | `JdbcType.VARCHAR` | 特殊类型才需 |
| `typeHandler` | 自定义类型转换 | `MyJsonTypeHandler.class` | 处理 JSON/枚举等 |

---

## 五、最佳实践建议

1. **优先使用默认映射**：能用驼峰转下划线就不要加 `value`。
2. **非表字段务必标记 `exist = false`**：避免运行时报错。
3. **敏感字段设 `select = false`**：安全第一。
4. **自动填充统一管理**：写一个 `MetaObjectHandler` 处理所有时间字段。
5. **复杂类型用 `typeHandler`**：避免手动序列化/反序列化。

---