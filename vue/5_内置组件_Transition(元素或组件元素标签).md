# **Vue 3 `<Transition>` 组件详解**

## **概述**

`<Transition>` 是 Vue 3 提供的内置组件，用于在**单个元素或组件进入/离开 DOM 时自动应用过渡或动画效果**。它能智能地结合 CSS 过渡/动画 或 JavaScript 钩子函数，在状态切换（如 `v-if`、`v-show`、动态组件）时提供流畅的视觉反馈，而无需手动操作 DOM 或管理动画生命周期。

## **前置知识**

- Vue 3 基础：响应式数据、指令（`v-if`、`v-show`）、组件系统
- 动态组件：`<component :is="...">` 的用法
- CSS 基础：`transition` 和 `animation` 属性
- 浏览器事件：`transitionend`、`animationend`
- （可选）JavaScript 动画库基础（如 GSAP、Anime.js）

## **关键词**

- `<Transition>`
- 过渡类名（enter/leave classes）
- `name` prop
- `mode="out-in"`
- JavaScript 钩子（`@before-enter`, `@enter`, `@after-leave` 等）
- `:css="false"`
- 动态过渡
- 出现时过渡（`appear`）

## **正式内容**

### 1. `<Transition>` 组件

- **作用**：为**单个元素或组件**包裹一层过渡逻辑。
- **触发条件**：
  - `v-if` / `v-show` 切换
  - 动态组件（`<component :is>`）切换
  - 元素 `key` 变化（强制重新创建）
- **限制**：插槽内只能有一个根元素（组件也需有单一根节点）。
- **自动探测**：Vue 会自动检测是否有 CSS 过渡/动画，或是否提供了 JS 钩子，从而决定如何处理动画生命周期。

> ✅ 示例：
> ```vue
> <Transition>
>   <p v-if="show">Hello</p>
> </Transition>
> ```

---

### 2. 基于 CSS 的过渡效果

#### 过渡类名（6 个）

| 类名 | 时机 | 说明 |
|------|------|------|
| `v-enter-from` | 进入开始前 | 起始状态（下一帧移除） |
| `v-enter-active` | 进入过程中 | 定义 `transition` 属性（持续时间、缓动函数） |
| `v-enter-to` | 进入完成后 | 结束状态（与 `v-enter-from` 同时切换） |
| `v-leave-from` | 离开开始时 | 起始状态（一帧后移除） |
| `v-leave-active` | 离开过程中 | 定义离开动画的 `transition` |
| `v-leave-to` | 离开结束时 | 结束状态 |

> 💡 提示：`v-enter-to` 和 `v-leave-from` 通常样式相同（即“静止”状态），而 `v-enter-from` 和 `v-leave-to` 是“隐藏”状态。

#### 自定义过渡名称

通过 `name="fade"`，类名前缀从 `v-` 变为 `fade-`：

```css
.fade-enter-active { transition: opacity 0.3s; }
.fade-enter-from { opacity: 0; }
/* ... */
```

#### 支持 CSS `transition` 与 `animation`

- **`transition`**：基于属性变化（如 `opacity`、`transform`）。
- **`animation`**：基于 `@keyframes`，此时 `*-enter-from` 在 `animationend` 时移除。

#### 自定义类名（集成第三方库）

可通过 props 覆盖默认类名，便于使用 Animate.css 等库：

```vue
<Transition
  enter-active-class="animate__animated animate__fadeIn"
  leave-active-class="animate__animated animate__fadeOut"
>
  <div v-if="show">...</div>
</Transition>
```

#### 深层级过渡 & 显式时长

- 使用深层选择器（如 `.my-transition .inner`）可对子元素做动画。
- 若动画嵌套复杂，Vue 可能无法准确判断结束时间，此时用 `:duration="{ enter: 500, leave: 800 }"` 显式指定。

#### 性能建议

优先使用 `transform` 和 `opacity`，避免触发布局重排（如 `height`、`margin`）。

