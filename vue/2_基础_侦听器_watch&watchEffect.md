# Vue 3 侦听器（Watchers）新手完全指南

> 本文基于 Vue 3 官方文档整理，旨在帮助初学者系统理解侦听器的使用方法。侦听器是 Vue 响应式系统的重要组成部分，让我们能够在数据变化时执行特定操作。

## 一、什么是侦听器？

在 Vue 中，**侦听器**（Watchers）是一种让我们能够在数据变化时**执行副作用操作**的机制。当你需要在数据变化时执行异步操作、复杂的计算或直接操作 DOM 时，侦听器就派上用场了。

> **小贴士**：计算属性（Computed）适合处理简单的派生状态，而侦听器则适合处理更复杂的逻辑，尤其是需要执行异步操作或产生副作用的场景。

## 二、基本使用

### 1. 选项式 API 中的 watch

在选项式 API 中，我们使用 `watch` 选项来定义侦听器：

```javascript
export default {
  data() {
    return {
      question: "",
      answer: "Questions usually contain a question mark. ;-)",
      loading: false,
    };
  },
  watch: {
    // 每当 question 改变时，这个函数就会执行
    question(newQuestion, oldQuestion) {
      if (newQuestion.includes("?")) {
        this.getAnswer();
      }
    },
  },
  methods: {
    async getAnswer() {
      this.loading = true;
      this.answer = "Thinking...";
      try {
        const res = await fetch("https://yesno.wtf/api");
        this.answer = (await res.json()).answer;
      } catch (error) {
        this.answer = "Error! Could not reach the API. " + error;
      } finally {
        this.loading = false;
      }
    },
  },
};
```

对应的模板：

```html
<p>Ask a yes/no question: <input v-model="question" :disabled="loading" /></p>
<p>{{ answer }}</p>
```

这个例子中，当用户在输入框中输入问题（`question`）并且包含问号时，会自动调用 API 获取答案。

### 2. 组合式 API 中的 watch

在组合式 API 中，使用 `watch` 函数来实现相同的功能：

```vue
<script setup>
import { ref, watch } from "vue";

const question = ref("");
const answer = ref("Questions usually contain a question mark. ;-)");
const loading = ref(false);

// 侦听 question ref
watch(question, async (newQuestion, oldQuestion) => {
  if (newQuestion.includes("?")) {
    loading.value = true;
    answer.value = "Thinking...";
    try {
      const res = await fetch("https://yesno.wtf/api");
      answer.value = (await res.json()).answer;
    } catch (error) {
      answer.value = "Error! Could not reach the API. " + error;
    } finally {
      loading.value = false;
    }
  }
});
</script>

<template>
  <p>Ask a yes/no question: <input v-model="question" :disabled="loading" /></p>
  <p>{{ answer }}</p>
</template>
```

## 三、侦听数据源的类型

`watch` 函数的第一个参数可以是多种类型的"数据源"：

1. **单个 ref**：

```javascript
const x = ref(0);
watch(x, (newX) => {
  console.log(`x is ${newX}`);
});
```

2. **getter 函数**：

```javascript
const x = ref(0);
const y = ref(0);
watch(
  () => x.value + y.value,
  (sum) => {
    console.log(`sum of x + y is: ${sum}`);
  },
);
```

3. **多个数据源组成的数组**：

```javascript
watch([x, () => y.value], ([newX, newY]) => {
  console.log(`x is ${newX} and y is ${newY}`);
});
```

4. **注意点**：不能直接侦听响应式对象的属性值：

```javascript
const obj = reactive({ count: 0 });

// 错误示例，因为 watch() 得到的参数是一个 number
watch(obj.count, (count) => {
  console.log(`Count is: ${count}`);
});

// 正确做法：使用 getter 函数
watch(
  () => obj.count,
  (count) => {
    console.log(`Count is: ${count}`);
  },
);
```

## 四、深层侦听器

默认情况下，`watch` 是**浅层**的，只会在被侦听的属性被赋新值时触发，而不会侦听嵌套属性的变化。

### 1. 选项式 API 实现深层侦听

```javascript
export default {
  watch: {
    someObject: {
      handler(newValue, oldValue) {
        // 嵌套属性变更时也会触发
      },
      deep: true,
    },
  },
};
```

### 2. 组合式 API 实现深层侦听

```javascript
const obj = reactive({ count: 0 });

// 侦听整个响应式对象
watch(
  obj,
  (newValue, oldValue) => {
    // 嵌套属性变更时会触发
    // 注意：newValue 和 oldValue 是同一个对象！
  },
  { deep: true },
);

obj.count++; // 会触发侦听器
```

### 3. 性能注意事项

