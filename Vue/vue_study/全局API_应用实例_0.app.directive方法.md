# app.directive() - directive【指令】
`app.drecitve`用于**全局注册自定义指令(Custom Directive)** 的方法。它属于`App`实例的一个方法，可以在创建应用后通过`createApp()`获取实例，然后调用`directive()`注册全局自定义指令
>✅ 全局指令：一旦注册，可以在整个应用的所有组件中使用。
>❌ 局部指令：只能在单个组件中通过`directive`选项注册

## 📚 一、app.directive() 的两种用法
**1. 注册一个全局指令（传入名字和定义）**

```ts
app.directive(name, directiveDefinition)
```
* name: 指令名称（不带 v- 前缀）
* directiveDefinition: 指令对象或函数

**2. 获取已注册的指令**
```ts
const directive = app.directive(name)
```
如果该指令已注册，则返回其定义；否则返回 undefined

## 🔧二、指令钩子函数(Hook Function)
[Vue 自定义指令支持多个生命周期钩子](https://cn.vuejs.org/guide/reusability/custom-directives.html)ctrl+f搜搜指令钩子：
|钩子	|触发时机|
|---|---|
|beforeMount|	指令绑定到元素前|
|mounted	|指令绑定到元素后（DOM 已插入）|
|beforeUpdate	|组件更新前|
|updated	|组件更新后|
|beforeUnmount	|指令从元素上移除前|
|unmounted|	指令从元素上移除后|

这些钩子接受以下参数
* `el`: 指令绑定的 DOM 元素
* `binding`: 包含值、旧值、表达式等信息的对象
    * `value`：传递给指令的值。例如在 v-my-directive="1 + 1" 中，值是 2。
    * `oldValue`：之前的值，仅在 beforeUpdate 和 updated 中可用。无论值是否更改，它都可用。
    * `arg`：传递给指令的参数 (如果有的话)。例如在 v-my-directive:foo 中，参数是 "foo"。
    * `modifiers`：一个包含修饰符的对象 (如果有的话)。例如在 v-my-directive.foo.bar 中，修饰符对象是 { foo: true, bar: true }。
    * `instance`：使用该指令的组件实例。
    * `dir`：指令的定义对象。
* `vnode`: 当前组件的虚拟节点
* `prevVNode`S: 上一次的虚拟节点（仅在 updated 和 beforeUpdate 中存在）

## 🧪 四、完整示例项目结构
**✅ 1.文件结构**
```
my-vue-app/
├── index.html
├── main.js
├── App.vue
└── components/
    └── MyComponent.vue
```

## 📄 五、代码实现
**`1.main.ts`**
```ts
import { createApp } from 'vue'
import App from './App.vue'

// 创建应用实例
const app = createApp(App)

// ✅ 注册全局自定义指令：v-focus（点击时自动聚焦）
app.directive('focus', {
  mounted(el) {
    el.focus()
  },
  updated(el, binding) {
    if (binding.value) {
      el.focus()
    }
  }
})

// ✅ 注册另一个指令：v-highlight（高亮文本颜色）
app.directive('highlight', {
  mounted(el, binding) {
    el.style.backgroundColor = binding.value || 'yellow'
  },
  updated(el, binding) {
    el.style.backgroundColor = binding.value || 'yellow'
  }
})

// 启动应用
app.mount('#app')
```

**`2.App.vue`中使用**
```ts
<template>
  <div class="app">
    <h1>Vue 3 自定义指令示例</h1>

    <!-- 使用 v-focus 指令 -->
    <input type="text" v-focus placeholder="输入框，自动聚焦" />

    <!-- 使用 v-highlight 指令 -->
    <p v-highlight="'lightblue'">这个段落背景是浅蓝色</p>
    <p v-highlight="'pink'">这个段落背景是粉红色</p>

    <!-- 在组件中也有效 -->
    <MyComponent />
  </div>
</template>

<script>
import MyComponent from './components/MyComponent.vue'
export default {
  components: { MyComponent }
}
</script>

<style>
.app {
  padding: 20px;
  font-family: Arial, sans-serif;
}
</style>
```

## ➕六、扩展学习
* [自定义指令](https://cn.vuejs.org/guide/reusability/custom-directives.html)-学习指令的定义，以及更多扩展