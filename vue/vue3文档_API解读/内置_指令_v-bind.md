# **内置指令v-bind**
---


> ✅ **适用场景**：Vue 3 + `<script setup>`（组合式 API）  
> 📚 官方文档参考：[https://cn.vuejs.org/api/built-in-directives.html#v-bind](https://cn.vuejs.org/api/built-in-directives.html#v-bind)

---

## **一、什么是 `v-bind`？**

- **作用**：动态地将数据绑定到 HTML 元素的 **attribute（属性）** 或组件的 **prop** 上。
- **本质**：让 HTML attribute 的值变成“响应式的”——当数据变化时，attribute 自动更新。
- **缩写**：`:`（冒号），这是最常用的写法！

```html
<!-- 完整写法 -->
<img v-bind:src="imageSrc" />

<!-- 缩写写法（推荐） -->
<img :src="imageSrc" />
```

---

## **二、基本用法**

### **1. 绑定普通 attribute**

```html
<template>
  <div :id="dynamicId" :title="tooltipText">鼠标悬停看提示</div>
</template>

<script setup>
const dynamicId = 'main-content'
const tooltipText = '这是动态标题'
</script>
```

> 💡 注意：`id`、`class`、`style`、`href`、`disabled` 等都可以用 `v-bind` 动态设置。

---

### **2. 绑定布尔 attribute（如 `disabled`）**

```html
<button :disabled="isDisabled">提交</button>
```

- 当 `isDisabled` 为 `true` → `<button disabled>`
- 当 `isDisabled` 为 `false`/`null`/`undefined` → `<button>`（无 `disabled` 属性）

> ✅ Vue 会自动处理布尔 attribute 的存在与否，无需手动写 `'true'` 或 `'false'` 字符串。

---

## **三、动态 attribute 名（高级但实用）**

使用 `[ ]` 包裹变量作为 attribute 名：

```html
<template>
  <button :[attrName]="attrValue">动态属性</button>
</template>

<script setup>
const attrName = 'data-user-id' // 可以是任意字符串
const attrValue = 123
</script>
```

渲染结果：
```html
<button data-user-id="123">动态属性</button>
```

> ⚠️ 注意：`attrName` 必须是合法的 attribute 名称（不能包含空格或特殊字符）。

---

## **四、绑定对象：一次绑定多个 attribute**

```html
<template>
  <div v-bind="attrsObject">批量绑定</div>
</template>

<script setup>
const attrsObject = {
  id: 'container',
  class: 'box',
  title: '这是一个盒子',
  'data-role': 'widget'
}
</script>
```

等价于：
```html
<div id="container" class="box" title="这是一个盒子" data-role="widget">批量绑定</div>
```

> ✅ 非常适合封装通用组件时透传多个 props 或 attributes。

---

## **五、特殊绑定：`class` 和 `style`**

`v-bind` 对 `class` 和 `style` 有**增强支持**，可使用对象或数组语法。

### **1. Class 绑定**

```html
<template>
  <!-- 对象语法 -->
  <div :class="{ active: isActive, error: hasError }"></div>

  <!-- 数组语法 -->
  <div :class="[baseClass, { active: isActive }]"></div>
</template>

<script setup>
const isActive = true
const hasError = false
const baseClass = 'btn'
</script>
```

### **2. Style 绑定**

```html
<template>
  <!-- 对象语法 -->
  <div :style="{ color: textColor, fontSize: fontSize + 'px' }"></div>

  <!-- 数组语法（合并多个样式对象） -->
  <div :style="[baseStyle, overrideStyle]"></div>
</template>

<script setup>
const textColor = 'red'
const fontSize = 16
const baseStyle = { padding: '10px' }
const overrideStyle = { backgroundColor: 'yellow' }
</script>
```

---

## **六、绑定到组件的 props**

在自定义组件中，`v-bind` 用于传递 **props**：

```html
<!-- Parent.vue -->
<template>
  <UserCard 
    :name="userName"
    :age="userAge"
    :is-admin="isAdmin"
  />
</template>

<script setup>
import UserCard from './UserCard.vue'
const userName = '张三'
const userAge = 25
const isAdmin = true
</script>
```

> ✅ 子组件 `UserCard` 必须用 `defineProps` 声明这些 prop。

---

## 七、修饰符（Modifier）

虽然不常用，但 `v-bind` 有三个修饰符：

| 修饰符 | 作用 |
|--------|------|
| `.camel` | 将 kebab-case attribute 转为 camelCase（主要用于 DOM 模板中的 SVG） |
| `.prop`  | 强制绑定为 DOM **property** 而非 attribute（3.2+） |
| `.attr`  | 强制绑定为 DOM **attribute**（3.2+） |

### 示例：`.camel`

```html
<svg :view-box.camel="viewBoxData"></svg>
```

> 在 DOM 模板中，HTML 不区分大小写，`viewBox` 会被转成 `view-box`，用 `.camel` 可还原。

### 示例：`.prop`（缩写为 `.`）

```vue
<div :textContent.prop="message"></div>
<!-- 等价于 -->
<div .textContent="message"></div>
```

> 这会设置 `el.textContent = message`，而不是 `el.setAttribute('textcontent', message)`

---

## 八、常见误区 & 注意事项

| 问题 | 正确做法 |
|------|--------|
| ❌ 写成 `:class="isActive ? 'active' : null"` | ✅ 可以，但更推荐 `:class="{ active: isActive }"` |
| ❌ 用 `v-bind` 绑定事件（如 `:onclick`） | ❌ 错！事件要用 `v-on`（或 `@click`） |
| ❌ 把用户输入直接用 `v-bind:innerHTML` 渲染 | ❌ 危险！会导致 XSS 攻击，应使用 `v-text` 或插值 `{{}}` |

---

## 九、速查表（Cheat Sheet）

```vue
<!-- 基础绑定 -->
<img :src="imgUrl" />

<!-- 动态 attribute -->
<div :[key]="value"></div>

<!-- 多属性对象绑定 -->
<div v-bind="{ id, class, title }"></div>

<!-- class/style 增强 -->
<div :class="{ red: isRed }" :style="{ fontSize: size + 'px' }"></div>

<!-- 组件 prop -->
<MyComp :title="post.title" :published="post.isPublished" />

<!-- 修饰符 -->
<svg :view-box.camel="box"></svg>
<div .value="inputValue"></div>
```

---

## 十、总结一句话

> **`v-bind`（或 `:`）就是让 HTML 属性“活起来”——把 JS 变量的值动态塞进标签的属性里，并且能随数据变化自动更新。**

---