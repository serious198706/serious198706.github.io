---
layout: post
title: "Windows Store Apps 开发手札 – 13（启动、恢复以及执行多任务）"
description: 
headline: 
modified: 2013-01-07
category: development
tags: [windows, windows store apps]
imagefeature: 
mathjax: 
chart: 
comments: true
featured: true
--- 


Windows Store App采用了新的多任务开发机制。

在Windows Store App的开发过程中，多任务通常用于应用程序之间切换时，后台应用的挂起、切换回某个应用时状态和数据的恢复、以及后台应用可能会有的异步操作。在某个应用被切换至后台时（通常是用户进行了某种操作，比如Alt+Tab，比如从Windows Store App应用切换到桌面应用），应用会暂时被挂起，再切换回来时，会恢复切换前的状态，除非应用有后台的异步操作，否则该应用是被完全“停止运行”的。

### 了解几种状态

当你激活应用时要发生的事件取决于应用的执行历史记录。

需要考虑五种情况。这些情况分别对应于 [Windows::ActivationModel::Activation::ApplicationExecutionState]("msdn.microsoft.com/zh-CN/library/windows/apps/windows.applicationmodel.activation.applicationexecutionstate") 枚举的值。

### 如何挂起应用 

<pre class="prettyprint linenums:1 lang-cpp">
using namespace Windows::ApplicationModel;
using namespace Windows::ApplicationModel::Activation;
using namespace Windows::Foundation;
using namespace Windows::UI::Xaml;
using namespace AppName;

MainPage::MainPage()
{
   InitializeComponent();
   Application::Current->Suspending += 
       ref new SuspendingEventHandler(this, &MainPage::App_Suspending);
}
void MainPage::App_Suspending(Object^ sender, SuspendingEventArgs^ e)
{
    // TODO: This is the time to save app data in case the process is terminated
}
</pre>

### 如何恢复应用-

恢复应用一般会用在当系统恢复你的应用时，刷新显示的内容。
1.  注册 Resuming事件处理程序
注册以处理 Resuming事件，该事件指示用户从你的应用切换到桌面或其他应用，而后又切换回你的应用。

<pre class="prettyprint linenums:1 lang-cpp">
MainPage::MainPage()
{
    InitializeComponent();
    Application::Current->Resuming += 
        ref new EventHandler<Platform::Object^>(this, &MainPage::App_Resuming);
}
void MainPage::App_Resuming(Object^ sender, Object^ e)
{
    // TODO: Refresh network data
}
</pre>

### 如何激活应用

<pre class="prettyprint linenums:1 lang-cpp">
using namespace Windows::ApplicationModel::Activation;
using namespace Windows::Foundation;
using namespace Windows::UI::Xaml;
using namespace AppName;

void App::OnLaunched(LaunchActivatedEventArgs^ args)
{
   EnsurePageCreatedAndActivate();
}

// Creates the MainPage if it isn't already created.  Also activates
// the window so it takes foreground and input focus.
void App::EnsurePageCreatedAndActivate()
{
    if (_mainPage == nullptr)
    {
        // Save the MainPage for use if we get activated later
        _mainPage = ref new MainPage();
    }
    Window::Current->Content = _mainPage;
    Window::Current->Activate();
}
void App::OnLaunched(Windows::ApplicationModel::Activation::LaunchActivatedEventArgs^ args)
{
   if (args->PreviousExecutionState == ApplicationExecutionState::Terminated ||
       args->PreviousExecutionState == ApplicationExecutionState::ClosedByUser)
   {
      // TODO: Populate the UI with the previously saved application data
   }
   else
   {
      // TODO: Populate the UI with defaults
   }

   EnsurePageCreatedAndActivate();
}
</pre>

### 创建和注册后台任务

#### 创建后台任务类

<pre class="prettyprint linenums:1 lang-cpp">
//
// ExampleBackgroundTask.h
//
#pragma once

using namespace Windows::ApplicationModel::Background;

namespace Tasks
{
    public ref class ExampleBackgroundTask sealed : public IBackgroundTask
    {

    public:
        ExampleBackgroundTask();
        virtual void Run(IBackgroundTaskInstance^ taskInstance);
    };
}

//
// ExampleBackgroundTask.cpp
//
#include "ExampleBackgroundTask.h"

using namespace Tasks;

void ExampleBackgroundTask::Run(IBackgroundTaskInstance^ taskInstance)
{

}
void ExampleBackgroundTask::Run(IBackgroundTaskInstance^ taskInstance)
{
    BackgroundTaskDeferral^ deferral = taskInstance->GetDeferral();
   
    //
    // TODO: Modify the following line of code to call a real async function.
    //       Note that the task<void> return type applies only to async
    //       actions. If you need to call an async operation instead, replace
    //       task<void> with the correct return type.
    //
    task<void> myTask(ExampleFunctionAsync());
   
    myTask.then([=] () {
        deferral->Complete();
    });
}
</pre>

