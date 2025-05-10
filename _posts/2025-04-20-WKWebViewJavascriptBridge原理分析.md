---
title: "WKWebViewJavascriptBridge"
date: 2025-04-20 14:00:00 +0800
categories: [iOS,WKWebViewJavascriptBridge]
tags: [iOS, WKWebViewJavascriptBridge]
---
# WKWebViewJavascriptBridge 实现 iOS 与 JS 通信的原理分析

## 引言

在混合式移动应用开发中，原生代码（如 Swift 或 Objective-C）与 JavaScript 之间的无缝通信是关键。WKWebViewJavascriptBridge 是一个轻量级、高性能的开源库，专为 iOS 应用的 WKWebView 提供 Swift 和 JavaScript 的双向通信支持。它通过抽象底层的消息传递机制，让开发者能够专注于混合式功能的实现，而无需处理复杂的通信细节。本文将深入剖析 WKWebViewJavascriptBridge 的实现原理，结合代码分析其工作机制，并提供使用示例和最佳实践。

## 什么是 WKWebViewJavascriptBridge？

WKWebViewJavascriptBridge 是一个为 WKWebView 设计的通信桥梁，允许在 Swift（或 Objective-C）与 JavaScript 之间发送消息。它受到 WebViewJavascriptBridge 的启发，专注于 WKWebView 的支持，放弃了对 UIWebView 的兼容，以提升安全性和性能。其主要特点包括：

- **高性能**：基于 WKWebView 的脚本消息处理，优于传统的请求拦截方式。
- **高速度**：无需担心警框安全超时问题。
- **轻量级**：仅包含 3 个核心文件，易于集成。
- **非侵入式**：无需继承其他基类。
- **兼容性**：支持 Swift 3.2 至 5.0，适用于 iOS 9.0 及以上版本。

WKWebView 相较于 UIWebView 具有更快的加载速度、更低的内存开销，并且默认关闭了文件访问权限（`allowFileAccessFromFileURLs` 和 `allowUniversalAccessFromFileURLs`），从而避免了跨域访问的安全漏洞。

## 高层架构

WKWebViewJavascriptBridge 的核心在于通过 WKWebView 的脚本消息处理机制（`WKScriptMessageHandler`）实现双向通信。其架构分为两个主要部分：

1. **原生侧（Swift/Objective-C）**：
   - 通过 `WKWebViewJavascriptBridge` 类注册处理器，接收 JavaScript 发来的消息。
   - 调用 JavaScript 中的处理器，发送消息到网页。
2. **JavaScript 侧**：
   - 通过注入的 `WKWebViewJavascriptBridgeJS` 脚本注册处理器，接收原生侧的消息。
   - 调用原生侧的处理器，发送消息到 iOS。

通信依赖于 WKWebView 的用户内容控制器（`WKUserContentController`），通过添加脚本消息处理器（如 `iOS_Native_InjectJavascript` 和 `iOS_Native_FlushMessageQueue`）捕获消息。消息以 JSON 格式传递，并通过队列机制确保可靠性和顺序。

## JavaScript 侧的实现

JavaScript 侧的实现主要在 `WKWebViewJavascriptBridgeJS` 文件中，定义了桥梁的核心功能，包括初始化、注册处理器、调用处理器和消息处理。

### 1. 桥梁初始化

JavaScript 侧通过 `setupWKWebViewJavascriptBridge` 函数初始化桥梁：

```javascript
function setupWKWebViewJavascriptBridge(callback) {
    if (window.WKWebViewJavascriptBridge) { return callback(WKWebViewJavascriptBridge); }
    if (window.WKWVJBCallbacks) { return window.WKWVJBCallbacks.push(callback); }
    window.WKWVJBCallbacks = [callback];
    window.webkit.messageHandlers.iOS_Native_InjectJavascript.postMessage(null);
}
```

- **逻辑**：
  - 检查 `WKWebViewJavascriptBridge` 是否已存在，若存在，直接调用回调。
  - 检查是否存在回调队列（`WKWVJBCallbacks`），若存在，将回调加入队列。
  - 否则，创建回调队列并通过 `window.webkit.messageHandlers.iOS_Native_InjectJavascript.postMessage(null)` 通知原生侧注入桥梁脚本.
- **WKWVJBCallbacks**：这是一个回调队列，用于在桥梁初始化完成前存储回调函数。初始化完成后，`_callWVJBCallbacks` 函数会遍历队列，依次执行所有回调，确保异步初始化不丢失注册操作。
- **作用**：触发原生侧注入 `WKWebViewJavascriptBridgeJS` 脚本，设置 `WKWebViewJavascriptBridge` 对象。

