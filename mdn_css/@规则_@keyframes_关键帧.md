# **@keyframes关键帧**
`@keyframes`属于`css`的`@`规则，在官方文档的解释是：
>"关键帧 `@keyframes` 通过在动画序列中定义关键帧（或 waypoints）的样式来控制 CSS 动画序列中的中间步骤。"

简单来说，关键帧就是**动画过程中重要的几个时间点**，以**一个完整动画流程**讲解，**百分比代表了动画进行的进度**，比如一个动画有2s演绎：
* 0%（动画开始时）
* 50%（动画进行到一半时）到达1s
* 64% (动画进度到64%时执行)
* 100%（动画结束时）  

## 🧩 **关键帧动画的基本结构**
CSS关键帧动画由两部分组成：
### **①定义关键帧动画**
```css
@keyframes 名称 {
  百分比(进度)% { /*动画*/ }
}
```
**MDN提示**：`0%`可以用`from`代替，`100%`可以用`to`代替，这样写更直观：
```css
@keyframes 旋转 {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}
```

### **②应用动画到元素上**
```css
.元素 {
  animation-name: 旋转;
  animation-duration: 2s; /* 动画持续2秒 */
  animation-iteration-count: infinite; /* 无限循环 */
}
```
**🧠 动画的"魔法参数"（MDN推荐属性）**
MDN文档中提到的动画属性，我来解释一下：
|属性|	作用|	MDN建议|	例子|
|---|---|---|---|
|animation-name	|指定动画的名字	|必须	|animation-name: 膨胀;
|animation-duration	|动画持续时间	|必须	|animation-duration: 2s;
|animation-iteration-count	|动画重复次数	|infinite表示无限	|animation-iteration-count: infinite;
|animation-direction	|动画播放方向	|normal/reverse/alternate	|animation-direction: alternate;
|animation-timing-function	|动画速度曲线	|ease/linear/ease-in-out	|animation-timing-function: ease-in-out;
|animation-delay	|动画延迟	|等待多久开始	|animation-delay: 1s;

**它们整体可以简写**
```css
.元素 {
  animation: 旋转 2s ease infinite;
}
```
#### **animation-timing-funciotn**
`linear`：匀速
`ease`：默认，先慢后快
`ease-in`：开始慢，结束快
`ease-out`：开始快，结束慢
`ease-in-out`：开始慢，中间快，结束慢

## **举个例子🌰**
```css

```

## **最后**
`@keyframes`看完上述，已经能够基本使用@keyframes了，当然如果想做稍微复杂点的动画，就得使用`transform`,`transition`,`opacity`等属性了