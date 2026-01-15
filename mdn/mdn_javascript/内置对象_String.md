# **String类**
string内置数据类型也可以使用String的方法

## **实例方法讲解**
### **String.prototype.replace(...)**
语法`String.prototype.replace(pattern,replacement)`
其中一个、多个或所有匹配的 pattern 被替换为 replacement。pattern 可以是字符串或 RegExp【正则表达式】
* `pattern`：被匹配字符或正则表达式
* `replacement`：被替换字符串

**举例🌰**
`使用正则`
```js
const str="I think Ruth's dog is cuter than your dog!";
const regex=/Dog/i;
console.log(str.replace(regex,"ferret"));
// I think Ruth's ferret is cuter than your dog!
```
`使用字符串替换`
```js
const str="I think Ruth's dog is cuter than your dog!";
console.log(str.replace("Ruth's","my"));
// I think my dog is cuter than your dog!
```