# Vue.js 应用实例 API：全局注册指令详解

## 一、为什么要学习全局注册指令？

在学习了**自定义指令**，了解到指令可以**局部注册**（只在当前组件使用）或**全局注册**（在整个应用中都能使用）。

当你有一个指令需要在**多个组件中重复使用**时，每次都局部注册会很麻烦。这时，**全局注册**就派上用场了！

> 📌 举个例子：你开发了一个 `v-loading` 指令用于显示加载状态，这个功能在应用的很多页面都需要。如果每个组件都写一遍注册代码，不仅重复，还难以维护。全局注册一次，所有组件都能直接使用！

## 二、什么是应用实例？

在 Vue 3 中，每个 Vue 应用都是通过 `createApp()` 创建的**应用实例**：

```javascript
import { createApp } from 'vue'
import App from './App.vue'

// 创建应用实例
const app = createApp(App)

// 挂载应用
app.mount('#app')
```

这个 `app` 对象就是**应用实例**，它提供了很多方法来配置整个应用，包括**全局注册指令**。

## 三、全局注册指令的方法

### 1. 基本语法

使用 `app.directive()` 方法来全局注册指令：

```javascript
app.directive(name, directive)
```

- **`name`**：指令的名称（字符串），在模板中使用时要加 `v-` 前缀
- **`directive`**：指令的定义（对象或函数）

### 2. 完整示例

```javascript
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)

// 全局注册 v-focus 指令
app.directive('focus', {
  mounted(el) {
    el.focus()
  }
})

// 全局注册 v-highlight 指令（简写形式）
app.directive('highlight', (el, binding) => {
  el.style.backgroundColor = binding.value || 'yellow'
})

app.mount('#app')
```

现在，在应用的**任何组件**中都可以直接使用这些指令：

```vue
<template>
  <div>
    <!-- 自动聚焦 -->
    <input v-focus />
    
    <!-- 高亮显示 -->
    <p v-highlight="'lightblue'">这段文字会被高亮</p>
  </div>
</template>
```

## 四、app.directive() 的两种用法

### 1. 注册指令（传入名称和定义）

```javascript
// 注册一个对象形式的指令
app.directive('myDirective', {
  mounted(el) {
    // 指令逻辑
  }
})

// 注册一个函数形式的指令（简写）
app.directive('mySimpleDirective', (el, binding) => {
  // 简单的指令逻辑
})
```

### 2. 获取已注册的指令（只传入名称）

```javascript
// 获取名为 'myDirective' 的指令定义
const myDirective = app.directive('myDirective')

if (myDirective) {
  console.log('指令已注册')
}
```

> 💡 这个功能通常用于插件开发或调试，普通应用开发中较少使用。

## 五、指令定义的详细说明

### 1. 对象形式（完整功能）

```javascript
app.directive('example', {
  // 元素创建后调用（在元素插入 DOM 前）
  created(el, binding, vnode) {
    // 只在客户端调用
  },
  
  // 元素插入 DOM 前调用
  beforeMount(el, binding, vnode) {
    // 可以在这里做准备工作
  },
  
  // 元素插入 DOM 后调用
  mounted(el, binding, vnode) {
    // 最常用的钩子，执行主要逻辑
  },
  
  // 父组件更新前调用
  beforeUpdate(el, binding, vnode, prevVnode) {
    // 可以在这里清理之前的副作用
  },
  
  // 父组件更新后调用
  updated(el, binding, vnode, prevVnode) {
    // 处理指令值变化后的逻辑
  },
  
  // 父组件卸载前调用
  beforeUnmount(el, binding, vnode) {
    // 清理定时器、事件监听器等
  },
  
  // 父组件卸载后调用
  unmounted(el, binding, vnode) {
    // 最终清理工作
  }
})
```

### 2. 函数形式（简写）

当只需要在 `mounted` 和 `updated` 时执行相同逻辑时，可以直接用函数：

```javascript
// 等价于上面的对象形式中的 mounted 和 updated 钩子
app.directive('color', (el, binding) => {
  el.style.color = binding.value
})
```

## 六、参数详解

所有钩子函数都会接收相同的参数：

