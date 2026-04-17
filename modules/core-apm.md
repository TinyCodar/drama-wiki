---
title: core_apm deep dive
created: 2026-04-16
updated: 2026-04-16
type: module
layer: core
status: verified
tags: [core, apm, perf, crash, memory, deep-dive]
sources:
  - core/core_apm/CLAUDE.md
  - app/src/main/java/com/dramawave/app/startup/component/CommonInitializer.kt
---

# core_apm deep dive

## 定位
`core_apm` 是 DramaWave 的基础监控模块，承载崩溃上报、性能追踪、设备性能分级、内存泄漏接入、线程治理与超低端机判断等能力。

## 关键职责
- Firebase Crashlytics（含 NDK）封装
- Firebase Performance trace 开关与指标埋点
- 设备性能评分（CPU + Memory 责任链）
- Debug 场景 LeakCanary + KOOM 接入
- wild-thread 治理与线程峰值监控
- FreeReels 超低端机检测

## 目录结构
- `crash/`：`CrashReporter`、`RxJavaPluginErrorHandler`
- `firebase/`：`FirebasePerfWrapper`、`PerfTrace`、`PerfTraceTag`
- `detector/`：性能分级责任链及 CPU/内存检测实现
- `performance/`：`PerformanceHelper`、`ILeakHelper`、`UltraLowDeviceCheck`
- `thread/`：线程峰值监控与对比
- `src/debug/.../LeakHelperDebug.kt`：debug 专用泄漏检测实现

## 关键类
- `CrashReporter`：Crashlytics 包装，支持 init、setUser、postCatchedException
- `PerfTrace`：业务 trace 开始/结束 API
- `FirebasePerfWrapper`：通过 Remote Config 开关控制性能追踪
- `PerformanceScoreDetector`：设备性能评分入口
- `DeviceBlacklistManager`：问题设备降级
- `PerformanceHelper`：泄漏检测包装入口
- `ThreadComparator`：线程治理对比上报

## 运行时接入
根据当前工程上下文，APM 初始化由 `app/src/main/java/com/dramawave/app/startup/component/CommonInitializer.kt` 中的 `PerformanceHelper.initHelper(context)` 触发。当前栈不是 Matrix，而是 Firebase Perf + Crashlytics，加上 debug 下的 LeakCanary/KOOM。

## 依赖关系
下游依赖：
- `core_common`
- `core_config`
- `core_kv`
- Firebase Crashlytics / Perf
- debug 下 leakcanary、koom

上游使用者：
- `app` 启动初始化链路
- 可能的播放器、网络、业务模块性能埋点调用方

## 风险与备注
- 泄漏检测为 compile-time 注入接口，不同 build variant 行为不同
- 文档与当前 Matrix 适配研究形成鲜明对照：现网 APM 基线仍是 Firebase + LeakCanary/KOOM
- 进行 Matrix 迁移评估时，`core_apm` 是最关键替换落点之一

## 相关页面
- [[startup-and-apm-init-flow]]
- [[dramawave-overview]]
- [[shared-player]]
