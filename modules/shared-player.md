---
title: shared_player deep dive
created: 2026-04-16
updated: 2026-04-16
type: module
layer: shared
status: verified
tags: [shared, player, state-machine, cache, download, deep-dive]
sources:
  - shared/shared_player/AGENTS.md
  - shared/shared_player/CLAUDE.md
  - feature/feature_home/AGENTS.md
---

# shared_player deep dive

## 定位
`shared_player` 是 DramaWave 的播放器包装层，围绕 Tencent LiteAVSDK Premium 构建，支持短剧竖滑播放、全局状态机、播放器控制器复用、CDN 重试、缓存/下载与叠层系统。

## 架构概览
- `PlayerSDK`：播放器能力入口与配置初始化
- `PlayerManager`：控制器生命周期总管
- `PlayerControllerCache`：LRU 缓存，默认最多 2 个 controller
- `PlayerController`：对 `IPlayer` 的操作包装
- `PlayerStateManager`：全局状态机
- `PlaybackEventDispatcher`：播放事件到状态流转的桥
- `cdn/`：CDN 重试与健康监控
- `manager/`：下载、缓存、字幕、单视频缓存、全局播放场景等管理器
- `layer/`：水印、弹窗等叠层管理
- `view/`：`ShortVideoSceneView`、`DirectionalVideoPager`

## 关键能力
- 短剧 feed/详情播放
- LRU 复用控制器，降低内存压力
- 全局状态机统一播放状态
- 播放事件 typed event 化
- CDN retry / health 管理
- 观看时长和集数统计
- 视频缓存与下载
- 播放器叠层系统

## 关键类
- `PlayerSDK`：初始化与配置
- `PlayerManager`：获取/释放 controller
- `PlayerControllerCache`：LinkedHashMap LRU 容器
- `PlayerController`：播放控制封装
- `PlayerStateManager`：PLAY/PAUSE/STOP/LOADING/ERROR/BIND/UNBIND 等状态
- `TraceableVodPlayer`：带分析追踪的播放器代理
- `WatchStatsManager`：观看统计持久化
- `GlobalPlayerManager`：单播放场景协调与 AppsFlyer 触发
- `VideoCacheManager` / `VideoDownloadManager`：缓存与下载
- `LayerManager`：叠层系统

## 业务接入面
- `feature_home` 中的 `ForYouPlayerFragment`、`SeriesPlayerFragment` 等页面是典型上游场景
- 首页、详情、剧场等业务通过 `PlayerManager.getController(...)` 获取控制器
- UI 层监听 `PlayerStateManager` 状态变化驱动播放交互

## 依赖关系
下游依赖：
- `core/core_player_api`
- Tencent LiteAVSDK
- `core_common`、`core_apm`、`core_kv`、`core_config`、`core_db`、`core_network`
- `shared_ui`、`shared_models`、`shared_analytics`

上游使用者：
- `feature_home`
- 可能的 `feature_theater`、其他视频/详情/推荐场景

## 风险与备注
- LRU 默认上限为 2，属于关键运行时策略，直接影响切页体验与内存占用
- CDN retry、播放状态机、缓存下载、统计采集耦合在同一模块，阅读时需按主题分层
- 若继续扩 wiki，建议拆出 `player-state-machine`、`player-cdn-retry`、`player-download-and-cache` 子页

## 相关页面
- [[player-system-overview]]
- [[feature-home-and-play-entry]]
- [[shared-ad]]
