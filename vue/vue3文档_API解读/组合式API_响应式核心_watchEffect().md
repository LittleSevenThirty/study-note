# Vue 3 响应式 API 核心：watchEffect 详解指南

> **适用人群**：Vue 3 初学者、一段时间未使用 Vue 需要复习的开发者
>
> **核心目标**：全面理解 Vue 3 中 `watchEffect` API 的使用方法与应用场景

## 一、什么是 watchEffect？

`watchEffect` 是 Vue 3 响应式系统中的一个核心 API，它会 **<font color=red>立即运行一个函数，同时自动追踪函数内部使用的响应式数据</font>** ，并在这些数据发生变化时重新执行该函数。**这就是和watch的重要区别**

### 与 watch 的区别

- `watchEffect`：自动收集函数内部使用的响应式依赖，立即执行，无法直接获取旧值
- `watch`：需要明确指定要侦听的数据源，懒执行（默认不立即执行），可以获取新值和旧值

简单来说，当你想让一段代码"自动感知"它使用的响应式数据并在数据变化时重新运行，`watchEffect` 就是一个理想选择。

## 二、基础语法与用法

### 1. 基本用法

```javascript
import { ref, watchEffect } from "vue";

const count = ref(0);

// watchEffect 会立即执行一次
watchEffect(() => {
  console.log(`当前 count 值: ${count.value}`);
  // 自动追踪 count 作为依赖
});

// 当 count 变化时，会再次执行
count.value++; // 输出：当前 count 值: 1
```

### 2. 返回值与停止侦听

`watchEffect` 返回一个停止函数，用于停止侦听器：

```javascript
const stop = watchEffect(() => {
  console.log(count.value);
});

// 当不再需要此侦听器时
stop();
```

### 3. 暂停/恢复侦听 (3.4+)

```javascript
const { stop, pause, resume } = watchEffect(() => {
  console.log(count.value);
});

// 暂停侦听
pause();

count.value++; // 不会触发 console.log

// 恢复侦听
resume();

count.value++; // 会再次触发 console.log

// 停止侦听
stop();
```

## 三、副作用清理

在某些情况下，需要在副作用重新运行前清理上一次的副作用，比如取消未完成的异步操作：

### 1. 传统方式：通过 onCleanup 参数

```javascript
watchEffect(async (onCleanup) => {
  // 假设这是个耗时的异步API调用
  const { response, cancel } = fetchData(someId.value);

  // 注册清理函数
  onCleanup(() => {
    cancel(); // 取消之前的请求
  });

  data.value = await response;
});
```

清理函数 `onCleanup` 会在以下情况被调用：

- 副作用即将重新执行时
- 侦听器被停止时（组件卸载或手动调用 stop()）

### 2. Vue 3.5+ 新方式：使用 onWatcherCleanup

```javascript
import { watchEffect, onWatcherCleanup } from "vue";

watchEffect(async () => {
  const { response, cancel } = fetchData(someId.value);

  // 使用独立API注册清理函数
  onWatcherCleanup(() => {
    cancel();
  });

  data.value = await response;
});
```

这种方式更灵活，可以在异步操作中使用，不受同步执行的限制。

## 四、配置选项详解

`watchEffect` 的第二个参数是一个选项对象，用于配置侦听行为：

### 1. flush: 控制执行时机

```javascript
watchEffect(
  () => {
    console.log(count.value);
  },
  {
    flush: "pre", // 默认值，在组件更新前执行
  },
);

watchEffect(
  () => {
    console.log(count.value);
  },
  {
    flush: "post", // 在组件更新后执行
  },
);

watchEffect(
  () => {
    console.log(count.value);
  },
  {
    flush: "sync", // 同步执行，数据变化后立即触发
  },
);
```

- `'pre'` (默认)：在组件更新前执行，可以访问更新前的 DOM
- `'post'`：在组件更新后执行，可以访问更新后的 DOM
- `'sync'`：同步执行，慎用，可能导致性能和数据一致性问题

### 2. 调试选项：onTrack 和 onTrigger

用于调试依赖追踪过程：

```javascript
watchEffect(
  () => {
    console.log(count.value);
  },
  {
    onTrack(e) {
      debugger; // 当响应式属性被追踪时触发
    },
    onTrigger(e) {
      debugger; // 当依赖触发侦听器重新运行时触发
    },
  },
);
```

## 五、常用别名

Vue 提供了两个 `watchEffect` 的常用别名，对应不同的刷新时机：

### 1. watchPostEffect

