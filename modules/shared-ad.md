---
title: shared_ad deep dive
created: 2026-04-16
updated: 2026-04-16
type: module
layer: shared
status: verified
tags: [shared, ad, cache, state-machine, deep-dive]
sources:
  - shared/shared_ad/AGENTS.md
  - shared/shared_ad/CLAUDE.md
---

# shared_ad deep dive

## 定位
`shared_ad` 是 DramaWave 的多广告平台聚合与统一调度层，对业务暴露单一入口 `AdSDK`，内部整合点位、场景、缓存池、回退规则、平台适配与 UI/业务联动。

## 架构概览
核心层次可概括为：
- `AdSDK`：统一入口
- `AdManager`：核心调度器
- `AdCachePool -> AdCacheQueue`：缓存池与单点位队列
- `AdService`：广告数据加载、缓存、重试、广告免打扰倒计时
- `biz/`：事件驱动场景分发、拦截器链、业务场景组装
- `platform/`：MAX、AdMob、Meta、Pangle 等平台接入
- `util/interactionstyle/`：广告交互样式策略
- `viewmodel/AdViewModel`：结合 MVI 的解锁/广告业务编排

## 关键能力
- Rewarded / Interstitial / Banner / Native / AppOpen / Offerwall 等广告类型
- scene/site/type 多维度建模
- 预加载与水位补充
- polling 等待广告 ready
- 失败 fallback 与 replaceAd 机制
- Push-to-player 延迟初始化
- GDPR consent 管理
- 小说、VIP、首页外流等差异化业务场景

## 关键类
- `AdSDK`：init、loadAdData、getAd 统一入口
- `AdBuilder`：Builder 模式构建取广告请求
- `AdManager`：平台注册、状态流转、缓存池调度
- `AdService`：广告数据服务与场景数据管理
- `AdCachePool` / `AdCacheQueue`：缓存调度核心
- `AdHandler`：事件驱动场景分发
- `AdRules`：fallback 链路规则
- `DelayAdInitManager`：特殊启动场景延迟初始化
- `AdInteractionStyleHelper`：交互样式门面
- `AdViewModel`：广告业务与解锁流程 ViewModel

## 关键场景
文档显示已建模场景包括但不限于：
- `DRAMA_FREE`
- `DRAMA_VIP_ADS`
- `HOME_OUT_FLOW`
- `NOVEL_FREE`
- `UNLOCK_IAP`
- `CHECK_IN`
- `APP_OPEN`
- `WATCH_ADS`
- `QUIT_PLAYER`

## 依赖关系
下游依赖十分广：
- 广告平台 SDK 与 adapter
- `shared_resource`、`shared_analytics`、`shared_api`、`shared_user`
- `shared_models`、`shared_toast`、`shared_base`、`shared_push`、`shared_purchase`
- `core_mvi`、`core_network`、`core_bus`、`core_player_api` 等多个基础模块

上游使用者：
- 首页、详情页、奖励页、小说场景、退出播放、开屏等业务模块

## 典型使用方式
- 启动或合适时机先 `AdSDK.init()` + `AdSDK.loadAdData()`
- 业务使用 `AdBuilder` 或 `AdSDK.getRewardedAdWithPolling()` 等入口取广告
- 取到广告后通过统一 `show(...)` 回调消费奖励/关闭事件

## 风险与备注
- 这是一个高耦合、高平台适配密度模块，改造成本高
- 场景枚举、点位枚举、回退规则与缓存策略共同决定线上行为，单点修改风险大
- 若继续扩 wiki，适合拆出 `ad-scenes-and-sites`、`ad-platform-layer`、`reward-ad-flow` 等子页

## 相关页面
- [[ad-system-overview]]
- [[shared-player]]
- [[module-map]]
