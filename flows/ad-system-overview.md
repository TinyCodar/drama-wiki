---
title: 广告系统总览
created: 2026-04-16
updated: 2026-04-16
type: flow
layer: shared
status: active
tags: [shared, ad, flow, architecture, cache]
sources:
  - shared/shared_ad/AGENTS.md
  - shared/shared_ad/CLAUDE.md
---

# 广告系统总览

## 一句话说明
DramaWave 广告系统以 `shared_ad` 为核心，对外提供统一广告入口，对内同时管理平台适配、缓存水位、场景/点位、数据服务、回退策略与业务事件分发。

## 典型链路
1. 应用或合适时机初始化 `AdSDK`
2. `AdService` 拉取广告策略、场景配置、点位数据
3. `AdManager` 注册平台并准备 `AdCachePool`
4. 业务侧按 `scene + site + type` 发起取广告请求
5. `AdCacheQueue` 判断是否已有 ready 广告，不足则补水
6. 如主平台失败，依据 `AdRules` / `replaceAd` 执行 fallback
7. 广告展示后通过统一 callback 回传奖励、关闭、失败等结果

## 系统分层
- 入口层：`AdSDK`、`AdBuilder`
- 调度层：`AdManager`、`AdCachePool`、`AdCacheQueue`
- 数据层：`AdService`
- 业务层：`AdHandler`、场景工厂、拦截器链
- 平台层：MAX / AdMob / Meta / Pangle / 更多平台
- 展示与交互层：dialog、widget、interaction style

## 核心抽象
- `AdScene`：业务场景
- `AdSite`：广告点位
- `AdType`：广告类型
- `AdPlatform`：广告平台

## 适用业务
- 剧集解锁
- 签到
- 首页外流
- 小说阅读相关广告
- 退出播放器
- 开屏广告
- 奖励页任务

## 理解建议
阅读该系统时，不要只看 `AdSDK`。至少要同时看：
- `AdManager`
- `AdService`
- `AdCachePool / AdCacheQueue`
- `biz/scene`
- `platform/*`

## 相关页面
- [[shared-ad]]
- [[module-map]]
- [[startup-and-apm-init-flow]]
