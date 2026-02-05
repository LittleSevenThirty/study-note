# **条件构造器Wrapper**

MyBatis-Plus 提供了一套强大的条件构造器（Wrapper），用于构建复杂的数据库查询条件。Wrapper 类允许开发者以链式调用的方式构造查询条件，无需编写繁琐的 SQL 语句，从而提高开发效率并减少 SQL 注入的风险。Java（编程语言）

在 MyBatis-Plus 中，Wrapper 类是构建查询和更新条件的核心工具。以下是主要的 Wrapper 类及其功能：

- **AbstractWrapper** ：这是一个抽象基类，提供了所有 Wrapper 类共有的方法和属性。它定义了条件构造的基本逻辑，包括字段（column）、值（value）、操作符（condition）等。所有的 QueryWrapper、UpdateWrapper、LambdaQueryWrapper 和 LambdaUpdateWrapper 都继承自 AbstractWrapper。
- **QueryWrapper**：专门用于构造查询条件，支持基本的等于、不等于、大于、小于等各种常见操作。它允许你以链式调用的方式添加多个查询条件，并且可以组合使用 `and` 和 `or` 逻辑。
- **UpdateWrapper**：用于构造更新条件，可以在更新数据时指定条件。与 QueryWrapper 类似，它也支持链式调用和逻辑组合。使用 UpdateWrapper 可以在不创建实体对象的情况下，直接设置更新字段和条件
- **LambdaQueryWrapper**：这是一个基于 Lambda 表达式的查询条件构造器，它通过 Lambda 表达式来引用实体类的属性，从而避免了硬编码字段名。这种方式提高了代码的可读性和可维护性，尤其是在字段名可能发生变化的情况下。
- **LambdaUpdateWrapper**：类似于 LambdaQueryWrapper，LambdaUpdateWrapper 是基于 Lambda 表达式的更新条件构造器。它允许你使用 Lambda 表达式来指定更新字段和条件，同样避免了硬编码字段名的问题。

## **功能详解**

**提示**

> **条件判断**： Wrapper 方法通常接受一个 `boolean` 类型的参数，用于决定是否将该条件加入到最终的 SQL 中。例如：
>
> ```java
> queryWrapper.like(StringUtils.isNotBlank(name), Entity::getName, name)
>            .eq(age != null && age >= 0, Entity::getAge, age);
> ```
>
> **默认行为**：如果某个方法没有显式提供`boolean`类型的参数，则默认为 `true`，即条件总是会被加入到 SQL 中。
> **字段引用**：字段引用：在 LambdaWrapper 中，`R `代表的是一个函数，用于引用实体类的属性，例如 `Entity::getId`。而在普通 Wrapper 中，`R `代表的是数据库字段名。
> **字段名注意事项**：当`R`具体类型为`String`时，表示的是数据库字段名，而不是实体类数据字段名。如果字段名是数据库关键字，需要使用转义符包裹> **集合参数**：如果方法的参数是 `Map `或` List`，当它们为空时，对应的 SQL 条件不会被加入到最终的 SQL 中。

### **ne方法-不等于**

`ne `方法是 MyBatis-Plus 中用于构建查询条件的基本方法之一，它用于设置单个字段的不相等条件

#### **使用范围**

- **QueryWrapper**
- **LambdaQueryWrapper**
- **UpdateWrapper**
- **LambdaUpdateWrapper**

#### **方法签名**

```java
// 设置指定字段的不相等条件
ne(R column, Object val)

// 根据条件设置指定字段的不相等条件
ne(boolean condition, R column, Object val)
```

#### **参数说明**

- `column`：数据库字段名或使用 `Lambda` 表达式的字段名。
- `val`：与字段名对应的值。
- `condition`：一个布尔值，用于控制是否应用这个不相等条件。

#### **示例**

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.ne("name", "老王");
```

**Lambda Wrapper**

```java
LambdaQueryWrapper<User> lambdaQueryWrapper = new LambdaQueryWrapper<>();
lambdaQueryWrapper.ne(User::getName, "老王");
```

**生成的SQL**

```sql
-- 普通 Wrapper 和 Lambda Wrapper 生成的 SQL 相同
SELECT * FROM user WHERE name <> '老王'
```

**普通Wrapper**

### **and方法**

`and `方法是 MyBatis-Plus 中用于构建查询条件的基本方法之一，它用于在查询条件中添加 AND 逻辑。通过调用 `and `方法，可以创建 AND 嵌套条件，即在一个 AND 逻辑块()中包含多个查询条件。（简单的说就是 `queryWrapper.eq("name","李白").and(wrapper->wrapper.eq("status","活着").or().eq("史书","存在"))`被转换成`name='李白' and ()`

这个 `wrapper -> ...` 其实就是 `and `方法的核心参数。你可以这样理解它的逻辑：

- **外层**：负责把括号 () 撑起来。
- **括号内（Lambda 内部）**：就像一个新的小画布，你在这个小画布上按顺序写 `gt, or, eq`

#### **使用范围**

- **QueryWrapper**
- **LambdaQueryWrapper**
- **UpdateWrapper**
- **LambdaUpdateWrapper**

#### **方法签名**

```java
// 添加 AND 嵌套条件
and(Consumer<Param> consumer)
and(boolean condition, Consumer<Param> consumer)
```

#### **参数说明**

- `consumer`：一个 `Consumer` 函数式接口，它接受一个 `Param` 类型的参数，并可以调用 `Param` 对象上的方法来构建 AND 嵌套条件。

* `condition`：一个布尔值，用于控制是否应用这个 AND 嵌套逻辑。

#### **示例**

**普通Wrapper**

```java
QueryWrapper<User> queryWrapper = new QueryWrapper<>();
queryWrapper.and(i -> i.and(j -> j.eq("name", "李白").eq("status", "alive"))
                         .and(j -> j.eq("name", "杜甫").eq("status", "alive")));
```

**LambdaWrapper**

```java
LambdaQueryWrapper<User> lambdaQueryWrapper = new LambdaQueryWrapper<>();
lambdaQueryWrapper.and(i -> i.and(j -> j.eq(User::getName, "李白").eq(User::getStatus, "alive"))
                              .and(j -> j.eq(User::getName, "杜甫").eq(User::getStatus, "alive")));
```

**生成的SQL**

```sql
-- 普通 Wrapper 和 Lambda Wrapper 生成的 SQL 相同
SELECT * FROM user WHERE ((name = '李白' AND status = 'alive') AND (name = '杜甫' AND status = 'alive'))
```
