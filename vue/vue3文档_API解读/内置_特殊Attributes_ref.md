# **Vue 3 中特殊 Attributes 之 `ref` 学习笔记**

---

## 概述

在 Vue 3 中，`ref` 是一个**内置的特殊 attribute（特殊属性）**，用于在模板中**注册对 DOM 元素或子组件实例的引用**。通过 `ref`，你可以在 JavaScript 代码中直接访问真实的 DOM 节点或子组件对象，从而执行原生操作（如聚焦输入框、读取元素尺寸、调用子组件方法等）。

简单来说：  
> **`ref` 就像是给模板中的某个元素“贴标签”，让你在脚本中能准确找到并操作它。**

需要注意的是，`ref` 本身只是模板中的一个 attribute（写在 HTML 标签上），而如何在逻辑中获取这个引用，则取决于你使用的是**组合式 API**还是**选项式 API**。

---

## 关键词

- **`ref`（模板 attribute）**：写在模板标签上的特殊属性，用于标记引用目标。
- **模板引用（Template Ref）**：通过 `ref` 注册的对 DOM 或组件的引用。
- **组合式 API**：Vue 3 推荐的逻辑组织方式，使用 `<script setup>` 和响应式函数。
- **选项式 API**：Vue 2 风格的写法，使用 `data`、`methods`、`mounted` 等选项。
- **`useTemplateRef()`**：Vue 3.5+ 新增的组合式 API 函数，用于在 `<script setup>` 中获取模板引用。
- **`this.$refs`**：选项式 API 中访问所有模板引用的对象。

---

## 前置准备

### 1. 确定 API 风格
你需要明确当前项目使用的是哪种 API 风格：

| API 风格 | 获取 `ref` 引用的方式 |
|----------|------------------------|
| **组合式 API（推荐）** | 使用 `useTemplateRef('name')`（Vue 3.5+）或通过 `ref` 变量直接绑定（传统方式） |
| **选项式 API** | 通过 `this.$refs.name` 访问 |

> 📌 本文将重点讲解 **组合式 API 下的现代用法（`useTemplateRef`）**，同时简要对比选项式 API。

### 2. 安装与版本要求
- Vue 版本 ≥ **3.5.0**（若使用 `useTemplateRef`）
- 若使用旧版 Vue 3（<3.5），仍可通过 `ref()` + 模板绑定实现类似功能（见附录说明）

---

## 核心讲解

### 1. `ref` 在模板中的作用

在模板中，`ref` 是一个特殊的 attribute，语法如下：

```html
<input ref="inputRef" />
<MyComponent ref="childRef" />
```

- 当 `ref` 用于**普通 HTML 元素**（如 `<div>`、`<input>`）时，引用指向**真实的 DOM 元素**。
- 当 `ref` 用于**子组件**时，引用指向**该子组件的实例对象**（可调用其暴露的方法或访问其数据）。

> ⚠️ 注意：`ref` 的值必须是**字符串字面量**（如 `ref="myInput"`），不能是动态表达式（如 `:ref="someVar"` 在模板 attribute 中不被支持——但可通过函数形式实现，见高级用法）。

---

### 2. 如何在逻辑中获取引用？

#### ✅ 组合式 API（Vue 3.5+ 推荐方式）：使用 `useTemplateRef`

```vue
<script setup>
import { onMounted } from 'vue'
import { useTemplateRef } from 'vue' // Vue 3.5+

// 声明一个模板引用，名称需与模板中的 ref 值一致
const inputRef = useTemplateRef('inputRef')

onMounted(() => {
  // 组件挂载后，inputRef 自动指向 <input> 元素
  inputRef.value.focus() // 聚焦输入框
})
</script>

<template>
  <input ref="inputRef" placeholder="点击后自动聚焦" />
</template>
```

> 🔍 `useTemplateRef('name')` 返回一个 **只读的 ref 对象**，其 `.value` 在组件挂载后自动填充为对应元素或组件实例。

