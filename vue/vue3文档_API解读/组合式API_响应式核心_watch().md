# Vue 3 响应式 API 核心：watch 详解指南

> **适用人群**：Vue 3 初学者、一段时间未使用 Vue 需要复习的开发者
>
> **核心目标**：全面理解 Vue 3 中 `watch` API 的使用方法与应用场景
> 侦听一个或多个响应式数据源，并在数据源变化时调用所给的回调函数。

## 一、什么是 watch？

在 Vue 3 的响应式系统中，`watch` 是一个核心 API，用于**侦听响应式数据的变化**，并在数据变化时执行特定的操作。简单来说，当你需要在某个数据改变时执行一些副作用（如发送网络请求、更新 DOM、记录日志等）时，`watch` 就是你的好帮手。

### 与 watchEffect 的区别

- `watch`：明确指定要侦听的数据源，在这些数据变化时才触发回调，可以获取新值和旧值
- `watchEffect`：自动收集依赖，立即运行并追踪内部使用的响应式数据，无法直接获取旧值

`watch` 更适合需要精确控制、需要访问变化前后值的场景。

## 二、基础用法

```ts
// 侦听单个来源
function watch<T>(
  source: WatchSource<T>,
  callback: WatchCallback<T>,
  options?: WatchOptions,
): WatchHandle;

// 侦听多个来源
function watch<T>(
  sources: WatchSource<T>[],
  callback: WatchCallback<T[]>,
  options?: WatchOptions,
): WatchHandle;

type WatchCallback<T> = (
  value: T,
  oldValue: T,
  onCleanup: (cleanupFn: () => void) => void,
) => void;

type WatchSource<T> =
  | Ref<T> // ref
  | (() => T) // getter
  | (T extends object ? T : never); // 响应式对象

interface WatchOptions extends WatchEffectOptions {
  immediate?: boolean; // 默认：false
  deep?: boolean | number; // 默认：false
  flush?: "pre" | "post" | "sync"; // 默认：'pre'
  onTrack?: (event: DebuggerEvent) => void;
  onTrigger?: (event: DebuggerEvent) => void;
  once?: boolean; // 默认：false (3.4+)
}

interface WatchHandle {
  (): void; // 可调用，与 `stop` 相同
  pause: () => void;
  resume: () => void;
  stop: () => void;
}
```

### 1. 侦听单个数据源

```javascript
import { ref, watch } from "vue";

const count = ref(0);

// 侦听 ref
watch(count, (newCount, oldCount) => {
  console.log(`count 从 ${oldCount} 变为 ${newCount}`);
});

// 也可以侦听一个 getter 函数
const state = reactive({ count: 0 });
watch(
  () => state.count,
  (newCount, oldCount) => {
    console.log(`state.count 从 ${oldCount} 变为 ${newCount}`);
  },
);
```

### 2. 侦听多个数据源

```javascript
const firstName = ref("John");
const lastName = ref("Doe");

watch(
  [firstName, lastName],
  ([newFirstName, newLastName], [oldFirstName, oldLastName]) => {
    console.log(
      `姓名从 ${oldFirstName} ${oldLastName} 变为 ${newFirstName} ${newLastName}`,
    );
  },
);
```

## 三、watch 选项详解

`watch` 的第三个参数是一个选项对象，可以用来配置侦听行为：

### 1. immediate: 立即执行

默认情况下，`watch` 是懒执行的（只在侦听源变化时才执行回调）。设置 `immediate: true` 可以在创建侦听器时立即执行一次回调。

```javascript
watch(
  count,
  (newCount, oldCount) => {
    // 第一次执行时，oldCount 是 undefined
    console.log(`count changed from ${oldCount} to ${newCount}`);
  },
  {
    immediate: true, // 创建后立即执行一次
  },
);
```

### 2. deep: 深度侦听

当侦听对象或数组时，有时需要侦听其内部属性的变化。这时需要设置 `deep: true`。

```javascript
const user = reactive({
  name: "John",
  address: {
    city: "New York",
  },
});

// 普通侦听，只在 user 引用变化时触发（内部属性变化不会触发）
watch(user, () => {
  console.log("user changed");
});

// 深度侦听，内部属性变化也会触发
watch(
  user,
  (newUser, oldUser) => {
    console.log("user or its nested properties changed");
    // 注意：深度侦听时，newUser 和 oldUser 实际上是同一个对象
    console.log(newUser === oldUser); // true
  },
  {
    deep: true,
  },
);

// 3.5+ 版本支持设置深度级别
watch(
  user,
  () => {
    console.log("仅侦听两层深度的变化");
  },
  {
    deep: 2,
  },
);
```

### 3. flush: 控制回调刷新时机

控制回调函数的执行时机，有三个选项：

- `'pre'`（默认）：在组件更新前执行，可以访问更新前的 DOM
- `'post'`：在组件更新后执行，可以访问更新后的 DOM
- `'sync'`：同步执行，数据变化后立即触发（谨慎使用，可能影响性能）

```javascript
watch(
  count,
  () => {
    console.log("在组件更新前执行");
  },
  {
    flush: "pre", // 默认值
  },
);

watch(
  count,
  () => {
    console.log("在组件更新后执行");
  },
  {
    flush: "post",
  },
);
```

