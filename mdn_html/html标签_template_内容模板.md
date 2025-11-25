# 📝 原生 HTML 中的 `<template>` 标签详解

> ✅ 适用场景：纯 HTML + JavaScript（无 Vue/React 等框架）  
> 📚 标准来源：[HTML Living Standard - `<template>`](https://html.spec.whatwg.org/multipage/scripting.html#the-template-element)

---

## 一、核心作用一句话总结

> **`<template>` 是一个“惰性容器”——它内部的 HTML 内容在页面加载时不会被渲染、不会执行脚本、不会发出网络请求，但可以被 JavaScript 按需“取出并激活”。**

---

## 二、关键特性（原生行为）

| 特性 | 说明 |
|------|------|
| ❌ **不渲染** | 浏览器不会把 `<template>` 及其内容显示在页面上 |
| ⏸️ **惰性解析** | 内部的 `<img>` 不会加载图片，`<script>` 不会执行，`<audio>` 不会播放 |
| 🧱 **保留结构** | 内容以文档片段（DocumentFragment）形式保存在内存中 |
| 🔓 **可克隆复用** | 通过 JS 获取后，可多次克隆插入 DOM，实现高效模板复用 |

---

## 三、原生 HTML 使用示例

### 示例 1：定义一个可复用的卡片模板

```html
<!DOCTYPE html>
<html>
<head>
  <title>原生 template 示例</title>
</head>
<body>
  <h1>用户列表</h1>
  <div id="user-container"></div>

  <!-- 定义模板（不会显示） -->
  <template id="user-card-template">
    <div class="user-card">
      <h3 class="name"></h3>
      <p class="email"></p>
      <button class="btn">查看详情</button>
    </div>
  </template>

  <script>
    const users = [
      { name: '张三', email: 'zhang@example.com' },
      { name: '李四', email: 'li@example.com' }
    ];

    const container = document.getElementById('user-container');
    const template = document.getElementById('user-card-template');

    users.forEach(user => {
      // 1. 克隆模板内容（deep = true 表示深拷贝）
      const clone = template.content.cloneNode(true);

      // 2. 填充数据
      clone.querySelector('.name').textContent = user.name;
      clone.querySelector('.email').textContent = user.email;

      // 3. 绑定事件（可选）
      clone.querySelector('.btn').addEventListener('click', () => {
        alert(`查看 ${user.name} 的详情`);
      });

      // 4. 插入到页面
      container.appendChild(clone);
    });
  </script>

  <style>
    .user-card {
      border: 1px solid #ccc;
      padding: 10px;
      margin: 10px 0;
      border-radius: 4px;
    }
  </style>
</body>
</html>
```

✅ 最终效果：页面显示两个用户卡片，但 `<template>` 自身完全不可见。

---

### 示例 2：验证 `<template>` 的“惰性”

```html
<template id="test-template">
  <img src="https://example.com/large-image.jpg" alt="不会加载">
  <script>
    console.log('这段脚本不会执行！');
  </script>
  <audio src="music.mp3" autoplay></audio> <!-- 不会播放 -->
</template>
```

打开浏览器开发者工具的 **Network 面板**，你会发现：
- 图片未请求
- 脚本未执行
- 音频未加载

只有当你用 JS 把它 `cloneNode()` 并 `appendChild()` 到 DOM 后，这些资源才会被激活！

---

## 四、为什么需要 `<template>`？（对比传统做法）

### ❌ 传统“伪模板”写法（不推荐）
```html
<div id="hidden-template" style="display: none;">
  <div class="card">...</div>
</div>
```

**问题**：
- 虽然隐藏了，但浏览器仍会：
  - 解析所有 HTML
  - 加载 `<img>`、`<link>` 等资源
  - 执行内联 `<script>`
  - 影响 SEO（搜索引擎可能索引隐藏内容）
- CSS 选择器可能意外匹配到它

### ✅ 使用 `<template>` 的优势
- **零副作用**：内容完全惰性，不消耗资源
- **语义清晰**：明确表示“这是模板，不是内容”
- **性能更好**：避免不必要的网络请求和脚本执行
- **标准支持**：现代浏览器原生支持（IE 不支持，但现代项目通常不考虑 IE）

---

## 五、浏览器兼容性

| 浏览器 | 支持情况 |
|--------|--------|
| Chrome | ✅ 26+ (2013) |
| Firefox | ✅ 22+ |
| Safari | ✅ 7.1+ |
| Edge | ✅ 13+ |
| IE | ❌ 不支持 |

> 💡 如果你需要支持 IE，可以用 `<script type="text/html">` 模拟，但无法获得惰性优势。

---

## 六、与 Vue/React 等框架的关系

- **Vue 和 React 都借鉴了 `<template>` 的思想**
  - Vue 的 `<template>` 在编译阶段被转换为渲染函数
  - React 的 JSX 本质也是“声明式模板”
- 但在 **原生 HTML 中**，`<template>` 是**真实存在的 DOM 元素**，你可以用 `document.getElementById()` 直接操作它

---

## 七、一句话总结（原生场景）

> **在原生 HTML 中，`<template>` 是一个“待命的 HTML 片段仓库”——内容静默存放，直到 JavaScript 主动取出并插入页面，才真正“活”过来。**

---