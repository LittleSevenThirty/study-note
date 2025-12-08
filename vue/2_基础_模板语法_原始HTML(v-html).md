# **优先阅读这篇--关于v-html使用的前置**
双大括号会将数据解释为纯文本，而不是 HTML。若想插入 HTML，你需要使用[v-html](./内置_指令_v-html.md)指令：
```html
<p>Using text interpolation: {{ rawHtml }}</p>
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```
**效果展示**
>Using text interpolation: \<span style="color: red">This should be red.\</span>
>Using v-html directive: <span style="color:red">This should be red.</span>

这里我们做的事情简单来说就是：在当前组件实例上，将此元素的 `innerHTML` 与 `rawHtml` 属性保持同步。

`span` 的内容将会被替换为 `rawHtml` 属性的值，插值为纯 HTML——数据绑定将会被忽略。注意，你不能使用 `v-html` 来拼接组合模板，因为 `Vue` 不是一个基于字符串的模板引擎。在使用 `Vue` 时，应当使用组件作为 UI 重用和组合的基本单元