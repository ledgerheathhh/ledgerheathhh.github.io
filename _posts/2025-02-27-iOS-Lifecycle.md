---
title: "iOS Lifecycle"
date: 2025-02-27 00:50:00 +0800
categories: [iOS,Lifecycle]
tags: [iOS, Lifecycle]
---
- iOS 应用的生命周期包括未运行、非活跃、活跃、后台和挂起五种状态，研究表明这些状态转换对管理应用行为至关重要。
- 关键方法包括 `application(_:didFinishLaunchingWithOptions:)`（启动时初始化）、`applicationWillResignActive(_:)`（即将非活跃）、`applicationDidEnterBackground(_:)`（进入后台）、`applicationWillEnterForeground(_:)`（即将进入前台）、`applicationDidBecomeActive(_:)`（恢复活跃）和 `applicationWillTerminate(_:)`（即将终止）。
- UIViewController 的生命周期包括初始化、视图加载、视图出现、布局调整、视图消失和内存管理等阶段，这些方法在界面管理中起到关键作用。
- 对于 SwiftUI 应用，生命周期管理有所不同，但本文重点讨论 UIKit 方式。

### 应用生命周期概述

#### 状态解释

- **未运行（Not Running）**：应用未在内存中，可能未启动或已被系统终止。
- **非活跃（Inactive）**：应用在前台运行但不接收事件，如用户锁屏或接电话时。
- **活跃（Active）**：应用在前台运行并接收用户输入，这是正常交互状态。
- **后台（Background）**：应用在后台运行，可执行代码但时间有限，之后可能挂起。
- **挂起（Suspended）**：应用在内存中但不执行代码，系统可能随时终止以释放内存。

#### 关键方法

以下是 `UIApplicationDelegate` 协议中的主要方法及其用途：

| 方法名                                            | 用途                                                             |
| ------------------------------------------------- | ---------------------------------------------------------------- |
| `application(_:didFinishLaunchingWithOptions:)` | 启动时初始化，配置 UI，处理启动选项如推送通知或 URL 方案。       |
| `applicationWillResignActive(_:)`               | 应用即将非活跃，暂停需要用户交互的任务，如接电话或按 Home 键时。 |
| `applicationDidEnterBackground(_:)`             | 应用进入后台，保存数据，释放不必要的资源，约有 5 秒完成任务。    |
| `applicationWillEnterForeground(_:)`            | 应用即将从后台返回前台，准备用户交互，如重新加载数据。           |
| `applicationDidBecomeActive(_:)`                | 应用恢复活跃，恢复之前暂停的任务，更新 UI。                      |
| `applicationWillTerminate(_:)`                  | 应用即将被终止，执行最终清理，如保存未保存数据。                 |

这些方法确保应用在状态转换时能够正确响应系统事件。例如，`applicationDidEnterBackground(_:)` 通常用于保存用户数据，因为应用可能被系统挂起或终止。

### UIViewController 生命周期

UIViewController 的生命周期管理了视图控制器的创建、显示和销毁过程。以下是关键方法及其作用：

1. **初始化阶段**：

   - `alloc`：分配内存为视图控制器对象。
   - `init`（包括 `initWithNibName` 或 `initWithCoder`）：初始化视图控制器，`initWithNibName` 用于代码创建，`initWithCoder` 用于从 nib 或 storyboard 加载。
   - `awakeFromNib`：从 nib 文件加载后调用，所有 outlets 和 actions 已连接。
2. **视图加载阶段**：

   - `loadView`：加载视图控制器管理的视图，通常由系统处理，但可重写以编程方式创建视图。注意：如果使用 Interface Builder 创建视图，勿重写此方法，否则 xib 或 storyboard 设置会失效。
   - `viewDidLoad`：视图加载到内存后调用，进行额外的视图设置，如添加子视图或设置属性。
3. **视图出现与消失**：

   - `viewWillAppear(_:)`：视图即将显示时调用，适合弹出键盘或设置状态栏颜色。
   - `viewDidAppear(_:)`：视图已显示时调用。
   - `viewWillDisappear(_:)`：视图即将从屏幕移除时调用。
   - `viewDidDisappear(_:)`：视图已从屏幕移除时调用。
