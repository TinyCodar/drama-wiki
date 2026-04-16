---
title: DramaWave 模块地图
created: 2026-04-16
updated: 2026-04-16
type: concept
tags: [project, architecture, module, dependency, android, research]
sources: [raw/articles/dramawave-codebase-scan-2026-04-16.md]
---

# DramaWave 模块地图

## 模块总览
从 `settings.gradle.kts` 看，DramaWave 是高度模块化工程，至少包含：
- 1 个 app 模块
- 13 个 feature 模块
- 11 个 interface 模块
- 22 个 shared 模块
- 20 个 core 模块
- 以及 build-logic、plugins、baselineprofile、libraries:utils 等附属模块

## 分层职责
### app
- `app`
- 职责：应用壳层、全局初始化、启动装配、主导航、构建入口

### feature
主要业务模块包括：
- `feature_home`
- `feature_theater`
- `feature_mylist`
- `feature_profile`
- `feature_reward`
- `feature_search`
- `feature_develop`
- `feature_login`
- `feature_ability`
- `feature_web`
- `feature_ashes`
- `feature_novel`
- `feature_widget`

从文件量看，`feature` 是最大业务层，约 1542 个 Kotlin 文件。

### interface
桥接 API 层：
- `interface_login`
- `interface_language`
- `interface_ability`
- `interface_purchase`
- `interface_reward`
- `interface_home`
- `interface_theater`
- `interface_main`
- `interface_report`
- `interface_profile`
- `interface_widget`

该层的意义不是“放公共工具”，而是“放跨 feature 协议”。

### shared
共享业务能力层，模块最密集：
- `shared_ad`
- `shared_af`
- `shared_analytics`
- `shared_api`
- `shared_base`
- `shared_general`
- `shared_im`
- `shared_injector`
- `shared_kv`
- `shared_models`
- `shared_navigation`
- `shared_novel`
- `shared_player`
- `shared_purchase`
- `shared_push`
- `shared_resource`
- `shared_share`
- `shared_skin`
- `shared_toast`
- `shared_ui`
- `shared_user`
- `shared_web`

其中最像“业务中台”的模块是：
- `shared_player`
- `shared_ad`
- `shared_af`
- `shared_purchase`
- `shared_user`
- `shared_analytics`

### core
技术基础设施层：
- `core_analytics`
- `core_apm`
- `core_bus`
- `core_common`
- `core_config`
- `core_db`
- `core_device_locale`
- `core_im`
- `core_image`
- `core_json`
- `core_kit`
- `core_kv`
- `core_log`
- `core_mvi`
- `core_network`
- `core_player_api`
- `core_router`
- `core_startup`
- `core_toast`
- `core_web`

## 重点模块信号
### feature_home
- 首页/ForYou/剧详情/播放器相关逻辑高度集中
- 同时依赖播放器、广告、API、analytics、purchase、push、AF、general 等多个共享模块
- 存在 `dramawaveImplementation(shared_im)`，说明 flavor 差异直达业务模块级别

### feature_reward
- 依赖面极大，很多依赖是 `api(...)`
- 说明奖励体系可能被多个业务链路复用，具有平台型业务属性

### shared_ad
- 聚合 AppLovin MAX、Google AdMob、Pangle、Tapjoy、FairBid、Unity、Opera、TaurusX 等多广告平台与适配器
- 是典型的第三方 SDK 聚合重灾区

### shared_player
- 包装腾讯 LiteAVSDK Premium
- 同时依赖 `core_apm`、`core_db`、`core_network`、`shared_models`、`shared_analytics`
- 是播放体验、缓存、下载、状态机的中心模块

### core_network
- 集成 Retrofit + OkHttp + Cronet GMS
- 是基础网络链路的汇总点

### core_apm
- 集成 Firebase Crashlytics / Perf
- debug 下接 LeakCanary + KOOM

## 文件量与复杂度分布
粗略统计：
- `feature`: 3147 文件 / 1542 Kotlin
- `shared`: 2661 文件 / 1318 Kotlin
- `core`: 622 文件 / 413 Kotlin / 77 Java

这说明：
- 业务逻辑主量级在 `feature + shared`
- 技术基建在 `core`
- Java 仍未完全退出舞台，主要留在基础设施或历史兼容区域

## 模块地图的阅读建议
理解整个工程时，推荐按这个顺序阅读：
1. `app`：看壳层如何组织系统
2. `core`：理解网络、路由、KV、MVI、启动框架
3. `shared`：理解播放器、广告、归因、用户、埋点等共享业务系统
4. `feature_home` / `feature_reward` / `feature_theater`：理解主要业务闭环
5. `interface`：理解 feature 之间是如何解耦的

## 关联页面
- [[dramawave-codebase-overview]]
- [[dramawave-build-and-architecture]]
- [[dramawave-startup-and-entry]]
- [[dramawave-technology-stack]]
- [[dramawave-codebase-risks]]
