# **指令v-html**
更新元素的`innerHTML`属性，这是`mdn_javascript`的知识
* **期望绑定值的类型：**`string` 形如-> `<标签>内容</标签>`
* **详细信息**

`v-html` 的内容直接作为普通 HTML 插入—— Vue 模板语法是不会被解析的。如果你发现自己正打算用 `v-html` 来编写模板，不如重新想想怎么使用组件来代替。

在`单文件组件`，`scoped` 样式将不会作用于 `v-html` 里的内容，因为 `HTML` 内容不会被 `Vue` 的模板编译器解析。如果你想让 `v-html` 的内容也支持 `scoped CSS`，你可以使用 `CSS modules` 或使用一个额外的全局 <style> 元素，手动设置类似 BEM 的作用域策略。