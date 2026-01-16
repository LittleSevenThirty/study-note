# **实例方法getAttribute**
## **概要**
**`getAttribute()`** 返回元素上一个指定的属性值。如果指定的属性不存在，则返回 null 或 "" （空字符串）；具体细节，请参阅 Notes 部分

## **语法**
```js
let attribute = element.getAttribute(attributeName);
```
* **attribute** 是一个包含 attributeName 属性值的字符串。
* **attributeName** 是你想要获取的属性值的属性名称。

## **例子**
这个元素可以是通过`getElement...`系列方法获取，也可以通过一个`元素自己的this`获取比如：
**case1**
```js
class MyElement extends HTMLElement{
  constructor(){
    let attribute=this.getAttribute("attribute");
  }
}
```
**case2**
```js
let div1 = document.getElementById("div1");
let align = div1.getAttribute("align");

alert(align);
// shows the value of align for the element with id="div1"
```