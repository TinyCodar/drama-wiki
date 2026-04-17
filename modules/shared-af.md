---
title: shared_af deep dive
created: 2026-04-17
updated: 2026-04-17
type: module
layer: shared
status: verified
tags: [shared, attribution, appsflyer, deeplink, deep-dive]
sources:
  - shared/shared_af/src/main/java/com/dramawave/shared/af/component/AppsFlyerInitializer.kt
  - shared/shared_af/src/main/java/com/dramawave/shared/af/manager/AttributionManager.kt
  - shared/shared_af/src/main/java/com/dramawave/shared/af/task/AttributionTaskManager.kt
  - shared/shared_af/src/main/java/com/dramawave/shared/af/viewmodel/AttributionViewModel.kt
  - shared/shared_af/src/main/java/com/dramawave/shared/af/component/AttributionResult.kt
  - app/src/main/java/com/dramawave/app/startup/loader/ApplicationLoader.kt
  - app/src/main/java/com/dramawave/app/MainActivity.kt
  - app/src/main/java/com/dramawave/app/main/viewmodel/MainAttributionViewModel.kt
---

# shared_af deep dive

## 定位
`shared_af` 是 DramaWave 的统一归因中枢模块。它上承 AppsFlyer、Meta、Google、剪切板等多种归因来源，下接 `MainActivity`、剧场、播放详情、奖励、小说等业务场景，负责归因结果汇总、优先级决策、首启倒计时与 fallback 开剧，以及奖励/邀请/DDL 等承接流程。

它不只是 AppsFlyer SDK 封装，更是“归因流程编排层”。

## 核心职责
- 初始化 AppsFlyer 及多渠道归因任务
- 汇总多来源归因结果为 `AttributionResult`
- 按策略选择 top attribution（预设优先级或 last-click）
- 处理首启必要任务等待、超时、兜底与实验分流
- 分发归因总线事件给首页、剧场、详情、奖励等业务
- 承接社交邀请、奖励归因、Growth Deeplink、DDL 等链路

## 架构概览
- `AppsFlyerInitializer`：启动时注册各归因渠道任务
- `AttributionTaskManager`：必要/非必要任务调度与超时管理
- `AttributionManager`：归因结果聚合、优先级判定、兜底结果缓存
- `AttributionResult`：统一归因结果模型与派生语义
- `AttributionViewModel`：首启倒计时、新手引导、fallback 开剧编排
- `MainAttributionViewModel`：主壳层最终承接与业务分发
- `TheaterInfoProvider`：通过 TheaterProxy 向 shared_af 提供推荐剧集信息，避免直接依赖 feature 实现

## 关键源码
### 启动初始化
- `shared/shared_af/src/main/java/com/dramawave/shared/af/component/AppsFlyerInitializer.kt`
- `app/src/main/java/com/dramawave/app/startup/loader/ApplicationLoader.kt`

启动阶段会把 CLIPBOARD、APPSFLYER、META_LINK、META_INSTALL_REFERRER、GOOGLE_INSTALL_REFERRER、GOOGLE_DDL、TIKTOK_DDL 等任务注册到 `AttributionTaskManager`。同时支持从 `/data/local/tmp/attr_mock.json` 读取 mock 配置，便于调试归因链路。

### 结果聚合与优先级
- `shared/shared_af/src/main/java/com/dramawave/shared/af/manager/AttributionManager.kt`
- `shared/shared_af/src/main/java/com/dramawave/shared/af/component/AttributionResult.kt`

`AttributionManager` 维护进程内归因结果列表，负责：
- `addAttr(...)` 增量追加结果
- 发出 `AttributionResultEvent`
- 修正 MetaLink / MetaInstallReferrer 时间戳
- 根据 PRESET 或 LAST_CLICK 规则选 top attribution
- 提供 `getCurrentTrialVipCampaign()`、`shouldShowForYouFlow()`、`getValidSocialAttributionResult()` 等高层语义 API
- 维护 server probabilistic attribution 的 pending fallback 结果

`AttributionResult` 统一封装：
- `source`
- `deeplink`
- `campaign`
- `channel`
- `clickTimestamp`
- `extra`

并派生：
- `type`
- `contentId`
- `supportJump`
- `isTrialVip`
- `isZeroGift`
- `shouldShowForYouFlow`
- `isValidSocialAttribution`

