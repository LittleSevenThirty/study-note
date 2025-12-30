# **服务器端控件**
形如`<asp:??? runat="server"></asp:???>都是服务器端的控件`
任何html标签加上了`runat="server"`都会变成服务器可控制的标签
**正规的`<asp:>`服务器控件，核心规则是 ——「顶级控件必须加 runat="server"，嵌套的子控件不用加」**

## **<asp:Content runat="server">** 控件
母版页（Site.Master）里会有一个`<asp:ContentPlaceHolder ID="MainContent" runat="server">`的 “占位框”，你这个`<asp:Content>`的`ContentPlaceHolderID="MainContent"`就是 “填充” 到母版页的这个位置；
简单说：母版页定好了网站的统一布局（比如顶部导航、底部版权），每个子页面（比如 Default.aspx）只需要写<asp:Content>里的 “主体内容”，不用重复写布局代码。
### **属性功能查询**
|属性|通俗含义|示例+讲解|
|---|---|---|
|`ContentPlaceHolderID`|填充到母版页的位置的ID|`ContentPlaceHolderID="MainContent"`|

## **


## **自定义注册控件**
Web Forms 里 **注册“用户控件（.ascx 文件）”** 的指令`<%@ Register>` ——`.ascx`是 Web Forms 的 “复用组件”（比如多个页面都要显示 “审批日志”，不用重复写代码，做一个`ApplicationLog.ascx`，哪里需要就调哪里），Register就是告诉当前.aspx 页面：“我要用这些自定义控件，你先认识它们”
### **属性功能查询**
|属性|通俗含义|示例+讲解|
|---|---|---|
|`Src`|告诉页面 “这个自定义控件的文件存在哪”（控件的物理路径）|`Src="~/ProcessCenter/Controls/RequestBasicInfo.ascx"`|
|`TagPrefix`|给这组控件起个 “家族前缀”（自定义，比如 uc1、myCtrl 都可以），避免不同控件重名|`TagPrefix="uc1"	`|
|`TagName`|给这个控件起个 “专属名字”（和控件功能对应，方便识别）|`TagName="RequestBasicInfo"	`|

**后面能写 <uc1:ProcessTitle ... />**
之后就是调用刚才注册的控件！`<uc1:ProcessTitle runat="server" ID="ProcessTitle" />` 拆解：
* `uc1`：对应TagPrefix（家族前缀）；
* `ProcessTitle`：对应TagName（控件专属名）；
* `runat="server"`：因为.ascx是服务器控件，必须加；
* `ID="ProcessTitle"`：给这个控件实例起个唯一 ID，方便后台代码（.cs 文件）控制它（比如隐藏、改内容）。
简单类比：`Register`就像给快递员说 “我要收的包裹，寄件人叫`「uc1」`，包裹名是`「ProcessTitle」`，地址在`～/ProcessCenter/Controls/...”`；`<uc1:ProcessTitle />`就是告诉快递员 “把这个包裹送到当前页面来”。