---
title: DramaWave 代码库总览
created: 2026-04-16
updated: 2026-04-16
type: concept
tags: [project, wiki, android, architecture, module, research]
sources: [raw/articles/dramawave-codebase-scan-2026-04-16.md]
---

# DramaWave 代码库总览

`DramaWave` 是一个大型 Android 短剧应用工程，采用 Kotlin 多模块架构，构建与运行时能力高度分层。它不是单体 app 模块堆叠，而是围绕 `app → feature → interface → shared → core` 的层级组织业务、基础设施与跨模块 API。

## 一句话判断
- 这是一个“重业务、多入口、多 SDK、强工程约束”的 Android 商业化项目。
- 代码库核心复杂度主要来自：多模块依赖治理、启动链路、广告与归因体系、播放器体系、奖励体系，以及 flavor 差异化。

## 分层结构
- `app`：应用入口与全局编排，承载 `DramaApp`、`MainActivity`、启动链路、路由接线与产品级配置。
- `feature`：业务功能模块，如首页、剧场、奖励、登录、个人页、搜索、小说、Web、小组件等。
- `interface`：跨 feature 调用的桥接 API，避免 feature 之间直接互相依赖。
- `shared`：共享业务能力，包含播放器、广告、归因、埋点、UI、用户、支付、推送、资源、导航等。
- `core`：纯基础设施层，包含网络、路由、MVI、总线、KV、数据库、APM、日志、图片、启动框架等。
- `plugins` / `build-logic`：构建插件与工程约定层，负责统一模块规则、flavor 感知、调试插件等。

## 规模信号
基于 2026-04-16 的本地扫描结果：
- `feature` 目录约 3147 个文件，其中 1542 个 Kotlin 文件，是业务主战场。
- `shared` 目录约 2661 个文件，其中 1318 个 Kotlin 文件，是共享业务能力核心区。
- `core` 目录约 622 个文件，其中 413 个 Kotlin、77 个 Java，是技术基础设施中心。
- 全仓库粗略扩展名统计中，`.kt` 约 3457 个，`.xml` 约 1721 个，说明该项目仍以 View 系 UI 与资源系统为主体，同时局部引入 Compose。

## 关键页面
- [[dramawave-build-and-architecture]]
- [[dramawave-startup-and-entry]]
- [[dramawave-module-map]]
- [[dramawave-technology-stack]]
- [[dramawave-codebase-risks]]

## 结论
对于后续 codebase wiki 建设，最值得优先沉淀的不是单个页面逻辑，而是：
1. 工程构建与模块边界
2. 启动链路与 Application/MainActivity 编排
3. 首页/奖励/播放器/广告四大高复杂业务簇
4. 网络、路由、KV、埋点、APM 五大基础设施簇

这些内容决定了阅读整个工程时的认知效率。
