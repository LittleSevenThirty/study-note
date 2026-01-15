# **Math类**
Math 是一个内置对象，它拥有一些数学常数属性和数学函数方法。Math 不是一个函数对象。
Math 用于 Number 类型。它不支持 BigInt
**它无法实例化，使用的都是其静态方法**

## **静态方法讲解**
### **Math.trunc(...)**
语法`Math.trunc(value)`
返回一个数值的整数部分，直接去除其小数点及以后部分，即使是负数也一样
* `value`：任意数值

**举例🌰**
```js
Math.trunc(13.37); // 13
Math.trunc(42.84); // 42
Math.trunc(0.123); //  0
Math.trunc(-0.123); // -0
Math.trunc("-1.123"); // -1
Math.trunc(NaN); // NaN
Math.trunc("foo"); // NaN
Math.trunc(); // NaN
```