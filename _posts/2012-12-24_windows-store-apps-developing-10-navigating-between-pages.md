---
layout: post
title: "Windows Store Apps 开发手札 - 10（在页面之间导航）"
description: 
headline: 
modified: 2012-12-24
category: development
tags: [windows, windows store apps]
imagefeature: 
mathjax: 
chart: 
comments: true
featured: true
---

在Windows Store App中，页面导航是一个非常重要部分。我们可以为 Windows Store App创建多个页面，并支持用户在应用中的页面之间进行导航，类似于在一个网站的网页之间进行导航。Visual Studio 11 包含很多页面模板，这些模板可为我们的应用提供基本的导航支持。在这篇文章中，我们使用基本页面模板创建一个支持导航的简单应用。

## 谁在工作？

XAMLUI 框架提供内置的导航模型，该模型使用 Frames 和 Pages，并且其工作方式与在 Web 浏览器中的导航十分类似。Frame 控件可托管 Pages，并且包含导航历史记录，你可以通过该历史记录在访问过的页面中前进和后退。在导航时，你可以在页面之间传递数据。

应用启动时，随即开始导航。支持导航的基础结构位于 App 类中。在 Visual Studio 项目模板中，一个名为 rootFrame 的 Frame 被设为应用窗口的内容。我们可以看一下 App.xaml.cpp 中的默认代码。

<pre class="prettyprint linenums:1">
void App::OnLaunched(Windows::ApplicationModel::Activation::LaunchActivatedEventArgs^ args)
{
	auto rootFrame = dynamic_cast<Frame^>(Window::Current->Content);

	// Do not repeat app initialization when the Window already has content,
	// just ensure that the window is active
	if (rootFrame == nullptr)
	{
		// Create a Frame to act as the navigation context and associate it with
		// a SuspensionManager key
		rootFrame = ref new Frame();

		if (args->PreviousExecutionState == ApplicationExecutionState::Terminated)
		{
			// TODO: Restore the saved session state only when appropriate, scheduling the
			// final launch steps after the restore is complete

		}

		if (rootFrame->Content == nullptr)
		{
			// When the navigation stack isn't restored navigate to the first page,
			// configuring the new page by passing required information as a navigation
			// parameter
			if (!rootFrame->Navigate(TypeName(MainPage::typeid), args->Arguments))     // 就是这儿
			{
				throw ref new FailureException("Failed to create initial page");
			}
		}
		// Place the frame in the current Window
		Window::Current->Content = rootFrame;
		// Ensure the current window is active
		Window::Current->Activate();
	}
	else
	{
		if (rootFrame->Content == nullptr)
		{
			// When the navigation stack isn't restored navigate to the first page,
			// configuring the new page by passing required information as a navigation
			// parameter
			if (!rootFrame->Navigate(TypeName(MainPage::typeid), args->Arguments))
			{
				throw ref new FailureException("Failed to create initial page");
			}
		}
		// Ensure the current window is active
		Window::Current->Activate();
	}
}
</pre>

请注意，设置 rootFrame 后，应用会检查它当前的状态，因为它可能正从关闭状态启动或正从挂起状态恢复，并且已经将自己的内容存储在内存中。如果它正从终止状态启动，则必须加载终止时所保存的状态。对上述各种情况进行处理之后，应用将导航到第一个窗口。

### Frame 和 Page 类

在建立应用之前，让我们来看看我们所添加的页面如何为应用提供导航支持。 Frame 类主要负责导航和实现方法，例如 Navigate、GoBack 和 GoForward。 使用 Navigate 方法在 Frame 中显示内容。 在前一个示例中，App.OnLaunched 方法将会创建一个 Frame，并将 BasicPage1 传递给 Navigate 方法。 然后，该方法将应用的当前窗口的内容设置为 Frame。其结果是，应用的窗口包含一个包含 BasicPage1 的 Frame。

BasicPage1 是 Page 类的间接子类。 Page 类具有 Frame 属性，它是一种用于获得包含 Page 的 Frame 的只读属性。 当 Button 的 Click 事件处理程序调用 Frame->Navigate(TypeName(BasicPage2::typeid)) 时，应用窗口中的 Frame 将会显示 BasicPage2 的内容。

