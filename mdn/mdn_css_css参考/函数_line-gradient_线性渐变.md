# 🎨 CSS 线性渐变 `linear-gradient()` 完全入门指南（新手友好版）

## 一、什么是线性渐变？

`linear-gradient()` 是 CSS 中用来创建**颜色平滑过渡背景**的一个函数。它属于 `<gradient>` 类型，而 `<gradient>` 本身是一种特殊的 **图像（`<image>`）**。

> ✅ 简单理解：  
> 它不是一张图片文件，而是用代码“画”出一条从一种颜色慢慢变成另一种（或多种）颜色的背景。

比如：

- 从上到下：蓝色 → 白色
- 从左到右：红色 → 黄色 → 绿色
- 对角线：紫红 → 深蓝

这些都可以用 `linear-gradient()` 实现！

---

## 二、基本语法

最简单的写法：

```css
background: linear-gradient(red, blue);
```

这会创建一个**从上到下**（默认方向）的渐变：顶部是红色，底部是蓝色。

### 完整语法结构：

```css
linear-gradient(
  [ <角度> | to <方向> ]?,   /* 可选：控制渐变方向 */
  <颜色1> [<位置1>]?,
  <颜色2> [<位置2>]?,
  ...
)
```

> ⚠️ 注意：**第一个参数是方向（可选），后面全是“颜色 + 位置”组合**。

---

## 三、控制渐变方向

你可以通过两种方式指定颜色“流动”的方向：

### 方式 1：使用关键词（推荐给新手）

```css
/* 默认：从上到下 */
linear-gradient(blue, red);

/* 明确写出方向 */
linear-gradient(to bottom, blue, red);    /* 同上 */
linear-gradient(to top, blue, red);       /* 从下到上 */
linear-gradient(to right, blue, red);     /* 从左到右 */
linear-gradient(to left, blue, red);      /* 从右到左 */

/* 对角线方向 */
linear-gradient(to bottom right, blue, red);  /* 左上 → 右下 */
linear-gradient(to top left, green, yellow);  /* 右下 → 左上 */
```

> ✅ 小技巧：`to X` 表示“朝 X 方向去”，所以 `to right` = 颜色从左往右变。

### 方式 2：使用角度（更精确）

**注意这个指的是顺时针旋转，12点时是0deg，6点是180deg，3点是90deg，指向哪，朝哪变**

```css
linear-gradient(0deg, blue, red);     /* 0° = to top（向上）*/
linear-gradient(90deg, blue, red);    /* 90° = to right */
linear-gradient(180deg, blue, red);   /* 180° = to bottom */
linear-gradient(45deg, blue, red);    /* 45° = 右上方向 */
```

> 🔁 角度规则：
>
> - `0deg` 指向**正上方**（等价于 `to top`）
> - 角度值**顺时针增加**（90° 是右，180° 是下，270° 是左）

---

## 四、添加多个颜色（多色渐变）

你可以在一个渐变中使用**任意多个颜色**：

```css
background: linear-gradient(
  to right,
  red,
  orange,
  yellow,
  green,
  blue,
  indigo,
  violet
);
```

这会创建一道“彩虹”效果，从左到右依次过渡。

> 💡 默认情况下，颜色会**均匀分布**在整个渐变区域。

---

## 五、精确控制颜色位置（色标）

有时候你不想让颜色均匀分布，而是想在特定位置切换颜色。这时就要用到**色标（color stop）的位置值**。

### 位置单位：

- `%`（百分比）：相对于整个渐变轴长度（0% = 起点，100% = 终点）
- `px`、`em` 等长度单位（较少用，但支持）

### 示例：

```css
/* 红色从 0% 开始，绿色在 30%，蓝色在 100% */
linear-gradient(to right, red, green 30%, blue);

/* 红色占前 20%，然后瞬间变为橙色（硬边效果）*/
linear-gradient(to right, red 20%, orange 20%);
```

> ✅ 关键点：  
> 如果两个颜色**在同一位置**（如都写 `20%`），就会形成**硬边（无过渡）**，常用于制作条纹效果。

---

## 六、高级技巧：颜色提示（Color Hint）

除了指定颜色位置，你还可以控制**颜色过渡的“节奏”**。

