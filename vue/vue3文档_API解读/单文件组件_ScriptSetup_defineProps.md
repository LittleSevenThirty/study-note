# 📝 Vue 组件传参（Props）学习笔记

> 适用版本：Vue 3（尤其是 `<script setup>` 语法）

---

## 一、什么是 Props？

- **Props（属性）** 是父组件向子组件**传递数据**的方式。
- 子组件通过 `props` 接收这些数据，并在模板或逻辑中使用。
- **核心原则：单向数据流** → 父传子，子不能直接改父的数据。

> ✅ 类比：就像函数的参数。  
> ```js
> function ChildComponent(props) { ... }
> ```

---

## 二、基础使用步骤（最简示例）

### 1. 父组件（Parent.vue）
```vue
<template>
  <ChildComponent message="Hello from parent!" />
</template>

<script setup>
import ChildComponent from './ChildComponent.vue'
</script>
```

### 2. 子组件（ChildComponent.vue）
```vue
<template>
  <p>{{ message }}</p>
</template>

<script setup>
// 声明接收哪些 props
const props = defineProps(['message'])
</script>
```

> 🔍 注意：
> - `defineProps` 是 Vue 3 的宏（macro），只能在 `<script setup>` 中使用。
> - `['message']` 表示只接收一个叫 `message` 的 prop。

---

## 三、声明 Props 的两种方式

### 方式 1️⃣：数组形式（简单，只指定名字）
```js
const props = defineProps(['title', 'count'])
```
✅ 适合快速开发，不校验类型。

---

### 方式 2️⃣：对象形式（推荐！可校验类型、设默认值等）
```js
const props = defineProps({
  title: String,        // 必须是字符串
  count: {
    type: Number,       // 类型
    required: true,     // 必填
    default: 0          // 默认值（非必填时用）
  },
  userInfo: {
    type: Object,
    default() {
      return { name: 'Guest' } // 对象/数组必须用函数返回！
    }
  }
})
```

> ⚠️ 重要规则：
> - `default` 如果是 **对象或数组**，必须写成 **函数**（避免多个组件实例共享同一个引用）。
> - `required: true` 表示调用组件时必须传这个 prop。

---

## 四、传递不同类型的数据

| 数据类型 | 父组件写法 | 说明 |
|--------|-----------|------|
| 字符串 | `<Child msg="hello" />` | 静态值，不用 `:` |
| 数字   | `<Child :age="18" />` | 必须加 `:`，否则会被当字符串 |
| 布尔值 | `<Child visible />` | 不写值 = `true`<br>`<Child :visible="false" />` 显式传 false |
| 数组   | `<Child :list="[1,2,3]" />` | 必须用 `:` |
| 对象   | `<Child :user="{name:'Tom'}" />` | 必须用 `:` |

> 💡 记忆口诀：**除了纯字符串，其他都要加 `:`（即 `v-bind`）**

---

## 五、动态传多个 Prop（批量绑定）

如果有一个对象包含多个 prop：

```js
// Parent.vue
const post = {
  id: 1,
  title: 'My Post',
  published: true
}
```

模板中可用 `v-bind` 一次性传入：

```vue
<ChildComponent v-bind="post" />
<!-- 等价于 -->
<ChildComponent :id="post.id" :title="post.title" :published="post.published" />
```

---

## 六、子组件不能直接修改 props！

❌ 错误做法：
```js
const props = defineProps(['count'])
props.count++ // 报错！props 是只读的！
```

✅ 正确做法：

### 场景 1：想把 prop 当初始值，后续自己维护
```js
import { ref } from 'vue'
const props = defineProps(['initialCount'])
const count = ref(props.initialCount) // 转为自己的响应式数据
```

### 场景 2：想对 prop 做转换（如格式化）
```js
import { computed } from 'vue'
const props = defineProps(['rawSize'])
const displaySize = computed(() => props.rawSize.trim().toUpperCase())
```

> 📌 原则：**子组件要改数据？先拷贝 or 用 computed，或者通知父组件改（emit 事件）**

---

## 七、特殊规则：Boolean 类型的自动转换

如果你声明了 `Boolean` 类型的 prop：

```js
defineProps({ disabled: Boolean })
```

那么以下写法会自动转换：

```vue
<MyButton disabled />        <!-- 相当于 :disabled="true" -->
<MyButton />                <!-- 相当于 disabled=false -->
<MyButton :disabled="false" /> <!-- 显式传 false -->
```

> ⚠️ 注意：如果同时允许 `String` 和 `Boolean`，顺序很重要！
> ```js
> disabled: [Boolean, String] // ✅ 布尔转换生效
> disabled: [String, Boolean] // ❌ 布尔转换失效！
> ```

---

## 八、TypeScript 用户（可选）

如果你用 TS，可以用类型注解声明 props：

```vue
<script setup lang="ts">
interface Props {
  title?: string
  likes: number
}
const props = defineProps<Props>()
</script>
```

Vue 会自动推导出运行时校验（如 `likes` 是必填 number）。

---

## 九、常见新手疑问解答

### Q1：为什么我传数字没加 `:`，结果变成字符串了？
A：HTML 属性值默认都是字符串。`<Comp age="18" />` → `age === "18"`。要用 `:age="18"` 才是数字。

### Q2：props 变了，子组件怎么响应？
A：props 本身就是响应式的！你可以在 `computed`、`watch` 或模板中直接用，Vue 会自动更新。

### Q3：能传函数吗？
A：可以！但通常建议用 **事件（emit）** 替代。如果非要传函数：
```js
// Parent
<Child :on-click="handleClick" />

// Child
const props = defineProps({ onClick: Function })
props.onClick()
```

---

## 十、最佳实践总结 ✅

1. **始终显式声明 props**（用对象形式更好）。
2. **不要直接修改 props** → 用 `ref` 拷贝 or `computed` 转换。
3. **复杂数据变动，用 `$emit` 通知父组件**。
4. **对象/数组的 default 必须是函数**。
5. **非字符串值记得加 `:`**。

---

## 附：完整示例代码

```vue
<!-- Parent.vue -->
<template>
  <UserProfile 
    :name="'Alice'"
    :age="25"
    :is-active="true"
    :hobbies="['reading', 'coding']"
    :settings="{ theme: 'dark' }"
  />
</template>
```

```vue
<!-- UserProfile.vue -->
<template>
  <div>
    <h2>{{ name }} ({{ age }}岁)</h2>
    <p>状态: {{ isActive ? '活跃' : '离线' }}</p>
    <p>爱好: {{ hobbies.join(', ') }}</p>
    <p>主题: {{ settings.theme }}</p>
  </div>
</template>

<script setup>
const props = defineProps({
  name: { type: String, required: true },
  age: Number,
  isActive: { type: Boolean, default: false },
  hobbies: {
    type: Array,
    default: () => []
  },
  settings: {
    type: Object,
    default: () => ({ theme: 'light' })
  }
})
</script>
```