深层侦听需要遍历对象中的所有嵌套属性，在大型数据结构上使用时开销很大。应只在必要时使用，并留意性能影响。

> **提示**：在 Vue 3.5+ 中，`deep` 选项还可以是一个数字，表示最大遍历深度，可以更精细地控制侦听范围。

## 五、即时回调的侦听器

默认情况下，`watch` 是懒执行的，只会在侦听的数据变化时执行回调。但在某些场景中，我们希望在创建侦听器时立即执行一次回调，例如请求初始数据。

### 1. 选项式 API

```javascript
export default {
  watch: {
    question: {
      handler(newQuestion) {
        // 组件创建时立即调用
      },
      immediate: true, // 强制立即执行
    },
  },
};
```

### 2. 组合式 API

```javascript
watch(
  question,
  (newQuestion) => {
    // 立即执行，且当 question 改变时再次执行
  },
  { immediate: true },
);
```

> **注意**：回调函数的初次执行发生在 `created` 钩子之前。Vue 此时已经处理了 `data`、`computed` 和 `methods` 选项，所以这些属性在第一次调用时是可用的。

## 六、一次性侦听器

有时我们只希望在数据源首次变化时触发一次回调。在 Vue 3.4+ 中，可以使用 `once: true` 选项：

### 1. 选项式 API

```javascript
export default {
  watch: {
    source: {
      handler(newValue, oldValue) {
        // 仅触发一次
      },
      once: true,
    },
  },
};
```

### 2. 组合式 API

```javascript
watch(
  source,
  (newValue, oldValue) => {
    // 仅触发一次
  },
  { once: true },
);
```

## 七、watchEffect 介绍

`watchEffect` 是一个特殊的 API，它会自动追踪回调函数中的响应式依赖，不需要显式指定侦听的数据源。

### 基本用法

```javascript
import { ref, watchEffect } from "vue";

const todoId = ref(1);
const data = ref(null);

// 使用 watch
watch(
  todoId,
  async () => {
    const response = await fetch(
      `https://jsonplaceholder.typicode.com/todos/${todoId.value}`,
    );
    data.value = await response.json();
  },
  { immediate: true },
);

