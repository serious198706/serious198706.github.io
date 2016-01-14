---
layout: post
title: "VS2012下开发基于ATL的ActiveX控件"
description: 
headline: 
modified: 2013-01-16
category: development
tags: [windows, ActiveX]
imagefeature: 
mathjax: 
chart: 
comments: true
featured: true
--- 
  

好久没更新，这段时间实在又忙翻了。

这段时间一直在忙ActiveX控件，做一个登录模块，在网页上登录的时候，会记录登录者的硬盘序列号，然后将其发送到服务器并记录，下次登录的时候就会验证登录者的身份了。

听起来很简单，但做起来让人崩溃。

首先，ActiveX技术在几年前还是门热门的技术，但现在微软已经开始逐渐抛弃这门技术了，因为它太过繁琐，而且兼容性很差。而且现在像PHP，JavaScript等都可以获得硬盘的序列号，完全可以无视ActiveX控件的功能了。再加之用高版本IDE开发出来的ActiveX控件是不能直接在低版本的操作系统上运行的，需要附带库。那样就会让插件变得臃肿不堪。还有，因为ActiveX技术是由微软提出的，所以只支持IE下安装使用，其他浏览器连门都没有。虽然可以通过一些手段让ActiveX插件在其他浏览器上运行，但总归不是“明媒正娶”。。

但是公司下达的命令，我也没法抗议，再说了，在深入了解这门技术之前，我也没想到这东西的兼容性会这么差。

万金油：

>ActiveX是Microsoft对于一系列策略性面向对象程序技术和工具的称呼，其中主要的技术是组件对象模型（COM）。在有目录和其它支持的网络中，COM变成了分布式COM（DCOM）。在创建包括ActiveX程序时，主要的工作就是组件，一个可以自足的在ActiveX网络（现在的网络主要包括Windows和Mac）中任意运行的程序。这个组件就是ActiveX控件。ActiveX是Microsoft为抗衡Sun Microsystems的JAVA技术而提出的，此控件的功能和java applet功能类似。

我的需求比较简单，开发出一个控件，能够在Web Service上发布，并提供给远程用户使用，但就这个需求折腾了我整整三天，虽然最后解决了，但解决的办法非常的令人不爽，后面再叙。

<!-- more -->

### 准备工作

如果要开发一个ActiveX控件，放在网络上供用户下载、安装并运行的话，是需要经过签名的，而且还需要打包成特定格式并且签名。这儿我们就需要一些工具了。工具在文章末有提供。附上任意门，签名教程在任意门里可以找到，签名不提供。。一年1300大洋不能随便给。。

我是在Win8，Visual Studio 2012下开发的，网上关于ActiveX的资料里，VS2003的教程最多，其次是VC6。

### 创建工程

建立一个新的工程，选择ATL模版下的ATL Project，并给工程命名为ActiveXDemo。

<img alt="" src="http://i1352.photobucket.com/albums/q645/cy198706/Programming/20130116100714.jpg" />

在接下来弹出的页面中，点击Next，进入如下页面：

<img alt="" src="http://i1352.photobucket.com/albums/q645/cy198706/Programming/20130116101001.jpg" />

左边一列是选择要开发的控件类型，我们选择DLL，Excutable(EXE)是可执行EXE文件，Service(EXE)是放在服务器端的可执行文件，都不是我需要的。

右边一列是该控件所支持的特性选项。

- Allow merging of proxy/stub code：允许在DLL服务器中捆绑自定义的代理/存根代码，选择此选项ATL将为我们提供一个&lt;projectname&gt;ps.mk的编译文件，如过我们要编译这个文件则在Project菜单中选择Setting，然后找到Post-build step选项卡在Post-build command(s)下填入：

