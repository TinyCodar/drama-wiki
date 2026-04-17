---
title: 启动与 APM 初始化链路
created: 2026-04-16
updated: 2026-04-16
type: flow
layer: app
status: active
tags: [startup, apm, app, flow, perf]
sources:
  - AGENTS.md
  - core/core_apm/CLAUDE.md
  - app/src/main/java/com/dramawave/app/startup/component/CommonInitializer.kt
---

# 启动与 APM 初始化链路

## 结论
当前 DramaWave 的 APM 初始化基线位于应用启动链路中，由 `CommonInitializer.kt` 调用 `PerformanceHelper.initHelper(context)` 完成。运行时监控主体为 Firebase Crashlytics + Firebase Performance，debug 下补充 LeakCanary/KOOM。

## 链路概览
1. `app/` 作为装配层启动应用
2. 启动初始化组件 `CommonInitializer` 被触发
3. 初始化阶段调用 `PerformanceHelper.initHelper(context)`
4. `core_apm` 内部继续接管泄漏检测、性能埋点、设备评分等能力
5. 业务模块随后可通过 `CrashReporter`、`PerfTrace` 等 API 使用 APM 能力

## 为什么关键
- 这是 Matrix 或其他 APM 体系替换/接入时的主落点
- APM 不只是 crash，还包括 perf trace、低端机识别、线程治理等能力
- 与 `shared_player`、网络、首页加载等高频链路天然相关

## 现状备注
- 当前并未使用 Matrix 作为现行主 APM 栈
- debug 泄漏检测采用 LeakCanary + KOOM
- 若将来演进 Matrix，需要保持 DramaWave 当前构建栈兼容，不能直接照搬旧 Matrix 工程结构

## 相关页面
- [[core-apm]]
- [[dramawave-overview]]
- [[player-system-overview]]
