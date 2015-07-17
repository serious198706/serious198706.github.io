---
layout: post
title: "Windows Store Apps 开发手札 - 9（添加应用栏AppBar）"
description: 
headline: 
modified: 2012-12-21
category: development
tags: [windows, windows store apps]
imagefeature: 
mathjax: 
chart: 
comments: true
featured: true
---

在第7节中提到过，在Windows Store App中，有一种新的控件叫做AppBar。AppBar为Windows Store App增色不少。

使用应用栏可以显示导航、命令以及始终隐藏不需要使用的工具。 你可以将AppBar置于页面顶部、底部、也可以同时放置在顶部和底部。在默认情况下，AppBar是隐藏的，当用户单击右键，按 Windows+Z，或从屏幕的顶部或底部边缘轻扫时，可显示或关闭AppBar。你可以通过编程方式将其设置为当用户做选择或与应用交互时显示。这点在Surface RT上操作感很强，用手指在屏幕边缘一扫，就能划出AppBar，感觉很棒。

我们假设你已经会利用Visual Studio添加控件并处理控件事件了。如果还不会，查看第7节。[传送门]("http://cy198706.com/blog/tech/windows-store-apps-developing-7-adding-control-and-handel-events/")

### 添加AppBar-

要向你的应用添加应用栏，请将一个 AppBar 控件指定给 Page 的 TopAppBar 或 BottomAppBar 属性。

使用顶部应用栏可在你的应用中显示导航。使用按钮应用栏以显示命令和工具。当然了，顶部AppBar也不是不可以添加些命令和工具。

接下来这个小例子显示了一个拥有两列，分别包含3个按钮和2个按钮的BottomAppBar。将下面这段代码加入到MainPage.xaml的Grid元素的外面，即与Grid元素同级：

<pre class="prettyprint linenums:1">
&lt;Grid&gt;
...
&lt;/Grid&gt;
&lt;Page.BottomAppBar&gt;
    &lt;AppBar x:Name="bottomAppBar" Padding="10,0,10,0"&gt;
        &lt;Grid&gt;
            &lt;StackPanel Orientation="Horizontal" HorizontalAlignment="Left"&gt;
                &lt;Button Style="{StaticResource EditAppBarButtonStyle}" Click="Button_Click"/&gt;
                &lt;Button Style="{StaticResource RemoveAppBarButtonStyle}" Click="Button_Click"/&gt;
                &lt;Button Style="{StaticResource AddAppBarButtonStyle}" Click="Button_Click"/&gt;
            &lt;/StackPanel&gt;
            &lt;StackPanel Orientation="Horizontal" HorizontalAlignment="Right"&gt;
                &lt;Button Style="{StaticResource RefreshAppBarButtonStyle}" Click="Button_Click"/&gt;
                &lt;Button Style="{StaticResource HelpAppBarButtonStyle}" Click="Button_Click"/&gt;
            &lt;/StackPanel&gt;
        &lt;/Grid&gt;
    &lt;/AppBar&gt;
&lt;/Page.BottomAppBar&gt;
</pre>

效果如下：

<img alt="" src="http://images.cy198706.com/Programming/20121220173556.jpg" />

我们可以看到，5个按钮分别使用了EditAppBarButtonStyle、RemoveAppBarButtonStyle、AddAppBarButtonStyle、RefreshAppBarButtonStyle和HelpAppBarButtonStyle样式。这5种样式是Windows Store App自带的样式，但默认是“关闭”的，需要“开启”一下。

找到Common文件夹里的StandardStyles.xaml，并搜索如下内容：

<pre class="prettyprint linenums:1">
Standard AppBarButton Styles for use with Button and ToggleButton
</pre>
会看到，下面很多很多很多的样式，都被注释掉了。我们就象征性地反注释掉一部分吧，大约去掉两三对注释符号，上述的5种样式就都可以使用了。

### 使用编程方式打开AppBar

除了使用硬件或者交互方式打开AppBar之外，我们也可以使用编程方式打开AppBar，比如点击一个按钮，就出现AppBar。我们可以通过将 AppBar的IsOpen 属性设置为 true 以编程方式打开应用栏。 当你执行此操作时，仅会打开你为其设置属性值的特定应用栏（顶部或底部）。

设置IsOpen属性有两种方式：

1. 在 XAML 中将 IsOpen 属性设置为 true。

 <pre class="prettyprint linenums:1">
 &lt;AppBar IsOpen="True"&gt;
    &lt;Grid&gt;
        &lt;StackPanel Orientation="Horizontal" HorizontalAlignment="Right"&gt;
            Button Style="{StaticResource PreviousAppBarButtonStyle}"
                    &lt;Click="Button_Click"/&gt;
            Button Style="{StaticResource NextAppBarButtonStyle}"
                    &lt;Click="Button_Click"/&gt;
        &lt;/StackPanel&gt;
    &lt;/Grid&gt;
 &lt;/AppBar&gt;
