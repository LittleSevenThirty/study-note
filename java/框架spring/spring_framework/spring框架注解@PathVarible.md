# @PathVariable 注解详解笔记

## 1. 什么是 @PathVariable？（核心概念）

@PathVariable 是从 URL 路径里**提取动态参数**的注解。

**通俗比喻**：  
你的朋友给你发了一个快递地址："中国北京市朝阳区某某街道，收件人：张三"。你要从这个地址里**提取出**"北京"、"朝阳区"、"张三"这些具体信息，@PathVariable 就是做这个工作的！

## 2. 基本语法（一看就懂）

```java
@GetMapping("/用户/{id}")  // {id} 是占位符
public String 获取用户(@PathVariable int id) {
    return "用户" + id;  // id 自动被Spring从路径提取出来
}
```

**访问过程**：

- 你访问：`http://localhost:8080/用户/123`
- Spring 自动提取：`id = 123`
- 方法收到：`id = 123`
- 返回：`"用户123"`

## 3. @PathVariable 的工作原理（深度理解）

```java
@GetMapping("/用户/{id}/订单/{orderId}")
public String 获取订单(
    @PathVariable int id,        // 自动从{id}提取
    @PathVariable int orderId    // 自动从{orderId}提取
) {
    return "用户" + id + "的订单" + orderId;
}
```

**请求流程**：

```
请求地址：http://localhost:8080/用户/123/订单/456

↓ Spring 解析路径

@PathVariable int id         → 123（来自{id}）
@PathVariable int orderId    → 456（来自{orderId}）

↓ 方法执行

return "用户123的订单456"
```

## 4. @PathVariable 的完整写法

### 4.1 简化写法（变量名必须和{占位符}相同）

```java
@GetMapping("/书/{bookId}")
public String 获取书(
    @PathVariable int bookId  // 直接提取{bookId}
) {
    return "书ID：" + bookId;
}
```

### 4.2 完整写法（变量名和占位符可以不同）

```java
@GetMapping("/书/{bookId}")
public String 获取书(
    @PathVariable(name = "bookId") int bid  // 指定从{bookId}提取给bid
) {
    return "书ID：" + bid;
}
```

### 4.3 必需 vs 可选

```java
// 必需的（默认）
@PathVariable int id
// 访问 /用户 → 404错误（路径不匹配）
// 访问 /用户/123 → 正常

// 可选的
@PathVariable(required = false) Integer id
// 访问 /用户 → id = null
// 访问 /用户/123 → id = 123
```

## 5. 数据类型支持（常用类型）

```java
@GetMapping("/商品/{id}")
public void 演示数据类型(
    @PathVariable int id,           // ✓ 整数
    @PathVariable Long productId,   // ✓ 长整数
    @PathVariable String name,      // ✓ 字符串
    @PathVariable double price,     // ✓ 浮点数
    @PathVariable LocalDate date    // ✓ 日期
) {}
```

**实例**：

```java
@GetMapping("/商品/{id}")
public String 获取商品(@PathVariable int id) {
    return "商品" + id;
}
// 访问：/商品/123 → "商品123"
// 访问：/商品/abc → 400 Bad Request（id必须是数字）

@GetMapping("/用户/{name}")
public String 获取用户(@PathVariable String name) {
    return "用户" + name;
}
// 访问：/用户/李明 → "用户李明"
// 访问：/用户/王五 → "用户王五"
```

## 6. 多个 @PathVariable（工作中最常见！）

```java
@GetMapping("/公司/{companyId}/部门/{departmentId}/员工/{employeeId}")
public String 获取员工信息(
    @PathVariable int companyId,      // 公司ID
    @PathVariable int departmentId,   // 部门ID
    @PathVariable int employeeId      // 员工ID
) {
    return String.format(
        "公司%d，部门%d，员工%d",
        companyId, departmentId, employeeId
    );
}
```

**访问**：`http://localhost:8080/公司/100/部门/20/员工/5`

```
companyId = 100
departmentId = 20
employeeId = 5
```

## 7. @PathVariable vs @RequestParam（重要区别！）

| 特性       | @PathVariable  | @RequestParam          |
| ---------- | -------------- | ---------------------- |
| 在哪里？   | URL路径里      | URL查询字符串里        |
| 写法       | `/用户/{id}`   | `/用户?id=123`         |
| 是否必需？ | **是**（默认） | 否（默认可选）         |
| 用途       | **资源标识**   | **过滤/搜索**          |
| 例子       | `/用户/123`    | `/用户?page=1&size=10` |

**完整对比例子**：

```java
// @PathVariable - 获取特定资源
@GetMapping("/用户/{id}")
public String 获取用户(@PathVariable int id) {
    // id 是"哪个用户"的标识
    return "用户" + id;
}
// 访问：/用户/123 → "用户123"

// @RequestParam - 过滤/搜索
@GetMapping("/用户")
public String 搜索用户(
    @RequestParam String keyword,
    @RequestParam(defaultValue="1") int page
) {
    // keyword和page是搜索条件
    return "搜索'" + keyword + "'，第" + page + "页";
}
// 访问：/用户?keyword=李&page=2 → "搜索'李'，第2页"

// 结合使用
@GetMapping("/用户/{id}/订单")
public String 获取用户订单(
    @PathVariable int id,                           // 用户是谁
    @RequestParam(defaultValue="0") int pageNum,    // 订单第几页
    @RequestParam(defaultValue="10") int pageSize   // 每页几条
) {
    return String.format(
        "用户%d的订单，第%d页，每页%d条",
        id, pageNum, pageSize
    );
}
// 访问：/用户/123/订单?pageNum=1&pageSize=20
```

## 8. 正则表达式匹配（进阶用法）