| 参数 | 类型 | 说明 |
|------|------|------|
| `el` | `HTMLElement` | 指令绑定的 DOM 元素 |
| `binding` | `Object` | 包含指令相关信息的对象 |
| `vnode` | `VNode` | 代表绑定元素的底层虚拟节点 |
| `prevVnode` | `VNode` \| `undefined` | 之前的虚拟节点（仅在 `beforeUpdate`/`updated` 中可用） |

### `binding` 对象的属性

| 属性 | 说明 | 示例 |
|------|------|------|
| `value` | 传递给指令的值 | `v-example="123"` → `123` |
| `oldValue` | 之前的值 | 更新前的值 |
| `arg` | 指令参数 | `v-example:foo` → `'foo'` |
| `modifiers` | 修饰符 | `v-example.foo.bar` → `{ foo: true, bar: true }` |
| `instance` | 使用指令的组件实例 | 组件实例对象 |
| `dir` | 指令定义对象 | 定义指令时传入的对象 |

## 七、实际应用场景

### 场景 1：权限控制指令

```javascript
// 全局注册权限指令
app.directive('permission', {
  mounted(el, binding) {
    const requiredPermission = binding.value
    const userPermissions = getUserPermissions() // 获取用户权限
    
    if (!userPermissions.includes(requiredPermission)) {
      el.style.display = 'none'
    }
  }
})
```

```vue
<template>
  <!-- 只有拥有 'admin' 权限的用户才能看到这个按钮 -->
  <button v-permission="'admin'">管理员操作</button>
</template>
```

### 场景 2：防抖点击指令

```javascript
app.directive('debounce-click', {
  mounted(el, binding) {
    const delay = binding.arg || 300 // 默认300ms
    let timer
    
    el.addEventListener('click', () => {
      if (timer) clearTimeout(timer)
      timer = setTimeout(() => {
        binding.value()
      }, delay)
    })
  }
})
```

```vue
<template>
  <!-- 防抖500ms的点击 -->
  <button v-debounce-click:500="handleClick">点击我</button>
</template>
```

## 八、最佳实践和注意事项

### 1. 命名规范

- 使用**小写+短横线**命名：`v-my-directive`
- 避免与内置指令冲突：不要使用 `v-model`、`v-show` 等

### 2. 内存泄漏防范

在 `unmounted` 钩子中清理资源：

```javascript
app.directive('resize', {
  mounted(el, binding) {
    const handler = () => binding.value(el)
    window.addEventListener('resize', handler)
    el._resizeHandler = handler // 保存引用
  },
  
  unmounted(el) {
    window.removeEventListener('resize', el._resizeHandler)
  }
})
```

### 3. 与组合式函数的区别

| 特性 | 全局指令 | 组合式函数 |
|------|----------|------------|
| 用途 | 直接操作 DOM | 逻辑复用 |
| 访问 DOM | 直接访问 | 通过 ref 间接访问 |
| 性能 | 每次渲染都执行 | 按需调用 |
| 推荐度 | 特定场景使用 | 优先推荐 |

> 📌 **建议**：除非确实需要直接操作 DOM，否则优先使用组合式函数进行逻辑复用。

### 4. 插件中的使用

在 Vue 插件中注册全局指令是很常见的做法：

```javascript
// MyPlugin.js
export default {
  install(app) {
    app.directive('loading', {
      // 指令逻辑
    })
  }
}

// main.js
import MyPlugin from './MyPlugin.js'
app.use(MyPlugin)
```

## 九、与其他全局 API 的关系

Vue 应用实例还提供了其他全局注册方法：

| 方法 | 用途 |
|------|------|
| `app.component()` | 全局注册组件 |
| `app.directive()` | 全局注册指令 |
| `app.use()` | 安装插件 |
| `app.provide()` | 全局提供依赖 |

这些方法共同构成了 Vue 应用的**全局配置体系**。

## 十、总结

全局注册指令是 Vue 应用配置的重要组成部分，它让你能够：

✅ **一次注册，处处使用** - 避免重复代码  
✅ **统一管理** - 所有全局指令集中在入口文件  
✅ **插件友好** - 方便封装成可复用的插件  

## **参考**
https://cn.vuejs.org/api/application.html#app-directive