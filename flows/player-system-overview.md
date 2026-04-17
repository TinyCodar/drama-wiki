---
title: 播放器系统总览
created: 2026-04-16
updated: 2026-04-16
type: flow
layer: shared
status: active
tags: [shared, player, flow, state-machine, cache]
sources:
  - shared/shared_player/AGENTS.md
  - shared/shared_player/CLAUDE.md
  - feature/feature_home/AGENTS.md
---

# 播放器系统总览

## 一句话说明
DramaWave 播放器系统以 `shared_player` 为中心，为首页 ForYou、剧集详情等短剧场景提供统一播放控制、状态机、控制器复用、缓存下载和 CDN 重试能力。

## 典型链路
1. 业务页面（如 `feature_home` 的播放器 Fragment）请求播放某个视频源
2. 通过 `PlayerManager.getController(dataSource)` 获取或复用 controller
3. `PlayerControllerCache` 按 LRU 策略管理 controller 实例
4. controller 底层驱动 `IPlayer` / LiteAVSDK
5. 播放过程中通过 `PlaybackEventDispatcher` 将事件映射到 `PlayerStateManager`
6. UI 层监听状态变化，驱动 loading、pause、resume、error 等交互
7. 统计、缓存、CDN retry、下载等配套管理器在旁路协同工作

## 关键组成
- 入口：`PlayerSDK`
- 控制器管理：`PlayerManager`、`PlayerControllerCache`
- 状态：`PlayerStateManager`
- 事件桥：`PlaybackEventDispatcher`
- 播放代理：`TraceableVodPlayer`
- 缓存下载：`VideoCacheManager`、`VideoDownloadManager`
- 容器视图：`ShortVideoSceneView`、`DirectionalVideoPager`
- 叠层：`LayerManager`

## 上游业务场景
- 首页推荐视频流
- 剧集详情页播放
- 可能的剧场页视频内容

## 阅读建议
如果目的是排查“为什么这条视频没有播起来”，通常需要同时检查：
- 业务 Fragment / Adapter 是否正确请求 controller
- `PlayerManager` 是否复用到预期实例
- `PlayerStateManager` 是否进入异常状态
- CDN retry 是否触发
- 视频源与缓存策略是否命中

## 相关页面
- [[shared-player]]
- [[startup-and-apm-init-flow]]
- [[dramawave-overview]]
