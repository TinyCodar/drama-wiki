# DramaWave Wiki Index

> DramaWave 工程 LLM Wiki 内容索引。
> 用于快速导航代码库知识、模块 deep dive、运行时链路与工程约束。
> Last updated: 2026-04-16 | Total pages: 12

## Overview
- [[dramawave-overview]] - DramaWave 工程总览，说明分层结构、技术栈、构建方式与知识库使用方式。
- [[architecture-layering]] - 分层规则与依赖约束，概括 app/feature/interface/shared/core 的边界。
- [[home-theater-attribution-network-mvi-overview]] - 串联 feature_home、feature_theater、shared_af、core_network、core_mvi 的跨模块主链路总览。

## Inventory
- [[module-map]] - 顶层模块地图，列出 app、feature、interface、shared、core、docs、scripts 的主要入口。
- [[docs-and-scripts-entrypoints]] - 项目内文档与脚本入口清单，方便继续扩展 wiki 时查源。

## Modules
- [[core-apm]] - core_apm deep dive，覆盖 Crashlytics、Firebase Perf、设备评分、LeakCanary/KOOM 与线程治理。
- [[shared-ad]] - shared_ad deep dive，覆盖广告聚合 SDK、缓存池、场景/点位、平台适配与业务入口。
- [[shared-player]] - shared_player deep dive，覆盖播放器包装、LRU 控制器、全局状态机、CDN 重试与下载缓存。
- [[feature-home-and-play-entry]] - feature_home 播放入口总览，聚焦首页、ForYou、剧集详情与 shared_player 的衔接。

## Flows
- [[startup-and-apm-init-flow]] - 启动与 APM 初始化路径，说明应用启动时 APM、播放器、广告等基础能力的接入位置。
- [[ad-system-overview]] - 广告系统总览，串联 AdSDK、AdManager、AdService、场景/点位与业务调用方式。
- [[player-system-overview]] - 播放器系统总览，串联 PlayerSDK、PlayerManager、PlayerStateManager 与页面侧接入方式。
