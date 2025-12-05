# **Vue3文档解读emit_组件事件**
## **一、为什么需要 `emit`？**

- 在 Vue 中，**数据流是单向的**：父组件 → 子组件（通过 `props`）
- 但子组件有时需要 **通知父组件发生了某些事情**（比如“用户点击了提交按钮”、“输入内容改变了”）
- 这时就需要 **子组件主动“抛出”一个事件**，由父组件“监听”并处理
- 这个“抛出”动作就是 `emit`

> 💡 类比：`props` 是父给子传话；`emit` 是子给父传话

---

## 二、基础用法：如何触发和监听事件？

### 1. 子组件：使用 `$emit` 或 `emit()` 触发事件

#### ✅ 在模板中直接使用 `$emit`
```vue
<!-- Child.vue -->
<template>
  <button @click="$emit('close')">关闭</button>
</template>

<script setup>
// 不需要额外定义，可以直接 $emit
</script>
```

> ⚠️ 注意：`$emit` 是模板中的内置方法，只能在 `<template>` 里用

---

#### ✅ 在 `<script setup>` 中使用 `defineEmits`

```vue
<!-- Child.vue -->
<script setup>
const emit = defineEmits(['close', 'update:title'])

function handleClose() {
  emit('close') // 触发 close 事件
}

function handleUpdate() {
  emit('update:title', '新标题') // 带参数
}
</script>

<template>
  <button @click="handleClose">关闭</button>
  <button @click="handleUpdate">更新标题</button>
</template>
```

> ✅ `defineEmits` 是编译宏（compiler macro），**不需要 import**，只能在 `<script setup>` 顶层使用

---

### 2. 父组件：用 `@事件名` 监听

```vue
<!-- Parent.vue -->
<template>
  <Child 
    @close="onClose" 
    @update:title="onTitleUpdate"
  />
</template>

<script setup>
import Child from './Child.vue'

function onClose() {
  console.log('子组件要求关闭')
}

function onTitleUpdate(newTitle) {
  console.log('新标题：', newTitle)
}
</script>
```

> 🔁 事件名自动转换：
> - 子组件 emit 的事件名推荐用 **camelCase**（如 `updateTitle`）
> - 父组件监听时可用 **kebab-case**（如 `@update-title`）
> - Vue 会自动匹配（类似 props）

---

## 三、`defineEmits` 的两种写法

### 1. 数组形式（简单声明事件名）
```ts
const emit = defineEmits(['submit', 'cancel'])
```
- 仅声明事件名称
- 无参数校验
- 推荐用于简单场景

---

### 2. 对象形式（带参数验证）
```ts
const emit = defineEmits({
  submit(payload: { email: string; password: string }) {
    // 返回 true 表示有效，false 表示无效（会警告）
    if (payload.email && payload.password) {
      return true
    } else {
      console.warn('submit 事件参数不合法')
      return false
    }
  },
  cancel: null // 不校验
})
```
- 可对事件携带的参数做运行时校验
- 校验函数返回 `true`/`false`
- 类似 `props` 的 validator

---

## 四、TypeScript 下的类型安全写法（强烈推荐）

### 方式 1：使用函数重载签名（最常用）
```ts
const emit = defineEmits<{
  (e: 'close'): void
  (e: 'update:title', title: string): void
  (e: 'change', id: number, value: string): void
}>()
```

### 方式 2：使用具名元组语法（Vue 3.3+）
```ts
const emit = defineEmits<{
  close: []
  'update:title': [title: string]
  change: [id: number, value: string]
}>()
```

> ✅ 优势：
> - 编辑器能智能提示事件名和参数
> - 调用 `emit` 时如果参数不对，TS 会报错
> - 自动生成运行时声明（无需重复写数组或对象）

> ❌ 注意：**不能同时使用类型声明 + 运行时声明**
> ```ts
> // ❌ 错误！会报错
> const emit = defineEmits<{ ... }>(['close'])
> ```

---

## 五、事件参数传递