#### 🔄 选项式 API 方式（兼容 Vue 2/3）

```js
export default {
  mounted() {
    // 通过 this.$refs 访问
    this.$refs.inputRef.focus()
  }
}
```

```html
<template>
  <input ref="inputRef" />
</template>
```

> 💡 `this.$refs` 是一个对象，键名即为模板中 `ref` 的值。

---

### 3. 使用注意事项

| 问题 | 说明 |
|------|------|
| **何时能访问 `ref`？** | 必须在 **组件挂载之后**（如 `onMounted` 或 `mounted` 中）。在 `setup()` 或 `created` 阶段访问会得到 `null`。 |
| **`ref` 是响应式的吗？** | ❌ **不是**。`useTemplateRef()` 返回的 ref **不会触发视图更新**，也不应直接用于模板绑定（如 `{{ inputRef }}` 无意义）。 |
| **多个同名 `ref` 会怎样？** | 后声明的会覆盖前面的，导致不可预测行为。**确保同一父组件下 `ref` 名称唯一**。 |
| **能否用于 `v-for`？** | 可以，但需注意：每个元素都会注册引用，通常配合函数形式更安全（见高级用法）。 |

---

### 4. 典型使用场景与示例

#### 示例 1：聚焦输入框（最常见场景）

```vue
<script setup>
import { onMounted } from 'vue'
import { useTemplateRef } from 'vue'

const searchInput = useTemplateRef('searchInput')

onMounted(() => {
  searchInput.value?.focus()
})
</script>

<template>
  <input ref="searchInput" type="text" placeholder="搜索..." />
</template>
```

#### 示例 2：调用子组件方法

假设子组件通过 `defineExpose` 暴露了方法：

```vue
<!-- Child.vue -->
<script setup>
const play = () => {
  console.log('播放视频')
}
// 暴露方法给父组件
defineExpose({ play })
</script>
```

父组件调用：

```vue
<script setup>
import { onMounted } from 'vue'
import { useTemplateRef } from 'vue'
import Child from './Child.vue'

const videoPlayer = useTemplateRef('videoPlayer')

onMounted(() => {
  videoPlayer.value.play() // 调用子组件的 play 方法
})
</script>

<template>
  <Child ref="videoPlayer" />
</template>
```

#### 示例 3：获取元素尺寸

```vue
<script setup>
import { onMounted } from 'vue'
import { useTemplateRef } from 'vue'

const box = useTemplateRef('box')

onMounted(() => {
  const rect = box.value.getBoundingClientRect()
  console.log('宽度:', rect.width)
})
</script>

<template>
  <div ref="box" style="width: 200px; height: 100px; background: lightblue;"></div>
</template>
```

---

## 高级提示（可选了解）

- **函数形式的 `ref`**：你也可以将 `ref` 设为函数，用于动态控制引用存储位置：
  ```html
  <input :ref="(el) => inputEl = el" />
  ```
  此时 `inputEl` 直接是 DOM 元素（非 ref 对象），但需自行管理响应性。

- **与响应式 `ref()` 的区别**：不要混淆模板 attribute `ref` 和 `import { ref } from 'vue'`。后者用于创建响应式变量，前者用于标记模板元素。

---

## 总结速查表

| 问题 | 答案 |
|------|------|
| **`ref` 是什么？** | 模板中的特殊 attribute，用于标记元素或组件以便脚本访问。 |
| **如何获取引用？** | 组合式 API 用 `useTemplateRef('name')`；选项式 API 用 `this.$refs.name`。 |
| **什么时候能用？** | 组件挂载后（`onMounted` / `mounted`）。 |
| **能用于响应式吗？** | 不能，它仅用于副作用操作（DOM 操作、调用方法等）。 |
| **最佳实践** | 仅在必要时使用（如第三方库集成、焦点管理），避免过度依赖 DOM 操作。 |

---