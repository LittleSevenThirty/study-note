# **配置键值css.preprocessorOptions**
* **类型：** `Record<string, object>`
指定传递给 CSS 预处理器的选项。文件扩展名用作选项的**键**，例如：
```ts
export default defineConfig({
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `$injectedColor: orange;`
      },
      styl: {
        additionalData: `$injectedColor ?= orange`
      }
    }
  }
})

```
**scss是键,值是一个只包含一个键值对的对象：**
**`addtionalData` 自动注入公共代码**