#### 注册要运行的后台服务

<pre class="prettyprint linenums:1 lang-cpp">
boolean taskRegistered = false;
Platform::String exampleTaskName = "ExampleBackgroundTask";

auto iter = BackgroundTaskRegistration::AllTasks->First();
auto hascur = iter->HasCurrent;

while (hascur)
{
    auto cur = iter->Current->Value;

    if(cur->Name == name)
    {
        taskRegistered = true;
        break;
    }

    hascur = iter->MoveNext();
}
auto builder = ref new BackgroundTaskBuilder();

builder->Name = exampleTaskName;
builder->TaskEntryPoint = "Task.ExampleBackgroundTask";
builder->SetTrigger(ref new SystemTrigger(SystemTriggerType::TimeZoneChange, false));
BackgroundTaskRegistration^ task = builder->Register();
</pre>

#### 使用事件处理程序处理后台任务完成

<pre class="prettyprint linenums:1 lang-cpp">
void ExampleBackgroundTask::OnCompleted(BackgroundTaskRegistration^ task, BackgroundTaskCompletedEventArgs^ args)
{
    auto settings = ApplicationData::Current->LocalSettings->Values;
    auto key = task->TaskId.ToString();
    auto message = dynamic_cast<String^>(settings->Lookup(key));
    UpdateUIExampleMethod(message);
}
auto uiDelegate = [this]()
{
    RegisterButton->IsEnabled = !BackgroundTask::TimeTriggeredTaskRegistered;
    UnregisterButton->IsEnabled = BackgroundTask::TimeTriggeredTaskRegistered;
    Progress->Text = BackgroundTask::TimeTriggeredTaskProgress;
    Status->Text = BackgroundTask::GetBackgroundTaskStatus(TimeTriggeredTaskName);
};
auto handler = ref new Windows::UI::Core::DispatchedHandler(uiDelegate, Platform::CallbackContext::Any);

Dispatcher->RunAsync(CoreDispatcherPriority::Normal, handler);
task->Completed += ref new BackgroundTaskCompletedEventHandler(this, &MainPage::OnCompleted);
</pre>

#### 在应用清单中声明你的应用使用后台任务

<pre class="prettyprint linenums:1 lang-cpp">
&lt;Extensions&gt;
    <Extension Category="windows.backgroundTasks" EntryPoint="Tasks.ExampleBackgroundTask">
        <BackgroundTasks>
            <Task Type="systemEvent" />
        </BackgroundTasks>
    </Extension>
</Extensions>
</pre>

### 退出应用

在 Windows Store App 的生存周期中，有五种状态，它们分别是：

- NotRunning
- Terminated
- ClosedByUser
- Suspended
- Running

下面是显示 Windows 如何确定应用的执行状态的图。在该图中，蓝色矩形表示应用未加载到系统内存中。白色句柄表示应用位于内存中。虚线弧是在没有通知正在运行的应用的情况下发生的更改。实线弧是包含应用通知的操作。


执行状态取决于应用的历史记录。例如，当用户在安装应用或重新启动 Windows 后首次启动应用时，之前的执行状态为 NotRunning，激活后的状态为 Running。激活事件参数包括一个 PreviousExecutionState 属性，该属性告诉你应用在激活之前处于哪种状态。

如果用户切换到其他应用或者系统进入电量不足操作模式，则 Windows 通知应用它正在被挂起。此时，你必须保存导航状态以及表示用户会话的所有用户数据。你还应该释放独占的系统资源，如打开的文件和网络连接。

Windows 允许应用在 5 秒之内处理 Suspending 事件。如果 Suspending 事件处理程序未在该时间内完成，那么 Windows 假定该应用已停止响应并且终止该应用。

应用响应 Suspending 事件之后，其状态为 Suspended。如果用户切换回该应用，那么 Windows 恢复该应用并允许该应用再次运行。

应用被挂起之后，Windows 可能会终止该应用（没有通知）。例如，如果系统资源不足，那么系统可能会决定回收被挂起的应用占用的资源。

如果用户在 Windows 终止该应用之后启动该应用，那么在激活时该应用之前的执行状态为 Terminated。

你可以通过之前的执行状态确定你的应用是否需要还原其在上次挂起时保存的数据，或者是否必须加载应用的默认数据。通常，如果应用崩溃或者用户将其关闭，那么重新启动应用应该会将用户带回应用的默认初始导航状态。当应用确定它在终止后被激活时，它应该加载它在挂起期间保存的应用数据，使应用显示为它在挂起时的状态。

