---
title: DramaWave 技术栈
created: 2026-04-16
updated: 2026-04-16
type: concept
tags: [project, android, dependency, network, player, ads, analytics, storage]
sources: [raw/articles/dramawave-codebase-scan-2026-04-16.md]
---

# DramaWave 技术栈

## 核心平台栈
- Android Gradle Plugin：8.8.0
- Kotlin：2.1.0
- compileSdk / targetSdk：35
- minSdk：23
- Gradle Kotlin DSL
- Hilt 作为依赖注入主方案

## UI 栈
该工程并非单一 UI 技术，而是混合栈：
- ViewBinding
- DataBinding
- Jetpack Compose
- Material Components
- ConstraintLayout
- 自定义 View / 传统 XML 页面仍占主体

说明项目处于渐进式 Compose 引入阶段，而不是全面 Compose 化。

## 架构与基础设施
- MVI：`core_mvi`
- EventBus：`core_bus`
- 路由：TheRouter + 自定义全局拦截器
- 启动框架：`core_startup` 自研 `AndroidStartup`
- 日志与追踪：`core_log`、`StartupPerformanceTracker`

## 网络栈
- OkHttp 4.12
- Retrofit 2.9
- Cronet GMS + cronet-okhttp
- RxJava 2
- Gzip 请求压缩
- DDNS / backup domain / 业务错误拦截 / QUIC 切换等增强策略

这说明网络层不是轻量包装，而是强业务定制的中台能力。

## 存储与状态
- MMKV：主 KV 存储方案，`core_kv` 下存在大量 `*Store`
- Room：数据库能力由 `core_db` 承载
- DataStore：版本目录中已声明，但从当前高频信号看，MMKV 仍是更主流状态存储手段

## 图片与媒体
- Coil 3：图片加载主方案
- Tencent LiteAVSDK Premium：播放核心 SDK
- `shared_player` 作为播放器包装层，支持状态机、缓存、下载、预加载等

## 商业化与增长
### 广告
`shared_ad` 聚合了大量广告平台与适配器：
- AppLovin MAX
- Google AdMob
- Pangle / FillBooster
- Tapjoy
- FairBid
- Unity Ads
- Opera Ads
- TaurusX
- 以及多个 mediation adapter

### 归因
- AppsFlyer
- `shared_af` 模块承载归因系统

### 支付
- Google Play Billing
- `shared_purchase` 负责 IAP/订阅业务

## 可观测性与稳定性
- Firebase Crashlytics
- Firebase Performance
- LeakCanary（debug）
- KOOM（debug）
- 自定义设备性能评分 / 超低端机检测 / 线程治理 / 启动耗时追踪

这说明项目非常关注线上稳定性与设备分层治理。

## Web 与混合能力
- `core_web` + `shared_web` + `feature_web`
- WebView 页面并不是边缘能力，而是完整业务域

## 构建与工程化插件
- `build-logic` 约定插件：统一 Android application/library/buildconfig/flavor aware
- `plugins/view-monitor`：调试用 ASM 监控插件
- baseline profile 模块：性能优化已进入构建体系

## 总体判断
DramaWave 的技术栈不是“现代 Android 教科书式最小集合”，而是典型商业 app 的“全栈混合体”：
- 新旧 UI 并存
- 强第三方 SDK 集成
- 强广告与增长中台
- 强基础设施沉淀
- 强构建治理与模块边界约束

## 关联页面
- [[dramawave-codebase-overview]]
- [[dramawave-build-and-architecture]]
- [[dramawave-startup-and-entry]]
- [[dramawave-module-map]]
- [[dramawave-codebase-risks]]
