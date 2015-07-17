---
layout: post
title: "Windows Store Apps 开发手札 - 12（应用功能声明）"
description:
headline:
modified: 2012-12-31
category: development   
tags: [windows, windows store apps]
imagefeature:
mathjax:
chart:
comments: true
featured: true
---

Windows Store App是个权限非常非常低的东西，它运行在一个安全沙盒中，通过这个沙盒与WinRT进行交互，然后WinRT再与系统进行交互。这个沙盒其中的一个功能就是放低Windows Store App的权限，如果有需要，请向沙盒申请权限。

例如，如果 Windows Store App需要对用户资源（如“图片库”）或连接的设备（如摄像头）进行编程访问，必须声明相应的capability。
我们可以使用 Visual Studio 中的清单设计器来声明许可范围。

清单设计器其实就在我们工程里的package.appxmanifest中。双击打开package.appxmanifest，然后选中选项卡中的Capbilities，就可以看到所有的权限了：

<img alt="" src="http://images.cy198706.com/Programming/20121227095040.jpg" />

在列表中，网络（Internet(client)）权限默认是选中的。

简单介绍一下这些权限。

- 文档库

 DocumentsLibrary 许可范围可提供对用户文档库的编程访问能力，这些文档将按照包清单中声明的文件类型关联进行筛选。例如，如果一个DOC阅读器应用仅声明了一个 .doc 文件类型关联，则它可以打开文档库中的 .doc 文件，而无法打开其他类型的文件。这种特性类似MFC中文件对话框（CFileDialog）对文件扩展名的限制。

 你只能将 DocumentsLibrary 许可范围用于支持打开其他文档中的嵌入内容。

 声明 DocumentsLibrary 许可范围的应用无法访问家庭组计算机中的文档库。文件选取器提供了一种强大的 UI 机制，让用户可以打开要通过某个应用处理的文件。

 仅当应用需要进行编程访问，而使用文件选取器无法实现编程访问时，才应声明 DocumentsLibrary 许可范围。 

 视频库、音乐库、图片库、可移动存储类似。

 但是可移动存储要注意一下，声明此许可范围时请务必小心，因为用户的可移动存储中可能包含多种信息，它们需要应用提供有效的理由才能对该可移动存储中所有文件类型进行编程访问。

- 企业身份验证

 Windows 域凭据支持用户使用其凭据登录远程资源，向用户提供了他们的用户名和密码一样操作。Enterprise Authentication 许可范围通常用在连接到企业内服务器的业务线应用中。对于一般的网络通信无需使用此许可范围。

 Enterprise Authentication 许可范围专用于支持一些常用的业务线应用。请勿在无需访问企业资源的应用中声明此许可范围。 文件选取器提供了一种强大的 UI 机制，让用户可以打开网络共享中要通过某个应用处理的文件。 仅当应用需要进行编程访问，而使用文件选取器无法实现编程访问时，才应声明 Enterprise Authentication 许可范围。

- Internet 和公共网络

 Internet (Client) 许可范围可提供通过防火墙对 Internet 和公共网络的出站访问能力。几乎所有 Web 应用都使用此许可范围。 Internet (Client) 许可范围可提供通过防火墙对 Internet 和公共网络的入站和出站访问。

 Internet (Client & Server) 许可范围通常用在使用文件共享和 VOIP 的应用的对等 (P2P) 场景中。Internet (Client & Server) 许可范围包括 Internet (Client)许可范围所提供的访问能力，因此，在指定 Internet (Client & Server) 时无需指定 Internet (Client)。

 如果你声明这些功能，则一个重要认证要求是在“设置”窗格中包含指向“隐私策略”的链接。阻止认证的最常见原因就是忘记执行此操作！

- 位置

 Location 许可范围可提供对定位功能的访问能力，定位功能通常从专用硬件（如电脑中的 GPS 传感器）或从可用的网络信息中获得。应用必须处理用户从“设置”超级按钮禁用了定位服务的情况。有关如何检测用户位置的示例，会在后面的文章中提到。 

- 麦克风

 Microphone 许可范围可提供对麦克风音频源的访问能力，让应用可以录制来自所连接麦克风的音频。应用必须处理用户从“设置”超级按钮禁用了麦克风的情况。有关如何录制音频的示例，会在后面的文章中提到。 

- 家庭和工作网络

 Private Network (Client & Server) 许可范围可提供通过防火墙对家庭和工作网络的入站和出站访问能力。此功能通常用于通过局域网 (LAN) 通信的游戏和在各种本地设备上共享数据的应用。
如果你的应用指定了 MusicLibrary、PicturesLibrary 或 VideosLibrary，则无需使用此许可范围即可访问家庭组中相应的库。 

- 临近设备通信

 Proximity 功能支持临近的多个设备彼此通信。此许可范围通常用在多用户休闲游戏和交换信息的应用中。 设备会尝试使用可提供最佳连接的通信技术，包括蓝牙、WiFi 和 Internet。此功能仅用于发起设备间的通信。 有关如何使用邻近感应连接应用的示例，请参阅快速入门：使用点击或浏览连接应用。 


- 网络摄像机

 Webcam许可范围可提供对摄像头视频源的访问能力，让应用可以捕获来自所连接摄像头的快照和视频。 此功能通常在视频聊天或会议应用中使用。 应用必须处理用户从“设置”超级按钮禁用了摄像头的情况。 有关如何录制视频的示例，请参阅如何录制音频或视频。
Webcam许可范围仅授予对视频流的访问权限。若也要授予对音频流的访问权限，必须添加 Microphone 许可范围。

- 共享用户证书

 Shared User Certificates 功能支持应用访问软件和硬件证书，例如存储在智能卡上的证书。此功能通常用于需要智能卡来验证身份的财务或企业应用。

在编写Windows Store App的时候，要注意权限的选中，但也不要乱选一气，权限太高对于 Windows8 来说，也不是什么好事。