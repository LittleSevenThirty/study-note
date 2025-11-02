# MyBatis XML映射文件中 `<select>` 标签的深度解析

---

## 一、`<select>` 标签基础

### 1. 核心作用
- 用于定义SQL查询语句
- 将查询结果映射到Java对象（POJO、Map、基本类型等）

### 2. 基础属性
| 属性名          | 作用                                    | 示例值               |
| --------------- | --------------------------------------- | -------------------- |
| `id`            | 唯一标识，与Mapper接口方法名对应        | `getUserById`        |
| `resultType`    | 自动映射的返回类型（全限定类名/别名）   | `com.example.User`   |
| `resultMap`     | 自定义结果映射的ID                      | `userResultMap`      |
| `parameterType` | 传入参数类型（可省略，MyBatis自动推断） | `int`, `map`, `User` |
| `flushCache`    | 是否清空本地缓存（默认`false`）         | `true`/`false`       |
| `useCache`      | 是否存入二级缓存（默认`true`）          | `true`/`false`       |
| `timeout`       | 超时时间（秒）                          | `10`                 |

---

## 二、参数传递的三种方式

### 1. 单个基本类型参数
```xml
<select id="getUserById" resultType="User">
    SELECT * FROM user WHERE id = #{id}
</select>
```

### 2. 多个参数（使用`@Param`注解）

```java
// Mapper接口方法
User getUserByCondition(@Param("name") String name, @Param("age") int age);
```

```xml
<select id="getUserByCondition" resultType="User">
    SELECT * FROM user 
    WHERE username = #{name} AND age > #{age}
</select>
```

### 3. 对象参数（推荐）
```java
public class QueryVO {
    private String keyword;
    private Date startDate;
    // getters/setters
}
```

```xml
<select id="searchUsers" resultType="User" parameterType="QueryVO">
    SELECT * FROM user 
    WHERE username LIKE CONCAT('%', #{keyword}, '%')
    AND create_time >= #{startDate}
</select>
```

---

## 三、结果映射的两种策略

### 1. `resultType` 自动映射
```xml
<!-- 字段名与属性名完全一致时 -->
<select id="getAllUsers" resultType="com.example.User">
    SELECT id, name, email FROM user
</select>

<!-- 返回单个字段 -->
<select id="countUsers" resultType="int">
    SELECT COUNT(*) FROM user
</select>

<!-- 返回Map -->
<select id="getUserMap" resultType="map">
    SELECT * FROM user WHERE id = #{id}
</select>
```

### 2. ⭐`resultMap` 自定义映射[指向](.\resultMap标签解释.md)
```xml
<!-- 定义 -->
<resultMap id="userResultMap" type="User">
    <id property="id" column="user_id"/>
    <result property="username" column="name"/>
    <result property="email" column="email_address"/>
</resultMap>

<!-- 使用 -->
<select id="getComplexUser" resultMap="userResultMap">
    SELECT user_id, name, email_address FROM user
</select>
```

---

## 四、动态SQL拼接

### 1. `<if>` 条件判断
```xml
<select id="dynamicSearch" resultType="User">
    SELECT * FROM user
    <where>
        <if test="name != null">
            AND username LIKE #{name}
        </if>
        <if test="minAge != null">
            AND age >= #{minAge}
        </if>
    </where>
</select>
```

### 2. `<choose>` 多分支选择
```xml
<select id="conditionalQuery" resultType="User">
    SELECT * FROM user
    <where>
        <choose>
            <when test="status == 1">
                AND is_vip = true
            </when>
            <when test="status == 2">
                AND is_admin = true
            </when>
            <otherwise>
                AND active = true
            </otherwise>
        </choose>
    </where>
</select>
```

### 3. `<foreach>` 遍历集合
```xml
<select id="batchGetUsers" resultType="User">
    SELECT * FROM user
    WHERE id IN
    <foreach item="id" collection="ids" 
             open="(" separator="," close=")">
        #{id}
    </foreach>
</select>
```

---

## 五、高级查询场景

### 1. 分页查询（逻辑分页）
```xml
<select id="getUsersByPage" resultType="User">
    SELECT * FROM user
    LIMIT #{offset}, #{pageSize}
</select>
```

### 2. 联合查询（一对一）
```xml
<resultMap id="userWithProfileMap" type="User">
    <id property="id" column="id"/>
    <result property="username" column="username"/>
    <association property="profile" javaType="UserProfile">
        <id property="profileId" column="profile_id"/>
        <result property="address" column="address"/>
    </association>
</resultMap>

<select id="getUserWithProfile" resultMap="userWithProfileMap">
    SELECT u.*, p.profile_id, p.address 
    FROM user u
    LEFT JOIN user_profile p ON u.id = p.user_id
    WHERE u.id = #{id}
</select>
```

### 3. 嵌套集合（一对多）
```xml
<resultMap id="userWithOrdersMap" type="User">
    <id property="id" column="id"/>
    <collection property="orders" ofType="Order">
        <id property="orderId" column="order_id"/>
        <result property="amount" column="amount"/>
    </collection>
</resultMap>

<select id="getUserWithOrders" resultMap="userWithOrdersMap">
    SELECT u.*, o.order_id, o.amount
    FROM user u
    LEFT JOIN orders o ON u.id = o.user_id
    WHERE u.id = #{id}
</select>
```

---

## 六、最佳实践

1. **优先使用`resultMap`**：  
   即使字段名与属性名一致，显式映射能提高可维护性

2. **参数传递规范**：  
   - 多个参数必须使用`@Param`注解或包装对象
   - 优先使用`#{}`防止SQL注入

3. **性能优化**：  
   ```xml
   <!-- 关闭自动缓存 -->
   <select id="getRealTimeData" 
           flushCache="true" 
           useCache="false">
       SELECT * FROM realtime_table
   </select>
   ```

4. **复杂查询拆分**：  
   对于多层嵌套查询，可拆分为多个`<resultMap>`分别定义

5. **类型别名**：  
   在`mybatis-config.xml`中配置类型别名简化配置：
   ```xml
   <typeAliases>
       <typeAlias type="com.example.User" alias="User"/>
   </typeAliases>
   ```

---

## 七、常见问题解决方案

### Q1：如何返回插入后的自增主键？
```xml
<insert id="insertUser" useGeneratedKeys="true" keyProperty="id">
    INSERT INTO user(username) VALUES(#{username})
</insert>
```

### Q2：如何处理字段名与属性名不一致？
- 方案1：使用`resultMap`显式映射
- 方案2：SQL中使用别名
  ```xml
  <select id="getUser" resultType="User">
      SELECT 
          user_id AS id, 
          user_name AS username
      FROM t_user
  </select>
  ```

### Q3：如何执行模糊查询？
```xml
<select id="search" resultType="User">
    SELECT * FROM user
    WHERE username LIKE CONCAT('%', #{keyword}, '%')
</select>
```

---

通过以上体系化的学习，您应该能够：  
✅ 熟练配置各种查询场景  
✅ 理解结果映射的底层逻辑  
✅ 掌握动态SQL的灵活运用  
✅ 规避常见的配置错误