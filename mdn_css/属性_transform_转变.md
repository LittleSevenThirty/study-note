# **transform转变**
## **🌀transform：元素的"变形"魔法**

### **什么是transform？**

transform是CSS中的一个属性，用来对元素进行**2D或3D变换**，比如旋转、缩放、移动和倾斜等。

MDN文档描述：

> "transform 属性用于对元素进行2D或3D转换。"

### **为什么用transform做动画？**

MDN文档特别强调：

> "transform 属性对性能影响小，不会引起页面重排。"

这意味着使用transform进行动画（如旋转、缩放）时，浏览器不需要重新计算页面布局，动画会非常流畅。

### **常见的transform函数：**

| 函数 | 作用 | 例子 |
|------|------|------|
| `translate` | 移动元素 | `transform: translate(50px, 100px);` |
| `rotate` | 旋转元素 | `transform: rotate(45deg);` |
| `scale` | 缩放元素 | `transform: scale(1.5);` |
| `skew` | 倾斜元素 | `transform: skew(20deg, 10deg);` |

**MDN重要提示**：transform函数可以组合使用，例如：
```css
transform: translate(50px, 100px) rotate(45deg) scale(1.5);
```

### **实际例子：**

```css
/* 让元素从左边滑动到右边，同时旋转 */
@keyframes 滑动旋转 {
  0% { transform: translateX(0) rotate(0deg); }
  100% { transform: translateX(200px) rotate(360deg); }
}

.box {
  width: 50px;
  height: 50px;
  background-color: #3498db;
  animation: 滑动旋转 3s linear infinite;
}
```