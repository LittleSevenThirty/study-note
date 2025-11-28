---

# Sass `@mixin` 与 `@include` 笔记

> ✅ 适用人群：未系统学过 Sass，但已了解 CSS 基础  
> 📌 目标：掌握 `@mixin` 是什么、怎么写、怎么用、常见场景

---

## 一、什么是 `@mixin`？

- **`@mixin`** 是 Sass 提供的一种**代码复用机制**。
- 它类似于编程语言中的“函数”或“宏”，可以定义一段可重复使用的样式代码块。
- 使用 `@include` 来调用（即“插入”）这个 mixin。

> 💡 类比：CSS 中没有“函数”，但你可以把 `@mixin` 想象成一个“样式模板”。

---

## 二、基础语法

### 1. 定义一个 mixin

```scss
@mixin 名称 {
  // 样式规则
}
```

✅ 示例：

```scss
@mixin center-flex {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

### 2. 使用 mixin（调用）

```scss
@include 名称;
```

✅ 示例：

```scss
.container {
  @include center-flex;
  width: 100%;
}
```

编译后生成的 CSS：

```css
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
}
```

---

## 三、带参数的 mixin（更强大！）

### 1. 定义带参数的 mixin

```scss
@mixin 名称($参数1, $参数2: 默认值) {
  // 使用 $参数1, $参数2...
}
```

> 🔸 参数可以有默认值（类似函数默认参数）

✅ 示例：带圆角和背景色的按钮样式

```scss
@mixin btn-style($bg-color, $radius: 4px) {
  background-color: $bg-color;
  border-radius: $radius;
  padding: 8px 16px;
  border: none;
}
```

### 2. 调用带参 mixin

```scss
.primary-btn {
  @include btn-style(#007bff);
}

.round-btn {
  @include btn-style(#28a745, 20px);
}
```

编译后：

```css
.primary-btn {
  background-color: #007bff;
  border-radius: 4px;
  padding: 8px 16px;
  border: none;
}

.round-btn {
  background-color: #28a745;
  border-radius: 20px;
  padding: 8px 16px;
  border: none;
}
```

---

## 四、高级用法（进阶但实用）

### 1. 可变参数（`...`）

当参数数量不确定时，可用 `...` 接收多个值（称为“参数列表” arglist）。

```scss
@mixin box-shadow($shadows...) {
  -webkit-box-shadow: $shadows;
  box-shadow: $shadows;
}

.card {
  @include box-shadow(0 2px 4px rgba(0,0,0,0.1), inset 0 1px 0 white);
}
```

### 2. 内容块 mixin（`@content`）

允许在调用 mixin 时传入一段自定义样式（类似“插槽”）。

```scss
@mixin media-mobile {
  @media (max-width: 768px) {
    @content; // 这里插入用户传入的内容
  }
}

.sidebar {
  width: 300px;
  @include media-mobile {
    width: 100%;
  }
}
```

编译后：

```css
.sidebar {
  width: 300px;
}
@media (max-width: 768px) {
  .sidebar {
    width: 100%;
  }
}
```

> ✅ 非常适合做响应式、主题切换等场景！

---

## 五、最佳实践 & 注意事项

| 项目 | 建议 |
|------|------|
| 命名 | 使用语义化名称，如 `@mixin responsive-text` |
| 复用性 | 把常用样式（如 flex 居中、清除浮动、按钮状态）封装成 mixin |
| 不要滥用 | 简单样式直接写 CSS；只有**重复+可配置**的才用 mixin |
| 与 `@function` 区分 | `@function` 返回值（用于计算），`@mixin` 输出样式块 |
| 与 CSS 自定义属性对比 | CSS 变量适合运行时动态修改，mixin 适合编译时生成静态 CSS |

---

## 六、常见使用场景举例

| 场景 | mixin 示例 |
|------|-----------|
| Flex 居中 | `@mixin center { display: flex; justify-content: center; align-items: center; }` |
| 清除浮动 | `@mixin clearfix { ... }` |
| 响应式断点 | `@mixin respond-to($breakpoint) { @media ... }` |
| 文字截断 | `@mixin ellipsis { white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }` |
| 高亮动画 | 带参数的颜色/持续时间 mixin |

---

## 七、快速复习卡片（可截图保存）

```
@mixin 名称($参数: 默认值) {
  样式...
  @content; // 如果需要插入外部内容
}

.selector {
  @include 名称(值);
  @include 名称 { 额外样式 }; // 仅当 mixin 含 @content 时有效
}
```

---

## 八、参考资料（官方）

- Sass 官方文档 mixin 章节：https://sass-lang.com/documentation/at-rules/mixin
- `@content` 说明：https://sass-lang.com/documentation/at-rules/mixin#passing-content-blocks-to-a-mixin