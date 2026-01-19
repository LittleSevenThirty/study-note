# **Vue 3 动态组件：`<component>` 与 `is` 属性详解**

## **概述**

在 Vue 应用中，有时我们希望**根据数据或用户操作，在多个组件之间动态切换显示**（例如选项卡、步骤向导、主题切换等）。这种“运行时决定渲染哪个组件”的能力，称为**动态组件**。

Vue 提供了两种方式实现动态组件：
1. 使用内置特殊元素 `<component>` 配合 `:is` prop；
2. 在原生 HTML 元素上使用内置特殊属性 `is`（主要用于兼容浏览器解析限制）。

本笔记将从零开始解释“什么是动态组件”“为什么需要它”，并详细说明 `<component>` 和 `is` 的用法、区别及常见场景，帮助你建立清晰的认知框架。

---

## **前置知识**

即使你是新手，也建议先了解以下基础概念：

1. **Vue 组件（Component）**  
   可复用的 UI 单元，如 `Button.vue`、`Modal.vue`。每个组件有自己独立的模板、逻辑和样式。

2. **组件注册**  
   - **局部注册**：在父组件的 `components` 选项或 `<script setup>` 中导入并使用。
   - **全局注册**：通过 `app.component()` 注册，整个应用可用。
   - 注册后，组件可通过**注册名**（字符串）在模板中使用。

3. **响应式数据**  
   使用 `ref()` 或 `data()` 定义的变量，当值变化时，视图自动更新。

4. **HTML 自定义元素限制**  
   浏览器对某些标签（如 `<table>` 内只能放 `<tr>`）有严格解析规则，直接写自定义组件可能被忽略。

---

## **关键词**

- 动态组件（Dynamic Component）
- `<component>`（内置特殊元素）
- `is`（内置特殊属性）
- `:is` 绑定
- 组件注册名（字符串）
- 组件对象（Component Definition）
- DOM 模板解析限制
- `vue:` 前缀

---

## **正式内容**

### 1. 什么是动态组件？为什么需要它？

**场景举例**：
- 一个设置页面有多个选项卡：“通用”、“账户”、“安全”，点击不同 tab 显示不同组件。
- 一个向导有三步，每步是一个组件，根据当前步骤 `currentStep` 决定显示哪一个。

**核心需求**：  
> **不写死组件标签，而是用一个变量控制“此刻该渲染哪个组件”**。

如果不用动态组件，你可能写出：
```vue
<General v-if="tab === 'general'" />
<Account v-else-if="tab === 'account'" />
<Security v-else />
```
这在组件少时可行，但**难以维护、扩展性差**。

✅ 动态组件提供更简洁、可扩展的方案。

---

### 2. `<component>`：Vue 的动态组件“容器”

#### 基本语法

```vue
<component :is="currentComponent" />
```

- `<component>` 是 Vue 提供的**内置特殊元素**，不是真实 DOM 标签，编译时会被替换。
- `:is` 是它的唯一 prop，值可以是：
  - **已注册的组件名（字符串）**
  - **组件对象（import 进来的组件定义）**

#### 示例 1：使用组件对象（推荐，尤其在 `<script setup>` 中）

```vue
<script setup>
import Home from './Home.vue'
import About from './About.vue'
import { ref } from 'vue'

const currentView = ref(Home) // 初始为 Home 组件对象
</script>

<template>
  <button @click="currentView = About">Go to About</button>
  <component :is="currentView" />
</template>
```

> ✅ 优点：无需额外注册，直接使用 import 的组件。

#### 示例 2：使用注册名（字符串）

```vue
<script>
import Home from './Home.vue'
import About from './About.vue'

export default {
  components: { Home, About }, // 局部注册
  data() {
    return { currentView: 'Home' } // 字符串 'Home'
  }
}
</script>

<template>
  <component :is="currentView" />
</template>
```

> ⚠️ 注意：必须先通过 `components` 注册，否则 Vue 找不到 `'Home'` 对应的组件。

#### 示例 3：渲染原生 HTML 元素

`:is` 也可以是 HTML 标签名：

```vue
<component :is="href ? 'a' : 'span'" :href="href">
  Click me
</component>
```

> ✅ 用途：根据条件动态决定渲染 `<a>` 还是 `<span>`。

---

### 3. `is` 属性：用于原生 HTML 元素的特殊情况

#### 问题背景：DOM 模板解析限制

当你在**浏览器原生 HTML 中写模板**（如 `<div id="app"><my-component></my-component></div>`），浏览器会先解析 HTML，再交给 Vue。

但某些 HTML 结构有严格限制，例如：

```html
<!-- ❌ 无效！浏览器会忽略 <blog-post-row> -->
<table>
  <blog-post-row></blog-post-row>
</table>
```

因为 `<table>` 内只允许特定标签（如 `<tr>`）。

#### 解决方案：使用 `is` 属性

```html
<!-- ✅ 正确：告诉浏览器这是 <tr>，但由 Vue 渲染为组件 -->
<table>
  <tr is="vue:blog-post-row"></tr>
</table>
```

- `is="vue:组件名"`：`vue:` 前缀告诉 Vue “这不是原生 custom element，而是我的组件”。
- 必须使用**已注册的组件名（字符串）**，不能传组件对象。

> 💡 注意：此用法**仅在 DOM 内模板（非单文件组件）中需要**。在 `.vue` 文件中，直接写 `<BlogPostRow />` 即可，无需 `is`。

---

### 4. `<component>` vs `is`：关键区别总结

| 特性 | `<component :is="...">` | 原生元素上的 `is="..."` |
|------|------------------------|------------------------|
| **使用位置** | 任意地方（推荐在 `.vue` 文件中） | 仅限受 HTML 限制的原生标签内（如 `<table>`、`<select>`） |
| **值类型** | 组件对象 或 注册名（字符串） | 仅注册名（字符串），且需加 `vue:` 前缀 |
| **是否 Vue 特有** | 是（编译时替换） | 否（利用 HTML 原生 `is` 属性） |
| **典型场景** | 选项卡、动态布局、路由过渡 | 在 `<table>` 等结构中插入组件 |

---

### 5. 常见组合：动态组件 + 过渡动画

动态组件常与 `<Transition>` 配合使用：

```vue
<Transition name="fade" mode="out-in">
  <component :is="currentView" />
</Transition>
```

每次 `currentView` 变化，都会触发动画。

> ✅ 这也是路由过渡（`<router-view>` 插槽）的底层原理。

---

### 6. 注意事项

- **单一根节点**：动态组件本身仍需遵守 Vue 组件规则——模板必须有单一根元素。
- **Props 和事件**：可像普通组件一样传递：
  ```vue
  <component :is="MyComp" :title="msg" @click="handle" />
  ```
- **避免直接绑定原始标签名到响应式变量**（除非你明确知道自己在做什么）：
  ```js
  // 不推荐：容易混淆组件与 HTML 元素
  current = 'div'
  ```

---

## **参考**

- [Vue 3 官方文档 - 动态组件（基础指南）](https://cn.vuejs.org/guide/essentials/component-basics.html#dynamic-components)
- [Vue 3 API 文档 - 内置特殊元素 `<component>`](https://cn.vuejs.org/api/built-in-special-elements.html#component)
- [Vue 3 API 文档 - 内置特殊属性 `is`](https://cn.vuejs.org/api/built-in-special-attributes.html#is)
- [Vue 3 官方文档 - DOM 内模板解析注意事项](https://cn.vuejs.org/guide/essentials/component-basics.html#dom-template-parsing-caveats)