### 4. once: 只执行一次 (3.4+)

设置 `once: true` 使回调只执行一次，之后自动停止侦听。

```javascript
watch(
  count,
  () => {
    console.log("这只会在 count 第一次变化时执行");
  },
  {
    once: true,
  },
);
```

### 5. 调试选项：onTrack 和 onTrigger

用于调试侦听器的依赖关系：

```javascript
watch(
  count,
  () => {
    console.log("count changed");
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

## 四、高级用法

### 1. 副作用清理

有时需要在侦听器重新运行前清理上一次的副作用，比如取消未完成的网络请求：

```javascript
watch(id, async (newId, oldId, onCleanup) => {
  const { response, cancel } = fetchData(newId);

  // 注册清理函数
  onCleanup(() => {
    cancel(); // 取消之前的请求
  });

  data.value = await response;
});
```

在 Vue 3.5+ 中，也可以使用 `onWatcherCleanup` API：

```javascript
import { watch, onWatcherCleanup } from "vue";

watch(id, async (newId) => {
  const { response, cancel } = fetchData(newId);

  // 使用独立 API 注册清理函数
  onWatcherCleanup(() => {
    cancel();
  });

  data.value = await response;
});
```

### 2. 停止侦听器

有时需要手动停止侦听器，比如在组件卸载时：

```javascript
const stopWatch = watch(count, () => {
  console.log("count changed");
});

// 当不再需要该侦听器时
stopWatch();
```

### 3. 暂停/恢复侦听器

Vue 3.4+ 支持暂停和恢复侦听器：

```javascript
const { stop, pause, resume } = watch(count, () => {
  console.log("count changed");
});

// 暂停侦听
pause();

// 恢复侦听
resume();

// 停止侦听
stop();
```

## 五、实用示例

### 1. 表单验证

```javascript
const email = ref("");
const emailError = ref("");

watch(
  email,
  (newEmail) => {
    if (!/^\S+@\S+\.\S+$/.test(newEmail)) {
      emailError.value = "请输入有效的邮箱地址";
    } else {
      emailError.value = "";
    }
  },
  {
    immediate: true, // 立即验证初始值
  },
);
```

### 2. 搜索优化

```javascript
const searchTerm = ref("");
const searchResults = ref([]);

watch(
  searchTerm,
  async (newTerm, oldTerm, onCleanup) => {
    if (newTerm.length < 2) {
      searchResults.value = [];
      return;
    }

    // 取消之前的请求
    let cancelled = false;
    onCleanup(() => {
      cancelled = true;
    });

    // 添加防抖
    await new Promise((resolve) => setTimeout(resolve, 300));

    if (cancelled) return;

    const results = await searchAPI(newTerm);
    if (!cancelled) {
      searchResults.value = results;
    }
  },
  {
    immediate: true,
  },
);
```

### 3. 数据同步

```javascript
// 将本地状态同步到 localStorage
const preferences = reactive({
  theme: "light",
  fontSize: 14,
});

watch(
  preferences,
  () => {
    localStorage.setItem("preferences", JSON.stringify(preferences));
  },
  {
    deep: true, // 深度侦听，内部属性变化也触发
  },
);
```

## 六、最佳实践与注意事项

1. **选择合适的侦听器**：
   - 需要访问旧值和新值时，使用 `watch`
   - 只需要响应数据变化执行操作，不关心旧值，可考虑 `watchEffect`

2. **避免过度使用深度侦听**：
   - 深度侦听 (`deep: true`) 会带来性能开销
   - 仅在必要时使用，或使用 `deep: number` 限制深度

3. **合理处理异步操作**：
   - 始终考虑清理未完成的异步操作
   - 使用 `onCleanup` 或 `onWatcherCleanup` 防止内存泄漏

4. **组件卸载时清理**：
   - 组件卸载时，Vue 会自动清理侦听器
   - 但手动创建的侦听器（如在 setup() 外部）需要手动停止

5. **性能考虑**：
   - 避免在侦听回调中执行昂贵操作
   - 考虑使用防抖（debounce）或节流（throttle）优化频繁变化的数据

## 七、常见问题解答

**Q: 什么时候应该用 watch 而不是 computed？**

A:

- 使用 `computed`：当需要基于其他响应式数据派生出新值，且这个过程是同步、无副作用的
- 使用 `watch`：当需要在数据变化时执行异步操作或执行复杂、有副作用的操作

**Q: 为什么在深度侦听对象时，新值和旧值是相同的？**

A: 深度侦听时，Vue 不会创建对象的副本，而是直接比较引用。因此当对象内部属性变化时，新值和旧值引用的是同一个对象。如果你需要比较具体属性的变化，可以在回调中手动比较特定属性。

**Q: 如何只侦听对象的特定属性？**

A: 使用 getter 函数返回需要侦听的属性：

```javascript
watch(
  () => user.name, // 只侦听 name 属性
  (newName, oldName) => {
    console.log(`name changed from ${oldName} to ${newName}`);
  },
);
```
