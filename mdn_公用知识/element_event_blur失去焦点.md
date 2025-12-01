# 📝 **事件系统与 `blur` 事件学习笔记**

## **一、前置知识：什么是“事件”（Event）？**

- **事件**是用户或浏览器执行的某种操作，例如：
  - 用户点击按钮（`click`）
  - 鼠标移动（`mousemove`）
  - 元素获得焦点（`focus`）
  - 元素失去焦点（`blur`）
- 浏览器会**自动触发**这些事件，开发者可以通过 JavaScript **监听**（listen to）这些事件并执行代码。

> 💡 类比：事件就像“铃声”，你（JavaScript）可以“听到铃声”后做某件事（比如接电话）。

---

## **二、什么是“焦点”（Focus）？**

- **焦点（focus）** 是指当前正在接收键盘输入或交互的元素。
- 常见可聚焦元素：
  - `<input>`、`<textarea>`、`<select>`
  - `<button>`
  - 任何设置了 `tabindex` 属性的元素（如 `<div tabindex="0">`）
- 当一个元素获得焦点时，会触发 `focus` 事件；失去焦点时，触发 `blur` 事件。

> ✅ 小测试：在网页上点击一个输入框，它周围出现蓝色边框 → 这就是获得了焦点！

---

## **三、`blur` 事件详解**

### **1. 定义**
- **`blur` 事件**在**元素失去焦点**时触发。
- 与之对应的是 `focus` 事件（获得焦点时触发）。

### **2. 触发场景举例**
| 操作 | 是否触发 `blur` |
|------|----------------|
| 点击另一个输入框 | ✅ 是（原输入框失去焦点） |
| 按 Tab 键跳到下一个元素 | ✅ 是 |
| 点击页面空白处 | ✅ 是（如果之前有焦点） |
| 页面失去窗口焦点（切换标签页） | ❌ 否（这是 `window.blur`，不是元素的 `blur`） |

### **3. 语法（如何监听）**

```javascript
element.addEventListener('blur', function(event) {
  console.log('元素失去焦点了！');
});
```

或者使用 HTML 内联（不推荐用于复杂逻辑）：
```html
<input onblur="console.log('失去焦点')">
```

### **4. 实际用途**
- 表单验证（用户离开输入框时检查内容）
- 自动保存草稿
- 隐藏提示信息

### **5. 示例：表单验证**

```html
<input type="email" id="email">

<script>
  const emailInput = document.getElementById('email');

  emailInput.addEventListener('blur', function() {
    if (!this.value.includes('@')) {
      alert('请输入有效的邮箱地址！');
    }
  });
</script>
```

---

## **四、常见误区 & 注意事项**

### ❌ 误区1：所有元素都能触发 `blur`
- **错误！** 只有**可聚焦元素**才能触发 `blur`。
- 普通 `<div>` 默认不能聚焦，除非加上 `tabindex="0"`。

✅ 正确做法：
```html
<div tabindex="0" id="myDiv">点我后按 Tab 或点击别处会触发 blur</div>
<script>
  document.getElementById('myDiv').addEventListener('blur', () => {
    console.log('div 失去焦点');
  });
</script>
```

### ❌ **误区2：`blur` 会冒泡（bubbling）**
- **`blur` 事件不会冒泡！**
- 如果你想在父元素监听子元素的 `blur`，需要用 `focusout`（IE 标准，但现代浏览器也支持）。

✅ 替代方案（支持冒泡）：
```javascript
parent.addEventListener('focusout', (e) => {
  if (e.target === childElement) {
    console.log('子元素失去焦点');
  }
});
```

> 🔍 补充：`focusin` / `focusout` 是 `focus` / `blur` 的可冒泡版本。

---

## **五、相关事件对比表**

| 事件名      | 触发时机         | 是否冒泡 | 常见用途             |
|------------|------------------|--------|--------------------|
| `focus`    | 元素获得焦点       | ❌      | 显示提示、高亮        |
| `blur`     | 元素失去焦点       | ❌      | 验证、隐藏提示        |
| `focusin`  | 元素获得焦点       | ✅      | 在父级监听子元素聚焦（冒泡用）   |
| `focusout` | 元素失去焦点       | ✅      | 在父级监听子元素失焦（冒泡用）  |

---

## 六、总结（速记卡片）

- ✅ `blur` = 元素**失去焦点**时触发。
- ✅ 只有**可聚焦元素**才有 `blur`。
- ❌ `blur` **不会冒泡** → 用 `focusout` 代替。
- 💡 常用于**表单验证**、**自动保存**等场景。
- 🔧 普通元素想支持 focus/blur？加 `tabindex="0"`！

---