# CSS 属性参考：`background-attachment`

**简介**：
`background-attachment` 是一个决定背景图像位置的 CSS 属性。它控制背景图像是在视口（Viewport）内固定，还是随着包含它的区块滚动。这一属性在实现视差滚动等视觉效果时非常关键。

---

### 1. 语法

* **【原文要点】**：
  该属性的语法定义了背景图像如何随页面的其余部分滚动。它接受三个关键字值：`scroll`、`fixed` 和 `local`。
* **【深度拆解】**：
  * **`scroll`**：这是默认值。背景图像相对于元素本身固定。如果元素有滚动条，背景不会随内容滚动，而是固定在元素的边框上。
  * **`fixed`**：背景图像相对于视口（浏览器窗口）固定。即使元素本身有滚动机制，背景也不会随着元素的内容滚动，而是随着整个页面滚动。
  * **`local`**：背景图像相对于元素的内容固定。如果元素有滚动条，背景图像会随着内容一起滚动，并且背景的绘制区域和定位区域是相对于可滚动的内容区域，而不是边框。
* **【标准语法】**：
  ```css
  background-attachment = <attachment>#
  <attachment> = scroll | fixed | local
  ```
  * **解释**：`<attachment>#` 表示可以接受一个或多个 `<attachment>` 值，用逗号分隔。`<attachment>` 本身只能是 `scroll`、`fixed` 或 `local` 中的一个。

---

### 2. 示例

#### 2.1 简单的例子

* **【实例教学】**：
  以下是一个简单的例子，展示了 `fixed` 和 `scroll` 的区别。

  **CSS 样式表**：
  ```css
  body {
    background-image: url("images/star.png");
    background-attachment: fixed;
  }
  ```
  * **代码解释**：
    * `body`：选择 HTML 文档的主体元素。
    * `background-image: url("images/star.png");`：设置背景图像为 `images` 文件夹下的 `star.png` 文件。
    * `background-attachment: fixed;`：设置背景图像相对于视口固定。这意味着，当你滚动页面时，背景图像会保持在视口的同一位置，不会随页面内容移动。

  **效果**：
  页面背景图像固定在视口中，即使页面内容滚动，背景图像也保持不动。

#### 2.2 多背景图支持

* **【实例教学】**：
  `background-attachment` 属性支持多张背景图像。你可以用逗号分隔来为每一张背景图片指定不同的 `<attachment>` 属性值。每一张背景图片的顺序对应相应的 attachment 属性。

  **CSS 样式表**：
  ```css
  body {
    background-image: url("images/star.png"), url("images/stripes.png");
    background-attachment: fixed, scroll;
  }
  ```
  * **代码解释**：
    * `background-image: url("images/star.png"), url("images/stripes.png");`：设置两张背景图像，第一张是 `star.png`，第二张是 `stripes.png`。
    * `background-attachment: fixed, scroll;`：为第一张图像 `star.png` 指定 `fixed`，使其相对于视口固定；为第二张图像 `stripes.png` 指定 `scroll`，使其相对于元素本身固定。

  **效果**：
  页面背景中，星星图像固定在视口中，而条纹图像随页面内容滚动。

---

### 3. 规范

* **【原文要点】**：
  该属性的规范定义了其初始值、适用元素、继承性、计算值和动画类型。
* **【深度拆解】**：
  * **Specification**：CSS Backgrounds and Borders Module Level 3。
  * **初始值**：`scroll`。这意味着，如果不指定 `background-attachment`，背景图像将默认相对于元素本身固定。
  * **适用元素**：所有元素。它也适用于 `::first-letter` 和 `::first-line` 伪元素。
  * **是否是继承属性**：否。子元素不会继承父元素的 `background-attachment` 属性。
  * **计算值**：as specified。即计算值与指定值相同。
  * **动画类型**：离散值。这意味着 `background-attachment` 属性不能进行平滑的动画过渡，只能在不同值之间切换。

---

### 4. 浏览器兼容性

* **【原文要点】**：
  该属性是广泛可用的（Widely available），自 2015 年 7 月起在所有主流浏览器中得到支持。
* **【深度拆解】**：
  * **Baseline**：该功能已经成熟，可以在大多数设备和浏览器版本中正常工作。
  * **注意**：虽然该属性本身广泛支持，但某些部分（如多背景图的复杂组合）可能在不同浏览器中有不同的支持级别。建议在使用时查阅具体的浏览器兼容性表。

---

### 5. 参见

* **【原文要点】**：
  更多关于背景图的属性和用法。
* **【深度拆解】**：
  * **相关属性**：
    * `background-image`：设置背景图像。
    * `background-position`：设置背景图像的位置。
    * `background-repeat`：设置背景图像的重复方式。
    * `background-size`：设置背景图像的大小。
  * **建议**：了解更多关于 CSS 背景和边框模块的内容，以充分利用这些属性。