- 所有传给 `emit` 的额外参数都会原样传给监听器
```js
emit('foo', 1, 'hello', { a: 1 })
```
父组件监听：
```vue
<Child @foo="(a, b, c) => { /* a=1, b='hello', c={a:1} */ }" />
```

---

## 六、重要注意事项

### 1. **组件事件不会冒泡**
- 只能监听**直接子组件**发出的事件
- 不能像 DOM 事件那样冒泡到祖父组件
- 跨层级通信请用：
  - `provide/inject`
  - 全局状态（如 Pinia）
  - 事件总线（不推荐，除非简单项目）

### 2. **`emits` 选项的作用**
- 显式声明事件有助于：
  - 代码自文档化
  - 让 Vue 区分“透传 attribute”和“监听的事件”
  - 避免第三方库触发的同名 DOM 事件被误认为是组件事件

### 3. **如果在 `emits` 中声明了原生事件名（如 `click`）**
- 父组件的 `@click` 将**只响应组件 emit 的 click**，不再响应原生 DOM click
- 除非你确实想覆盖原生行为，否则不要这样做

---

## 七、对比：选项式 API vs `<script setup>`

| 场景 | 选项式 API | `<script setup>` |
|------|-----------|------------------|
| 定义 emit | `emits: ['xxx']` | `defineEmits(['xxx'])` |
| 触发事件 | `this.$emit('xxx')` | `emit('xxx')` |
| 带参数 | `this.$emit('xxx', arg)` | `emit('xxx', arg)` |
| TS 类型 | `emits: { xxx(...) { ... } }` | `defineEmits<{ (e: 'xxx', ...): void }>()` |

> 💡 `<script setup>` 更简洁，类型推导更强

---

## 八、常见错误 & 解决方案

### ❌ 错误 1：在 `<script setup>` 里直接用 `$emit`
```ts
// ❌ 报错：$emit is not defined
function handleClick() {
  $emit('test') // 模板外不能用 $emit
}
```
✅ 正确做法：用 `defineEmits` 获取 `emit` 函数

---

### ❌ 错误 2：在函数内部调用 `defineEmits`
```ts
// ❌ 编译错误
function bad() {
  const emit = defineEmits(['x']) // 必须在顶层！
}
```
✅ 正确：`defineEmits` 必须在 `<script setup>` **顶层作用域**

---

### ❌ 错误 3：混合使用类型声明和运行时声明
```ts
// ❌ 报错
const emit = defineEmits<{ ... }>(['a', 'b'])
```
✅ 正确：二选一

---

## 九、最佳实践建议

1. **始终显式声明 `emits`**（用 `defineEmits`）
   - 提高可读性和维护性
2. **优先使用 TypeScript 类型声明**
   - 获得完整类型安全
3. **事件命名语义清晰**
   - 如 `submit`, `delete-item`, `update:modelValue`
4. **避免过度使用 emit 跨多层通信**
   - 超过两层考虑状态管理

---

## 十、速查模板（可直接复制使用）

### 基础 emit（JS）
```vue
<script setup>
const emit = defineEmits(['confirm', 'cancel'])
</script>

<template>
  <button @click="emit('confirm')">确定</button>
  <button @click="emit('cancel')">取消</button>
</template>
```

### 带参数 + TS 类型
```vue
<script setup lang="ts">
const emit = defineEmits<{
  (e: 'select', id: number): void
  (e: 'search', keyword: string, page: number): void
}>()

function onSelect(id: number) {
  emit('select', id)
}
</script>
```

### 带验证（运行时）
```ts
const emit = defineEmits({
  login(user: { name: string; age: number }) {
    return !!user.name && user.age > 0
  }
})
```

---

## 总结一句话

> **`emit` 是子组件向父组件“喊话”的方式，通过 `defineEmits` 声明和触发，父组件用 `@事件名` 监听，配合 TypeScript 可实现类型安全通信。**

---

希望这份笔记能帮你彻底掌握 Vue 3 的 `emit`！你可以把它保存为 Markdown 文件，随时查阅。如有后续疑问（比如和 `v-model` 结合、与 `defineModel` 的关系等），也可以继续问我！