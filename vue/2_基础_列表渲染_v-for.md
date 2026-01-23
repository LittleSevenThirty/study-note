# Vue 3 列表渲染（`v-for`）新手友好笔记

> **依据文档**：[Vue 3 官方指南 · 列表渲染](https://cn.vuejs.org/guide/essentials/list.html)  
> **目标**：用生活化语言 + 清晰结构，帮你真正理解 `v-for` 的核心机制与最佳实践。

---

## 📌 一、用 `v-for` 渲染列表（对应指南开头）

### ✨ 核心思想

`v-for` 就像“模板复印机”——你提供一个数据列表（比如商品、用户、消息），它就自动复制一段 HTML 模板，每份填入不同的数据。

### 🔧 基本语法

```html
<li v-for="item in items">{{ item.message }}</li>
```

- `items`：你的数据源（通常是数组）
- `item`：当前正在处理的那一项（你可以叫它 `fruit`、`user` 等）

> 💡 这和 JavaScript 的 `for...of` 循环非常像：
>
> ```js
> for (const item of items) {
>   console.log(item.message);
> }
> ```

---

## 🔢 二、获取当前项的索引（Index）

有时候你需要知道“这是第几项”，比如加序号、隔行变色。

### ✅ 写法

```html
<li v-for="(item, index) in items">{{ index + 1 }}. {{ item.message }}</li>
```

### 📝 注意

- `index` 从 **0 开始**（和 JS 数组一致）
- 参数顺序固定：**`(值, 索引)`**
- 你也可以只用解构取部分字段：
  ```html
  <!-- 只取 message 字段 -->
  <li v-for="{ message } in items">{{ message }}</li>
  ```

> 🌰 类比 JS：
>
> ```js
> items.forEach((item, index) => {
>   console.log(index, item.message);
> });
> ```

---

## 📦 三、在 `<template>` 上使用 `v-for`

当你想一次渲染**多个元素**（比如每个列表项包含标题+分隔线），但又不想多一层无意义的 `<div>`，就用 `<template>`。

### ✅ 示例

```html
<ul>
  <template v-for="item in items" :key="item.id">
    <li>{{ item.title }}</li>
    <li class="divider"></li>
  </template>
</ul>
```

### ⚠️ 关键点

- `<template>` 本身**不会出现在最终 DOM 中**，只是逻辑容器
- `:key` 要写在 `<template>` 上，而不是内部元素

---

## 🗝️ 四、为什么需要 `key`？（重点！）

### ❓ 问题场景

假设你有一个待办事项列表，每项都有一个输入框。当你删除中间一项时，**后面的输入框内容错位了**！

### 🔍 原因

Vue 默认采用“就地更新”策略：它认为“第一个 `<li>` 还是第一个”，直接复用 DOM 元素。但数据变了，状态（如输入框内容）却没跟着走。

### ✅ 解决方案：唯一 `key`

```html
<div v-for="item in list" :key="item.id">{{ item.text }}</div>
```

### 🧠 `key` 的作用

- 告诉 Vue：“这个 DOM 元素代表的是 id=5 的那条数据”
- 当列表变化时，Vue 能**精准追踪每个元素的身份**，正确移动/销毁/创建 DOM
- 避免组件状态错乱、动画异常等问题

### 🚫 错误做法

```html
<!-- 千万别用 index 当 key！ -->
<div v-for="(item, index) in list" :key="index"></div>
```

> 为什么？当插入/删除中间项时，后面所有 `index` 都会变，导致 Vue 误以为“所有项都变了”，失去优化意义。

---

## ⚠️ 五、`v-for` 与 `v-if` 一起用？小心！

### ❌ 不推荐写法

```html
<li v-for="todo in todos" v-if="!todo.isComplete">{{ todo.name }}</li>
```

### 🤔 为什么？

- `v-if` 优先级高于 `v-for`，意味着 `v-if` 先执行
- 此时 `todo` 变量还未定义 → **报错！**

### ✅ 正确做法（两种场景）

#### 场景1：过滤列表（如只显示“未完成”任务）

👉 用 **计算属性** 提前过滤：

```js
computed: {
  incompleteTodos() {
    return this.todos.filter(todo => !todo.isComplete)
  }
}
```

```html
<li v-for="todo in incompleteTodos" :key="todo.id">{{ todo.name }}</li>
```

#### 场景2：整体隐藏整个列表（如“暂无数据时不显示列表”）

👉 把 `v-if` 放到**外层容器**：

```html
<ul v-if="shouldShowTodos">
  <li v-for="todo in todos" :key="todo.id">{{ todo.name }}</li>
</ul>
```

---

## 🔄 六、响应式数组变更检测

Vue 能自动侦测以下**数组方法**的调用，并触发更新：

| 方法                    | 说明           |
| ----------------------- | -------------- |
| `push()` / `pop()`      | 末尾增删       |
| `shift()` / `unshift()` | 开头增删       |
| `splice()`              | 任意位置增删改 |
| `sort()` / `reverse()`  | 排序/反转      |

### ❗ 但这些操作**不会触发更新**：

```js
// 直接通过索引设置项
items[0] = newValue;

// 修改数组长度
items.length = 0;
```

### ✅ 替代方案

```js
// 用 splice 代替索引赋值
items.splice(0, 1, newValue)

// 用新数组替换（Vue 能高效复用 DOM）
items = items.filter(item => ...)
```

> 💡 Vue 3 使用 Proxy 实现响应式，已能监听大部分数组操作，但**直接索引赋值仍需谨慎**。

---

## 📊 七、显示过滤/排序后的列表（不修改原数据）

有时你想展示“筛选后”的结果，但又不想改动原始数据（比如保留完整列表用于其他功能）。

### ✅ 推荐：计算属性

```js
const numbers = ref([1, 2, 3, 4, 5]);

const evenNumbers = computed(() => {
  return numbers.value.filter((n) => n % 2 === 0);
});
```

```html
<li v-for="n in evenNumbers" :key="n">{{ n }}</li>
```

### ⚠️ 注意：不要在计算属性中修改原数组！

```js
// ❌ 错误：sort() 会改变原数组
computed: {
  sortedList() {
    return this.list.sort() // 副作用！
  }
}

// ✅ 正确：先复制再排序
computed: {
  sortedList() {
    return [...this.list].sort()
  }
}
```

---

## 🧩 八、遍历对象

除了数组，`v-for` 也能遍历对象的属性：

```js
const myObject = {
  title: "Vue 教程",
  author: "小明",
  publishedAt: "2026-01-22",
};
```

### 三种写法

```html
<!-- 只取值 -->
<li v-for="value in myObject">{{ value }}</li>

<!-- 取键和值 -->
<li v-for="(value, key) in myObject">{{ key }}: {{ value }}</li>

<!-- 再加索引 -->
<li v-for="(value, key, index) in myObject">
  {{ index }}. {{ key }} = {{ value }}
</li>
```

> 📝 参数顺序：`(值, 键, 索引)`

---

## 📋 九、遍历数字（较少用）

```html
<!-- 渲染 1 到 10 -->
<span v-for="n in 10" :key="n">{{ n }}</span>
```

> 注意：`n` 从 **1 开始**，不是 0！

---

## 🧪 十、在组件上使用 `v-for`

你可以直接在自定义组件上用 `v-for`，但**必须手动传递数据**：

```html
<!-- ❌ 不会自动传 item！ -->
<MyComponent v-for="item in items" :key="item.id" />

<!-- ✅ 正确：显式传 props -->
<MyComponent
  v-for="item in items"
  :key="item.id"
  :item-data="item"
  :index="index"
/>
```

> 💡 为什么？为了让组件更通用，不和 `v-for` 强绑定。

---

## 📚 总结：对照官方指南的关键映射

| 官方指南小节               | 新手理解要点                                           |
| -------------------------- | ------------------------------------------------------ |
| **v-for 与数组**           | 像 JS 的 `for...of`，支持 `(item, index)`              |
| **v-for 与对象**           | 参数顺序 `(value, key, index)`                         |
| **在 `<template>` 上使用** | 用于包裹多个元素，`key` 写在 template 上               |
| **key 的作用**             | 唯一身份标识，避免状态错乱，**不用 index**             |
| **v-for 与 v-if**          | 不要同元素使用，用计算属性或外层容器替代               |
| **响应式变更检测**         | 用 Vue 能侦测的方法（push/splice等），避免直接索引赋值 |
| **显示过滤/排序结果**      | 用计算属性，注意不要修改原数组                         |

---

✅ **学习建议**：

1. 先通读 [官方指南 · 列表渲染](https://cn.vuejs.org/guide/essentials/list.html)
2. 对照本笔记，重点理解 **`key` 的作用** 和 **`v-for` + `v-if` 的陷阱**
3. 动手写几个小例子（如 Todo List），亲自验证每个知识点

这样，你就能从“会用”进阶到“真正掌握”！
