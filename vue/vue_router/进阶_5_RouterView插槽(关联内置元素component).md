# **Vue 3 中路由组件的过渡动画：从零理解 `<router-view>` 插槽与 `<Transition>` 的配合**

## **概述**

在 Vue 3 + Vue Router 项目中，当你从一个页面（如“首页”）跳转到另一个页面（如“关于”），默认是**瞬间切换**的。为了让这种切换更流畅、用户体验更好，我们希望加入**过渡动画**（比如淡入淡出、滑动等）。

但直接对 `<router-view />` 包裹 `<Transition>` 是无效的。正确做法是：**使用 `<router-view>` 的插槽（slot）获取当前要显示的组件，再手动用 `<component :is>` 渲染它，并在外层包裹 `<Transition>`**。

本笔记将从最基础的概念讲起，帮助你理解为什么需要这样做，以及每一步的作用是什么。

---

## **前置知识**

即使你是新手，也建议先了解以下概念（无需深入，知道“是什么”即可）：

1. **Vue 组件（Component）**  
   Vue 应用由多个可复用的组件构成，例如 `Home.vue`、`About.vue` 都是一个组件。

2. **Vue Router 路由**  
   通过 URL 决定显示哪个组件。`<router-view>` 是一个占位符，Vue Router 会在这里“插入”当前匹配的组件。

3. **动态组件 `<component :is="...">`**  
   Vue 提供的一种方式，可以根据变量的值动态决定渲染哪个组件。例如：
   ```vue
   <component :is="currentComponent" />
   ```
   如果 `currentComponent = Home`，就渲染 `Home` 组件。

4. **`<Transition>` 组件**  
   Vue 内置组件，用于给**单个元素或组件**添加进入/离开时的动画效果。它依赖 CSS 类名（如 `v-enter-from`）或 JavaScript 钩子。

5. **作用域插槽（Scoped Slot）**  
   父组件通过 `v-slot` 接收子组件传递的数据。例如：
   ```vue
   <Child v-slot="{ data }">
     {{ data }}
   </Child>
   ```
   这里 `data` 是 `Child` 组件主动“提供”给父组件使用的。

---

## **关键词**

- `<router-view>` 插槽（scoped slot）
- 动态组件（`<component :is>`）
- 路由组件（Route Component）
- `<Transition>` 组件
- 过渡类名（enter/leave classes）
- `mode="out-in"`
- 路由视图动画

---

## **正式内容**

### 1. 问题：为什么不能直接对 `<router-view>` 加 `<Transition>`？

```vue
<!-- ❌ 这样写没有动画效果！ -->
<Transition>
  <router-view />
</Transition>
```

**原因**：  
`<router-view>` 本身是一个**固定的组件容器**，它的“内部内容”变化对 Vue 的 `<Transition>` 来说是**不可见的**。  
`<Transition>` 只能感知**自己直接子元素是否被创建/销毁**，而 `<router-view>` 始终存在，只是内部换了内容。

> ✅ 类比：就像你给一个电视机（`<router-view>`）加外壳动画，但换频道（换节目）时，外壳不动，只有画面变——动画不会触发。

---

### 2. 解决方案：用 `<router-view>` 插槽“取出”组件

Vue Router 允许你通过 **作用域插槽** 获取当前要显示的组件对象：

```vue
<router-view v-slot="{ Component }">
  <!-- 此时 Component 就是当前路由对应的组件，比如 Home.vue -->
</router-view>
```

- `Component`（注意首字母大写）：代表当前路由匹配到的**组件定义对象**（不是实例）。
- 这是 Vue Router 主动“暴露”出来的数据，供你自定义渲染逻辑。

---

### 3. 手动渲染组件 + 添加过渡

现在，我们可以用 Vue 的**动态组件**语法来渲染它，并包裹 `<Transition>`：

```vue
<router-view v-slot="{ Component }">
  <Transition name="fade" mode="out-in">
    <component :is="Component" />
  </Transition>
</router-view>
```

#### 拆解说明：

| 代码 | 作用 |
|------|------|
| `v-slot="{ Component }"` | 从 `<router-view>` 获取当前路由组件 |
| `<component :is="Component" />` | 动态渲染该组件（每次路由变化，`Component` 会变，触发重新渲染） |
| `<Transition>` | 监听 `<component>` 的创建/销毁，自动应用动画 |

✅ **关键点**：  
现在 `<Transition>` 的直接子元素是 `<component>`，而 `<component>` 在路由切换时会被**销毁旧的、创建新的**，因此 `<Transition>` 能正确捕获这一过程并触发动画。

---

### 4. 关于 `mode="out-in"`

默认情况下，Vue 的 `<Transition>` 会让**新旧组件同时存在一瞬**（新组件进入的同时旧组件离开），可能导致布局抖动或重叠。

- `mode="out-in"`：**先完全移除旧组件 → 再插入新组件**，动画顺序清晰，推荐用于页面切换。
- `mode="in-out"`：先进入新组件，再移除旧组件（较少用）。

---

### 5. 如何编写 CSS 动画？

假设 `name="fade"`，你需要定义以下 CSS 类：

```css
/* 进入动画 */
.fade-enter-active {
  transition: opacity 0.3s;
}
.fade-enter-from {
  opacity: 0;
}
.fade-enter-to {
  opacity: 1;
}

/* 离开动画 */
.fade-leave-active {
  transition: opacity 0.3s;
}
.fade-leave-from {
  opacity: 1;
}
.fade-leave-to {
  opacity: 0;
}
```

> 💡 提示：`-active` 类控制动画属性（如 `transition`），`-from`/`-to` 控制起止状态。

---

### 6. 常见扩展场景

#### ✅ 场景 1：缓存页面（保留状态）

```vue
<router-view v-slot="{ Component }">
  <Transition name="fade" mode="out-in">
    <keep-alive>
      <component :is="Component" />
    </keep-alive>
  </Transition>
</router-view>
```

> ⚠️ 注意：被 `<keep-alive>` 缓存的组件**不会销毁**，因此**离开时无动画**。仅首次进入有动画。若需每次都有动画，不要缓存。

#### ✅ 场景 2：不同页面不同动画

通过路由的 `meta` 字段指定动画名：

```js
// router.js
{ path: '/home', component: Home, meta: { transition: 'slide' } }
```

```vue
<router-view v-slot="{ Component, route }">
  <Transition :name="route.meta.transition || 'fade'" mode="out-in">
    <component :is="Component" />
  </Transition>
</router-view>
```

---

### 7. 总结流程图（文字版）

1. 用户点击链接 → 路由改变  
2. Vue Router 计算出新路由对应的组件（如 `About.vue`）  
3. `<router-view>` 通过插槽将 `Component = About` 传递出来  
4. `<component :is="About" />` 被创建  
5. `<Transition>` 检测到“新元素进入”，应用 `fade-enter-*` 类  
6. 同时，旧组件（如 `Home`）被销毁，应用 `fade-leave-*` 类  
7. 动画结束后，页面完成切换

---

## **参考**

- [Vue Router 官方文档 - RouterView 插槽](https://router.vuejs.org/zh/guide/advanced/router-view-slot)
- [Vue 3 官方文档 - Transition 组件](https://cn.vuejs.org/guide/built-ins/transition.html)
- [Vue 3 官方文档 - 动态组件](https://cn.vuejs.org/guide/essentials/component-basics.html#dynamic-components)
- [Vue 3 官方文档 - 作用域插槽](https://cn.vuejs.org/guide/components/slots.html#scoped-slots)