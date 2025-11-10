# Axios的拦截器
此拦截器，可设置在发送请求前进行处理；和在收到响应后、业务代码前做处理

## 笔记
### 一、项目结构（清晰定位）

```
src/
├── utils/
│   └── request.js        ← 【核心】封装带拦截器的 Axios 实例
└── main.js               ← 应用入口（可选）
```

---

### 二、完整代码示例（带注释）

#### 文件：`src/utils/request.js`

```js
// src/utils/request.js
import axios from 'axios';

// 1. 创建实例（可选，也可直接用 axios）
const api = axios.create({
  baseURL: 'https://api.example.com',
  timeout: 10000
});

// --------------------------------------------------
// 🚦 请求拦截器：在请求发出前做处理
// --------------------------------------------------
api.interceptors.request.use(
  // ✅ 成功回调：可以修改 config
  (config) => {
    // 场景1：自动添加 token
    const token = localStorage.getItem('token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    // 场景2：统一日志（开发时用）
    console.log('【发送请求】', config.method?.toUpperCase(), config.url);

    // ⚠️ 必须 return config！否则请求不会发出
    return config;
  },

  // ❌ 错误回调：请求配置出错（如网络不通前的校验失败）
  (error) => {
    console.error('【请求配置错误】', error);
    return Promise.reject(error); // 抛出错误，让调用者 catch
  }
);

// --------------------------------------------------
// 📥 响应拦截器：在收到响应后、业务代码前做处理
// --------------------------------------------------
api.interceptors.response.use(
  // ✅ 成功回调：状态码 2xx
  (response) => {
    // 场景1：直接返回 data，省去 .data 冗余
    return response.data; // 👈 关键！以后调用直接拿到业务数据
  },

  // ❌ 错误回调：状态码非 2xx（如 401, 500）
  (error) => {
    // 场景1：统一错误提示
    const message = error.response?.data?.message || '网络请求失败';
    console.error('【接口错误】', message);

    // 场景2：token 过期，跳转登录
    if (error.response?.status === 401) {
      localStorage.removeItem('token');
      window.location.href = '/login';
    }

    // ⚠️ 必须 reject，否则调用者无法 catch 错误
    return Promise.reject(error);
  }
);

export default api;
```

#### 使用方式（任意组件中）：

```js
// 调用时直接拿到 data，无需 .data
const userData = await api.get('/user');

// 错误会自动被拦截器处理，也可手动 catch
try {
  await api.post('/login', credentials);
} catch (err) {
  // 只有你需要特殊处理时才写 catch
}
```

---

### 三、设计理念（为什么这么设计？）

#### 💡 核心动机：**横切关注点分离（Cross-Cutting Concerns）**

- 日志、认证、错误处理、数据格式化……这些逻辑**和具体业务无关**，但每个请求都需要。
- 如果每个 API 调用都手动加 token、处理错误，代码会重复且难以维护。

#### 🎯 设计哲学：
> **“把通用逻辑抽离到管道中，业务只关心核心数据流。”**

- **请求拦截器** = 请求发出前的“安检门”（加 token、日志、参数校验）
- **响应拦截器** = 响应回来后的“分拣站”（提取 data、统一报错、重试）

#### ⚙️ 关键机制：
- 拦截器是**链式调用**，支持多个（后加的先执行？不，先加的先执行！）
- 必须 `return config` 或 `return response`，否则流程中断
- 错误必须 `return Promise.reject(error)`，否则调用方无法感知失败

---

### 四、速记口诀（30 秒回忆）

> - **请求拦截器**：发请求前 → 加 token / 打日志 → **return config**  
> - **响应拦截器**：收响应后 → 提 data / 统一报错 → **return response.data**  
> - **错误都要 reject**，否则 catch 不到！  
> - 拦截器属于**实例**，不同实例互不影响

---

### 五、官方
>在请求或响应被 then 或 catch 处理前拦截它们。
>
>```js
>// 添加请求拦截器
>axios.interceptors.request.use(function (config) {
>    // 在发送请求之前做些什么
>    return config;
>  }, function (error) {
>    // 对请求错误做些什么
>    return Promise.reject(error);
>  });
>
>// 添加响应拦截器
>axios.interceptors.response.use(function (response) {
>    // 2xx 范围内的状态码都会触发该函数。
>    // 对响应数据做点什么
>    return response;
>  }, function (error) {
>    // 超出 2xx 范围的状态码都会触发该函数。
>    // 对响应错误做点什么
>    return Promise.reject(error);
>  });
>```
>
>
>如果你稍后需要移除拦截器，可以这样：
>```js
>const myInterceptor = axios.interceptors.request.use(function () {/*...*/});
>axios.interceptors.request.eject(myInterceptor);
>```
>可以给自定义的 axios 实例添加拦截器。
>```js
>const instance = axios.create();
>instance.interceptors.request.use(function () {/*...*/});
>```