注入的脚本定义了以下核心函数：

- `registerHandler`：注册处理器。
- `callHandler`：调用原生处理器。
- `_fetchQueue`：获取消息队列。
- `_handleMessageFromiOS`：处理原生侧消息。

### 2. 注册处理器

使用 `registerHandler` 注册处理器，接收原生侧的消息：

```javascript
function registerHandler(handlerName, handler) {
    messageHandlers[handlerName] = handler;
}
```

- **参数**：
  - `handlerName`：处理器名称。
  - `handler`：处理器函数，接收数据和回调。
- **存储**：处理器存储在 `messageHandlers` 对象中，键为处理器名称，值为函数.

### 3. 调用原生处理器

使用 `callHandler` 调用原生侧的处理器：

```javascript
function callHandler(handlerName, data, responseCallback) {
    if (arguments.length == 2 && typeof data == 'function') {
        responseCallback = data;
        data = null;
    }
    _doSend({ handlerName:handlerName, data:data }, responseCallback);
}
```

- **参数**：
  - `handlerName`：原生处理器名称。
  - `data`：传递的数据。
  - `responseCallback`：响应回调函数。
- **逻辑**：
  - 处理参数兼容性（如果只有两个参数且第二个是函数，则视为回调）。
  - 调用 `_doSend` 发送消息。

`_doSend` 函数：

```javascript
function _doSend(message, responseCallback) {
    if (responseCallback) {
        var callbackID = 'cb_'+(uniqueId++)+'_'+new Date().getTime();
        responseCallbacks[callbackID] = responseCallback;
        message['callbackID'] = callbackID;
    }
    sendMessageQueue.push(message);
    window.webkit.messageHandlers.iOS_Native_FlushMessageQueue.postMessage(null);
}
```

- **逻辑**：
  - 如果有回调，生成唯一回调 ID 并存储。
  - 将消息添加到 `sendMessageQueue` 队列。
  - 通知原生侧通过 `iOS_Native_FlushMessageQueue` 刷新队列。

### 4. 处理原native消息

原生侧的消息通过 `_handleMessageFromiOS` 处理：

```javascript
function _handleMessageFromiOS(messageJSON) {
    _dispatchMessageFromiOS(messageJSON);
}
```

`_dispatchMessageFromiOS` 函数解析消息并分发：

```javascript
function _dispatchMessageFromiOS(messageJSON) {
    var message = JSON.parse(messageJSON);
    var messageHandler;
    var responseCallback;

    if (message.responseID) {
        responseCallback = responseCallbacks[message.responseID];
        if (!responseCallback) { return; }
        responseCallback(message.responseData);
        delete responseCallbacks[message.responseID];
    } else {
        if (message.callbackID) {
            var callbackResponseId = message.callbackID;
            responseCallback = function(responseData) {
                _doSend({ handlerName:message.handlerName, responseID:callbackResponseId, responseData:responseData });
            };
        }
        var handler = messageHandlers[message.handlerName];
        if (!handler) {
            console.log("WKWebViewJavascriptBridge: WARNING: no handler for message from iOS:", message);
        } else {
            handler(message.data, responseCallback);
        }
    }
}
```

- **逻辑**：
  - 解析 JSON 消息。
  - 如果包含 `responseID`，调用对应的回调并移除。
  - 否则，查找注册的处理器并调用，若有 `callbackID`，创建响应回调。

## Swift 侧的实现

Swift 侧的实现主要由 `WKWebViewJavascriptBridgeBase` 和 `WKWebViewJavascriptBridge` 两个类组成。

### 1. 桥梁初始化

创建 `WKWebViewJavascriptBridge` 实例：

```swift
public init(webView: WKWebView) {
    super.init()
    self.webView = webView
    base = WKWebViewJavascriptBridgeBase()
    base.delegate = self
    addScriptMessageHandlers()
}
```

- **逻辑**：
  - 初始化 `WKWebViewJavascriptBridgeBase` 实例。
  - 设置委托为自身。
  - 调用 `addScriptMessageHandlers` 添加脚本消息处理器。

`addScriptMessageHandlers` 方法：

```swift
private func addScriptMessageHandlers() {
    webView?.configuration.userContentController.add(LeakAvoider(delegate: self), name: iOS_Native_InjectJavascript)
    webView?.configuration.userContentController.add(LeakAvoider(delegate: self), name: iOS_Native_FlushMessageQueue)
}
```

