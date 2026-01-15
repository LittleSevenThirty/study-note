# **📝 `focus` 事件学习笔记（小白友好版）**

## **一、什么是 `focus` 事件？**

- **`focus` 事件**在**元素获得焦点**（focus）时触发。
- 它是用户与页面交互中最基础的事件之一，常与 `blur` 成对出现。

> 💡 类比：当你点击一个输入框开始打字，这个输入框就“被聚焦”了，此时会触发 `focus` 事件。

---

## **二、哪些元素能触发 `focus`？**

默认可聚焦的元素（无需额外设置）：
- `<input>`（所有类型）
- `<textarea>`
- `<select>`
- `<button>`
- `<a>`（带有 `href` 属性的链接）
- `<details>`

> ⚠️ 注意：普通元素如 `<div>`、`<span>` **默认不能聚焦**，但可通过添加 `tabindex` 属性使其可聚焦。

✅ 示例：让 div 可聚焦
```html
<div tabindex="0">点我或按 Tab 键可聚焦</div>
```

---

## **三、如何监听 `focus` 事件？**

### **方法1：使用 `addEventListener`**
```javascript
const input = document.querySelector('input');
input.addEventListener('focus', (event) => {
  console.log('输入框获得焦点！');
});
```

### **方法2：HTML 内联（不推荐用于复杂逻辑）**
```html
<input onfocus="console.log('获得焦点')">
```

---

## **四、典型使用场景**

| 场景 | 说明 |
|------|------|
| **表单增强** | 用户点击输入框时高亮边框、显示提示文字 |
| **自动选择内容** | 聚焦时自动全选输入框内容（如复制链接） |
| **动态加载数据** | 聚焦下拉框时才加载选项（懒加载） |
| **无障碍支持** | 配合键盘导航提升可访问性 |

### ✅**实用示例：聚焦时高亮 + 显示提示**

```html
<input type="text" id="username" placeholder="请输入用户名">
<p id="tip" style="display:none; color:gray;">用户名至少6位</p>

<script>
  const input = document.getElementById('username');
  const tip = document.getElementById('tip');

  input.addEventListener('focus', () => {
    input.style.borderColor = 'blue';
    tip.style.display = 'block';
  });

  input.addEventListener('blur', () => {
    input.style.borderColor = '';
    tip.style.display = 'none';
  });
</script>
```

---

## **五、重要特性 & 注意事项**

### 1. ❌ `focus` 事件**不会冒泡**
- 和 `blur` 一样，`focus` 是**非冒泡事件**。
- 如果你想在父容器监听子元素的聚焦行为，请使用 **`focusin`**（它是可冒泡版本）。

✅ 对比：
```javascript
// ❌ 无效：focus 不冒泡
document.body.addEventListener('focus', (e) => {
  console.log('某个子元素聚焦了？'); // 不会触发！
});

// ✅ 有效：使用 focusin
document.body.addEventListener('focusin', (e) => {
  console.log('聚焦的元素是：', e.target);
});
```

### 2. 🔒 `focus` 不能通过 JavaScript 阻止默认行为
- 你无法用 `event.preventDefault()` 阻止元素获得焦点。
- 但可以通过逻辑控制是否允许聚焦（例如禁用按钮）。

### 3. 🖱️ 聚焦方式不同，体验也不同
- **鼠标点击**：直接聚焦
- **Tab 键导航**：按 DOM 顺序或 `tabindex` 顺序聚焦
- **JavaScript 调用 `.focus()`**：程序化聚焦（无用户交互）

✅ 程序化聚焦示例：
```javascript
document.getElementById('myInput').focus(); // 立即聚焦
```

> 💡 小技巧：页面加载后自动聚焦搜索框，提升用户体验。

---

## 六、**`focus` vs `focusin` vs `blur` vs `focusout` 对照表**

| 事件名      | 触发时机         | 是否冒泡 | 适用场景                     |
|------------|------------------|--------|----------------------------|
| `focus`    | 元素获得焦点       | ❌      | 直接监听特定元素             |
| `blur`     | 元素失去焦点       | ❌      | 表单验证、清理状态           |
| `focusin`  | 元素获得焦点       | ✅      | 在父级统一处理子元素聚焦行为 |
| `focusout` | 元素失去焦点       | ✅      | 在父级统一处理子元素失焦行为 |

---

## 七、总结（速记卡片）

- ✅ `focus` = 元素**获得焦点**时触发。
- ✅ 默认只有**表单控件和链接**可聚焦；其他元素需加 `tabindex`。
- ❌ `focus` **不会冒泡** → 用 `focusin` 实现冒泡监听。
- 💡 常用于**高亮、提示、自动选择、无障碍导航**。
- 🔧 可通过 `.focus()` 方法**程序化聚焦**元素。

---