```css
/* 默认：红→蓝 在 50% 处是中间色 */
linear-gradient(red, blue);

/* 自定义：过渡中点移到 10% 处 → 前面红得久，后面快速变蓝 */
linear-gradient(red, 10%, blue);
```

这里的 `10%` 就是**颜色提示** —— 它表示“红到蓝的中间色出现在 10% 的位置”。

> 🎯 用途：制造非线性过渡效果，比如让某种颜色“停留更久”。

---

## 七、多位置色标（简化写法）

CSS 允许一个颜色同时指定**起始和结束位置**，用于创建色块：

```css
/* 传统写法 */
linear-gradient(to right,
  red 0%, red 20%,
  orange 20%, orange 40%,
  yellow 40%, yellow 60%
);

/* 简化写法（推荐！）*/
linear-gradient(to right,
  red 0% 20%,
  orange 20% 40%，
  yellow 40% 60%
);
```

> ✅ 这种写法特别适合做**彩色条纹背景**或**进度条样式**。

---

## 八、重要注意事项

### 1. `linear-gradient()` 是图像，不是颜色

- ❌ 不能用在 `color`、`border-color` 等需要 `<color>` 的地方
- ✅ 只能用在需要 `<image>` 的地方，比如：
  - `background`
  - `background-image`
  - `list-style-image`
  - `mask-image` 等

### 2. 渐变没有“固有尺寸”

- 它会自动**拉伸填满容器**
- 如果想重复平铺，要用 [`repeating-linear-gradient()`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/Reference/Values/gradient/repeating-linear-gradient)

### 3. 色标顺序很重要

- 如果后面的色标位置**小于前面的**，浏览器会自动调整，但可能导致意外硬边。
- ✅ 建议按**位置递增**顺序书写颜色。

---

## 九、实用示例合集

### 示例 1：简单垂直渐变

```css
.hero {
  background: linear-gradient(to bottom, #6a11cb, #2575fc);
}
```

### 示例 2：水平彩虹条纹

```css
.stripes {
  background: linear-gradient(
    to right,
    red 0% 20%,
    orange 20% 40%,
    yellow 40% 60%,
    green 60% 80%,
    blue 80% 100%
  );
}
```

### 示例 3：柔和焦点光效（多层渐变）

```css
.glow {
  background:
    linear-gradient(135deg, rgba(255, 255, 255, 0.1), transparent),
    linear-gradient(to bottom, #ff7e5f, #feb47b);
}
```

---

## 十、浏览器兼容性

- ✅ **广泛支持**：所有现代浏览器（Chrome、Firefox、Safari、Edge）自 **2015 年起**已完全支持。
- 旧版 IE（≤9）不支持，但如今基本无需考虑。
- 无需加 `-webkit-` 等前缀（除非要兼容非常老的移动端浏览器）。

---

## 十一、与其他渐变函数的关系

| 函数                          | 作用                                |
| ----------------------------- | ----------------------------------- |
| `linear-gradient()`           | **线性渐变**（直线方向）✅ 本文主角 |
| `radial-gradient()`           | 径向渐变（圆形/椭圆扩散）           |
| `conic-gradient()`            | 圆锥渐变（绕圈旋转）                |
| `repeating-linear-gradient()` | **重复线性渐变**（自动平铺）        |

> 学完 `linear-gradient()`，再学其他就很容易了！

---

## 十二、小结：关键要点速记

- `linear-gradient()` 创建**直线方向的颜色过渡背景**。
- 默认方向：**从上到下**（`to bottom`）。
- 方向可用 `to right` / `45deg` 等方式指定。
- 颜色可多个，位置用 `%` 或长度单位控制。
- 同一位置放两个颜色 = **硬边（条纹）**。
- 使用 `颜色 起始% 结束%` 可快速定义色块。
- 它是**图像**，只能用于 `background` 等图像属性。
- **不要用在文字颜色或边框颜色上**。

---

## 十三、动手试试！

打开浏览器开发者工具，粘贴以下代码到某个元素的 `style` 中，实时观察效果：

```css
background: linear-gradient(45deg, #ff9a9e, #fad0c4);
```

然后尝试修改角度、颜色、位置，感受渐变的变化！

---