- <pre class="prettyprint linenums:1">
nmake -f mkfile
</pre>
<br />
其中mkfile即为生成的.mk编译文件的名字。这个文件用于建立一个独立的代理/存根DLL，该DLL将被分发到所有需要列集和散集自定义接口的COM客户和服务器所在的机器上，如果我们希望把代理/存根DLL捆绑到DLL服务器中(因而在服务器所在的主机上可以节省一个独立的代理/存根DLL),就需要选中此项，在生成工程之后，还需要完成以下操作才能真正合并代理/存个代码。
  - 在Project Setting 对话框中为所有配置添加预处理器定义_MERGE_PROXYSTUB
  - 在dlldatax.c文件的设置中不要选中"Exclude file from build"。
  - 把dlldatax.c的预编译头文件的设置改为"Not using precompiled headers"
- Support MFC：支持MFC，我们不要选。本来就是要开发ATL下的控件，不需要它支持MFC。
- Support COM+ 1.0及Support component register：COM+ 1.0旨在开发基于组件的分布式应用程序。它还提供用于部署和管理这些应用程序的运行时结构。如果选择“支持 COM+ 1.0”复选框，向导将修改链接步骤中的生成脚本。具体说来，COM+ 1.0 项目链接到这两个库：comsvcs.lib、Mtxguid.lib。如果选择“支持 COM+ 1.0”复选框，则也可以选择“Support component register（组件注册器）”。 组件注册器使 COM+ 1.0 对象得以获取组件列表、注册组件或注销组件（个别或同时）。
- Security Development Lifecycle(SDL) checks：开启安全开发生命周期检查。安全开发生命周期是个很大的课题，涉世未深，我一点都不懂。。有兴趣的自行Google，或者看看[这儿](http://msdn.microsoft.com/zh-cn/library/ms995349.aspx)。现在只说明这是个帮助你的软件更加安全的特性。

点击Finish，就完成了工程的创建。

### 添加组件

VS为我们创建了很多文件。接下来我们先添加一个控件类，再来解释这些文件。

选中我们的工程，右键，Add->Add Class，会弹出如下的页面：

<img alt="" src="http://i1352.photobucket.com/albums/q645/cy198706/Programming/20130116152422.jpg" />

选择ATL Simple Object就可以了。这种对象不可见，通常用于完成一些后台操作。ATL Control是控件型对象，通常用于在网页上显示一个控件，如文本框、按钮、绘制文字等。

点击Add。进入下一个页面。

<img alt="" src="http://i1352.photobucket.com/albums/q645/cy198706/Programming/20130116153039.jpg" />


填入Short Name，Alert，然后向导会帮你生成其他的名字。注意一点，在VS2005之后，ProgID就必须手动填写了，这通常方便组件注册使用。一般的命名规则是“Project_Name.Short_Name”。再点击Next。

<img alt="" src="http://i1352.photobucket.com/albums/q645/cy198706/Programming/20130116153051.jpg" />


直接点击Next。

<img alt="" src="http://i1352.photobucket.com/albums/q645/cy198706/Programming/20130116153123.jpg" />

这一个页面也比较有学问，来一一解释。

- Threading Model：对于大多数用于Web的ActiveX控件来说，Single足够了，但是为了以防万一，有多线程功能，我们还是以默认为佳，选择Apartment（单元线程）。
- Aggregation：是否允许其他COM组件与此组件进行聚合。属于可扩展性部分，以默认，选择Yes。
- Interface：接口模式，选择Dual（双重接口）。
- Support：额外支持特性。
- ISupportErrorInfo是支持返回错误信息。
- Connection Points支持Web向ActiveX控件发送事件。
- IObjectWithSite(IE Object support)。一个站点（site）是一个中间对象，它位于容器对象（比如IE）与被包容对象之间。通过它，容器对象管理被包容对象的内容，也因此使得对象的内部功能可用。为此，容器方要实现接口IoleClientSite，被包容对象要实现接口IOleObject 。通过调用IOleObject提供的方法，容器对象使得被包容对象清楚地了解其宿主的环境。

 一旦容器对象成为IE（或是具有WEB能力的Windows资源管理器），被包容对象只需实现一个轻型的IObjectWithSite接口。这个接口包括两个函数：

- <pre class="prettyprint linenums:1">
HRESULT SetSite(IUnknown* pUnkSite)                 // 接收ie浏览器的IUnknown指针
                                                    // 典型实现是保存该指针以备将来使用
HRESULT GetSite(REFIID riid， void** ppvSite)       // 从通过SetSite()方法设置的场所中接收并返回指定的接口
                                                    // 典型实现是查询前面保存的接口指针以进一步取得指定的接口
</pre>

各个选项的选择，就依照上图的选择即可。点击Finish，就添加完成我们的控件类了。

来看看VS帮我们生成的文件。

<img alt="" src="http://i1352.photobucket.com/albums/q645/cy198706/Programming/20130116160619.jpg" />

不要看文件很多，我们需要更改的只有Alert.h和Alert.cpp而已。其余的可以通过向导来帮我们实现，比如添加方法，添加属性等。

转到Class视图，展开ActiveXDemo类，在IAlert上点击右键，选择Add->Add Method。

<img alt="" src="http://i1352.photobucket.com/albums/q645/cy198706/Programming/20130116160924.jpg" />

在添加方法的界面，输入方法的名称GetString，在下方添加一个BSTR*类型的变量，并在上方勾选retval，表示该变量为返回值型变量，命名为pVal，点击Add添加到方法中，最后点击Finish。

<img alt="" src="http://i1352.photobucket.com/albums/q645/cy198706/Programming/20130116161038.jpg" />

添加完方法后，最直观的表现，就是在名为ActiveXDemo.idl的文件中，多了如下一行代码：

<pre class="prettyprint linenums:1">
[id(1)] HRESULT GetString([out,retval] BSTR* pVal);
</pre>
<br />
当然一行代码不可能起什么作用，还有些后台偷偷添加的代码，我们就不管了
<img alt="" src="http://cy198706.com/wp-content/plugins/ck-and-syntaxhighlighter/ckeditor/plugins/smiley/images/6.gif" />

打开Alert.cpp，在我们的GetString方法中，修改代码：

<pre class="prettyprint linenums:1">
STDMETHODIMP CAlert::GetString(BSTR* pVal)
{
	// TODO: Add your implementation code here
	USES_CONVERSION;
	char szStr[20] = "Hello World!";
	*pVal = ::SysAllocString(A2W(szStr));
	return S_OK;
}
</pre>
<br />
功能的部分就完成了，但是接下来要实现一个关于“安全脚本调用”和“安全执行”的功能，都是因为IE的诸多限制，要想去掉烦人的提示，就必须在控件中标明“偶是安全的，偶是无公害的，偶是可以被脚本调用的”。

在ATL3.0版本以后，标记一个ActiveX控件为安全已经变成了很简单的事件，只需要两行代码即可。顺带一提，VC6.0集成的就是ATL3.0，所以完全不用担心VS2012的ATL版本。

打开Alert.h，为CAlert类多添加一个继承类：

<pre class="prettyprint linenums:1">
public IObjectSafetyImpl&lt;CAlert, INTERFACESAFE_FOR_UNTRUSTED_CALLER | INTERFACESAFE_FOR_UNTRUSTED_DATA&gt;
</pre>
<br />
并在下方的BEGIN_COM_MAP和END_COM_MAP中间插入一句：

<pre class="prettyprint linenums:1">
COM_INTERFACE_ENTRY(IObjectSafety)
</pre>
<br />
就可以了。

好了，编译吧。这儿要提个醒，由于是在Windows8下，在注册dll的时候是需要管理员权限的，如果有必要的话，VS2012要使用管理员身份打开。

工作只完成了一半哟。找到我们的DLL文件，拖到一个“干净”的地方。现在需要用到上面提到的打包签名教程里的工具MakeCab.exe及SignCode.exe。

先对DLL文件进行签名，否则在交互时会提示不安全的交互，麻烦至极。

建立一个ActiveXDemo.inf文件，这个文件描述了控件的CLSID及下载成功后DLL文件的安装位置，还提供DLL的版本，供客户端检查是否应该更新控件。inf的文件内容如下：

<pre class="prettyprint linenums:1">
[version] 
signature="$CHICAGO$"
AdvancedINF=2.0 

[Add.Code] 
ActiveXDemo.dll=ActiveXDemo.dll 

[ActiveXDemo.dll] 
clsid={7EE749D3-E73A-4A7B-A94B-95C931ECEDB9} 
file-win32-x86=thiscab 
FileVersion=1,0,0,1
DestDir=11 
RegisterServer=yes 
</pre>
<br />
clsid的值取自工程的ActiveXDemo.idl文件：

<pre class="prettyprint linenums:1">
importlib("stdole2.tlb");
[
    uuid(7EE749D3-E73A-4A7B-A94B-95C931ECEDB9)		
]
</pre>
<br />
DestDir表示安装路径为C:\Windows\SysWoW64\，Windows XP为C:\Windows\System32\。

接下来新建一个list.txt文件，在里面输入：

<pre class="prettyprint linenums:1">
ActiveXDemo.dll
ActiveXDemo.inf
</pre>
<br />
在Dll的旁边，放下MakeCab.exe。为了方便，我建立一个.bat文件，只要双击之，就可以自动打包，免去在命令提示符中打包的痛苦。.bat文件内容如下：

<pre class="prettyprint linenums:1">
cd /d %~dp0
MAKECAB.EXE /f list.txt /d compressiontype=lzx /d compressionmemory=21 /d cabinetnametemplate=result.cab
pause
</pre>

双击这个bat文件，就自动生成了一个disk1文件夹，里面就是我们的Cab压缩包。对这个Cab压缩包进行签名。

建立一个测试用的Web页面test.html吧。内容如下即可：

<pre class="prettyprint linenums:1 lang-html">
&lt;HTML&gt;
&lt;HEAD&gt; 
&lt;TITLE&gt;New Page&lt;/TITLE&gt; 
	&lt;OBJECT id=obj align="CENTER" WIDTH=0 HEIGHT=0 
	codeBase="ActiveXDemo.CAB#version=1,0,0,1" 
	classid="CLSID:7EE749D3-E73A-4A7B-A94B-95C931ECEDB9"&gt;
	&lt;/OBJECT&gt; 
	&lt;script language="javascript"&gt; 
		function doTest() 
		{ 
			var sum = obj.GetString(); 
			alert(sum); 
		} 
	&lt;/script&gt; 
&lt;/HEAD&gt; 
&lt;BODY&gt; 
	&lt;input type="button" value="TEST" id="btnOK" onclick="doTest();"&gt;&lt;/input&gt; 
&lt;/BODY&gt; 
&lt;/HTML&gt; 
</pre>

在页面上加入了一个Object元素。codeBase是指CAB包应该从哪里下载，我写的是相对路径，那么CAB包就应该与页面文件放在同一个目录下。classid就是刚才inf文件里的clsid了。

我选择使用Web Service来实验结果，将Cab包与test.html一起放到wwwroot目录下，在IE中运行，按照步骤来安装控件，点击按钮，出现“Hello World！”，任务完成。

<img alt="" src="http://i1352.photobucket.com/albums/q645/cy198706/Programming/20130116173218.jpg" />

这是最普通的ActiveX插件，几乎“对操作系统没有操作”，如果要用到一些库的话，那么打包的时候，就必须把库文件也一起签名打包进去了，Cab包就会变得巨大无比。还有，如果在VS2012下开发的话，很可能会出现本地可以访问但客户端电脑（无论是什么操作系统）无法成功安装的情况，我解决了三天就是在解决这个问题，最后在虚拟机的XP+VC6下开发了一个，才成功的。。有经验的大虾们，烦请指教。

打包及签名工具：

<a href="pan.baidu.com/share/link?shareid=215150&uk=151049050"><img alt="" src="http://icons.iconarchive.com/icons/robsonbillponte/sinem/256/File-Downloads-icon.png" width="128" height="128" /></a>

