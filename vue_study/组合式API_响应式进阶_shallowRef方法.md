# `shallowRef`方法
## 🔍一、什么是`shallowRef`？
**官方文档**
```ts
shallowRef<T>(value: T): ShallowRef<T>
```
* **作用**：创建一个“浅层”的`ref`
* **解释**：只有`.value`这一层是响应式的，**内部对象的属性变化不会被追踪**

## 🛠️二、如何使用
### ☀️1.文件结构
```txt
my-vue-app/
├── index.html
├── main.js
├── App.vue
└── components/
    └── ProductList.vue
```

### ✅2.实例场景：商品列表
**`1.components/ProductList.vue`**
```js
<template>
    <div class="product-list">
        <h3>商品列表，共{{products.value.length}}件</h3>
        <ul>
            <li v-for="p in products" :key="p.id">
                {{p.name}}:{{p.value}}
                <button @click="updatePrice(p)">涨价</button>
            </li>
        </ul>
        <button @click="fetchNewData">刷新数据</button>
    </div>
</template>

<!-- 文件：components/ProductList.vue -->
<template>
  <div class="product-list">
    <h3>商品列表（共 {{ products.value.length }} 件）</h3>
    <ul>
      <li v-for="p in products.value" :key="p.id">
        {{ p.name }} - ¥{{ p.price }}
        <button @click="updatePrice(p)">涨价</button>
      </li>
    </ul>
    <button @click="fetchNewData">刷新数据</button>
  </div>
</template>

<script setup>
import { shallowRef, triggerRef } from 'vue'

// ✅ 使用 shallowRef 包裹大数据
const products = shallowRef([])

// 模拟从 API 获取数据
async function fetchNewData() {
  const res = await fetch('/api/products') // 假设返回 10000 条数据
  const data = [
    { id: 1, name: 'iPhone', price: 5999 },
    { id: 2, name: 'iPad', price: 3999 },
    { id: 3, name: 'MacBook', price: 9999 }
  ]
  // ✅ 替换整个 .value → 触发响应式更新
  products.value = data
}

// ❌ 直接修改内部属性不会触发更新（因为是 shallow）
function updatePrice(product) {
  product.price += 100

  // ✅ 手动触发更新（告诉 Vue：虽然内部变了，但请更新）
  triggerRef(products)
}

// 首次加载
fetchNewData()
</script>

```

## 📖三、其它
### **🧠 设计哲学：为什么叫 shallowRef？**
|单词	|含义	|设计意图|
|---|---|---|
|`shallow`	|浅的，不深的	|只监听 .value 是否被替换，不深入监听内部属性|
|`Ref`	|Reference（引用）	|依然是一个 ref，可以 .value 访问|

👉 所以 `shallowRef` = “**只在表层建立响应性的** ref”

>💡 类比：你有一个盒子（`.value`），Vue 只关心盒子有没有换，不关心盒子里的东西变没变。

### **🎯 设计动机：为什么要设计 shallowRef？**
❌ **问题：深度响应式太“重”**
当你用 `ref({ bigObject })` 时，Vue 会递归把` bigObject` 的每一个属性都变成响应式（通过 `reactive`），这在以下场景会很慢：

* 对象非常大（如：10000 条商品数据）
* 数据是只读的（不需要监听内部变化）
* 频繁替换整个对象（如：从接口拉新数据）
这时，**深度响应式就成了性能负担。**

✅ **解法：shallowRef —— 只监听 .value 被替换**
* 如果你只是替换整个对象（如 `state.value = newData`），那` .value` 变了，视图更新 ✅
* 如果你改对象内部属性（如 `state.value.items[0].name = 'new'`），**不会触发更新** ❌（除非手动 `triggerRef`）
>🚀 优势：避免不必要的递归监听，性能极高

###  📌 六、关键点解析
|方法|	作用|	使用场景|
|---|---|---|
|`shallowRef(obj)`|	创建浅响应 ref	|大对象、整块替换|
|`triggerRef(shallowRef)`	|手动触发更新|	修改了内部属性但想更新视图|
|`triggerRef `|不是响应式	是副作用触发	|必须手动调用|