等同于 `watchEffect` 使用 `flush: 'post'` 选项：

```javascript
import { watchPostEffect } from "vue";

// 等同于 watchEffect(fn, { flush: 'post' })
watchPostEffect(() => {
  console.log("在组件更新后执行");
});
```

### 2. watchSyncEffect

等同于 `watchEffect` 使用 `flush: 'sync'` 选项：

```javascript
import { watchSyncEffect } from "vue";

// 等同于 watchEffect(fn, { flush: 'sync' })
watchSyncEffect(() => {
  console.log("同步执行，数据变化后立即触发");
});
```

## 六、实用示例

### 1. 自动保存表单数据

```javascript
const formData = reactive({
  name: "",
  email: "",
});

watchEffect(() => {
  // 自动保存，当任何表单字段变化时触发
  localStorage.setItem("form-data", JSON.stringify(formData));
});
```

### 2. 动态获取数据

```javascript
const userId = ref(1);
const userData = ref(null);

watchEffect(async () => {
  // 当 userId 变化时，自动获取新用户数据
  userData.value = await fetchUser(userId.value);
});
```

### 3. DOM 操作与动画

```javascript
const element = ref(null);
const isVisible = ref(true);

// 在组件更新后操作DOM
watchPostEffect(() => {
  if (element.value) {
    if (isVisible.value) {
      element.value.style.opacity = 1;
    } else {
      element.value.style.opacity = 0;
    }
  }
});
```

### 4. 响应式条件请求

```javascript
const searchQuery = ref("");
const searchResults = ref([]);

watchEffect(async (onCleanup) => {
  if (searchQuery.value.length < 2) {
    searchResults.value = [];
    return;
  }

  let cancelled = false;
  onCleanup(() => {
    cancelled = true;
  });

  // 防抖处理
  await new Promise((resolve) => setTimeout(resolve, 300));

  if (cancelled) return;

  const results = await searchAPI(searchQuery.value);
  if (!cancelled) {
    searchResults.value = results;
  }
});
```

## 七、最佳实践与注意事项

### 1. 何时选择 watchEffect

- 需要立即执行副作用
- 依赖关系复杂或动态，难以明确指定
- 副作用函数逻辑简单，不需要旧值
- 响应式数据使用模式是"自动追踪"而非"明确指定"

### 2. 何时避免 watchEffect

- 需要访问变化前的旧值
- 需要精确控制何时重新运行（避免不必要的执行）
- 初始化时不需要立即执行
- 依赖关系非常明确且简单

### 3. 性能考量

- 避免在 `watchEffect` 中创建大量不必要的响应式依赖
- 对于复杂计算，考虑使用 `computed` 而非 `watchEffect`
- 频繁变化的数据应考虑防抖或节流
- 3.5+ 版本中，使用 `onWatcherCleanup` 代替参数方式可以更好地处理异步清理

### 4. 代码可维护性

- 避免在单个 `watchEffect` 中处理过多不相关的副作用
- 为复杂副作用添加注释，说明依赖关系
- 需要精确控制时，优先考虑 `watch` 而非 `watchEffect`

## 八、常见问题解答

**Q: watchEffect 和生命周期钩子有什么区别？**

A:

- `onMounted` 等生命周期钩子只执行一次
- `watchEffect` 会持续追踪依赖并在依赖变化时重新执行
- 通常在 `onMounted` 中初始化数据，在 `watchEffect` 中处理数据变化的副作用

**Q: 为什么我的 watchEffect 没有在数据变化时触发？**

A: 可能原因：

- 没有正确访问响应式数据的 `.value` 属性
- 依赖的数据不是响应式的
- 侦听器已被手动停止
- 在作用域外修改了响应式数据（例如在 setTimeout 中使用了过时的引用）

**Q: 如何在 setup() 外使用 watchEffect？**

A: 需要注意在组件卸载时手动清理：

```javascript
let stopWatchEffect = null;

onMounted(() => {
  stopWatchEffect = watchEffect(() => {
    // 你的代码
  });
});

onUnmounted(() => {
  if (stopWatchEffect) {
    stopWatchEffect();
  }
});
```

**Q: watchEffect 能否替代所有 watch 的使用场景？**

A: 不能。虽然功能有重叠，但它们有不同的用途：

- 需要访问旧值和精确控制依赖时，`watch` 更合适
- 初始化不需要执行且依赖明确时，`watch` 更合适
- 需要立即执行且依赖自动追踪时，`watchEffect` 更合适

---