## 页面导航

建立一个新的工程，并按Ctrl + Shift + A，添加一个Basic Page，并命名为BasicPage1.xaml，并如法炮制添加BasicPage2.xaml。

<img alt="" src="http://images.cy198706.com/Programming/20121221164818.jpg" />

为了方便分辨两个页面，我们先打开BasicPage1.xaml，点击上面的My Application，并在下方XAML代码中，将Text="{StaticResource AppName}"修改为Text="Basic Page 1"，接着去BasicPage2.xaml，修改为Text="Basic Page 2"。

然后我们去MainPage.xaml添加一个按钮，并添加一个事件Go_to_Page1：

<pre class="prettyprint linenums:1">
&lt;Grid Background="{StaticResource ApplicationPageBackgroundThemeBrush}">
    &lt;Button Content="Go to Page 1" Height="130" Width="200" HorizontalAlignment="Center" VerticalAlignment="Center"  Click="Go_to_Page1"/>
&lt;/Grid>
</pre>

并编写Go_to_Page1的代码：

<pre class="prettyprint linenums:1">
#include "BasicPage1.xaml.h"
using namespace Windows::UI::Xaml::Interop;

void NavigationTest::MainPage::Go_to_Page1(Platform::Object^ sender, Windows::UI::Xaml::RoutedEventArgs^ e)
{
	this->Frame->Navigate(TypeName(BasicPage1::typeid));   // 其实是Frame控件在帮我们进行导航工作
}
</pre>

TypeName(BasicPage1::typeid))是一个比较有趣的东西。typeid的用途是获得一个对象的类型，比如这儿的BasicPage1，它的类型就是BasicPage1，比如Float*，它的类型是System.Single*。而TypeName则是获得这个类型的名字。

接下来为BasicPage1.xaml添加一个按钮，并添加一个事件Go_to_Page2：（注意分清层次关系，哪个Grid与哪个Grid同级）

<pre class="prettyprint linenums:1">
&lt;!-- Back button and page title -->
&lt;Grid>
    &lt;Grid.ColumnDefinitions>
        &lt;ColumnDefinition Width="Auto"/>
        &lt;ColumnDefinition Width="*"/>
    &lt;/Grid.ColumnDefinitions>
    &lt;Button x:Name="backButton" Click="GoBack" IsEnabled="{Binding Frame.CanGoBack, ElementName=pageRoot}" Style="{StaticResource BackButtonStyle}"/>
    &lt;<TextBlock x:Name="pageTitle" Grid.Column="1" Text="Basic Page 1" Style="{StaticResource PageHeaderTextStyle}"/>
&lt;/Grid>

&lt;!-- Add this -->
&lt;Grid Grid.Row ="1">
    &lt;Button Content="Go to Page 2" Height="130" Width="200" HorizontalAlignment="Center" VerticalAlignment="Center" Click="Go_to_Page2"></Button>
&lt;/Grid>
</pre>

并编写Go_to_Page2的代码：

<pre class="prettyprint linenums:1">
void NavigationTest::BasicPage1::Go_to_Page2(Platform::Object^ sender, Windows::UI::Xaml::RoutedEventArgs^ e)
{
	this->Frame->Navigate(TypeName(BasicPage2::typeid, nullptr));
}
</pre>
咦，这次怎么变成两个参数了？

第二个参数是在页面之间传递数据时使用的，现在我们先不传任何东西过去。

编译，运行。
<img alt="" src="http://images.cy198706.com/Programming/20121221173809.jpg" />

<img alt="" src="http://images.cy198706.com/Programming/20121221173835.jpg" />

<img alt="" src="http://images.cy198706.com/Programming/20121221173856.jpg" />

可以使用页面上方的“返回”按钮来返回上一个页面。有人说了，我不想用页面上方的按钮，我想自己做个按钮回去行不？

行。

在BasicPage2.xaml中添加一个Grid和一个按钮

