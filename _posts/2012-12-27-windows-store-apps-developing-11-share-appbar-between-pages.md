---
layout: post
title: "Windows Store Apps 开发手札 - 11（如何跨页面共享应用栏）"
description: 
headline: 
modified: 2012-12-27
category: development
tags: [windows, windows store apps]
imagefeature: 
mathjax: 
chart: 
comments: true
featured: true
---

这篇文章的基础是第9篇（AppBar）与第10篇（页面间的导航）。

很多时候，多个页面之间是需要共享同一个AppBar的，一是方便调用，二是节省资源。让我们来看一下如何共享AppBar。

<!-- more -->

新建一个工程，添加两个空白页面，分别命名为Page1和Page2。
 
我们首先需要一个根页面来承载共享的AppBar，和一个Frame。Frame用于显示各个页面（Page）。让我们就把MainPage当作承载的页面吧。

在MainPage.xaml中添加如下代码：

<pre class="prettyprint linenums:1 lang-xml">
&lt;Page.Resources&gt;
    &lt;Style x:Key="BackAppBarButtonStyle" TargetType="Button" BasedOn="{StaticResource AppBarButtonStyle}"&gt;
        &lt;Setter Property="Content" Value="&#xE0C4;"/&gt;
        &lt;Setter Property="AutomationProperties.AutomationId" Value="SuperstarButton"/&gt;
        &lt;Setter Property="AutomationProperties.Name" Value="Superstar"/&gt;
    &lt;/Style&gt;
&lt;/Page.Resources&gt;

&lt;Page.TopAppBar&gt;
    &lt;AppBar x:Name="globalAppBar" Padding="10,0,10,0"&gt;
        &lt;Grid&gt;
            &lt;StackPanel x:Name="leftCommandPanel"
                        Orientation="Horizontal" HorizontalAlignment="Left"&gt;
                &lt;Button x:Name="Back" Style="{StaticResource BackAppBarButtonStyle}"
                        AutomationProperties.Name="Back"
                        Click="Back_Click"/&gt;
            &lt;/StackPanel&gt;
            &lt;StackPanel x:Name="rightCommandPanel"
                        Orientation="Horizontal" HorizontalAlignment="Right"&gt;
                &lt;Button x:Name="page1Button" Content="1"
                        Style="{StaticResource AppBarButtonStyle}"
                        AutomationProperties.Name="Page 1"
                        Click="Page1Button_Click"/&gt;
                &lt;Button x:Name="page2Button" Content="2"
                        Style="{StaticResource AppBarButtonStyle}"
                        AutomationProperties.Name="Page 2"
                        Click="Page2Button_Click"/&gt;
            &lt;/StackPanel&gt;
        &lt;/Grid&gt;
    &lt;/AppBar&gt;
&lt;/Page.TopAppBar&gt;
&lt;Grid Background="{StaticResource ApplicationPageBackgroundThemeBrush}"&gt;
    &lt;Frame x:Name="frame1"/&gt;
&lt;/Grid&gt;
</pre>
<br />
可以看到，我们定义了一个BackAppBarButtonStyle的样式（不定义此样式，使用StandardStyles里的样式也可以），定义了一个含有三个按钮的顶端AppBar，一个用于显示页面的Frame。

为MainPage类写下如下代码：

<pre class="prettyprint linenums:1 lang-cpp">
// MainPage.xaml.h
// private:
//     void Back_Click(Platform::Object^ sender, Windows::UI::Xaml::RoutedEventArgs^ e);
//     void Page1Button_Click(Platform::Object^ sender, Windows::UI::Xaml::RoutedEventArgs^ e);
//     void Page2Button_Click(Platform::Object^ sender, Windows::UI::Xaml::RoutedEventArgs^ e);

//     Page^ rootPage;

void MainPage::OnNavigatedTo(NavigationEventArgs^ e)
{
	(void) e;	// Unused parameter
	rootPage = safe_cast<Page^>(e->Parameter);
	frame1->Navigate(TypeName(BasicPage1::typeid), this);
}


void NavigationTest::MainPage::Back_Click(Platform::Object^ sender, Windows::UI::Xaml::RoutedEventArgs^ e)
{
	if(frame1->CanGoBack)
	{
		frame1->GoBack();
	}
	else if(rootPage && rootPage->Frame->CanGoBack)
	{
		rootPage->Frame->GoBack();
	}
}


void NavigationTest::MainPage::Page1Button_Click(Platform::Object^ sender, Windows::UI::Xaml::RoutedEventArgs^ e)
{
	frame1->Navigate(TypeName(BasicPage1::typeid), this);
}


void NavigationTest::MainPage::Page2Button_Click(Platform::Object^ sender, Windows::UI::Xaml::RoutedEventArgs^ e)
{
	frame1->Navigate(TypeName(BasicPage2::typeid), this);
}
</pre>

给Page1.xaml添加如下代码：

<pre class="prettyprint linenums:1 lang-cpp">
&lt;Grid>
    &lt;TextBlock x:Name="pageTitle" Grid.Column="1" Text="Page 1" Style="{StaticResource PageHeaderTextStyle}" HorizontalAlignment="Center"/>
&lt;/Grid>
</pre>

为Page1类写下如下代码：

<pre class="prettyprint linenums:1 lang-cpp">
// Page1.xaml.h
//	protected:
//		virtual void OnNavigatedTo(Windows::UI::Xaml::Navigation::NavigationEventArgs^ e) override;
//	private:
//		Page^ rootPage;

// Page1.xaml.cpp
void NavigationTest::BasicPage1::OnNavigatedTo( Windows::UI::Xaml::Navigation::NavigationEventArgs^ e )
{
	rootPage = safe_cast<Page^>(e->Parameter);
}
</pre>

对Page2做相同处理。

完成后就可以编译运行了！效果如下：

<img alt="" src="http://i1352.photobucket.com/albums/q645/cy198706/Programming/20121224172852.jpg" />

<img alt="" src="http://i1352.photobucket.com/albums/q645/cy198706/Programming/20121224172916.jpg" />
<br />

在Page2，可以通过AppBar左侧的“返回”按钮返回Page1。这样就实现了两个页面共享同一个AppBar。