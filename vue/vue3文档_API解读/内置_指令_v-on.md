# **v-on指令**
---
# 📝 Vue 3 内置指令：`v-on` 详解笔记

> ✅ **适用场景**：Vue 3 + `<script setup>`（组合式 API）  
> 📚 官方文档参考：[https://cn.vuejs.org/api/built-in-directives.html#v-on](https://cn.vuejs.org/api/built-in-directives.html#v-on)

---

## 一、什么是 `v-on`？

- **作用**：监听 DOM 事件或组件自定义事件，并在事件触发时执行 JavaScript 代码。
- **缩写**：`@`（这是最常用的写法！）
- **本质**：将事件处理器（函数）绑定到元素上。

```html
<!-- 完整写法 -->
<button v-on:click="handleClick">点击</button>

<!-- 缩写写法（推荐） -->
<button @click="handleClick">点击</button>
```

---

## 二、基本用法

### 1. 绑定方法（最常见）

```html
<template>
  <button @click="sayHello">打招呼</button>
</template>

<script setup>
function sayHello() {
  alert('Hello from Vue!')
}
</script>
```

> ✅ 方法名直接写，不需要加括号（除非要传参）。

---

### 2. 内联语句（带参数或简单逻辑）

```html
<template>
  <button @click="count++">点我加 1（当前：{{ count }}）</button>
  <button @click="greet('小明')">向小明问好</button>
</template>

<script setup>
import { ref } from 'vue'

const count = ref(0)

function greet(name) {
  alert(`你好，${name}！`)
}
</script>
```

> 💡 内联语句中可以访问：
> - 组件的响应式变量（如 `count`）
> - 方法（如 `greet`）
> - 特殊变量 `$event`（见下文）

---

### 3. 访问原生事件对象 `$event`

当使用内联语句时，如果需要访问原生 DOM 事件对象，用 `$event`：

```html
<template>
  <input @input="handleInput($event)" />
</template>

<script setup>
function handleInput(event) {
  console.log('输入内容：', event.target.value)
}
</script>
```

> ⚠️ 如果直接写方法名（如 `@input="handleInput"`），Vue 会自动把原生事件作为第一个参数传入，无需 `$event`。

---

## 三、事件修饰符（Modifier）

`v-on` 提供一系列 **修饰符**，用于更方便地处理事件细节，避免手动调用 `event.preventDefault()` 等。

| 修饰符 | 作用 |
|--------|------|
| `.stop` | 阻止事件冒泡（`event.stopPropagation()`） |
| `.prevent` | 阻止默认行为（`event.preventDefault()`） |
| `.capture` | 在捕获阶段触发处理器 |
| `.self` | 只有事件从元素本身触发才执行（不包括子元素） |
| `.once` | 事件最多触发一次 |
| `.passive` | 以 `{ passive: true }` 添加监听器（提升滚动性能） |

### 常见示例：

```html
<template>
  <!-- 阻止表单提交刷新页面 -->
  <form @submit.prevent="onSubmit">
    <button type="submit">提交</button>
  </form>

  <!-- 点击只触发一次 -->
  <button @click.once="showTip">点我显示提示（仅一次）</button>

  <!-- 阻止链接跳转 -->
  <a href="https://example.com" @click.prevent>这个链接不会跳转</a>

  <!-- 阻止冒泡 -->
  <div @click="outer">
    <button @click.stop="inner">点我不会触发外层 div 的点击</button>
  </div>
</template>

<script setup>
function onSubmit() { console.log('表单提交') }
function showTip() { alert('这是提示') }
function outer() { console.log('外层被点击') }
function inner() { console.log('内层被点击') }
</script>
```

> ✅ 修饰符可以链式使用：`@click.stop.prevent`

---

## 四、按键修饰符（用于键盘事件）

在 `keyup` / `keydown` 中，可指定具体按键：

```html
<template>
  <!-- 只在按 Enter 时触发 -->
  <input @keyup.enter="submitForm" />

  <!-- 支持常用别名：.enter .tab .delete .esc .space .up .down 等 -->
  <input @keyup.esc="clearInput" />
</template>

<script setup>
function submitForm() { console.log('提交表单') }
function clearInput() { console.log('清空输入') }
</script>
```

> 🔤 也可以直接使用键名（需 kebab-case）：
> ```html
> <input @keyup.page-down="nextPage" />
> ```

---

## 五、鼠标按钮修饰符

限定鼠标按键：

```html
<button @click.left="leftClick">左键</button>
<button @click.right="rightClick">右键</button>
<button @click.middle="middleClick">中键</button>
```

---

## 六、监听组件自定义事件

`v-on` 也可用于监听 **子组件 emit 的自定义事件**。

**子组件（Child.vue）**
```html
<template>
  <button @click="notifyParent">通知父组件</button>
</template>

<script setup>
const emit = defineEmits(['custom-event'])

function notifyParent() {
  emit('custom-event', '来自子组件的数据')
}
</script>
```

**父组件**
```html
<template>
  <Child @custom-event="handleCustomEvent" />
</template>

<script setup>
import Child from './Child.vue'

function handleCustomEvent(payload) {
  console.log('收到子组件消息：', payload) // 输出：来自子组件的数据
}
</script>
```

> ✅ 这是父子组件通信的核心方式之一。

---

## 七、动态事件名（高级）

使用 `[ ]` 动态指定事件类型：

```html
<template>
  <button @[event]="handler">动态事件</button>
</template>

<script setup>
const event = 'click' // 也可以是 'mouseover' 等
function handler() { alert('触发了动态事件！') }
</script>
```

---

## 八、对象语法（批量绑定多个事件）

```html
<template>
  <button v-on="{
    click: onClick,
    mouseenter: onHover,
    keyup: onKeyup
  }">多事件绑定</button>
</template>

<script setup>
function onClick() { /* ... */ }
function onHover() { /* ... */ }
function onKeyup() { /* ... */ }
</script>
```

> ⚠️ 注意：**对象语法不支持修饰符**（如 `.prevent`）。

---

## 九、常见误区 & 注意事项

| 问题 | 正确做法 |
|------|--------|
| ❌ 写成 `@click="myFunction()"`（带括号无参数） | ❌ 这会导致组件初始化时立即执行！应写 `@click="myFunction"` |
| ❌ 在内联语句中忘记 `$event` | ✅ 如需事件对象，必须显式写 `$event` |
| ❌ 试图用 `v-on` 监听非标准事件（如 `@myCustomEvent`）但未 emit | ✅ 自定义事件必须由子组件主动 `emit` 才能触发 |
| ❌ 混淆 `v-on` 和 `v-bind` | ✅ `v-on` 用于**事件**（@click），`v-bind` 用于**属性**（:src） |

---

## 十、速查表（Cheat Sheet）

```html
<!-- 基础 -->
<button @click="doSomething">点击</button>

<!-- 传参 -->
<button @click="handle(id)">处理 {{ id }}</button>

<!-- 修饰符 -->
<form @submit.prevent></form>
<button @click.stop.prevent="cancel"></button>

<!-- 键盘 -->
<input @keyup.enter="submit" />

<!-- 组件事件 -->
<Child @update="onUpdate" />

<!-- 动态事件 -->
<button @[eventType]="handler"></button>
```

---

## 十一、总结一句话

> **`v-on`（或 `@`）是 Vue 中监听用户交互（点击、输入、提交等）并作出响应的核心指令——它让静态页面“活”起来。**

---