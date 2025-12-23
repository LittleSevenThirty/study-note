# **服务器端代码在前端渲染语法**

## **渲染的三种语法**
这是Web Forms的「服务器端代码渲染语法」，核心分3种（你代码里都出现了），先看你最关心的：

| 语法 | 例子 | 通俗含义 |
|------|------|----------|
| `<%: 变量/表达式 %>` | `<%: Page.Title %>`、`<%: DateTime.Now.Year %>` | 「安全输出」：把服务器端的值渲染成HTML文本（自动防XSS攻击）<br>✅ `<%: Page.Title %>`：取子页面（比如Default.aspx）`<%@ Page Title="Home Page" %>`里的Title值，所以运行后标题是「Home Page - 我的 ASP.NET 应用程序」<br>✅ `<%: DateTime.Now.Year %>`：动态输出当前年份（比如2025），不用手动改代码，每年自动变 |
| `<% 代码块 %>` | （你代码里没直接出现，但常见）比如 `<% int a = 1; Response.Write(a); %>` | 「执行代码」：直接运行一段C#代码，不输出内容（除非主动Write） |
| `<%-- 注释 --%>` | `<%--To learn more about bundling scripts...--%>` | 「服务器端注释」：只在服务器解析时生效，不会被发送到浏览器（区别于`<!-- 客户端注释 -->`，后者会传到浏览器，能在F12里看到） |

补充：你看到的 `<%: Scripts.Render("~/bundles/modernizr") %>` 是ASP.NET的「脚本捆绑」——把多个JS文件打包成一个，减少浏览器请求，`Scripts.Render` 就是输出这个打包后的JS链接。