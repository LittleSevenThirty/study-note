# **Vue3文档解读emit_组件事件**
## **一、为什么需要 `emit`？**

- 在 Vue 中，**数据流是单向的**：父组件 → 子组件（通过 `props`）
- 但子组件有时需要 **通知父组件发生了某些事情**（比如“用户点击了提交按钮”、“输入内容改变了”）
- 这时就需要 **子组件主动“抛出”一个事件**，由父组件“监听”并处理
- 这个“抛出”动作就是 `emit`

> 💡 类比：`props` 是父给子传话；`emit` 是子给父传话

---

## 二、基础用法：如何触发和监听事件？
### 第一步：先搞懂“组件”是什么？

想象你在搭积木：

- **父组件** = 大盒子（比如一个“用户设置页面”）
- **子组件** = 小积木（比如一个“保存按钮”、一个“头像上传框”）

```text
[ 用户设置页面 ]  ← 父组件
   ├── [ 保存按钮 ]   ← 子组件
   └── [ 头像上传 ]   ← 子组件
```

✅ **数据传递方向**：
- 父 → 子：通过 **props**（比如父告诉按钮：“文字是‘保存’”）
- 子 → 父：通过 **事件（emit）**（比如按钮告诉父：“用户点我了！”）

> 🔑 核心思想：**子不能直接改父的数据，只能“通知”父，让父自己决定怎么做**

---

### 第二步：为什么不能直接改？举个反例 ❌

假设没有 `emit`，子组件直接改父的数据：

```js
// ❌ 错误做法（Vue 不允许！）
// 子组件里直接写：
parentData.isSaved = true
```

这会带来问题：
- 父组件不知道数据被谁改了
- 调试困难
- 组件耦合太紧（换个父组件就用不了）

所以 Vue 强制规定：**子只能“发消息”，不能“动手改”**

---

### 第三步：`emit` 就是“发消息”的方式 ✅

#### 比喻：子组件按了一个“呼叫铃”

- 你住酒店，按下床头的“服务铃”
- 铃不会自己给你送水，而是**通知前台**：“302房间需要服务！”
- 前台（父组件）收到后，决定派服务员送水

在 Vue 中：
- 子组件调用 `emit('eventName')` → 相当于按铃
- 父组件写 `@eventName="doSomething"` → 相当于前台监听铃声

---

### 第四步：手把手写一个例子 🛠️

#### 场景：点击子组件的按钮，让父组件弹出“你好！”

###### 1. 父组件（Parent.vue）
```html
<template>
  <div>
    <h1>我是父组件</h1>
    <!-- 使用子组件，并监听它发出的事件 -->
    <ChildComponent @say-hello="handleHello" />
  </div>
</template>

<script setup>
import ChildComponent from './ChildComponent.vue'

// 定义：当听到 "say-hello" 事件时，执行这个函数
function handleHello() {
  alert('你好！')
}
</script>
```

###### 2. 子组件（ChildComponent.vue）
```html
<template>
  <button @click="sayHello">点我打招呼</button>
</template>

<script setup>
// 声明：我会发出哪些事件（可选但推荐）
const emit = defineEmits(['sayHello'])

function sayHello() {
  // 发出事件！告诉父组件：“有人点我了！”
  emit('sayHello')
}
</script>
```

✅ 运行效果：
- 点击子组件的按钮
- 父组件弹出 “你好！”

---

### 第五步：`emit` 能带“附加信息”吗？能！

就像按服务铃时可以说：“我要一瓶水 + 一条毛巾”

#### 修改上面的例子：传递名字

###### 子组件发出带参数的事件：
```js
function sayHello() {
  emit('sayHello', '小明') // 第二个参数就是“附加信息”
}
```

###### 父组件接收参数：
```js
function handleHello(name) { // 参数自动传进来
  alert(`你好，${name}！`)
}
```

> 💡 `emit('事件名', 参数1, 参数2, ...)`  
> 父组件的处理函数会按顺序接收到这些参数

---

### 第六步：`defineEmits` 到底是什么？

你可以把它理解为 **“提前报备”**：

> “我要发出哪些事件，请帮我记录一下”

#### 不写 `defineEmits` 行不行？
- 技术上可以（Vue 允许隐式 emit）
- 但**强烈建议写**，原因：
  1. 代码更清晰（别人一看就知道这个组件会发什么事件）
  2. 避免拼写错误（比如写成 `emti` 不会报错，但事件无效）
  3. 在 TypeScript 中能获得类型检查

#### 最简写法：
```js
const emit = defineEmits(['事件1', '事件2'])
```

然后就可以用 `emit('事件1')` 了。

---

### 第七步：常见疑问解答 ❓

#### Q1：为什么事件名在模板里用 `@say-hello`（短横线），JS 里用 `'sayHello'`（驼峰）？

- Vue 自动帮你转换！这是约定。
- 推荐：**JS 里用驼峰（sayHello），模板里用短横线（say-hello）**
- 两者等价，Vue 能识别

#### Q2：`$emit` 和 `emit()` 有什么区别？

| 写法 | 适用场景 |
|------|--------|
| `$emit('xxx')` | 只能在 `<template>` 里用（模板语法糖） |
| `emit('xxx')` | 在 `<script setup>` 的 JS 代码里用 |

```html
<template>
  <!-- 模板里可以直接 $emit -->
  <button @click="$emit('test')">点我</button>
</template>

<script setup>
// JS 里必须用 defineEmits 得到的 emit
const emit = defineEmits(['test'])
function clickHandler() {
  emit('test')
}
</script>
```

#### Q3：父组件能同时监听多个子组件的相同事件吗？

可以！每个子组件是独立的：

```html
<Child @update="handleUpdate1" />
<Child @update="handleUpdate2" />
```
两个 `Child` 各自 emit，各自触发对应的处理函数。

---

### 第八步：总结成一句话 🧠

> **`emit` 是子组件向父组件“发通知”的方法。  
> 子说：“我干了某事！”  
> 父听到了，决定：“那我该做什么？”**

---

### 附：极简记忆模板

#### 子组件（发消息）：
```html
<script setup>
const emit = defineEmits(['myEvent'])
function doSomething() {
  emit('myEvent', 数据) // 可选带数据
}
</script>
```

#### 父组件（收消息）：
```html
<Child @my-event="处理函数" />

<script setup>
function 处理函数(数据) {
  // 做你想做的事
}
</script>
```

---