- **处理器**：
  - `iOS_Native_InjectJavascript`：处理初始化请求。
  - `iOS_Native_FlushMessageQueue`：处理消息队列刷新。
- **LeakAvoider**：`LeakAvoider` 是一个代理类，通过弱引用 `WKScriptMessageHandler` 避免 `WKUserContentController` 持有强引用导致内存泄漏。这种设计在长期运行的 WebView 中尤为重要，可防止控制器或桥梁对象无法释放。

### 2. 注入 JavaScript 脚本

当接收到 `iOS_Native_InjectJavascript` 消息时，调用 `injectJavascriptFile`：

```swift
func injectJavascriptFile() {
    let js = WKWebViewJavascriptBridgeJS
    delegate?.evaluateJavascript(javascript: js, completion: { [weak self] (_, error) in
        guard let self = self else { return }
        if let error = error {
            self.log(error)
            return
        }
        self.startupMessageQueue?.forEach({ (message) in
            self.dispatch(message: message)
        })
        self.startupMessageQueue = nil
    })
}
```

- **逻辑**：
  - 注入 `WKWebViewJavascriptBridgeJS` 脚本。
  - 如果有启动消息队列（`startupMessageQueue`），分发所有消息。
  - 清空启动消息队列。
- **startupMessageQueue**：用于在桥梁脚本注入完成前缓存消息，避免初始化期间的消息丢失。

### 3. 注册处理器

使用 `register` 方法注册处理器：

```swift
public func register(handlerName: String, handler: @escaping WKWebViewJavascriptBridgeBase.Handler) {
    base.messageHandlers[handlerName] = handler
}
```

- **存储**：处理器存储在 `messageHandlers` 字典中。

### 4. 调用 JavaScript 处理器

使用 `callHandler` 方法调用 JavaScript 处理器：

```swift
public func call(handlerName: String, data: Any? = nil, callback: WKWebViewJavascriptBridgeBase.Callback? = nil) {
    base.send(handlerName: handlerName, data: data, callback: callback)
}
```

`send` 方法：

```swift
func send(handlerName: String, data: Any?, callback: Callback?) {
    var message = [String: Any]()
    message["handlerName"] = handlerName
  
    if data != nil {
        message["data"] = data
    }
  
    if callback != nil {
        uniqueId += 1
        let callbackID = "native_iOS_cb_\(uniqueId)"
        responseCallbacks[callbackID] = callback
        message["callbackID"] = callbackID
    }
  
    queue(message: message)
}
```

- **逻辑**：
  - 创建消息字典，包含处理器名称和数据。
  - 如果有回调，生成唯一回调 ID 并存储。
  - 调用 `queue` 方法处理消息。

`queue` 方法：

```swift
private func queue(message: Message) {
    if startupMessageQueue == nil {
        dispatch(message: message)
    } else {
        startupMessageQueue?.append(message)
    }
}
```

`dispatch` 方法：

```swift
private func dispatch(message: Message) {
    guard var messageJSON = serialize(message: message, pretty: false) else { return }
  
    messageJSON = messageJSON.replacingOccurrences(of: "\\", with: "\\\\")
    // ... 其他转义处理 ...
  
    let javascriptCommand = "WKWebViewJavascriptBridge._handleMessageFromiOS('\(messageJSON)');"
    if Thread.current.isMainThread {
        delegate?.evaluateJavascript(javascript: javascriptCommand)
    } else {
        DispatchQueue.main.async {
            self.delegate?.evaluateJavascript(javascript: javascriptCommand)
        }
    }
}
```

- **逻辑**：
  - 序列化消息为 JSON。
  - 转义特殊字符。
  - 注入 JavaScript 代码，调用 `_handleMessageFromiOS`。
  - 检查当前线程，若非主线程，使用 `DispatchQueue.main.async` 确保 UI 操作（如 `evaluateJavaScript`）在主线程执行，防止线程安全问题。

### 5. 处理消息队列

当接收到 `iOS_Native_FlushMessageQueue` 消息时，调用 `flushMessageQueue`：

```swift
private func flushMessageQueue() {
    webView?.evaluateJavaScript("WKWebViewJavascriptBridge._fetchQueue();") { (result, error) in
        if error != nil {
            print("WKWebViewJavascriptBridge: WARNING: Error when trying to fetch data from WKWebView: \(String(describing: error))")
        }
      
        guard let resultStr = result as? String else { return }
        self.base.flush(messageQueueString: resultStr)
    }
}
```