4. **布局调整**：

   - `updateViewConstraints`：更新视图的约束，仅在 Auto Layout 时调用。初始化约束时，建议写在 `init` 或 `viewDidLoad` 中，`updateViewConstraints` 适合更新约束。
   - `viewWillLayoutSubviews`：视图即将布局子视图时调用，此时 Auto Layout 未起作用。
   - `viewDidLayoutSubviews`：视图布局子视图完成后调用，此时 frame 值最准确。如果用约束布局，在此设置 frame 无效；如果用 frame 布局，则有效。
5. **内存管理**：

   - `didReceiveMemoryWarning`：系统内存不足时调用，当前控制器及导航堆栈上的控制器都会调用此方法。如果视图未显示在 window 上，系统可能销毁视图以释放内存。
   - `dealloc`：视图控制器被销毁时调用，释放在 `init` 和 `viewDidLoad` 中创建的对象。

### Extra

#### 状态转换

以下是典型状态转换序列，帮助理解方法调用顺序：

- **启动**：从不运行到活跃，调用 `application(_:didFinishLaunchingWithOptions:)`，随后可能调用 `applicationDidBecomeActive(_:)`。
- **非活跃**：如用户按 Home 键，调用 `applicationWillResignActive(_:)`，然后可能进入后台。
- **后台**：调用 `applicationDidEnterBackground(_:)`，应用进入后台状态。
- **返回前台**：调用 `applicationWillEnterForeground(_:)`，随后 `applicationDidBecomeActive(_:)` 恢复活跃。
- **终止**：系统决定终止应用时，调用 `applicationWillTerminate(_:)`。

需要注意的是，`applicationWillTerminate(_:)` 并非总是被调用，例如设备重启时不会触发。

#### 实际应用

在开发中，正确处理这些方法至关重要。例如：

- 在 `applicationDidEnterBackground(_:)` 中，建议保存用户数据并释放缓存，以减少内存占用，避免被系统终止。
- 在 `applicationWillEnterForeground(_:)` 中，可检查网络状态或更新 UI，确保用户体验流畅。
- `applicationWillTerminate(_:)` 适合执行最终清理，但由于可能不被调用，关键数据应在 `applicationDidEnterBackground(_:)` 中保存。
- 在 `viewDidLoad` 中设置视图的子视图和属性。
- 在 `didReceiveMemoryWarning` 中释放非必要资源以应对内存压力。
- 注意 `loadView` 的重写：如果使用 Interface Builder 创建视图，勿重写此方法。

此外，证据显示，过度依赖 AppDelegate 会导致代码膨胀，建议通过扩展或单独类分离职责，提升可维护性。

#### 额外细节

- 在 `main` 函数执行前，objc 运行时环境初始化，会加载所有类并调用类的 `load` 方法，通常用于实现方法交换（Method Swizzle）。
- UIViewController 的 `self.view` 是通过懒加载创建的，每次访问 `view` 属性且 `view` 为 nil 时，调用 `loadView`。加载成功后调用 `viewDidLoad`。
- 如果重写 `loadView`，不应该调用父类的 `loadView`，因为默认逻辑会根据 xib 或 storyboard 初始化视图。如果未找到同名文件，会创建一个空白 UIView，frame 为屏幕大小。
- 在 `init` 中不要调用 `self.view`，应仅初始化关键数据。如果需要重写 `loadView`，只初始化视图，其他工作放在 `viewDidLoad` 中完成。

#### 引用来源

信息来源于以下资源：

- [Managing your app’s life cycle Apple Developer Documentation](https://developer.apple.com/documentation/uikit/managing-your-app-s-life-cycle)
- [Application life cycle in iOS Every iOS Developer Medium](https://manasaprema04.medium.com/application-life-cycle-in-ios-f7365d8c1636)
- [IOS app lifecycle Javatpoint](https://www.tpointtech.com/ios-app-lifecycle)
- [App Life Cycle and View Controller Life Cycle in Swift IOS Medium](https://medium.com/@kashif00527/app-life-cycle-and-view-controller-life-cycle-in-swift-ios-3a8393963ad4)
