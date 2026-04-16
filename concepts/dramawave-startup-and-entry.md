---
title: DramaWave 启动链路与入口
created: 2026-04-16
updated: 2026-04-16
type: concept
tags: [project, android, startup, manifest, architecture, analytics, network]
sources: [raw/articles/dramawave-codebase-scan-2026-04-16.md]
---

# DramaWave 启动链路与入口

## 入口文件
启动链路的关键文件包括：
- `app/src/main/AndroidManifest.xml`
- `app/src/main/java/com/dramawave/app/DramaApp.kt`
- `app/src/main/java/com/dramawave/app/startup/loader/ApplicationLoader.kt`
- `app/src/main/java/com/dramawave/app/startup/component/CommonInitializer.kt`
- `app/src/main/java/com/dramawave/app/MainActivity.kt`

## Manifest 视角
`AndroidManifest.xml` 透露出产品是典型的商业化内容应用：
- 基础权限：网络、相机、相册读取
- `queries` 声明大量外部包查询，包括竞品 app、韩国认证 app、广告拦截类 app
- `application` 为 `.DramaApp`
- `SplashActivity` 是 launcher，并负责 deeplink 入口
- `MainActivity` 为主壳 Activity，`singleTask`，`adjustNothing`，portrait
- 配置了 Google Ads 优化初始化与优化加载 meta-data
- 同时配置了 Google Analytics consent defaults

这些信息表明：应用非常重视冷启动、deeplink 接入、广告初始化与合规参数。

## DramaApp：真正的全局装配器
`DramaApp` 是 Hilt Application，职责非常重：

### attachBaseContext 阶段
- 初始化 `AppProvider`
- 主进程下接管多语言 `MultiLanguages.attach`
- 初始化 `AppConfig`

### onCreate 阶段
按顺序做了大量全局装配：
- 启动性能打点 `StartupPerformanceTracker`
- debug 条件下初始化 ViewMonitor
- 初始化语言系统
- 初始化路由 `RouteManager.init(this)`
- 注册全局路由拦截器与路径替换器
- 初始化 Firebase
- 初始化 CrashReporter
- 初始化图片加载 `Img.init(CoilImgLoader())`
- 初始化 Toaster
- 初始化 EventBus
- 初始化并启动 StartUpManager / ApplicationLoader
- 初始化 Activity 生命周期管理器 `LifecycleManager`
- 初始化 ServiceLocator
- 挂载 `ProcessLifecycleOwner` 观察器
- 初始化 UI 模块级配置（如是否允许截图）

结论：`DramaApp` 并非瘦 Application，而是工程级总控点。

## 路由链路
`DramaApp.initGlobalRouterInterceptor()` 表示路由体系不是简单地只靠注解跳转，而是带有统一拦截责任链：
- `InternalNavigationHandler`
- `PipRouteHandler`
- `ComingSoonRouteHandler`
- `PlayDetailRouteHandler`
- `LoginRouterHandler`

这说明 TheRouter 在工程里承担的不只是路径解析，还包含：
- 应用内导航标记
- PIP/播放器跳转治理
- 未上线功能兜底
- 播放详情重写
- 登录态路由修正

## ApplicationLoader：启动任务编排核心
`ApplicationLoader` 是 `BaseLoader` 子类，`execute()` 中调用 `initStartUp()`，再通过 `StartupManager.Builder()` 编排一批 `AndroidStartup` 任务。

### 启动任务清单
默认列表包含：
- `FirebaseAnalyticsInitializer`
- `AppsFlyerInitializer`
- `StarLoggerAnalyticsInitializer`
- `NetworkInitializer`
- `ViewInitializer`
- `RemoteConfigInitializer`
- `CommonInitializer`
- `PlayerInitializer`
- `NotificationInitializer`
- `AsyncInitializer`
- `SkinManagerInitializer`

条件性任务：
- `ChatInitializer`：仅 DramaWave flavor
- `SmInitializer`：仅首次启动时加入

### 启动框架特征
- 有自研 `core_startup` 框架
- 支持统计与超时配置
- `awaitTimeout` 为 8000ms
- 支持完成回调并上报主线程耗时
- 与 Firebase Perf 和 `StartupPerformanceTracker` 打通

这意味着项目把“启动”当成一个可观测、可编排、可拆分的系统，而不是 Application 里直接堆初始化代码。

## CommonInitializer：通用初始化中心
`CommonInitializer` 主要负责：
- 清理过期 AB 实验缓存
- 初始化性能监控 `PerformanceHelper.initHelper(context)`
- 注入网络日志回调到埋点系统
- 注册 RxJava 全局异常处理器
- 预热屏幕尺寸缓存
- 注入用户鉴权、cookie、user-agent、GAID 到网络代理层

它处于“APM / 网络 / 用户态基础上下文”交汇点，是典型的启动基建型 initializer。

## MainActivity：主壳协调中心
`MainActivity.kt` 体量约 2370 行，是当前 codebase 最强复杂度信号之一。

从 import 和职责看，它至少同时承担：
- 主导航容器
- 多 tab 页面切换
- 首页、剧场、书单、奖励、消息等业务协调
- 事件总线消费与发射
- deeplink/路由衔接
- 广告、归因、低活、福利、弹窗治理
- 播放器/观看状态与页面联动
- 站内消息、徽标、通知、埋点更新

这说明当前 MainActivity 已接近“超级壳层”模式：
- 优点：集中调度、入口统一
- 风险：维护面大、回归影响大、职责边界容易继续膨胀

## 启动链路总体判断
DramaWave 的启动链路体现出三个工程倾向：
1. 强可观测：通过 `StartupPerformanceTracker`、Firebase Perf、网络日志与埋点打通启动分析
2. 强模块化：通过自研 startup 框架把初始化拆成多个独立任务
3. 强商业化接线：广告、归因、远程配置、通知、播放器、用户态都在冷启动附近完成接线

## 关联页面
- [[dramawave-codebase-overview]]
- [[dramawave-build-and-architecture]]
- [[dramawave-module-map]]
- [[dramawave-technology-stack]]
- [[dramawave-codebase-risks]]
