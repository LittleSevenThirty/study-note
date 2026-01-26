# **Element.getBoundingClientRect方法获取相对于视口的位置属性**
**`Element.getBoundingClientRect()`** 方法返回一个 DOMRect 对象，其提供了元素的大小及其相对于视口的位置属性。

## **语法**
```js
getBoundingClientRect()
```
**返回值**
返回值是一个` DOMRect `对象，是包含整个元素的最小矩形（包括 `padding` 和 `border-width`）。该对象使用 `left、top、right、bottom、x、y、width` 和 `height` 这几个以像素为单位的只读属性描述整个矩形的位置和大小。除了 `width` 和 `height` 以外的属性是相对于视图窗口的左上角来计算的。

该方法返回的 `DOMRect` 对象中的 `width` 和 `height` 属性是包含了 `padding` 和 `border-width` 的，而不仅仅是内容部分的宽度和高度。在标准盒子模型中，这两个属性值分别与元素的 `width/height + padding + border-width` 相等。而如果是 `box-sizing: border-box`，两个属性则直接与元素的 `width` 或 `height` 相等。

这个对象是由该元素的 `getClientRects()` 方法返回的一组矩形的集合，就是该元素的 CSS 边框大小。

空边框盒（译者注：没有内容的边框）会被忽略。如果所有的元素边框都是空边框，那么这个矩形给该元素返回的 `width、height` 值为 `0，left、top` 值为第一个 CSS 盒子（按内容顺序）的 `top-left` 值。

如果你需要获得边界矩形相对于整个网页左上角的位置，则可以将当前的滚动位置（可通过 `window.scrollX` 和 `window.scrollY` 获得）添加到 `top` 和 `left` 属性上。获得的边界矩形与当前的滚动位置无关。