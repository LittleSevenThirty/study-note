# **Vue 3 组合式 API 中 `onMounted` 生命周期钩子学习笔记**

---

## 概述

在 Vue 3 的组合式 API（Composition API）中，`onMounted` 是一个**生命周期钩子函数**，用于在组件**挂载完成后**执行特定的逻辑。所谓“挂载完成”，是指 Vue 已经将组件的模板编译为真实的 DOM 元素，并成功插入到页面中。

简单来说：  
> **当你需要在组件渲染到页面后立即执行某些操作（比如获取 DOM 元素、启动定时器、发起网络请求等），就应该使用 `onMounted`。**

它替代了 Vue 2 选项式 API 中的 `mounted()` 钩子，但用法更灵活，通常与 `<script setup>` 语法配合使用。

---

## 关键词

- **组合式 API（Composition API）**：Vue 3 提供的一种组织组件逻辑的新方式。
- **生命周期钩子（Lifecycle Hook）**：在组件特定阶段自动调用的函数。
- **挂载（Mount）**：组件首次被渲染并插入到 DOM 中的过程。
- **副作用（Side Effect）**：依赖于 DOM 或浏览器环境的操作（如操作元素、添加事件监听器等）。
- **`<script setup>`**：Vue 3 中编写组合式 API 的推荐语法。

---

## 前置准备

要正确使用 `onMounted`，你需要：

1. **使用 Vue 3**（版本 ≥ 3.0）。
2. 在单文件组件（`.vue` 文件）中使用 `<script setup>` 语法（推荐），或在 `setup()` 函数中使用。
3. 从 `vue` 中导入 `onMounted`：
   ```js
   import { onMounted } from 'vue'
   ```

> ⚠️ 注意：所有生命周期钩子（包括 `onMounted`）**必须在 `setup()` 或 `<script setup>` 中同步调用**，不能在异步函数、条件语句或嵌套函数中调用。

---

## 核心讲解

### 1. 何时触发 `onMounted`？

- 组件的模板已编译为 DOM。
- DOM 已被插入到父容器中（即页面上可见）。
- 所有**同步子组件**也已完成挂载（异步组件或 `<Suspense>` 内的组件不在此列）。

此时你可以安全地访问组件的 DOM 元素或执行依赖浏览器环境的操作。

---

### 2. 典型使用场景

| 场景 | 说明 |
|------|------|
| **操作 DOM 元素** | 通过 `ref` 获取真实 DOM 并进行操作（如聚焦输入框、初始化第三方库）。 |
| **发起数据请求** | 在组件显示后立即加载数据（虽然也可在 `setup` 中直接请求，但若需依赖 DOM 则放在此处更合适）。 |
| **绑定全局事件监听器** | 如 `window.addEventListener('resize', ...)`，需在组件卸载时清理（配合 `onUnmounted`）。 |
| **启动定时器或动画** | 如 `setInterval`、`requestAnimationFrame` 等。 |

---

### 3. 示例代码

#### 示例 1：通过 `ref` 访问 DOM 元素

```vue
<script setup>
import { ref, onMounted } from 'vue'

const titleRef = ref(null)

onMounted(() => {
  // 此时 DOM 已存在，可以安全访问
  console.log(titleRef.value) // <h1>Hello Vue!</h1>
  titleRef.value.style.color = 'blue'
})
</script>

<template>
  <h1 ref="titleRef">Hello Vue!</h1>
</template>
```

> ✅ 说明：`ref` 在模板中绑定到元素后，`onMounted` 中可通过 `.value` 获取真实 DOM。

---

#### 示例 2：发起网络请求（结合响应式数据）

```vue
<script setup>
import { ref, onMounted } from 'vue'

const user = ref(null)

onMounted(async () => {
  const res = await fetch('/api/user')
  user.value = await res.json()
})
</script>

<template>
  <div v-if="user">
    欢迎，{{ user.name }}！
  </div>
  <div v-else>
    加载中...
  </div>
</template>
```

> 💡 提示：虽然数据请求也可以在 `setup` 顶层直接写，但如果逻辑复杂或需确保 DOM 已就绪，放在 `onMounted` 更清晰。

---

#### 示例 3：与 `onUnmounted` 配合清理资源

```vue
<script setup>
import { onMounted, onUnmounted } from 'vue'

let timerId = null

onMounted(() => {
  timerId = setInterval(() => {
    console.log('Tick...')
  }, 1000)
})

onUnmounted(() => {
  clearInterval(timerId) // 防止内存泄漏
})
</script>
```

> ✅ 最佳实践：在 `onMounted` 中创建的副作用（如定时器、监听器），应在 `onUnmounted` 中清理。

---

## 常见误区提醒

- ❌ **不要在 `onMounted` 中修改会触发重新渲染的状态来“修正”初始值**——这会导致不必要的更新。应尽量在 `setup` 中初始化状态。
- ❌ **不要在服务端渲染（SSR）中依赖 `onMounted` 执行关键逻辑**——它只在客户端运行。
- ✅ **如果只是获取数据且不依赖 DOM，可直接在 `setup` 中请求**，无需等待挂载。

---
