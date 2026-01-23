# **画布元素<canvas>**

`<canvas>` 元素可被用来通过 JavaScript绘制图形及图形动画

## **属性**

本元素支持全局属性。
`height`
该元素占用空间的高度，以 CSS 像素（px）表示，默认为 150。
`width`
该元素占用空间的宽度，以 CSS 像素（px）表示，默认为 300。

## **注意事项**

### **标签需要闭合**

不同于 \<img> 元素， \<canvas>元素需要有闭合标签 (\</canvas>).

### **设置画布 (canvas) 的大小**

直接在 html 标签中设置 width 和 height 属性或者使用 JavaScript 来指定画布尺寸，这将改变一个画布的水平像素和垂直像素数，就像定义了一张图片的大小一样。

可以使用 CSS 的 width 和 height 以在渲染期间缩放图像以适应样式大小，就像\<img>元素一样。如果你发现\<canvas>元素中展示的内容变形，你可以通过\<canvas>自带的 height 和 width 属性进行相关设置，而不要使用 CSS。

## **示例**

### **HTML**

```html
<canvas id="canvas" width="300" height="300">
  抱歉，你的浏览器不支持 canvas 元素
  （这些内容将会在不支持&lt;canvas%gt;元素的浏览器或是禁用了 JavaScript
  的浏览器内渲染并展现）
</canvas>
```

### **JavaScript**

使用`HTMLCanvasElement.getContext()`获得一个绘图上下文并开始绘制

```js
var canvas = document.getElementById("canvas");
var ctx = canvas.getContext("2d");
ctx.fillStyle = "green";
ctx.fillRect(10, 10, 100, 100);
```
