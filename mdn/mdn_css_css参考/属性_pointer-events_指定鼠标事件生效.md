# **css属性pointer-events**
**`pointer-events`**，css属性指定在什么情况下某个特定元素可以成为鼠标事件的target

## **语法**
`pointer-events`属性被指定为从下面的值列表中选择的一个关键字。
**值**
* `auto`
与pointer-events属性未指定时的表现效果相同，对于 SVG 内容，该值与``visible`Painted`效果相同

* `none`
元素永远不会成为鼠标事件的target。但是，当其后代元素的pointer-events属性指定其他值时，鼠标事件可以指向后代元素，在这种情况下，鼠标事件将在捕获或冒泡阶段触发父元素的事件侦听器。

* `visiblePainted`
只适用于 SVG。元素只有在以下情况才会成为鼠标事件的目标：

  * `visibility`属性值为`visible`，且鼠标指针在元素内部，且`fill`属性指定了`none`之外的值
  * `visibility`属性值为`visible`，鼠标指针在元素边界上，且`stroke`属性指定了`none`之外的值
`visibility`
只适用于 SVG。只有在元素`visibility`属性值为`visible`，且鼠标指针在元素内部时，元素才会成为鼠标事件的目标，`fill`属性的值不影响事件处理。

* `visibleStroke`
只适用于 SVG。只有在元素`visibility`属性值为`visible`，且鼠标指针在元素边界时，元素才会成为鼠标事件的目标，`stroke`属性的值不影响事件处理。

* `visible`
只适用于 SVG。只有在元素`visibility`属性值为`visible`，且鼠标指针在元素内部或边界时，元素才会成为鼠标事件的目标，`fill`和`stroke`属性的值不影响事件处理。

* `painted`
只适用于 SVG。元素只有在以下情况才会成为鼠标事件的目标：

鼠标指针在元素内部，且`fill`属性指定了`none`之外的值
鼠标指针在元素边界上，且`stroke`属性指定了`none`之外的值
`visibility`属性的值不影响事件处理。

* `fill`
只适用于 SVG。只有鼠标指针在元素内部时，元素才会成为鼠标事件的目标，`fill`和`visibility`属性的值不影响事件处理。

* `stroke`
只适用于 SVG。只有鼠标指针在元素边界上时，元素才会成为鼠标事件的目标，`stroke`和`visibility`属性的值不影响事件处理。

* `all`
只适用于 SVG。只有鼠标指针在元素内部或边界时，元素才会成为鼠标事件的目标，`fill`、`stroke`和`visibility`属性的值不影响事件处理。