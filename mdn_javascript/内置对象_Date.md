# **Date类**
[官方文档更加详细👉](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Date#%E6%96%B9%E6%B3%95)

## **实例对象**
```js
const d1=new Date();    // 默认是当前时间
const d2=new Date(value);
const d3=new Date(dateString);
const d4=new Date(year, monthIndex [, day [, hours [, minutes [, seconds [, milliseconds]]]]]);
```

## **实例方法讲解**
### **Date.prototype.getTime()**
返回一个数值，表示从1970年1月1日0时0分0秒（UTC，即协调世界时）距离该`Date`对象所代表时间的毫秒数。（更早的时间会用负数表示）

**举例🌰**
`简单使用`
```js
console.log(new Date().getTime());  // 
```
`复制相同日期对象`
```js
let d1=new Date(2025,9,1);
let d2=new Date();
d2.setTime(d1.getTime());
console.log(d2);    // Wed Oct 01 2025 00:00:00 GMT+0800 (China Standard Time)
```
`测量代码执行时间`
```js
var end, start, i;

start = new Date();
for (i = 0; i < 1000; i++) {
  Math.sqrt(i);
}
end = new Date();

console.log("操作耗时：" + (end.getTime() - start.getTime()) + "ms");
```