

# Vue.js 应用挂载：`app.mount()` 详解

## 一、为什么要学习应用挂载？

在 Vue.js 中，**挂载**是将 Vue 应用与浏览器 DOM 元素关联的过程。就像给房子安装门窗一样，挂载是让 Vue 应用"活起来"的关键步骤。

> 📌 举个生活化的例子：  
> 你建了一座房子（Vue 组件），但房子没有门（没有挂载到 DOM）。  
> 你需要在房子上开一扇门（`app.mount()`），让居民（用户）能进去住（交互）。

## 二、`app.mount()` 的核心作用

`app.mount()` 是 Vue 应用实例的**挂载方法**，用于：
- 将 Vue 应用绑定到指定的 DOM 元素
- 启动 Vue 的响应式系统
- 渲染应用的初始内容

```javascript
// 创建应用实例
const app = createApp(App)

// 挂载应用到 #app 元素
app.mount('#app')
```

## 三、基本用法详解

### 1. 最简单的挂载方式

```javascript
// 挂载到 ID 为 app 的元素
app.mount('#app')
```

### 2. 挂载到实际 DOM 元素

```javascript
// 挂载到第一个 <div> 元素
app.mount(document.querySelector('div'))
```

### 3. 挂载到文档碎片（Fragment）

```javascript
// 创建一个文档碎片
const container = document.createDocumentFragment()
container.appendChild(document.createElement('div'))

// 挂载到文档碎片
app.mount(container)
```

## 四、参数详解

### 1. 参数类型

`app.mount()` 接受一个参数，可以是：
- **DOM 元素**（`HTMLElement`）
- **CSS 选择器字符串**（如 `'#app'`、`'.container'`）

### 2. 选择器行为

当传入字符串时：
- 会使用 `document.querySelector()` 查找第一个匹配的元素
- 如果找不到元素，会抛出错误

```javascript
// 有效：查找 #app 元素
app.mount('#app')

// 有效：查找第一个 .container 元素
app.mount('.container')

// 无效：找不到元素时会报错
app.mount('#non-existent-id')
```

## 五、挂载后的行为

### 1. 基本流程

1. Vue 检查挂载容器
2. 如果容器有内容，Vue 会**替换**这些内容
3. Vue 将应用的根组件渲染到容器中

### 2. 两种挂载模式

| 模式 | 行为 | 适用场景 |
|------|------|----------|
| **有模板/渲染函数** | 直接渲染组件内容，替换容器内所有内容 | 99% 的 Vue 应用 |
| **无模板/渲染函数** | 使用容器的 `innerHTML` 作为模板 | 仅在开发中使用 |

> 💡 提示：在 Vue 3 中，你几乎总是会定义模板或渲染函数，所以第二种情况很少见。

### 3. SSR 激活模式（服务端渲染）

在服务端渲染（SSR）中，`app.mount()` 会**激活**已存在的服务器端渲染的 DOM：
- 如果客户端和服务器端渲染结果匹配，直接激活
- 如果不匹配，会调整 DOM 以匹配客户端渲染结果

## 六、返回值说明

`app.mount()` 返回**根组件的实例**（`ComponentPublicInstance`）：

```javascript
const rootInstance = app.mount('#app')

// 访问根组件实例
console.log(rootInstance.$data) // 根组件的数据
console.log(rootInstance.$el)   // 根组件的 DOM 元素
```

> 📌 重要：这个实例可以用于访问根组件的属性和方法。

## 七、实际应用场景

### 场景 1：标准应用挂载

```javascript
// main.js
import { createApp } from 'vue'
import App from './App.vue'

const app = createApp(App)
app.mount('#app')
```

```html
<!-- index.html -->
<div id="app"></div>
```

### 场景 2：动态挂载（多应用场景）

```javascript
// 根据路由动态挂载不同应用
function mountApp(route) {
  const app = createApp(App)
  
  if (route === 'admin') {
    app.mount('#admin-container')
  } else {
    app.mount('#user-container')
  }
}
```

### 场景 3：挂载到非 HTML 元素

```javascript
// 挂载到 iframe 中的元素
const iframe = document.getElementById('my-iframe')
const iframeDoc = iframe.contentDocument
const container = iframeDoc.getElementById('app')

app.mount(container)
```

## 八、常见问题与注意事项

### 1. 每个应用实例只能挂载一次

```javascript
const app = createApp(App)

// 第一次挂载
app.mount('#app')

// 第二次挂载会报错
app.mount('#app') // ❌ 错误：应用已挂载
```

> 💡 原因：Vue 应用一旦挂载，就无法再次挂载到其他容器。

### 2. 挂载前的准备工作

在挂载前，你可以：
- 注册全局组件
- 注册全局指令
- 安装插件
- 配置全局属性

```javascript
const app = createApp(App)

// 注册全局组件
app.component('MyButton', MyButton)

// 安装插件
app.use(VueRouter)

// 配置全局属性
app.config.globalProperties.$http = axios

// 最后挂载
app.mount('#app')
```

### 3. SSR 挂载的特殊处理

在 SSR 中，挂载会激活已存在的 DOM，但需要确保：
- 服务端渲染的 HTML 与客户端渲染一致
- 使用 `createSSRApp()` 创建应用实例

```javascript
// 服务端渲染时
import { createSSRApp } from 'vue'

const app = createSSRApp(App)
app.mount('#app') // 激活模式
```

## 九、最佳实践

### 1. 优先使用 ID 选择器

```javascript
// 推荐
app.mount('#app')

// 不推荐（如果页面有多个同名元素）
app.mount('.app-container')
```

### 2. 避免在挂载前操作 DOM

```javascript
// 错误：在挂载前操作 DOM
const container = document.getElementById('app')
container.innerHTML = '正在加载...' // 会被 Vue 覆盖

app.mount('#app')
```

### 3. 使用 `nextTick` 确保 DOM 更新

```javascript
app.mount('#app')

// 确保 DOM 已更新
app.config.globalProperties.$nextTick(() => {
  console.log('DOM 已更新')
})
```

## 十、总结

`app.mount()` 是 Vue 应用的**启动按钮**，它将应用与浏览器 DOM 关联起来。

**核心要点回顾：**
1. 语法：`app.mount(selector | DOMElement)`
2. 参数：可以是 CSS 选择器或 DOM 元素
3. 行为：替换容器内容并启动应用
4. 返回值：根组件实例
5. 限制：每个应用实例只能调用一次

> 📌 最后提醒：`app.mount()` 是 Vue 应用的**最后一步**，在它之前的所有配置（注册组件、指令、插件等）都会生效。
