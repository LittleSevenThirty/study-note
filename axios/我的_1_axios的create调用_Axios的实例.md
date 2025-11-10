# 🧠 Axios 实例（Instance）—— 长期记忆友好版

> ✅ 目标：即使半年没碰 Axios，也能通过这份笔记快速恢复上下文  
> ✅ 方法：真实项目结构 + 完整可运行代码 + 注释 + 设计理念说明

---

## 一、为什么需要 “Axios 实例”？（设计理念）

### 🔍 问题场景
在真实项目中，你通常有：
- 多个 API 接口地址（比如用户服务、订单服务）
- 统一的请求头（如 `Authorization: Bearer xxx`）
- 全局超时设置（如 5 秒）
- 拦截器（统一处理 token 刷新、错误提示等）

如果每次请求都手动写：
```js
axios.get('https://user-api.com/v1/profile', {
  headers: { Authorization: '...' },
  timeout: 5000
})
```
不仅重复，还难以维护。

### 💡 设计哲学
> **“一次配置，处处复用”**  
> Axios 实例允许你创建一个**预配置好的请求客户端**，就像“定制一把专属扳手”，以后所有请求都用这把扳手，不用每次都重新组装。

### 🎯 核心动机
- **解耦配置与调用**：配置集中管理，调用只关心业务逻辑
- **支持多环境/多服务**：可以创建多个实例（如 userApi、orderApi）
- **便于测试和拦截**：拦截器绑定到实例，不影响全局

---

## 二、项目结构示例（清晰定位文件作用）

假设我们正在开发一个 Vue 3 + Vite 的前端项目：

```
src/
├── api/
│   ├── index.js          ← 【核心】导出所有 API 实例
│   ├── userApi.js        ← 用户服务专用实例
│   └── orderApi.js       ← 订单服务专用实例
├── utils/
│   └── request.js        ← 【可选】通用基础实例（带拦截器）
└── views/
    └── Profile.vue       ← 使用 userApi 的组件
```

> ✅ 这种结构让每个 API 服务独立、可测试、可复用。

---

## 三、完整代码示例（带详细注释）

### 文件：`src/utils/request.js`  
> 📌 作用：创建一个**带拦截器的基础 Axios 实例**，作为其他实例的“基类”

```js
// src/utils/request.js
import axios from 'axios';

// 1. 创建一个基础实例，包含通用配置
const baseInstance = axios.create({
  // 所有请求自动加上这个前缀（避免在每个 URL 写完整域名）
  baseURL: import.meta.env.VITE_API_BASE_URL || 'http://localhost:3000/api',
  
  // 全局超时时间（毫秒）
  timeout: 10000,
  
  // 默认请求头
  headers: {
    'Content-Type': 'application/json'
  }
});

// 2. 添加响应拦截器（统一处理 token 过期、错误提示等）
baseInstance.interceptors.response.use(
  // 成功响应：直接返回 data，省去 .data 的麻烦
  (response) => response.data,
  
  // 错误响应：统一处理
  (error) => {
    if (error.response?.status === 401) {
      // token 过期，跳转登录页
      window.location.href = '/login';
    } else {
      // 其他错误，弹出提示（这里简化为 console）
      console.error('请求失败:', error.response?.data?.message || error.message);
    }
    return Promise.reject(error);
  }
);

export default baseInstance;
```

---

### 文件：`src/api/userApi.js`  
> 📌 作用：基于基础实例，创建**用户服务专用实例**

```js
// src/api/userApi.js
import baseInstance from '@/utils/request';

// 1. 创建用户服务实例（继承 baseInstance 的配置和拦截器）
const userApi = baseInstance;

// 2. 可以进一步覆盖或扩展配置（可选）
// userApi.defaults.baseURL = 'https://user-service.example.com'; // 如果用户服务是独立域名

// 3. 导出具体的 API 方法（封装业务逻辑）
export const getUserProfile = () => userApi.get('/user/profile');
export const updateUserProfile = (data) => userApi.put('/user/profile', data);
export const login = (credentials) => userApi.post('/auth/login', credentials);

// ✅ 现在这些方法都自动带有：
// - baseURL 前缀
// - 超时设置
// - 拦截器（自动处理 401、返回 .data）
```

