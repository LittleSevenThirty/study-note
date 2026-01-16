# **实例方法attachShadow**
平时我们写 `HTML/CSS`，样式是全局的。比如写了 `p { color: red; }`，页面上所有`<p>`标签都会变红。如果团队协作，写的样式可能不小心改了别人的，别人的也可能影响你的，特别乱。
而 `Shadow DOM` 就是解决这个问题的 —— 给每个组件单独建个 “小黑屋”，屋里的东西`（DOM 结构、CSS 样式）`和外面完全隔开。

## **概述**
**`Element.attachShadow`** 方法给指定的元素挂载一个 `Shadow DOM`，并且返回对 `ShadowRoot` 的引用。

**可以被挂载的shadow Dom元素**
要注意的是，不是每一种类型的元素都可以附加到 shadow root（影子根）下面。出于安全考虑，一些元素不能使用 shadow DOM（例如`<a>`），以及许多其他的元素。下面是一个可以挂载 shadow root 的元素列表：

* 任何带有有效的名称且可独立存在的（autonomous）自定义元素
* `<article>`
* `<aside>`
* `<blockquote>`
* `<body>`
* `<div>`
* `<footer>`
* `<h1>`
* `<h2>`
* `<h3>`
* `<h4>`
* `<h5>`
* `<h6>`
* `<header>`
* `<main>`
* `<nav>`
* `<p>`
* `<section>`
* `<span>`

## **语法**
```js
let shadow= attachShadow(options)
```
**参数**
`options`
一个包括下列字段的对象：
>`mode`
> 指定 Shadow DOM 树封装模式的字符串，可以是以下值：
>&nbsp;&nbsp;&nbsp;&nbsp; * `open` shadow root 元素可以从 js 外部访问根节点，例如使用 Element.shadowRoot:
>```js
>element.attachShadow({ mode: "open" });
>element.shadowRoot; // 返回一个 ShadowRoot 对象
>```
>&nbsp;&nbsp;&nbsp;&nbsp; * `closed` 拒绝从 js 外部访问关闭的 shadow root 节点
>```js
>element.attachShadow({ mode: "closed" });
>element.shadowRoot; // 返回 null
>```

**返回值**
返回一个 ShadowRoot 对象或者 null

## **示例**
下面的例子取至 word-count-web-component 片段 ( 现场看看 ). 你可以看到使用 attachShadow() 在代码中间创建一个 shadow root，然后我们可以将自定义元素的内容挂载添加到它上面。
```js
// 为新元素创建一个类
class WordCount extends HTMLParagraphElement {
  constructor() {
    // 在构造器中先调用一下 super
    super();

    // 计数器指向元素的父级
    var wcParent = this.parentNode;

    function countWords(node) {
      var text = node.innerText || node.textContent;
      return text.trim().split(/\s+/g).length;
    }

    var count = "Words: " + countWords(wcParent);

    // 创建一个 shadow root
    var shadow = this.attachShadow({ mode: "open" });

    // 创建文本节点并向其添加计数器
    var text = document.createElement("span");
    text.textContent = count;

    // 将其添加到 shadow root 上
    shadow.appendChild(text);

    // 当元素内容发生变化时更新计数
    setInterval(function () {
      var count = "Words: " + countWords(wcParent);
      text.textContent = count;
    }, 200);
  }
}

// 定义新元素
customElements.define("word-count", WordCount, { extends: "p" });
```