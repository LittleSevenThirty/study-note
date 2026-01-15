----------
author: little-seven-thirty
create-time: 2026-01-12
----------

# **localStorage 接口详解与速查笔记**

## **概述**

`localStorage` 是 Web Storage API 的一部分，提供了一种在浏览器中持久化存储键值对数据的机制。其数据不会因页面刷新或关闭而丢失（除非用户手动清除），适用于保存用户偏好、缓存配置、临时状态等轻量级本地数据。

与 `sessionStorage` 不同，`localStorage` 的数据在**同一源（origin）下所有标签页和窗口间共享**，且**无过期时间**。

> ⚠️ 注意：`localStorage` 中的所有键和值都以 **字符串形式** 存储。若需存储对象或数组，需配合 `JSON.stringify()` 和 `JSON.parse()` 使用。

---

## **前置知识**

- HTML 文档基础结构
- JavaScript 基础语法（变量、函数、对象）
- 浏览器同源策略（Same-origin policy）
- JSON 数据格式与基本操作（`JSON.stringify` / `JSON.parse`）

---

## **关键词**

- `localStorage`
- Web Storage API
- 持久化存储
- 同源（origin）
- 键值对（key-value）
- `setItem` / `getItem` / `removeItem` / `clear`
- 字符串序列化

---

## **正式内容**

### 1. **基本用法**

```javascript
// 存储数据（自动转为字符串）
localStorage.setItem("username", "Alice");
localStorage.setItem("theme", "dark");
localStorage.setItem("user", JSON.stringify({ id: 1, name: "Bob" }));

// 读取数据
const username = localStorage.getItem("username"); // "Alice"
const userStr = localStorage.getItem("user");
const user = JSON.parse(userStr); // { id: 1, name: "Bob" }

// 删除某项
localStorage.removeItem("theme");

// 清空所有数据（谨慎使用！）
localStorage.clear();
```

### 2. **重要特性**

| 特性 | 说明 |
|------|------|
| **持久性** | 数据永久保存，除非用户清除浏览器数据或代码调用 `clear()`/`removeItem()` |
| **作用域** | 仅限**同源**（协议 + 域名 + 端口相同）的页面共享 |
| **存储类型** | 所有 key 和 value 都是 **字符串** |
| **容量限制** | 通常为 5–10MB（因浏览器而异） |
| **同步阻塞** | 操作是同步的，大量读写可能影响性能（不适用于大数据） |

### 3. **常见陷阱与最佳实践**

- ❌ **不要直接存对象**  
  ```js
  localStorage.setItem("obj", { a: 1 }); // 实际存的是 "[object Object]"
  ```
  ✅ 正确做法：
  ```js
  localStorage.setItem("obj", JSON.stringify({ a: 1 }));
  ```

- ❌ **忽略解析错误**  
  若存储的字符串不是合法 JSON，`JSON.parse()` 会抛出异常。
  ✅ 安全读取：
  ```js
  function safeGet(key) {
    try {
      return JSON.parse(localStorage.getItem(key));
    } catch (e) {
      return null;
    }
  }
  ```

- 🛡️ **敏感数据勿存**  
  `localStorage` 数据可被 XSS 攻击窃取，**切勿存储密码、token 等敏感信息**。

- 🔁 **监听跨标签页变更**  
  可通过 `window.addEventListener('storage', callback)` 监听其他标签页对 `localStorage` 的修改（当前页修改不会触发）。

### 4. **封装工具函数（推荐）**

```js
const storage = {
  set(key, value) {
    localStorage.setItem(key, JSON.stringify(value));
  },
  get(key) {
    const val = localStorage.getItem(key);
    try {
      return val ? JSON.parse(val) : null;
    } catch {
      return val; // 兼容非 JSON 字符串
    }
  },
  remove(key) {
    localStorage.removeItem(key);
  },
  clear() {
    localStorage.clear();
  }
};

// 使用示例
storage.set("profile", { name: "Tom", age: 25 });
console.log(storage.get("profile")); // { name: "Tom", age: 25 }
```

---

## **参考**

- [MDN Web Docs: Window.localStorage](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/localStorage)（主参考）
- [Using the Web Storage API - MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Storage_API/Using_the_Web_Storage_API)
- [Web Storage API 规范 - WHATWG](https://html.spec.whatwg.org/multipage/webstorage.html)