</pre>
 这样，一打开这个页面，就会出现AppBar了。

2. 在代码中将 IsOpen 属性设置为 true。要在代码中引用AppBar，必须为其命名（x:Name）。

 <pre class="prettyprint linenums:1">
 &lt;Page.TopAppBar&gt;
    &lt;AppBar x:Name="topAppBar"&gt;
        &lt;Grid&gt;
            &lt;StackPanel Orientation="Horizontal" HorizontalAlignment="Right"&gt;
                &lt;Button Style="{StaticResource SaveAppBarButtonStyle}"
                        Click="Button_Click"/&gt;
                &lt;Button Style="{StaticResource UploadAppBarButtonStyle}"
                        Click="Button_Click"/&gt;
            &lt;/StackPanel&gt;
        &lt;/Grid&gt;
    &lt;/AppBar&gt;
&lt;/Page.TopAppBar&gt;
</pre>

如果我们想要单击按钮出现AppBar，就在代码中添加如下事件处理：

<pre class="prettyprint linenums:1">
// MainPage.xaml.h
private:
    void OpenButton_Click(Platform::Object^ sender, Windows::UI::Xaml::RoutedEventArgs^ e);

// MainPage.xaml.cpp
void AppBarSample::MainPage::OpenButton_Click(Platform::Object^ sender, Windows::UI::Xaml::RoutedEventArgs^ e)
{
    topAppBar->IsOpen = true;
}
</pre>
### 粘滞AppBar

默认情况下，当用户与你的应用在AppBar外部的任何位置交互时，会解除AppBar。有些时候，我们需要让AppBar一直显示，以达到“省事”的目的。

要保持命令可见，你可以通过将 IsSticky 属性设置为 true 来更改解除模式。当AppBar为粘滞时，仅当用户单击右键，按 Windows+Z，或从屏幕的顶部或底部边缘轻扫时，才会解除AppBar。

同样地，设置IsSticky时也有两种方式：

1. 在 XAML 中将 IsSticky 属性设置为 true。

<pre class="prettyprint linenums:1">
 &lt;AppBar IsSticky="True"&gt;
    &lt;Grid&gt;
        &lt;StackPanel Orientation="Horizontal" HorizontalAlignment="Right"&gt;
        &lt;Button Style="{StaticResource HelpAppBarButtonStyle}"
                Click="Button_Click"/&gt;
        &lt;/StackPanel&gt;
    &lt;/Grid&gt;
&lt;/AppBar&gt;
</pre>
2. 在代码中将 IsSticky 属性设置为 true。要在代码中引用AppBar，必须为其命名（x:Name）。

<pre class="prettyprint linenums:1">
 &lt;Page.BottomAppBar&gt;
    &lt;AppBar x:Name="bottomAppBar"&gt;
        &lt;Grid&gt;
            &lt;StackPanel Orientation="Horizontal" HorizontalAlignment="Right"&gt;
                &lt;Button Style="{StaticResource HelpAppBarButtonStyle}"
                        Click="Button_Click"/&gt;
            &lt;/StackPanel&gt;
        &lt;/Grid&gt;
    &lt;/AppBar&gt;
&lt;/Page.BottomAppBar&gt;
</pre>
代码中：

<pre class="prettyprint linenums:1">
// MainPage.xaml.h
private:
    void StickyButton_Click(Platform::Object^ sender, Windows::UI::Xaml::RoutedEventArgs^ e);

// MainPage.xaml.cpp
void AppBarSample::MainPage::StickyButton_Click
    (Platform::Object^ sender, Windows::UI::Xaml::RoutedEventArgs^ e)
{
    bottomAppBar->IsSticky = true;
}
</pre>

### 特殊情况

在Windows Store App中，也有类似MFC中HTMLViewDlg的WebView，可以显示网页并进行浏览。如果你的某一个页面使用了WebView，那恭喜了，你中了微软的“圈套”。

如果你使用了WebView，还使用了AppBar，比如说我想刷新页面，那么在这种情况下，WebView会遮挡住AppBar，就如下图所示：

<img alt="" src="http://images.cy198706.com/Programming/20121221092727.jpg" />

那遇见这种情况该肿么办呢？好在微软也提供了解决办法。

我们可以通过处理应用栏的 Opened 和 Closed 事件来解决此问题。当用户打开应用栏时，你可以创建一个 WebViewBrush 并将 WebView 设置为其源。然后调用 Redraw 方法，此操作会获取 WebView 的可视快照。然后，你可以使用该画笔填充 Rectangle 并隐藏 WebView。应用栏即可在 Rectangle 之上显示其内容。当用户关闭应用栏时，我们不再需要模拟的 WebView，因此我们将内容恢复为 Closed 事件处理程序中的原状。即截个屏，并填充到一个矩形框内，让你误以为这是网页。。