注意:当应用处于挂起状态但尚未被终止时，你可以恢复应用，而不还原任何状态。应用将仍然位于内存中。在这种情况下，你可能需要重新获取资源并更新 UI 以反映在挂起应用时对环境所进行的任何更改。

我们先看一下如何挂起应用吧。在Windows8中，当用户将应用移下屏幕时，Windows 8 会在内存中挂起应用。这样可允许其他应用在前台运行。当应用挂起时，它驻留在内存中，并且 Windows 已停止其运行。当挂起时，我们的应用应该保存用户的重要数据，以便在恢复时还原上次的状态，不至于丢失数据。

基本上，挂起应用可以分为三个步骤：

1. 向 Suspending 事件处理程序注册

  注册以处理 Suspending 事件，该事件指示在系统挂起你的应用之前，应用应该保存其应用程序数据。
2. 在挂起之前保存应用程序数据

  当你的应用处理 Suspending 事件时，它将有机会将其重要的应用程序数据保存到处理程序函数中。应用应该使用 LocalSettings 存储 API 来同步保存简单的应用程序数据。
3. 释放独占资源和文件句柄
 
  当你的应用处理 Suspending 事件时，它还将有机会释放独占资源和文件句柄。独占资源的示例为摄相机、I/O 设备、外部设备以及网络资源。显式释放独占资源和文件句柄有助于确保：你的应用未使用它们时其他应用可以访问它们。当在终止后又激活应用时，它应该打开其独占资源和文件句柄。

 每当用户切换到桌面或其他应用时，系统都会挂起你的应用。每当用户切回到你的应用时，系统就会恢复你的应用。当系统恢复你的应用时，你的变量和数据结构的内容与系统将你的应用挂起之前的内容相同。系统会将你的应用完全恢复到你离开时的状态，使用户感觉你的应用好像一直在后台运行一样。

  当你的应用被挂起后，系统会尝试将你的应用及其数据保留在内存中。但是，如果系统没有资源将你的应用保存在内存里，则将终止你的应用。当用户切换回已终止的挂起应用时，该应用会发送 Activated 事件，且应该在其 OnLaunched 方法中还原其应用程序数据。
当终止应用时系统不会通知应用，因此当挂起应用时，你的应用必须保存其应用程序数据并释放独占资源和文件句柄，并且当在终止后又激活应用时还原这些内容。

4. 挂起之后刷新显示的内容
当你的应用处理 Resuming事件时，它将有机会刷新其显示的内容。由于该事件并非在 UI 线程中引发，因此必须使用调度程序来将向 UI 中注入更新。

  如果你的应用没有任何需要刷新的显示内容，则它无需处理 Resuming 事件。

在首次安装完成第一次进入应用、应用被关闭（或崩溃）后再次开启时，会用到激活操作。

1. 替代启动处理程序
  
  当激活了应用时，无论任何原因，系统都会发送 Activated 事件。有关激活类型的列表，请参阅  枚举。
Windows.UI.Xaml.Application 类定义了为处理各种不同的激活类型而可以替代的一些方法。对于其中一些激活类型，有特定的方法可以替代。对于其他激活类型，则替代 OnActivated 方法。

  替代 OnLaunched 方法。无论何时当用户启动应用时都会调用此方法。LaunchActivatedEventArgs 参数包含你的应用之前的状态和激活参数。

2. 应用挂起而后终止时还原应用数据

  当用户切换到终止的应用时，系统将发送 Activated 事件，其中，（获取正在激活该应用程序的原因。） 设置为 Launch，（在应用程序激活前获取该应用程序的执行状态。）设置为 Terminated 或 ClosedByUser。应用应该加载其保存的应用程序数据并刷新其显示的内容。
如果 PreviousExecutionState 值为 NotRunning则应用无法成功保存其应用程序数据，应用必须如同初始启动一样重新启动。

这一小节会创建一个后台任务类并注册以便运行该任务，从而在应用不在前台运行时提供功能。 1. 你可以通过编写用于实现 IBackgroundTask 接口的类来在后台运行代码。在使用诸如 SystemTrigger 或 MaintenanceTrigger 等触发器触发特定事件时，将运行该代码。
以下步骤介绍如何编写实现 IBackgroundTask接口的一个新类。
创建一个新项目 BackgoundRunningTest，接着为后台任务添加一个新的空类，并导入 Windows.ApplicationModel.Background 命名空间。
创建一个实现 IBackgroundTask接口的一个新类。Run 方法是所需的入口点，触发指定的事件时将调用该方法；每个后台任务中都需要此方法。

注意：后台任务类本身--及后台任务项目中的所有其他类--都需要是具有 sealed 属性的 public 类。

