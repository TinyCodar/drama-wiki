---
title: feature_home 播放入口总览
created: 2026-04-16
updated: 2026-04-16
type: module
layer: feature
status: verified
tags: [feature, home, player, flow, deep-dive]
sources:
  - feature/feature_home/AGENTS.md
  - shared/shared_player/CLAUDE.md
---

# feature_home 播放入口总览

## 定位
`feature_home` 是 DramaWave 的首页业务模块，同时也是短剧播放主入口之一，覆盖首页 feed、ForYou 推荐播放、剧集详情播放、广告辅助逻辑与部分插件式能力。

## 与播放器的关系
该模块自身不实现底层播放器，而是作为上游业务层接入 `shared_player`：
- 页面/Fragment 决定何时开始播放、切换视频、暂停恢复
- `shared_player` 负责 controller、状态机、缓存、下载与 LiteAV SDK 封装

## 关键类
- `HomeFragment`：首页 tab 入口
- `HomeViewModel`：feed 数据加载与分页
- `AbstractPlayerFragment`：播放器 Fragment 基类
- `ForYouPlayerFragment`：推荐流播放
- `SeriesPlayerFragment`：剧集播放
- `VideoPagerAdapter`：ViewPager2 视频滑动适配
- `DramaSeriesActivity`：剧集详情主入口
- `PlayDetailActivity`：旧版详情页

## 目录关注点
- `player/`：播放器相关 Fragment 与 adapter
- `detail/`：剧集详情页及协调器
- `ad/`：详情页广告工具
- `architecture/plugins/`：VIP 引导、notice 等插件系统

## 典型链路
1. `HomeFragment` 或详情场景装载 feed / 剧集数据
2. `VideoPagerAdapter` 管理视频页切换
3. `ForYouPlayerFragment` / `SeriesPlayerFragment` 进入播放场景
4. 页面通过 `shared_player` 获取 controller 并订阅状态
5. 页面同时可能与 `shared_ad` 协作，处理解锁广告、详情广告等逻辑

## 风险与备注
- 这是“业务播放入口层”，阅读播放问题时不能只看 `feature_home` 或只看 `shared_player`，两边必须联动看
- 详情页还夹带广告、归因、插件系统等复杂逻辑，是重要复杂度聚集点

## 相关页面
- [[shared-player]]
- [[player-system-overview]]
- [[shared-ad]]
