# **Axios介绍**
> 本篇截取自官方文档🙂

Axios 是一个基于 promise 网络请求库，作用于node.js 和浏览器中。 它是 isomorphic 的(即同一套代码可以运行在浏览器和node.js中)。在服务端它使用原生 node.js http 模块, 而在客户端 (浏览端) 则使用 XMLHttpRequest。

## **安装**
使用 npm:
```cmd
$ npm install axios
```

## **特性**
从浏览器创建 XMLHttpRequest
从 node.js 创建 http 请求
支持 Promise API
拦截请求和响应
转换请求和响应数据
取消请求
超时处理
查询参数序列化支持嵌套项处理
自动将请求体序列化为：
JSON (application/json)
Multipart / FormData (multipart/form-data)
URL encoded form (application/x-www-form-urlencoded)
将 HTML Form 转换成 JSON 进行请求
自动转换JSON数据
获取浏览器和 node.js 的请求进度，并提供额外的信息（速度、剩余时间）
为 node.js 设置带宽限制
兼容符合规范的 FormData 和 Blob（包括 node.js）
客户端支持防御XSRF

## **其它安装**
使用 bower:
```cmd
$ bower install axios
```
使用 yarn:
```cmd
$ yarn add axios
```
使用 pnpm:
```cmd
$ pnpm add axios
```
使用 jsDelivr CDN:
```js
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
```
使用 unpkg CDN:
```js
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
```
为了直接使用 require 导入预构建的 CommonJS 模块（如果您的模块打包器无法自动解析它们），我们提供了以下预构建模块：
```js
const axios = require('axios/dist/browser/axios.cjs'); // browser
const axios = require('axios/dist/node/axios.cjs'); // node
```