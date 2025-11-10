# vite .env环境文件配置
## 🧠 一、核心概念速记

### ✅ 1. `import.meta.env` 是什么？
Vite 在客户端代码中通过 `import.meta.env` 对象提供环境变量，会自动关联将.env...相关文件。  
#### ⚠️ 注意：**只有以 `VITE_` 开头的变量才会暴露给前端代码**（出于安全考虑）。

### ✅ 2. 内建变量（始终可用）
```js
import.meta.env.MODE     // 当前运行模式（如 'development', 'production', 'staging'）
import.meta.env.BASE_URL // 应用部署的基础路径（由 vite.config.js 中的 base 决定）
import.meta.env.DEV      // 是否为开发环境（布尔值）
import.meta.env.PROD     // 是否为生产环境（布尔值，与 DEV 相反）
import.meta.env.SSR      // 是否在服务端渲染（SSR）
```

### ✅ 3. 环境变量加载规则（优先级从高到低）
1. **命令行传入的变量**（最高优先级，不会被 .env 覆盖）  
   ```bash
   VITE_API_URL=https://prod.api.com vite build
   ```
2. `.env.[mode].local` → `.env.[mode]`
3. `.env.local`
4. `.env`（最低优先级）

> 💡 例如：运行 `vite build --mode staging` 时，会加载：
> - `.env`
> - `.env.local`
> - `.env.staging`
> - `.env.staging.local`

---

## 📁 二、完整项目结构示例（便于定位）

```
my-vite-app/
├── .env                    # 所有模式通用
├── .env.local              # 本地覆盖（不提交 Git）
├── .env.development        # 仅开发模式
├── .env.production         # 仅生产模式
├── .env.staging            # 自定义 staging 模式
├── src/
│   ├── env.d.ts            # TypeScript 类型声明（可选但推荐）
│   └── main.ts             # 使用环境变量的地方
├── vite.config.ts          # Vite 配置（可指定 base 等）
└── package.json
```

---

## 📄 三、各文件详解 + 注释版代码

---

### 📄 `.env`（所有模式都加载）
```env
# 通用配置，所有模式都会读取
VITE_APP_NAME=My Awesome App
DB_PASSWORD=secret123  # ❌ 不会暴露给前端！因为没有 VITE_ 前缀
```

> 🔒 **设计理念**：  
> 为什么只暴露 `VITE_` 开头的变量？  
> **安全第一**！前端代码最终会被打包到浏览器，任何环境变量都会“泄露”给用户。  
> 所以 Vite 强制要求显式标记哪些变量可以暴露（`VITE_`），防止开发者不小心把数据库密码、API 密钥等敏感信息打包进前端。
---
### 📄 `.env.staging`
```env
# 自定义模式：预发布环境
NODE_ENV=production        # 告诉框架行为像生产环境（如 React 会启用优化）
VITE_API_URL=https://staging-api.myapp.com
VITE_APP_TITLE=My App (Staging)
```

---

### 📄 `.env.development`
```env
# 仅在开发模式（vite dev）时生效
VITE_API_URL=http://localhost:3000/api
```

---

### 📄 `.env.production`
```env
# 仅在生产构建（vite build）时生效
VITE_API_URL=https://api.myapp.com
```

---

> 💡 **为什么需要自定义 mode（如 staging）？**  
> 生产（production）和开发（development）是两个极端，但实际项目常有中间环境：测试、预发布、灰度等。  
> Vite 允许你通过 `--mode staging` 启动一个“类生产”环境，但使用不同的 API 地址或标题，方便测试上线前的效果。  
> 这体现了 **“配置即代码” + “环境隔离”** 的工程思想。

---

### 📄 `src/main.ts`（前端使用环境变量）
```ts
// src/main.ts

// ✅ 正确：静态访问 VITE_ 开头的变量
console.log('App Name:', import.meta.env.VITE_APP_NAME);
console.log('API URL:', import.meta.env.VITE_API_URL);
console.log('Mode:', import.meta.env.MODE); // e.g., 'development'

// ❌ 错误：动态 key 无法被 Vite 静态分析，结果是 undefined
const key = 'VITE_API_URL';
console.log(import.meta.env[key]); // undefined！

// ❌ 危险：尝试访问非 VITE_ 变量（如 DB_PASSWORD）→ undefined
console.log(import.meta.env.DB_PASSWORD); // undefined（安全保护生效）
```

> ⚠️ **为什么不能动态访问？**  
> Vite 在构建时会**静态替换** `import.meta.env.XXX` 为实际字符串（如 `"https://api.com"`）。  
> 它无法在编译时知道 `key` 的值是什么，所以跳过替换 → 运行时就是 `undefined`。  
> **这是为了性能和安全性做的权衡**：静态替换更快、更安全，但牺牲了动态性。

---

### 📄 `src/env.d.ts`（TypeScript 类型增强）
```ts
// src/env.d.ts

/// <reference types="vite/client" />

// 扩展 ImportMetaEnv 接口，让 TS 知道你有哪些自定义 VITE_ 变量
interface ImportMetaEnv {
  readonly VITE_APP_NAME: string;
  readonly VITE_API_URL: string;
  readonly VITE_APP_TITLE?: string; // staging 特有，可选
}

// 确保 import.meta.env 符合上述定义
interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

> ✅ **好处**：  
> - 代码自动补全  
> - 类型检查（写错变量名会报错）  
> - 团队协作更清晰

---

### 📄 `package.json`（脚本示例）
```json
{
  "scripts": {
    "dev": "vite",                          // 默认 mode: development
    "build": "vite build",                  // 默认 mode: production
    "build:staging": "vite build --mode staging",
    "preview": "vite preview"
  }
}
```

运行：
```bash
npm run build:staging  # 会加载 .env.staging，构建出预发布版本
```

---

## 🧩 四、常见陷阱 & 解决方案

### ❌ 问题：在字符串中写了 `import.meta.env.MODE`，结果报语法错误
```js
// 错误示例（在字符串里）
const msg = "Current mode is import.meta.env.MODE"; // 构建时会被替换成 "Current mode is "development""
// → 变成非法 JS 字符串！
```

### ✅ 解决方案：用零宽空格或 `<wbr>` 打断识别
```js
// JavaScript 中：
const msg = 'Current mode is import.meta\u200b.env.MODE';

// Vue 模板中：
<template>
  <div>Mode: import.meta.<wbr>env.MODE</div>
</template>
```

> 💡 **设计理念**：  
> Vite 的替换是“文本级”的，它会在所有 JS/Vue 文件中搜索 `import.meta.env.XXX` 并替换。  
> 如果你不小心把它写在字符串里，也会被替换，导致语法错误。  
> 所以提供“打断”技巧，本质是**让 Vite 看不到完整的关键词**。

---

## ✅ 五、总结记忆口诀

> 🔑 **VITE_ 开头才可见，  
> 静态访问是关键，  
> mode 决定加载谁，  
> staging 让上线更安全。**

---

希望这份重构后的笔记能让你**半年后回看也能秒懂**！如果还有哪部分觉得模糊，可以告诉我，我再针对性补充 😊