---

### 3. JavaScript 钩子

当需要完全控制动画（如使用 GSAP），可监听以下事件：

| 钩子 | 触发时机 | 用途 |
|------|--------|------|
| `@before-enter` | 元素插入前 | 设置初始状态 |
| `@enter` | 插入后下一帧 | 启动进入动画（需调用 `done()`） |
| `@after-enter` | 进入完成 | 清理工作 |
| `@enter-cancelled` | 进入被中断 | （如快速切换） |
| `@before-leave` | 离开前 | — |
| `@leave` | 离开开始 | 启动离开动画（需调用 `done()`） |
| `@after-leave` | 离开完成且 DOM 移除 | — |
| `@leave-cancelled` | 仅 `v-show` 下可能触发 | — |

> ⚠️ 注意：若仅用 JS 动画，建议加 `:css="false"`，防止 CSS 干扰。

> ✅ 示例（GSAP）：
> ```vue
> <Transition
>   @enter="(el, done) => gsap.to(el, { opacity: 1, onComplete: done })"
>   @leave="(el, done) => gsap.to(el, { opacity: 0, onComplete: done })"
>   :css="false"
> >
>   <div v-if="show">...</div>
> </Transition>
> ```

---

### 4. 可复用过渡效果

将 `<Transition>` 封装为自定义组件，便于跨项目复用：

```vue
<!-- MyFade.vue -->
<template>
  <Transition name="my-fade" @after-enter="onAfterEnter">
    <slot />
  </Transition>
</template>

<style>
.my-fade-enter-active { transition: opacity 0.3s; }
.my-fade-enter-from { opacity: 0; }
/* ... */
</style>
```

使用：
```vue
<MyFade><div v-if="show">Hello</div></MyFade>
```

---

### 5. 出现时过渡（`appear`）

默认 `<Transition>` 只在**后续切换**时生效。若希望**初次渲染**也带动画，加 `appear`：

```vue
<Transition appear>
  <div>Hello</div>
</Transition>
```

此时会额外应用 `v-appear-*` 类名（规则同 `v-enter-*`）。

---

### 6. 元素间过渡 & 过渡模式

- 多个元素用 `v-if`/`v-else` 切换时，`<Transition>` 也能工作。
- 默认进入和离开**同时发生**，可能导致布局抖动。
- 解决方案：使用 `mode`：
  - `mode="out-in"`：先出后进（推荐）
  - `mode="in-out"`：先进后出（较少用）

> ✅ 示例：
> ```vue
> <Transition mode="out-in">
>   <button v-if="type === 'A'">A</button>
>   <button v-else>B</button>
> </Transition>
> ```

---

### 7. 组件间过渡（对应VueRouter文档RouterView插槽）

直接用于动态组件：

```vue
<Transition name="slide" mode="out-in">
  <component :is="currentView" />
</Transition>
```

常用于 tabs、步骤向导等场景。

---

### 8. 动态过渡

`name` 或钩子函数可绑定响应式数据，实现根据状态切换不同动画：

```vue
<Transition :name="isMobile ? 'slide' : 'fade'">
  <div>...</div>
</Transition>
```

终极方案：封装为可配置的过渡组件。

---

### 9. 使用 `key` Attribute 触发过渡

当内容变化但 DOM 结构不变时（如计数器文本更新），Vue 不会认为是“新元素”，因此无过渡。

解决方法：给元素加 `:key="value"`，强制 Vue 重建元素：

```vue
<Transition>
  <span :key="count">{{ count }}</span>
</Transition>
```

每次 `count` 变化，`<span>` 被替换，触发过渡。

## **参考**

- [Vue 3 官方文档 - Transition](https://cn.vuejs.org/guide/built-ins/transition.html)
- [MDN - CSS Transitions](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Transitions/Using_CSS_transitions)
- [MDN - CSS Animations](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Animations/Using_CSS_animations)
- [GSAP 官网](https://gsap.com/)