---

### 文件：`src/api/orderApi.js`  
> 📌 作用：订单服务可能用不同域名，所以单独配置

```js
// src/api/orderApi.js
import axios from 'axios';

// 1. 订单服务使用独立实例（因为可能是不同后端团队维护）
const orderApi = axios.create({
  baseURL: 'https://order-service.example.com/v2', // 不同域名
  timeout: 15000, // 更长的超时（订单操作可能慢）
  headers: {
    'X-API-Key': 'your-order-service-key' // 特殊认证方式
  }
});

// 2. 也可以添加专属拦截器（比如记录订单日志）
orderApi.interceptors.request.use(config => {
  console.log('发送订单请求:', config.url);
  return config;
});

// 3. 导出订单相关方法
export const createOrder = (items) => orderApi.post('/orders', items);
export const getOrderList = () => orderApi.get('/orders');

export default orderApi;
```

---

### 文件：`src/api/index.js`  
> 📌 作用：统一导出，方便其他地方引入

```js
// src/api/index.js
export * from './userApi';
export { default as orderApi } from './orderApi';
```

---

### 文件：`src/views/Profile.vue`  
> 📌 作用：在组件中使用封装好的 API

```vue
<!-- src/views/Profile.vue -->
<template>
  <div v-if="profile">
    <h1>{{ profile.name }}</h1>
    <p>Email: {{ profile.email }}</p>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
// 从统一入口导入
import { getUserProfile, updateUserProfile } from '@/api';

const profile = ref(null);

onMounted(async () => {
  try {
    // ✅ 注意：这里直接拿到的是 data，不是 response 对象！
    profile.value = await getUserProfile();
  } catch (error) {
    // 错误会自动被拦截器处理，这里通常不需要再 catch
    // 除非你想做特殊处理
  }
});

// 更新示例
const saveProfile = async () => {
  await updateUserProfile({ name: 'New Name' });
};
</script>
```

---

## 四、关键知识点总结（便于快速回忆）

| 概念 | 说明 | 记忆锚点 |
|------|------|--------|
| `axios.create(config)` | 创建一个新实例，配置会合并到后续请求 | “定制一把专属扳手” |
| 实例继承 | 实例拥有自己的 `defaults` 和 `interceptors`，互不影响 | 多个服务 → 多个实例 |
| 拦截器作用域 | 在哪个实例上加拦截器，就只影响该实例的请求 | `userApi` 的拦截器不影响 `orderApi` |
| 配置合并规则 | 请求时传的 config 会和实例 config 合并（请求 config 优先级更高） | 局部覆盖全局 |
| 直接调用实例 | `instance({ method, url })` 等价于 `axios(config)` | 用于重试逻辑（如 token 刷新后重发） |

---

## 五、常见误区提醒

❌ **误区 1**：以为 `axios.create()` 返回的是函数  
✅ 正确：它返回的是一个 **对象**，这个对象本身也可调用（`instance(config)`），同时有 `.get()`, `.post()` 等方法。

❌ **误区 2**：在多个实例上重复写相同的拦截器  
✅ 正确：用“基础实例 + 继承”模式（如 `userApi = baseInstance`），避免重复。

❌ **误区 3**：把所有 API 都塞进一个实例  
✅ 正确：按业务域拆分实例（用户、订单、支付），更清晰、更易维护。

---

## 六、一句话回顾设计哲学

> **Axios 实例的本质是“配置+行为”的封装单元**，让你能把 HTTP 客户端当作“可配置的服务”来使用，而不是每次请求都从零开始拼装。

---

现在，即使你半年后打开这份笔记，也能：
1. 看懂项目结构 → 知道文件放哪  
2. 看懂代码注释 → 知道每行做什么  
3. 理解设计动机 → 知道为什么这么写  

这就是长期可维护的知识沉淀！如果你以后用 React 或其他框架，结构稍作调整即可复用这套思路。