<pre class="prettyprint linenums:1">
&lt;Grid Grid.Row="1">
    &lt;Button Content="Go Back" Height="130" Width="200" HorizontalAlignment="Center" VerticalAlignment="Center"  Click="Go_Back_to_Page1"/>
&lt;/Grid>
</pre>
为Go_Back_to_Page1编写代码：

<pre class="prettyprint linenums:1">
this->Frame->GoBack();
</pre>

看，多方便。编译运行，就可以使用BasicPage2上的按钮返回了。

这儿要注意一个问题，只有当页面可以GoBack时，“返回”按钮才会出现。对程序做如下处理：

在App.xaml.cpp中，在OnLaunched函数中，将MainPage::typeid修改为BasicPage1::typeid，同样地，要添加#include "BasicPage1.xaml.h"，然后编译，运行，就看不到BasicPage1里的“返回”按钮了。

<img alt="" src="http://images.cy198706.com/Programming/20121221175049.jpg" />


## 在导航时传递数据

之前所做的，只是让页面“跳来跳去”，没有什么实际意义，当一个应用包含多个页面时，这些页面经常需要共享信息。现在我们来让BasicPage1传递信息给BasicPage2。

在BasicPage1中添加一个TextBox：

<pre class="prettyprint linenums:1">
&lt;TextBox x:Name="textbox1"  HorizontalAlignment="Left" Margin="319,249,0,0" TextWrapping="Wrap" VerticalAlignment="Top" Height="77" Width="180" FontSize="24"/>
</pre>

在BasicPage2中添加一个TextBlock：

<pre class="prettyprint linenums:1">
&lt;TextBlock x:Name="block1" HorizontalAlignment="Left" Margin="313,249,0,0" TextWrapping="Wrap" Text="" VerticalAlignment="Top" Height="66" Width="134" FontSize="24"/>
</pre>

将BasicPage1的按钮事件修改成如下代码：

<pre class="prettyprint linenums:1">
void NavigationTest::BasicPage1::Go_to_Page2(Platform::Object^ sender, Windows::UI::Xaml::RoutedEventArgs^ e)
{
	TypeName pageType2 = {BasicPage2::typeid->FullName, TypeKind::Custom};

	this->Frame->Navigate(pageType2, textbox1->Text);
}
</pre>
为BasicPage2类添加一个重写函数：

<pre class="prettyprint linenums:1">
// BasicPage2.xaml.h
protected:
    virtual void OnNavigatedTo(Windows::UI::Xaml::Navigation::NavigationEventArgs^ e) override;
</pre>
这个函数将会在导航至BasicPage2时被执行。记得要加上override标识，否则在编译时会报错。

实现这个函数：

<pre class="prettyprint linenums:1">
void NavigationTest::BasicPage2::OnNavigatedTo( Windows::UI::Xaml::Navigation::NavigationEventArgs^ e )
{
	String^ str = (String^)e->Parameter;

	block1->Text = str;
}
</pre>

好了，编译运行，在BasicPage1的TextBox中输入“This is good!”，然后点击按钮，就会看到，在BasicPage2中显示出了传递过来的信息。

<img alt="" src="http://images.cy198706.com/Programming/20121224100536.jpg" />

<img alt="" src="http://images.cy198706.com/Programming/20121224100547.jpg" />

如果你点击一下BasicPage2的“返回按钮”，你会发现，BasicPage1的TextBox已经被清空了，很多时候这会造成一些麻烦。我们可以使用 NavigationCacheMode 属性来指定对一个页面进行缓存。这样在切换回该页面时，可以保留上次的状态。

在 BasicPage1的构造函数中，将 NavigationCacheMode 设置为 Enabled。可以在构造函数中开启此模式。

<pre class="prettyprint linenums:1">
BasicPage1::BasicPage1()
{
	InitializeComponent();

	this->NavigationCacheMode = Windows::UI::Xaml::Navigation::NavigationCacheMode::Enabled;
</pre>

我们每次加载页面时，都必须考虑两种视图状态：纵向模式、辅屏视图、填充视图或横向模式，而且还必须考虑是通过用户操作（前进或后退）还是通过系统恢复（系统在过去某个时刻终止应用后将其恢复）来调用导航。