// 使用 watchEffect 简化
watchEffect(async () => {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/todos/${todoId.value}`,
  );
  data.value = await response.json();
});
```

`watchEffect` 会立即执行一次，并自动追踪 `todoId.value` 作为依赖。当 `todoId.value` 变化时，回调会再次执行。

> **重要提示**：`watchEffect` 仅会在其同步执行期间追踪依赖。在使用异步回调时，只有在第一个 `await` 之前访问到的属性才会被追踪。

## 八、watch 与 watchEffect 的区别

| 特点          | watch                              | watchEffect                                |
| ------------- | ---------------------------------- | ------------------------------------------ |
| 依赖追踪      | 明确指定侦听的数据源               | 自动追踪回调中使用的响应式属性             |
| 回调触发时机  | 仅在侦听的数据变化时触发           | 依赖变化时触发，且会立即执行一次           |
| 访问旧值/新值 | 可以访问旧值和新值                 | 无法访问旧值，只有新值                     |
| 适用场景      | 需要精确控制触发时机，需要访问旧值 | 依赖关系简单，不需要旧值，需要自动追踪依赖 |

**选择建议**：

- 当需要访问变化前后的值，或需要精确控制触发时机时，使用 `watch`
- 当依赖关系较复杂，或想简化代码时，使用 `watchEffect`

## 九、副作用清理

在执行异步操作时，如果数据在前一个操作完成前发生变化，可能导致竞态条件。我们可以使用清理函数来解决这个问题。

### 1. Vue 3.5+ 中的 onWatcherCleanup

```javascript
import { watch, onWatcherCleanup } from "vue";

watch(id, (newId) => {
  const controller = new AbortController();
  fetch(`/api/${newId}`, { signal: controller.signal }).then(() => {
    /* 回调逻辑 */
  });

  // 注册清理函数
  onWatcherCleanup(() => {
    controller.abort(); // 取消过期请求
  });
});
```

### 2. 通用方法：通过参数传递清理函数

在 Vue 3.5 之前的版本或异步函数中，可以使用回调参数方式：

```javascript
watch(id, (newId, oldId, onCleanup) => {
  const controller = new AbortController();
  fetch(`/api/${newId}`, { signal: controller.signal }).then(() => {
    /* 回调逻辑 */
  });

  onCleanup(() => {
    controller.abort();
  });
});

// 对于 watchEffect
watchEffect((onCleanup) => {
  const controller = new AbortController();
  fetch(`/api/${id.value}`, { signal: controller.signal }).then(() => {
    /* 回调逻辑 */
  });

  onCleanup(() => {
    controller.abort();
  });
});
```

## 十、回调的触发时机

侦听器回调的触发时机可以通过 `flush` 选项控制：

### 1. 默认时机

默认情况下，侦听器回调会在：

- 父组件更新之后
- 当前组件 DOM 更新之前

这意味着在侦听器回调中访问 DOM，得到的是更新前的状态。

### 2. 后置刷新（post）

```javascript
// 选项式 API
watch: {
  key: {
    handler() { /* ... */ },
    flush: 'post'
  }
}

// 组合式 API
watch(source, callback, { flush: 'post' })
watchEffect(callback, { flush: 'post' })
```

当设置 `flush: 'post'` 时，回调会在组件 DOM 更新后执行，这时可以访问到更新后的 DOM。

> **便利函数**：`watchPostEffect()` 是 `watchEffect` 设置 `flush: 'post'` 的简写：
>
> ```javascript
> import { watchPostEffect } from "vue";
> watchPostEffect(() => {
>   /* 在 Vue 更新后执行 */
> });
> ```

### 3. 同步刷新（sync）

```javascript
// 选项式 API
watch: {
  key: {
    handler() { /* ... */ },
    flush: 'sync'
  }
}

// 组合式 API
watch(source, callback, { flush: 'sync' })
watchEffect(callback, { flush: 'sync' })
```

当设置 `flush: 'sync'` 时，侦听器会在响应式数据变化后立即同步执行，不进行批处理。

> **警告**：同步侦听器会降低性能，应谨慎使用。它们不会进行批处理，每当检测到响应式数据变化就会触发。适用于监视简单布尔值，但应避免用于可能多次同步修改的数据源（如数组）。
>
> **便利函数**：`watchSyncEffect()` 是 `watchEffect` 设置 `flush: 'sync'` 的简写：
>
> ```javascript
> import { watchSyncEffect } from "vue";
> watchSyncEffect(() => {
>   /* 在响应式数据变化时同步执行 */
> });
> ```

## 十一、命令式创建侦听器

除了声明式方法，还可以使用组件实例的 `$watch` 方法命令式创建侦听器：

```javascript
export default {
  created() {
    this.$watch("question", (newQuestion) => {
      // 可以动态创建侦听器
    });
  },
};
```

这种方法适用于需要在特定条件下设置侦听器，或者只侦听响应用户交互的内容。

## 十二、停止侦听器

通常情况下，侦听器会随着宿主组件的卸载而自动停止。但在某些情况下，需要手动停止侦听器。

### 1. 通过返回的取消函数

```javascript
// 选项式 API
const unwatch = this.$watch("foo", callback);
// 不再需要时
unwatch();

// 组合式 API
const unwatch = watch(source, callback);
// 不再需要时
unwatch();
```

### 2. 侦听器生命周期注意事项

- 在 `setup()` 或 `<script setup>` 中**同步**创建的侦听器，会自动绑定到组件实例，并在组件卸载时自动停止
- **异步**创建的侦听器（如 setTimeout 内部）不会自动绑定到组件，必须手动停止，防止内存泄漏

```vue
<script setup>
import { watchEffect } from "vue";

// 自动停止
watchEffect(() => {
  /* ... */
});

// 不会自动停止，需要手动清理
setTimeout(() => {
  const unwatch = watchEffect(() => {
    /* ... */
  });
  // 需要在适当时候调用 unwatch()
}, 100);
</script>
```

### 3. 最佳实践

尽量避免异步创建侦听器。如果需要等待异步数据，可以使用条件式侦听：

```javascript
const data = ref(null);

watchEffect(() => {
  if (data.value) {
    // 数据加载后执行操作
  }
});
```

## 十三、总结与最佳实践

1. **选择合适的侦听方式**：
   - 简单场景：使用 `watchEffect`
   - 需要访问旧值或精确控制：使用 `watch`

2. **避免过度使用深层侦听**：
   - 深层侦听性能开销大
   - 优先考虑侦听特定属性或使用 getter 函数

3. **管理异步操作**：
   - 使用清理函数处理过期的异步请求
   - 防止竞态条件导致的状态不一致

4. **注意侦听器的生命周期**：
   - 了解自动停止的条件
   - 手动创建的侦听器需要手动清理

5. **理解触发时机**：
   - 默认时机：父组件更新后，当前组件 DOM 更新前
   - 根据需要调整 `flush` 选项

侦听器是 Vue 响应式系统中强大而灵活的工具，合理使用可以帮助我们构建更高效、更响应式的应用。掌握这些概念和技巧，将使你能够更自信地处理各种数据变化场景。