```java
// 只接受纯数字ID
@GetMapping("/产品/{id:\\d+}")
public String 获取产品(@PathVariable int id) {
    return "产品" + id;
}
// 访问：/产品/123 ✓ 正常
// 访问：/产品/abc ✗ 404错误

// 接受带小数点的版本号
@GetMapping("/软件/{version:\\d+\\.\\d+}")
public String 获取版本(@PathVariable String version) {
    return "版本" + version;
}
// 访问：/软件/1.0 ✓ 正常
// 访问：/软件/1.0.0 ✗ 不匹配
```

## 9. 常见错误及解决方案

| 错误现象        | 原因                                | 解决方案                              |
| --------------- | ----------------------------------- | ------------------------------------- |
| 404 Not Found   | 路径不匹配                          | 检查URL和@GetMapping里的{占位符}      |
| 400 Bad Request | 类型转换失败                        | 确保传入的值能转换为指定类型（如int） |
| 参数为null      | 设置了required=false但没传值        | 检查路径是否完整                      |
| 变量名错误      | @PathVariable的名字和{占位符}不一致 | 保持一致或用name属性指定              |

**例子**：

```java
// ✗ 错误示范
@GetMapping("/用户/{userId}")
public String 错误示范(@PathVariable int id) {
    // 变量名id和占位符{userId}不匹配！
    return id;
}
// 结果：500错误

// ✓ 正确做法1
@GetMapping("/用户/{id}")
public String 正确示范1(@PathVariable int id) {
    return "用户" + id;
}

// ✓ 正确做法2
@GetMapping("/用户/{userId}")
public String 正确示范2(@PathVariable(name = "userId") int id) {
    return "用户" + id;
}
```

## 10. 实战完整例子（复制即用）

```java
@RestController
@RequestMapping("/api")
public class 在线商城Controller {

    // 1. 获取单个商品
    @GetMapping("/商品/{productId}")
    public String 获取商品(@PathVariable int productId) {
        return "商品ID：" + productId + "，价格：99.9元";
    }

    // 2. 获取分类下的商品
    @GetMapping("/分类/{categoryId}/商品/{productId}")
    public String 获取分类商品(
        @PathVariable int categoryId,
        @PathVariable int productId
    ) {
        return "分类" + categoryId + "的商品" + productId;
    }

    // 3. 获取订单详情
    @GetMapping("/用户/{userId}/订单/{orderId}")
    public String 获取订单详情(
        @PathVariable int userId,
        @PathVariable int orderId
    ) {
        return "用户" + userId + "的订单" + orderId;
    }

    // 4. 复杂场景：用户-店铺-商品
    @GetMapping("/用户/{uid}/店铺/{shopId}/商品/{productId}")
    public String 复杂查询(
        @PathVariable int uid,
        @PathVariable int shopId,
        @PathVariable int productId
    ) {
        return String.format(
            "用户%d在店铺%d购买商品%d",
            uid, shopId, productId
        );
    }
}
```

**测试**：

```
GET /api/商品/100
返回：商品ID：100，价格：99.9元

GET /api/分类/5/商品/100
返回：分类5的商品100

GET /api/用户/1/订单/999
返回：用户1的订单999

GET /api/用户/1/店铺/10/商品/100
返回：用户1在店铺10购买商品100
```

## 11. 高级技巧（工作必备）

### 11.1 类型自动转换

```java
@GetMapping("/文章/{date}")
public String 按日期查询(@PathVariable LocalDate date) {
    // Spring自动将"2026-02-04"转换为LocalDate对象
    return "日期：" + date;
}
// 访问：/文章/2026-02-04 ✓ 自动转换
```

### 11.2 可变路径长度（REST推荐做法）

```java
@GetMapping("/文件/**")
public String 获取文件(@PathVariable String 文件路径) {
    return "文件：" + 文件路径;
}
// 访问：/文件/documents/2024/报告.pdf
```

### 11.3 结合@ResponseEntity返回HTTP状态

```java
@GetMapping("/用户/{id}")
public ResponseEntity<用户> 获取用户(@PathVariable Long id) {
    用户 u = 用户服务.findById(id);
    return u != null ?
        ResponseEntity.ok(u) :           // 200 OK
        ResponseEntity.notFound().build(); // 404 Not Found
}
```

## 12. 记忆要点（考试/面试）

```
@PathVariable 三句话记住：
1. 从URL路径里提取参数
2. 用{占位符}标记，方法参数接收
3. 默认必需，类型自动转换
```

**快速判断题**：

```
1. /用户/123 中的123如何接收？
   答：@PathVariable int id（路径{id}）

2. /用户?id=123 中的123如何接收？
   答：@RequestParam int id（查询参数）

3. /用户/{userId} 匹配 /用户/abc 吗？
   答：如果userId是int，则400错误；如果是String，则正常

4. @PathVariable 默认必需吗？
   答：是（required=true）
```

## 13. 最容易混淆的点（再读3遍！）

```java
// ✗ 这样写就错了！
@GetMapping("/用户/{id}")
public String 错误写法(@PathVariable int userId) {
    // {id} 和 userId 不匹配！Spring找不到映射！
}

// ✓ 这样改对
@GetMapping("/用户/{id}")
public String 正确写法(@PathVariable int id) {
    // 名字一致！
}

// ✓ 或者这样也对
@GetMapping("/用户/{id}")
public String 也可以(@PathVariable(name="id") int userId) {
    // 用name属性显式指定映射关系
}
```

---

**核心记忆**：@PathVariable = 从URL路径{}里拿参数！变量名要匹配占位符，或用name属性指定！

**保存建议**：配合@GetMapping笔记一起看，这两个注解是 Spring REST API 的核心基础，掌握它们就掌握了80%的Web开发！
