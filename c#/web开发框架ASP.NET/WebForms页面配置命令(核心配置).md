# **页面配置命令**

## **通用指令属性功能查询**
|属性|通俗含义|示例+讲解|
|---|---|---|
|`Language="C#"`|后台代码用 C# 写||
|`AutoEventWireup="true"`|自动绑定页面的加载、初始化等事件||
|`CodeBehind="Site.master.cs"`|页面的后台逻辑文件||
|`Inherits="TestWebApplication1.SiteMaster"`|关联页面后台的 C# 类名，之后逻辑函数，参数都写在这个类里面||
|`MasterPageFile="~/ProcessCenter/ProcessPage.Master"`|关联母版页面提供位置信息||

## **<%@ Page %>指令**
告诉服务器怎么处理这个页面，一般出在`xxx.aspx`文件里

## **<%@ Master %>指令**
<%@ Master %> 就是母版页的 “身份证 + 配置单”，告诉服务器 “这是个母版页，按这规则解析

## **<%@ Register %>指令**
Web Forms 里 **注册“用户控件（.ascx 文件）”** 的指令 ——`.ascx`是 Web Forms 的 “复用组件”（比如多个页面都要显示 “审批日志”，不用重复写代码，做一个`ApplicationLog.ascx`，哪里需要就调哪里），Register就是告诉当前.aspx 页面：“我要用这些自定义控件，你先认识它们”
### **属性功能查询**
|属性|通俗含义|示例+讲解|
|---|---|---|
|`Src`|告诉页面 “这个自定义控件的文件存在哪”（控件的物理路径）|`Src="~/ProcessCenter/Controls/RequestBasicInfo.ascx"`|
|`TagPrefix`|给这组控件起个 “家族前缀”（自定义，比如 uc1、myCtrl 都可以），避免不同控件重名|`TagPrefix="uc1"	`|
|`TagName`|给这个控件起个 “专属名字”（和控件功能对应，方便识别）|`TagName="RequestBasicInfo"	`|

**后面能写 <uc1:ProcessTitle ... />**
这就是调用刚才注册的控件！`<uc1:ProcessTitle runat="server" ID="ProcessTitle" />` 拆解：
* `uc1`：对应TagPrefix（家族前缀）；
* `ProcessTitle`：对应TagName（控件专属名）；
* `runat="server"`：因为.ascx是服务器控件，必须加；
* `ID="ProcessTitle"`：给这个控件实例起个唯一 ID，方便后台代码（.cs 文件）控制它（比如隐藏、改内容）。
简单类比：`Register`就像给快递员说 “我要收的包裹，寄件人叫`「uc1」`，包裹名是`「ProcessTitle」`，地址在`～/ProcessCenter/Controls/...”`；`<uc1:ProcessTitle />`就是告诉快递员 “把这个包裹送到当前页面来”。