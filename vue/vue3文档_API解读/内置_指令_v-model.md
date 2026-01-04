# **内置指令v-model**
---
# 📝 Vue 3 内置指令：`v-model` 详解笔记

> ✅ **适用场景**：Vue 3 + `<script setup>`（组合式 API）  
> 📚 官方文档参考：[https://cn.vuejs.org/api/built-in-directives.html#v-model](https://cn.vuejs.org/api/built-in-directives.html#v-model)

---

## **一、什么是 `v-model`？**

- **作用**：在表单输入元素（如 `<input>`、`<textarea>`、`<select>`）或自定义组件上创建 **双向数据绑定**。
- **核心思想**：  
  - 数据 → 视图：当 JS 变量变化时，输入框内容自动更新；  
  - 视图 → 数据：当用户在输入框中输入内容时，JS 变量也自动更新。
- **本质**：`v-model` 是语法糖，它同时做了两件事：
  1. 用 `:value`（或对应 prop）传入当前值；
  2. 用 `@input`（或对应事件）监听变化并更新变量。

---

## **二、基本用法（原生表单元素）**

### **1. 文本输入框（`<input type="text">`）**

```html
<template>
  <input v-model="message" placeholder="请输入内容" />
  <p>你输入了：{{ message }}</p>
</template>

<script setup>
import { ref } from 'vue'

const message = ref('') // 初始为空字符串
</script>
```

> 💡 `message` 是一个响应式引用（`ref`），`v-model` 会自动读取 `.value` 并更新它。

---

### **2. 多行文本（`<textarea>`）**

```html
<template>
  <textarea v-model="comment"></textarea>
</template>

<script setup>
const comment = ref('默认评论')
</script>
```

---

### **3. 单选框（`<input type="radio">`）**

```html
<template>
  <label><input type="radio" v-model="picked" value="A" /> A</label>
  <label><input type="radio" v-model="picked" value="B" /> B</label>
  <p>选中：{{ picked }}</p>
</template>

<script setup>
const picked = ref('A') // 默认选 A
</script>
```

> ✅ 所有 radio 共享同一个 `v-model` 变量，`value` 决定选中时变量的值。

---

### 4. 复选框（`<input type="checkbox">`）

#### 单个复选框（布尔值）

```html
<template>
  <input type="checkbox" v-model="isChecked" />
  <span>{{ isChecked ? '已同意' : '未同意' }}</span>
</template>

<script setup>
const isChecked = ref(false)
</script>
```

#### 多个复选框（数组）

```html
<template>
  <label><input type="checkbox" v-model="selected" value="苹果" /> 苹果</label>
  <label><input type="checkbox" v-model="selected" value="香蕉" /> 香蕉</label>
  <label><input type="checkbox" v-model="selected" value="橙子" /> 橙子</label>
  <p>选中的水果：{{ selected }}</p>
</template>

<script setup>
const selected = ref(['苹果']) // 初始选中苹果
</script>
```

> ✅ `selected` 必须是数组，选中项的 `value` 会被加入/移出数组。

---

### **5. 下拉选择框（`<select>`）**

```html
<template>
  <select v-model="city">
    <option disabled value="">请选择城市</option>
    <option value="bj">北京</option>
    <option value="sh">上海</option>
    <option value="gz">广州</option>
  </select>
  <p>你选择了：{{ city }}</p>
</template>

<script setup>
const city = ref('') // 初始为空
</script>
```

> ⚠️ 如果 `v-model` 的值与所有 `<option>` 的 `value` 都不匹配，则显示空白。

---

## **三、修饰符（Modifier）**

`v-model` 提供三个常用修饰符，用于修改绑定行为：

| 修饰符 | 作用 |
|--------|------|
| `.lazy` | 将监听事件从 `input` 改为 `change`（失去焦点或回车才更新） |
| `.number` | 自动将输入值转为数字类型（注意：仅对 `type="text"` 有效） |
| `.trim` | 自动去除输入内容首尾空格 |

### **示例：**

```html
<template>
  <!-- 失去焦点才更新 -->
  <input v-model.lazy="msg" />

  <!-- 输入内容自动转为数字 -->
  <input v-model.number="age" type="text" />

  <!-- 自动去除空格 -->
  <input v-model.trim="name" />
</template>

<script setup>
const msg = ref('')
const age = ref(0)
const name = ref('')
</script>
```

> ⚠️ 注意：`type="number"` 的 `<input>` 返回的是字符串！若要数字，请配合 `.number` 使用 `type="text"`，或手动 `Number()` 转换。

---

## **四、在自定义组件中使用 `v-model`**

**这是进阶但非常重要的用法！**

### **1. 默认 `v-model`（绑定 `modelValue` prop + `update:modelValue` 事件）**

**子组件（CustomInput.vue）**
```html
<template>
  <input 
    :value="modelValue" 
    @input="$emit('update:modelValue', $event.target.value)"
    placeholder="自定义输入框"
  />
</template>

<script setup>
// 声明接收 modelValue
const props = defineProps({
  modelValue: String // 类型根据需求调整
})

// 声明 emit 事件
const emit = defineEmits(['update:modelValue'])
</script>
```

**父组件**
```html
<template>
  <CustomInput v-model="parentMsg" />
  <p>父组件数据：{{ parentMsg }}</p>
</template>

<script setup>
const parentMsg = ref('hello')
</script>
```

> ✅ 这就是 Vue 3 中 `v-model` 在组件上的标准实现方式。

---

### **2. 多个 `v-model`（命名模型）**

Vue 3 支持一个组件有多个 `v-model`：

**子组件**
```html
<template>
  <input :value="firstName" @input="$emit('update:firstName', $event.target.value)" />
  <input :value="lastName" @input="$emit('update:lastName', $event.target.value)" />
</template>

<script setup>
const props = defineProps({
  firstName: String,
  lastName: String
})
const emit = defineEmits(['update:firstName', 'update:lastName'])
</script>
```

**父组件**
```html
<template>
  <UserForm 
    v-model:first-name="user.first" 
    v-model:last-name="user.last" 
  />
</template>

<script setup>
const user = reactive({ first: '张', last: '三' })
</script>
```

> 🔑 语法：`v-model:propName` 对应 prop 名为 `propName`，事件名为 `update:propName`。

---

## **五、常见误区 & 注意事项**

| 问题 | 正确做法 |
|------|--------|
| ❌ 在非表单元素上直接用 `v-model` | ❌ 无效！必须是 `<input>`/`<textarea>`/`<select>` 或自定义组件 |
| ❌ 忘记在组件中声明 `modelValue` 和 `update:modelValue` | ✅ 必须通过 `defineProps` 和 `defineEmits` 显式声明 |
| ❌ 用 `v-model` 绑定计算属性但没写 setter | ✅ 计算属性需提供 `set` 函数才能双向绑定 |
| ❌ 以为 `type="number"` 的 input 返回数字 | ❌ 实际返回字符串！需用 `.number` 修饰符或手动转换 |

---

## **六、速查表（Cheat Sheet）**

```vue
<!-- 基础绑定 -->
<input v-model="text" />

<!-- 修饰符 -->
<input v-model.lazy.trim.number="value" />

<!-- 复选框数组 -->
<input type="checkbox" v-model="list" value="item" />

<!-- 自定义组件 -->
<CustomInput v-model="data" />
<MultiInput 
  v-model:first="first" 
  v-model:last="last" 
/>
```

---

## 七、总结一句话

> **`v-model` 是 Vue 中实现“数据 ↔ 表单”双向同步的快捷方式——你改数据，表单变；用户改表单，数据也变。**

---