`flush` 方法：

```swift
func flush(messageQueueString: String) {
    guard let messages = deserialize(messageJSON: messageQueueString) else {
        log(messageQueueString)
        return
    }
  
    for message in messages {
        log(message)
      
        if let responseID = message["responseID"] as? String {
            guard let callback = responseCallbacks[responseID] else { continue }
            callback(message["responseData"])
            responseCallbacks.removeValue(forKey: responseID)
        } else {
            var callback: Callback?
            if let callbackID = message["callbackID"] {
                callback = { (_ responseData: Any?) -> Void in
                    let msg = ["responseID": callbackID, "responseData": responseData ?? NSNull()] as Message
                    self.queue(message: msg)
                }
            } else {
                callback = { (_ responseData: Any?) -> Void in }
            }
          
            guard let handlerName = message["handlerName"] as? String else { continue }
            guard let handler = messageHandlers[handlerName] else {
                log("NoHandlerException, No handler for message from JS: \(message)")
                continue
            }
            handler(message["data"] as? [String : Any], callback)
        }
    }
}
```

- **逻辑**：
  - 反序列化消息队列。
  - 遍历消息：
    - 如果包含 `responseID`，调用对应回调。
    - 否则，查找处理器并调用，若有 `callbackID`，创建响应回调。

## 通信流程详解

以下是完整的通信流程：

### 1. 初始化桥梁

- **JavaScript 侧**：调用 `setupWKWebViewJavascriptBridge`，将回调加入 `WKWVJBCallbacks`，发送 `iOS_Native_InjectJavascript` 消息。
- **原生侧**：接收消息，调用 `injectJavascriptFile`，注入 `WKWebViewJavascriptBridgeJS` 脚本，执行 `WKWVJBCallbacks` 中的回调。
- **结果**：页面中设置 `WKWebViewJavascriptBridge` 对象，双方准备好通信。

### 2. JavaScript 调用原生处理器

- **JavaScript 侧**：
  - 调用 `bridge.callHandler`，添加消息到 `sendMessageQueue`。
  - 发送 `iOS_Native_FlushMessageQueue` 消息。
- **原生侧**：
  - 接收消息，调用 `flushMessageQueue`。
  - 执行 `WKWebViewJavascriptBridge._fetchQueue()` 获取队列。
  - 解析消息，调用注册的处理器。
- **回调**：如果有回调，处理器通过 `callback` 发送响应，JavaScript 侧通过 `responseCallbacks` 处理。

### 3. 原生调用 JavaScript 处理器

- **原生侧**：
  - 调用 `bridge.callHandler`，序列化消息。
  - 注入 JavaScript 代码，调用 `_handleMessageFromiOS`。
- **JavaScript 侧**：
  - 解析消息，调用注册的处理器。
  - 如果有回调，发送响应消息。
- **回调**：原生侧通过 `responseCallbacks` 处理响应，异步处理确保非阻塞。

## 使用示例

以下是一个完整的示例，展示如何使用 WKWebViewJavascriptBridge，包括错误处理。

### 原生侧（Swift）

```swift
import WKWebViewJavascriptBridge
import WebKit

class ViewController: UIViewController {
    var webView: WKWebView!
    var bridge: WKWebViewJavascriptBridge!

    override func viewDidLoad() {
        super.viewDidLoad()
      
        let configuration = WKWebViewConfiguration()
        webView = WKWebView(frame: view.bounds, configuration: configuration)
        view.addSubview(webView)
      
        bridge = WKWebViewJavascriptBridge(webView: webView)
        bridge.isLogEnable = true // 启用日志以调试
      
        // 注册处理器
        bridge.register(handlerName: "testiOSCallback") { (parameters, callback) in
            guard let params = parameters else {
                print("Error: No parameters received")
                callback?(["error": "Invalid parameters"])
                return
            }
            print("Native received data: \(params)")
            callback?(["response": "Native processed data"])
        }
      
        // 调用 JavaScript 处理器
        bridge.call(handlerName: "testJavascriptHandler", data: ["foo": "bar"]) { (response) in
            print("Native received response: \(response ?? [:])")
        }
      
        // 加载网页
        if let url = URL(string: "https://example.com") {
            webView.load(URLRequest(url: url))
        }
    }
}
```

### JavaScript 侧

