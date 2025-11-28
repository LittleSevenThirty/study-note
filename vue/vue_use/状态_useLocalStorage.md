# **useLocalStorage**
---

# **📝 VueUse `useLocalStorage` 学习笔记（小白友好版）**

> **目标**：理解并能独立使用 VueUse 中的 `useLocalStorage`  
> **前提知识**：了解基本的 Vue 3（Composition API）即可，无需提前掌握 VueUse

---

## **一、什么是 `localStorage`？**

- 浏览器提供的一种**持久化存储机制**（关闭浏览器后数据还在）。
- 只能存字符串（`setItem('key', 'value')`），所以存对象要先 `JSON.stringify`，取出来再 `JSON.parse`。
- 常用于保存用户设置、主题、登录状态等。

> ⚠️ 注意：`localStorage` 是同步的、有容量限制（通常 5~10MB），且只在同源下可用。

---

## **三、为什么需要 `useLocalStorage`？**

手动操作 `localStorage` 很麻烦：
```js
// 手动写法（繁琐且不响应式）
const saveTheme = (theme) => {
  localStorage.setItem('theme', theme);
};
const loadTheme = () => {
  return localStorage.getItem('theme') || 'light';
};
```

而 `useLocalStorage` 做了三件事：
1. **自动读写 `localStorage`**
2. **返回一个响应式 ref（和 `ref()` 一样用）**
3. **支持任意类型（自动序列化/反序列化）**

> 💡 简单说：你把它当成一个“会自动同步到本地存储的 ref”就行！

---

## **四、`useLocalStorage` 基本用法**

### **1. 导入**
```js
import { useLocalStorage } from '@vueuse/core'
```

### **2. 声明一个响应式本地存储变量**
```js
const theme = useLocalStorage('my-theme', 'light')
```
- `'my-theme'`：localStorage 中的 key（键名）
- `'light'`：默认值（如果 localStorage 里没有这个 key，就用这个值）

### **3. 使用它（完全像普通 ref）**
```vue
<template>
  <div>
    当前主题：{{ theme }}
    <button @click="theme = 'dark'">切换为暗色</button>
    <button @click="theme = 'light'">切换为亮色</button>
  </div>
</template>

<script setup>
import { useLocalStorage } from '@vueuse/core'

const theme = useLocalStorage('my-theme', 'light')
</script>
```

✅ 效果：
- 点击按钮修改 `theme`，页面立即更新（响应式）
- 同时自动保存到 `localStorage`
- 刷新页面后，值依然保留！

---

## **五、支持的数据类型**

`useLocalStorage` 支持：
- 字符串、数字、布尔值
- 对象、数组（自动 `JSON.stringify` / `JSON.parse`）

### 示例：保存用户设置对象
```js
const userSettings = useLocalStorage('user-settings', {
  lang: 'zh',
  notifications: true,
  fontSize: 16
})

// 修改某个属性
userSettings.value.fontSize = 18 // 自动保存！
```

> 🔒 注意：不能存函数、undefined、Symbol 等无法 JSON 序列化的值。

---

## **六、高级选项（可选）**

你可以传入第三个参数 `options` 来控制行为：

```js
const count = useLocalStorage('count', 0, {
  // 是否监听 storage 事件（其他标签页修改时同步）
  listenToStorageChanges: true,

  // 自定义序列化/反序列化（一般不需要）
  serializer: {
    read: (v) => v ? JSON.parse(v) : null,
    write: (v) => JSON.stringify(v)
  },

  // 是否在写入前做深比较（避免无意义写入）
  mergeDefaults: false
})
```

> 对初学者来说，**前两个参数就够用了**，`options` 暂时忽略也行。

---

## **七、常见问题 & 注意事项**

| 问题 | 说明 |
|------|------|
| ❓ 为什么修改值后 localStorage 没变？ | 确保你修改的是 `.value`（因为它是 ref）——但在模板中直接写 `theme` 就行，Vue 会自动解包 |
| 🔄 多个标签页同步？ | 默认不监听其他标签页的 localStorage 变化。如需同步，设 `listenToStorageChanges: true` |
| 🧹 如何清除？ | 直接 `localStorage.removeItem('key')`，或把 `ref` 设为 `null` 并重新赋默认值 |
| 📦 和 `useStorage` 有什么区别？ | `useLocalStorage` 是 `useStorage(localStorage, ...)` 的快捷方式；`useStorage` 更通用（可换 sessionStorage 等） |

---

## **八、完整示例：记住用户名**

```vue
<template>
  <div>
    <input v-model="username" placeholder="输入你的名字" />
    <p>你好，{{ username || '陌生人' }}！</p>
    <p>（刷新页面试试，名字还在！）</p>
  </div>
</template>

<script setup>
import { useLocalStorage } from '@vueuse/core'

// 自动从 localStorage 读取 'username'，默认空字符串
const username = useLocalStorage('username', '')
</script>
```

---

## **九、总结（速记卡片）**

```text
useLocalStorage(key, defaultValue, options?)

✅ 返回一个响应式 ref
✅ 自动读写 localStorage
✅ 支持对象/数组（自动 JSON 转换）
✅ 修改 .value 即可保存
✅ 刷新不丢失，开箱即用！

适合场景：主题、语言、用户偏好、表单草稿等
```

---