### 任务调度
- `shared/shared_af/src/main/java/com/dramawave/shared/af/task/AttributionTaskManager.kt`

`AttributionTaskManager` 负责：
- 注册全部 `AttributionTask`
- 区分必要任务与非必要任务
- 等待必要任务完成或超时
- 通过 `taskCompletionState` 暴露 `isNecessaryCompleted` / `isTimeout` / `isInterrupted` / `isProcessed`
- 在首启场景异步启动 server probabilistic attribution 作为兜底补偿

### 首启引导与 fallback 开剧
- `shared/shared_af/src/main/java/com/dramawave/shared/af/viewmodel/AttributionViewModel.kt`
- `shared/shared_af/src/main/java/com/dramawave/shared/af/provider/TheaterInfoProvider.kt`
- `shared/shared_af/src/main/java/com/dramawave/shared/af/provider/TheaterInfoProviderImpl.kt`

`AttributionViewModel` 负责：
- 新用户引导显示/隐藏
- 首启倒计时
- 归因完成后的收敛或继续倒计时
- 倒计时完成时尝试 `applyPendingServerProbAttr()`
- 按实验配置决定是否执行 fallback 开剧

fallback 开剧的数据来源不直接依赖 `feature_theater`，而是通过 `TheaterInfoProvider -> TheaterProxy` 间接读取推荐剧集信息。

## 关键事件
- `AttributionResultEvent`
- `AttributionRepairEvent`
- `RewardsAttributionEvent`
- `GrowthDeeplinkEvent`

这些事件把归因结果从 shared_af 分发给上层业务页面与业务协调器。

## 上游接入面
### 启动与主壳层
- `app/src/main/java/com/dramawave/app/startup/loader/ApplicationLoader.kt`
- `app/src/main/java/com/dramawave/app/MainActivity.kt`
- `app/src/main/java/com/dramawave/app/main/viewmodel/MainAttributionViewModel.kt`

典型链路：
1. `ApplicationLoader` 注册 AppsFlyer 初始化器
2. `MainActivity` 首次 layout 后触发 `executeClipboardTask()`
3. `MainAttributionViewModel` 调用 `AttributionTaskManager.startWaitingNecessary(...)`
4. 必要任务完成或超时后，读取 `AttributionManager.getTopAttrResult()`
5. 根据归因类型执行剧集跳转、DDL、奖励承接、ForYou 流等业务逻辑

### 剧场、首页、详情、奖励、小说
已验证的典型使用方包括：
- `feature/feature_theater/.../TheaterHomeFragment.kt`
- `feature/feature_home/.../DramaAttributionProcessor.kt`
- `feature/feature_home/.../PlayDetailFragment.kt`
- `feature/feature_home/.../HostLinker.kt`
- `feature/feature_reward/.../NewbieWelfareHintDialogNew.kt`
- `feature/feature_novel/.../ReaderFragment.kt`
- `feature/feature_novel/.../NovelContentDetailFragment.kt`

说明 shared_af 是真正的跨业务共享模块，不局限于安装归因，而是深入参与首页推荐、剧场引导、福利弹层与内容详情承接。

## 依赖关系
下游依赖：
- AppsFlyer / Meta / Google 相关归因渠道实现
- `shared_user`、`shared_general`、`shared_models`
- `core_bus`
- 通过 `TheaterProxy` 间接连接剧场推荐能力

上游使用者：
- `app`
- `feature_home`
- `feature_theater`
- `feature_reward`
- `feature_novel`
- 其他需要消费归因结果或引导状态的业务模块

## 风险与备注
- `AttributionManager` 的 `_attrResults` 是进程内内存态，进程重启后需依赖各渠道重新回流
- `AttributionResult.type` 的计算带有 `UserStore` 写入副作用，数据模型与状态写入耦合偏重
- LAST_CLICK 排序实现和“priority 越小越优先”的语义存在理解门槛，后续应复核排序意图
- Meta 时间戳修正逻辑既在 `addAttr` 做一次，也在排序时动态修正一次，维护成本较高
- server probabilistic attribution 不在普通任务列表中，而是首启时额外异步触发，时序理解成本较高
- 模块同时承担 SDK 初始化、任务调度、实验逻辑、UI 引导与业务承接，边界偏宽，后续继续演进时应警惕复杂度继续上升

## 相关页面
- [[dramawave-overview]]
- [[home-theater-attribution-network-mvi-overview]]
- [[feature-home-and-play-entry]]