2. 如果你在后台任务中运行任何异步代码，则你的后台任务需要使用延迟。如果不使用延迟，则后台任务进程可能会意外终止（如果 Run 方法在异步方法调用完成之前完成）。
调用异步方法之前，在 Run 方法中请求延迟。
将延迟保存为全局变量，这样异步方法可以对其进行访问。
完成异步代码之后声明延迟完成。
1. 通过在 BackgroundTaskRegistration.AllTasks 属性中迭代，找出后台任务是否已注册。此步骤非常重要；如果应用不检查现有后台任务注册，则它可能会多次注册该任务，这会导致性能问题和工作结束前超出任务的最大可用 CPU 时间（后台任务使用CPU是有限制的。使后台执行最少可确保前台应用的最佳用户体验以及最佳电池寿命。通过对后台任务应用资源限制来实施该操作，具体查看）。
下例将在 AllTasks 属性上进行迭代，并且如果任务已经注册，则将标志参数设置为 true：

2. 如果后台任务尚未注册，则使用 BackgroundTaskBuilder 创建你的后台任务的一个实例。
任务入口点应为命名空间为前缀的后台任务的名称。
后台任务触发器控制后台任务何时运行。有关可能的触发器的列表，请参阅 。
例如，此代码创建一个新后台任务并将其设置为在 TimeZoneChanged 触发器引发时运行：

3. 通过在 BackgroundTaskBuilder 对象上调用 Register 方法来注册后台任务。存储 BackgroundTaskRegistration 结果，以便可以在下一步中使用该结果。
以下代码注册后台任务并存储结果：
接下来我们使用 BackgroundTaskCompletedEventHandler 注册一个方法，以便应用可以从后台任务中获取结果。当启动或恢复应用时，将调用 OnCompleted 方法（如果自从应用位于前台的最后时间以来后台任务已完成）。（如果应用当前位于前台时后台任务完成，则会立即调用 OnCompleted 方法。）
1. 编写一个 OnCompleted 方法，以处理后台任务的完成。例如，后台任务结果可能导致 UI 更新。此处所示的方法足迹对于 OnCompleted 事件处理程序方法来说是必需的，即使该示例不使用 args 参数也是如此。
以下示例代码识别后台任务完成并调用获取消息字符串的方法。

注意：UI更新应该异步执行，为的是避免占用UI线程。可以用以下的方法来更新UI。

2. 回到已注册后台任务的位置。在该代码行之后，添加一个新的 BackgroundTaskCompletedEventHandler 对象。提供 OnCompleted 方法作为 BackgroundTaskCompletedEventHandler 构造函数的参数。
在Windows Store App中，必须先在应用清单 Package.appxmanifest 中声明各个后台任务，你的应用才能运行后台任务。
打开Package.appxmanifest，选中Declarations选项卡，点击“Available Declarations:”下方的下拉菜单，选中“Background Tasks”，再点击右侧的“Add”按钮。
在下方Supported Declarations就会出现一个新的项目“Background Tasks”，点击这个项目，右侧会出现一些信息及选项，让我们选中“System Event”，并在下方的“Entry Point”中填入“Tasks.ExampleBackgroundTask”，保存。


如果我们查看一下Package.appxmanifest的代码，会发现多出了下面的部分：

我们必须列出后台任务使用的每个触发器类型。如果你的应用尝试注册一个后台任务，其中有一个触发器未在清单中列出，那么注册将会失败。

MSDN中也给出了一个“”，告诉你如何才能让用户有更好的应用切换体验。

因为本篇比较零碎，可以下载这个例子，来查看完整的代码，来学习多任务机制。

像桌面应用，一般都会有“最小化”、“最大化”、“关闭”三个按钮，但Windows Store App不包含用于关闭应用的UI。用户可以通过按 Alt+F4、从“起始页”拖动应用或选择“关闭”上下文菜单的方式来关闭应用。当已使用这些方法中的任何方法关闭某个应用时，该应用将进入 NotRunning 状态大约 10 秒钟，然后转换到 ClosedByUser 状态。

正常执行时，应用不应采用编程方式关闭自身。当你以编程方式关闭应用时，Windows 会将此视为应用崩溃。应用将进入 NotRunning 状态并且保持该状态，直到用户再次激活该应用为止。
应用必须遵守系统崩溃体验，只需返回到“开始”页面即可。

下面是显示 Windows 如何确定应用的执行状态的图。Windows 将考虑应用崩溃和用户关闭操作，以及挂起或恢复状态。在该图中，蓝色矩形表示应用未加载到系统内存中。白色句柄表示应用位于内存中。虚线弧是在没有通知正在运行的应用的情况下发生的更改。实线弧是包含应用通知的操作。