具体操作如下：

新建一个页面WebView.xaml，然后在XAML文件中加入如下代码：

<pre class="prettyprint linenums:1">
&lt;Page&gt;
    &lt;Page.TopAppBar&gt;
        &lt;AppBar x:Name="topAppBar" Opened="AppBar_Opened" Closed="AppBar_Closed"&gt;
            &lt;Grid&gt;
                &lt;StackPanel Orientation="Horizontal" HorizontalAlignment="Left"&gt;
                    &lt;Button Style="{StaticResource RefreshAppBarButtonStyle}" Click="Refresh_Click"/&gt;
                    &lt;Button Style="{StaticResource SearchAppBarButtonStyle}"/&gt;
                &lt;/StackPanel&gt;
                &lt;StackPanel Orientation="Horizontal" HorizontalAlignment="Right"&gt;
                    &lt;Button Style="{StaticResource EditAppBarButtonStyle}"/&gt;
                    &lt;Button Style="{StaticResource HelpAppBarButtonStyle}"/&gt;
                &lt;/StackPanel&gt;
            &lt;/Grid&gt;
        &lt;/AppBar&gt;
    &lt;/Page.TopAppBar&gt;

    &lt;Grid Background="{StaticResource ApplicationPageBackgroundThemeBrush}"&gt;
        &lt;Border BorderBrush="Gray" BorderThickness="2" Margin="100,20,100,20"&gt;
            &lt;Grid&gt;
                &lt;WebView x:Name="contentView" Source="http://www.baidu.com"/&gt;
                &lt;Rectangle x:Name="contentViewRect"/&gt;
            &lt;/Grid&gt;
        &lt;/Border&gt;
    &lt;/Grid&gt;
&lt;/Page&gt;
</pre>
可以看到，在WebView的下方加入了一个矩形框contentViewRect。

然后就是代码实现“瞒天过海之计”：

<pre class="prettyprint linenums:1">
// WebView.xaml.h
private:
    void AppBar_Opened(Platform::Object^ sender, Platform::Object^ e);
    void AppBar_Closed(Platform::Object^ sender, Platform::Object^ e);
    void Refresh_Click(Platform::Object^ sender, Windows::UI::Xaml::RoutedEventArgs^ e);

void AppBarSample::WebViewPage::AppBar_Opened(Platform::Object^ sender, Platform::Object^ e)
{
    WebViewBrush^ wvb = ref new WebViewBrush();
    wvb->SourceName = "contentView";      // 将WebViewBrush的源指向WebView
    wvb->Redraw();
    contentViewRect->Fill = wvb;                 // 将矩形框填充为刚才WebView的“截图”
    contentView->Visibility = Windows::UI::Xaml::Visibility::Collapsed;      // 隐藏WebView
}

void AppBarSample::WebViewPage::AppBar_Closed(Platform::Object^ sender, Platform::Object^ e)
{
    contentView->Visibility = Windows::UI::Xaml::Visibility::Visible;           // 显示WebView
    contentViewRect->Fill = ref new SolidColorBrush(Windows::UI::Colors::Transparent);      // 将矩形框清空
}

void AppBarSample::WebViewPage::Refresh_Click(Platform::Object^ sender, Windows::UI::Xaml::RoutedEventArgs^ e)
{
    contentView->Navigate(ref new Uri("http://www.baidu.com"));      // 记住一定要加http:// ，否则会崩溃，原因是“www.baidu.com不是绝对有效的URI”
    topAppBar->IsOpen = false;                 // 上面讲过的，利用IsOpen属性关闭AppBar，顺道触发AppBar_Closed事件
}
</pre>
效果就如下了：

<img alt="" src="http://images.cy198706.com/Programming/20121221101131.jpg" />

### 特殊情况2

我们好像已经习惯了“特殊情况”了。

大多数时候，我们的程序都是多页面的，那么可否在多页面中共享同一个AppBar呢？答案是可以。

但是这部分涉及到页面的导航，又是一个很重要的部分，我们在后面拿出来单独讲。

关于AppBar，还有很多内容，比如[自定义按钮样式](http://msdn.microsoft.com/zh-cn/library/windows/apps/xaml/jj662743.aspx)、[为AppBar添加菜单](http://msdn.microsoft.com/zh-cn/library/windows/apps/xaml/jj150602.aspx)、[以不同视图使用AppBar](http://msdn.microsoft.com/zh-cn/library/windows/apps/xaml/jj662742.aspx)等，便不再叙述了，大家自行进入传送门观看吧。
