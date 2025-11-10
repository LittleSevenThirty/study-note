# **Axios的使用**
## **起步**
**导入**，若已经正确导入axios组件后可以使用
```js
import axios from 'axios';
```

**使用axios的API**
可以向` axios `传递相关配置来创建请求
```js
// 方式1
aios(config)
//// 举例
// 发起一个post请求
axios({
  method: 'post',
  url: '/user/12345',
  data: {
    firstName: 'Fred',
    lastName: 'Flintstone'
  }
});

// 方式2
//// 别名方式
axios.request(config)
axios.get(url[, config])    // 发送get请求
axios.delete(url[, config])
axios.head(url[, config])
axios.options(url[, config])
axios.post(url[, data[, config]])   // 发送post请求
axios.put(url[, data[, config]])
axios.patch(url[, data[, config]])
axios.postForm(url[, data[, config]])
axios.putForm(url[, data[, config]])
axios.patchForm(url[, data[, config]])
```
* **参数url**：字符串格式，网络地址
* **config**：配置额外参数，格式
```js
axios.get("http://...",{
    params: {
        id: 1,
        name: 'lisi'
    },
    headers: {
        Authorization: 'Bearer xxx'
    }
    timeout: 5000
})
```
* **data**：访问url提供的数据
```js
axios.post('/user', {
  name: 'Bob',
  email: 'bob@example.com'
},{
    params: {
        ...
    },
    timeout: ...S
})
// Content-Type 自动设为 application/json
```

## 其它
### ⭐. `axios.request(config)`
#### ✅ 作用：**Axios 的底层统一入口**
- 所有别名方法（`.get`, `.post` 等）内部最终都调用它
- 适合做 **动态请求**（比如方法名从变量来）

#### 🌰 例子：
```js
function apiCall(method, url, data) {
  return axios.request({
    method,
    url,
    data
  });
}

// 动态调用
apiCall('post', '/user', { name: 'Bob' });
apiCall('put', '/user/1', { name: 'Charlie' });
```

### 🔹 1. `axios.patch(url, data, config?)`
#### ✅ 作用：**局部更新资源**（只改部分字段）
- 和 `PUT`（全量替换）相反
- **只传要修改的字段**，其他字段保持不变

#### 🌰 例子：
```js
// 只更新用户的邮箱，不碰用户名、年龄等其他字段
axios.patch('/user/123', {
  email: 'new@example.com'
})
```

> 💡 **什么时候用？**  
> 当你有一个表单只允许修改“头像”或“密码”，不想把整个用户对象都传过去时，就用 `PATCH`。

---

### 🔹 2. `axios.head(url, config?)`
#### ✅ 作用：**只获取响应头，不下载响应体**
- 请求成功后，**没有 `response.data`**
- 常用于检查资源是否存在、获取文件大小、最后修改时间等

#### 🌰 例子：
```js
axios.head('/report.pdf')
  .then(res => {
    console.log('文件大小:', res.headers['content-length']);
    console.log('最后修改时间:', res.headers['last-modified']);
  });
```

> 💡 **什么时候用？**  
> 想知道一个大文件有多大，但又不想真的下载它；或者检查某个 URL 是否有效（返回 200 还是 404）。

---

### 🔹 3. `axios.options(url, config?)`
#### ✅ 作用：**获取服务器支持的 HTTP 方法和 CORS 信息**
- 主要用于 **CORS 预检请求（Preflight）**
- 浏览器在跨域复杂请求前会自动发 `OPTIONS`，但有时你也需要手动发

#### 🌰 例子：
```js
axios.options('/api/data')
  .then(res => {
    console.log('允许的方法:', res.headers['access-control-allow-methods']);
    console.log('允许的头部:', res.headers['access-control-allow-headers']);
  });
```

> 💡 **什么时候用？**  
> 调试跨域问题时，手动查看 API 支持哪些方法和头部；或者实现自定义的 CORS 策略。

---

### 🔹 4. `axios.postForm(url, data, config?)` 等（v0.27+ 新增）
### ✅ 作用：**自动处理表单数据（特别是文件上传）**
- 自动设置 `Content-Type: multipart/form-data`
- 自动把 JS 对象转成 `FormData`

#### 🌰 例子：
```html
<input type="file" id="avatar">
```
```js
const file = document.getElementById('avatar').files[0];
axios.postForm('/upload', {
  username: 'alice',
  avatar: file // 文件对象
});
// 不用手动 new FormData()！
```

> 💡 **什么时候用？**  
> 上传图片、文件、或包含文件的混合表单时，比手动构造 `FormData` 简洁得多。

> ⚠️ 注意：如果你的项目 Axios 版本低于 0.27，这些方法不存在！

---

> 💡 **什么时候用？**  
> 封装通用请求函数、写拦截器、或需要根据配置动态决定请求方法时。

---

### 📊 总结：所有方法用途速查表

| 方法 | 用途 | 使用频率 | 典型场景 |
|------|------|--------|--------|
| `get` | 获取数据 | ⭐⭐⭐⭐⭐ | 加载列表、详情 |
| `post` | 创建资源 | ⭐⭐⭐⭐⭐ | 注册、提交表单 |
| `put` | 全量更新 | ⭐⭐⭐ | 替换整个对象 |
| `delete` | 删除资源 | ⭐⭐⭐⭐ | 删除用户、订单 |
| `patch` | **局部更新** | ⭐⭐⭐ | 只改邮箱、密码等 |
| `head` | **只拿响应头** | ⭐ | 检查文件大小、存在性 |
| `options` | **获取 API 能力** | ⭐ | 调试 CORS、预检 |
| `xxxForm` | **自动表单编码** | ⭐⭐ | 文件上传 |
| `request` | **底层统一调用** | ⭐⭐ | 封装通用请求 |