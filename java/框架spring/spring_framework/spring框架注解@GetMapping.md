# @GetMapping 注解详解笔记

## 1. 什么是 @GetMapping？（超级简单理解）

@GetMapping 就是 Spring Boot（或 Spring MVC）里的一个“标签”，告诉程序：“嘿，这个方法专门用来处理浏览器发来的 GET 请求！”

**通俗比喻**：  
想象你开了一家小店，门口挂了个牌子“只卖水果”（GET），顾客进来就能买苹果香蕉（数据）。@GetMapping 就是这个牌子，告诉 Spring：“这个门只接待想买水果的顾客！”

## 2. 基本语法（一看就懂）

```java
@GetMapping("/水果")  // 顾客访问 http://你的网站/水果
public String 卖水果(){
    return "苹果5元/斤";
}
```

**完整例子**：

```java
@RestController  // 告诉Spring：我是个提供API的控制器
public class 水果店Controller {

    @GetMapping("/苹果")  // 顾客访问：http://localhost:8080/苹果
    public String 获取苹果列表() {
        return "红富士、青苹果，5元/斤";
    }
}
```

## 3. @GetMapping vs @RequestMapping（为什么要用它？）

| 写法                                                       | 优缺点                                  | 什么时候用             |
| ---------------------------------------------------------- | --------------------------------------- | ---------------------- |
| `@RequestMapping(value="/苹果", method=RequestMethod.GET)` | 功能最全，但写起来啰嗦                  | 需要同时支持GET+POST时 |
| `@GetMapping("/苹果")`                                     | **推荐！简洁明了，一看就知道是GET请求** | **99%情况都用这个**    |

**记忆口诀**：GET用GetMapping，POST用PostMapping，简洁第一！

## 4. 常见用法（按频率排序）

### 4.1 基础路径映射

```java
@GetMapping("/用户")           // http://localhost:8080/用户
@GetMapping("/用户/1")         // http://localhost:8080/用户/1
@GetMapping("/用户/{id}")      // 动态路径，id可以变化
```

### 4.2 带参数的 GET 请求

```java
@GetMapping("/搜索")
public String 搜索商品(String keyword, Integer page) {
    // keyword=手机, page=1
    return "找到10个手机";
}
```

浏览器访问：`http://localhost:8080/搜索?keyword=手机&page=1`

### 4.3 路径变量（最常用！）

```java
@GetMapping("/用户/{id}")  // {id}是占位符
public String 获取用户(@PathVariable int id) {
    return "用户" + id + "的信息";
}
```

访问：`http://localhost:8080/用户/123` → 返回“用户123的信息”

## 5. 高级用法（工作中必备）

### 5.1 多路径支持

```java
@GetMapping({"/旧路径", "/新路径"})  // 两个地址都能访问
public String 双路径方法() {
    return "OK";
}
```

### 5.2 指定媒体类型

```java
@GetMapping(value = "/数据", produces = "application/json")
public Map<String,Object> 返回JSON() {
    return Map.of("status", "success");
}
```

### 5.3 条件请求（HTTP状态码）

```java
@GetMapping("/商品/{id}")
public ResponseEntity<商品> 获取商品(@PathVariable Long id) {
    商品 g = 商品服务.findById(id);
    return g != null ?
        ResponseEntity.ok(g) :     // 200 OK
        ResponseEntity.notFound().build();  // 404 Not Found
}
```

## 6. 完整实战例子（复制粘贴就能跑）

```java
@RestController
@RequestMapping("/api")  // 所有方法都有共同前缀/api
public class 用户Controller {

    // 1. 获取所有用户列表
    @GetMapping("/用户")
    public List<用户> 获取用户列表() {
        return 用户列表;
    }

    // 2. 根据ID获取单个用户
    @GetMapping("/用户/{id}")
    public 用户 获取用户(@PathVariable Long id) {
        return 查找用户(id);
    }

    // 3. 分页查询
    @GetMapping("/用户")
    public Page<用户> 获取用户分页(
            @RequestParam(defaultValue="0") int page,
            @RequestParam(defaultValue="10") int size) {
        return 用户服务.findAll(page, size);
    }
}
```

**测试地址**：

```
GET http://localhost:8080/api/用户                    # 所有用户
GET http://localhost:8080/api/用户/123               # 用户123
GET http://localhost:8080/api/用户?page=0&size=5     # 第1页，每页5条
```

## 7. 记忆要点（考试/面试必背）

```
@GetMapping = GET请求专用注解
路径写在括号里：@GetMapping("/路径")
动态参数用{}：/用户/{id}
参数用@PathVariable接收
只读操作用GET（查数据，不改数据）
```

## 8. 常见错误及解决方案

| 错误                | 原因           | 解决                                      |
| ------------------- | -------------- | ----------------------------------------- |
| 404 Not Found       | 路径拼写错误   | 检查@GetMapping里的路径                   |
| 参数类型不匹配      | int接收String  | 用String接收或加@NumberFormat             |
| 缺少@RequestMapping | 类级别路径缺失 | 在Controller类上加@RequestMapping("/api") |

## 9. 快速复习题（自己默写一遍就记住了）

1. `@GetMapping` 处理什么HTTP方法？**答：GET**
2. 如何接收路径中的动态参数？**答：/用户/{id} + @PathVariable**
3. 怎么写分页接口？**答：@RequestParam page和size**

**恭喜！记住了这3点，80%的GET接口都会写了！**

---