```javascript
function setupWKWebViewJavascriptBridge(callback) {
    if (window.WKWebViewJavascriptBridge) { return callback(WKWebViewJavascriptBridge); }
    if (window.WKWVJBCallbacks) { return window.WKWVJBCallbacks.push(callback); }
    window.WKWVJBCallbacks = [callback];
    window.webkit.messageHandlers.iOS_Native_InjectJavascript.postMessage(null);
}

// 在网页加载时初始化
document.addEventListener('DOMContentLoaded', function() {
    setupWKWebViewJavascriptBridge(function(bridge) {
        // 注册处理器
        bridge.registerHandler('testJavascriptHandler', function(data, responseCallback) {
            if (!data) {
                console.log('Error: No data received');
                responseCallback({ error: 'Invalid data' });
                return;
            }
            console.log('JS received data: ' + JSON.stringify(data));
            responseCallback('JS processed data');
        });
      
        // 调用原生处理器
        bridge.callHandler('testiOSCallback', {'foo': 'bar'}, function(response) {
            console.log('JS received response: ' + JSON.stringify(response));
        });
    });
});
```

**注意**：

- 确保网页加载的 HTML 包含 `setupWKWebViewJavascriptBridge` 脚本，通常在 `DOMContentLoaded` 事件中执行。
- 示例中添加了参数检查，处理无效数据的情况。

## 实现原理总结

WKWebViewJavascriptBridge 的实现依赖于以下关键机制：

1. **脚本消息处理器**：
   - 原生侧通过 `WKUserContentController` 添加 `iOS_Native_InjectJavascript` 和 `iOS_Native_FlushMessageQueue` 处理器。
   - JavaScript 侧通过 `window.webkit.messageHandlers` 发送消息。
2. **消息队列**：
   - JavaScript 侧使用 `sendMessageQueue` 存储待发送消息。
   - 原生侧使用 `startupMessageQueue` 存储初始化期间的消息。
3. **JSON 序列化**：
   - 消息以 JSON 格式传递，确保跨语言兼容性。
   - 原生侧通过 `serialize` 和 `deserialize` 方法处理 JSON。
4. **回调机制**：
   - 双方通过唯一 ID（`callbackID` 和 `responseID`）管理回调。
   - 回调存储在 `responseCallbacks` 中，处理后移除。
5. **线程安全**：
   - 原生侧通过 `DispatchQueue.main.async` 确保 `evaluateJavaScript` 在主线程执行，避免 UI 线程问题。

## 优势与注意事项

### 优势

- **简单易用**：提供直观的 API，开发者无需深入了解底层机制。
- **高性能**：直接使用 `WKScriptMessageHandler` 进行消息传递，避免了请求拦截的网络开销。JSON 序列化确保高效的数据传输，消息队列机制减少重复的 JavaScript 注入操作。
- **轻量级**：仅 3 个文件，易于集成。
- **安全**：基于 WKWebView，默认关闭跨域访问权限，降低安全风险。

### 注意事项

- **初始化时机**：确保在网页加载前初始化桥梁，并在 `DOMContentLoaded` 事件中调用 `setupWKWebViewJavascriptBridge`，否则可能错过消息。
- **错误处理**：启用日志（`isLogEnable = true`）以调试问题。处理无效参数或处理器不存在的情况，避免异常。
- **内存管理**：`LeakAvoider` 类防止内存泄漏，开发者应避免直接将控制器作为 `WKScriptMessageHandler`。
- **线程安全**：消息分发在主线程执行，开发者无需额外处理线程问题，但应注意异步回调的非阻塞特性。
- **版本兼容性**：支持 iOS 9.0 及以上，但 WKWebView 在较低版本上的性能可能略有差异。建议测试目标系统版本以确保兼容性。

## 结论

WKWebViewJavascriptBridge 通过巧妙地利用 WKWebView 的脚本消息处理机制，实现了 iOS 和 JavaScript 之间的高效双向通信。其设计简洁、性能卓越，提供了易于使用的 API，让开发者能够轻松地在混合式应用中实现复杂的交互功能。理解其内部实现原理不仅有助于更好地使用该库，还能在需要时进行定制和优化。对于希望在 iOS 应用中集成 Web 内容的开发者来说，WKWebViewJavascriptBridge 是一个不可或缺的工具。

**引用**：

- [WKWebViewJavascriptBridge GitHub 仓库](https://github.com/Lision/WKWebViewJavascriptBridge)
- [在 iOS 应用中设置 WKWebView 和 JavaScript 之间的桥梁](https://medium.com/%40bahalek/setting-up-js-bridge-between-your-webpage-and-wkwebview-in-your-ios-app-4ec8ca8230f7)
