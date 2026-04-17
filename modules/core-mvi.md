---
title: core_mvi deep dive
created: 2026-04-17
updated: 2026-04-17
type: module
layer: core
status: verified
tags: [core, mvi, stateholder, hilt, compose, deep-dive]
sources:
  - core/core_mvi/AGENTS.md
  - core/core_mvi/build.gradle.kts
  - core/core_mvi/src/main/java/com/dramawave/core/mvi/BaseHilt.kt
  - core/core_mvi/src/main/java/com/dramawave/core/mvi/architecture/StateHolder.kt
  - core/core_mvi/src/main/java/com/dramawave/core/mvi/architecture/MviExt.kt
  - core/core_mvi/src/main/java/com/dramawave/core/mvi/architecture/StateHolderOwner.kt
  - shared/shared_base/src/main/java/com/dramawave/shared/base/activity/BaseA.kt
  - shared/shared_base/src/main/java/com/dramawave/shared/base/fragment/BaseF.kt
  - app/src/main/java/com/dramawave/app/MainActivity.kt
---

# core_mvi deep dive

## 定位
`core_mvi` 是 DramaWave 的项目级 MVI 基础设施模块，统一承载 ViewModel 状态、一次性事件、intent 执行 DSL，以及 Hilt Activity/Fragment/Dialog 基类。

它同时覆盖传统 View 页面和 Compose 页面，是整个工程里状态驱动 UI 的基础设施核心之一。

## 核心职责
- 提供 `StateHolder` 作为状态与事件的统一运行时容器
- 提供 `intent / awaitIntent / interceptIntent / intentChain` 等 DSL
- 提供 `observe`、`observeWithoutLifecycle`、`collectAsState`、`collectEvents` 等接入 API
- 提供 `BaseHiltActivity` / `BaseHiltFragment` / `BaseHiltDialog` 统一 Hilt 基类
- 为 ViewModel 的延迟初始化提供 `LazyCreateHolderDecorator`

## 核心结构
### BaseHilt 基类
- `core/core_mvi/src/main/java/com/dramawave/core/mvi/BaseHilt.kt`

包含：
- `BaseHiltActivity : AppCompatActivity`
- `BaseHiltFragment : Fragment`
- `BaseHiltDialog : DialogFragment`

三者都带 `@AndroidEntryPoint`，为页面层提供统一的 Hilt 注入入口。

### MVI 核心接口
- `core/core_mvi/src/main/java/com/dramawave/core/mvi/architecture/MviWrapper.kt`

定义：
- `stateFlow: StateFlow<STATE>`
- `eventsFlow: Flow<EVENT>`
- `run(...)`
- `awaitRun(...)`
- `interceptRun(...)`
- `cancel()`
- `joinIntents()`

它是 `StateHolder` 与装饰器体系共同遵循的核心契约。

### StateHolder
- `core/core_mvi/src/main/java/com/dramawave/core/mvi/architecture/StateHolder.kt`

`StateHolder` 是真实的 MVI 运行时容器：
- 用 `MutableStateFlow` 保存最新 state
- 用 `MutableSharedFlow` 发送一次性 event
- 用 `Channel.UNLIMITED` 分发 intent
- 使用 `CoroutineExceptionHandler` 兜底未处理异常并记录日志

调度常量：
- `EVENT_COROUTINE_CONTEXT = Dispatchers.Default`
- `INTENT_COROUTINE_CONTEXT = Dispatchers.IO`

### DSL 上下文
- `HolderContext.kt`
- `Dispatcher.kt`
- `StateContext.kt`
- `MviDsl.kt`

这组类共同支持以下写法：
- `reduce { state.copy(...) }`
- `postEvent(...)`
- `val current = state`

## 标准接入方式
### ViewModel 侧
典型模式是：
- ViewModel 实现 `StateHolderOwner<STATE, EVENT>`
- `override val holder = holder(InitialState()) { ... }`
- 业务动作中使用 `intent { reduce { ... }; postEvent(...) }`

代表性使用点：
- `feature/feature_home/.../HomeFeedViewModel.kt`
- `feature/feature_home/.../DramaSeriesViewModel.kt`
- `shared/shared_general/.../GlobalViewModel.kt`

### 页面侧观察
通过 `MviExt.kt` 提供：
- `observe(...)`
- `observeWithoutLifecycle(...)`
- `collectAsState()`
- `collectEvents()`
- `collectEventsWithoutLifecycle()`

代表性使用点：
- `app/src/main/java/com/dramawave/app/MainActivity.kt`
- `feature/feature_home/src/main/kotlin/com/dramawave/feature/home/HomeFragment.kt`
- `feature/feature_novel/src/main/kotlin/com/dramawave/feature/novel/ReaderFragment.kt`

### 通过 shared_base 扩散到页面体系
- `shared/shared_base/src/main/java/com/dramawave/shared/base/activity/BaseA.kt`
- `shared/shared_base/src/main/java/com/dramawave/shared/base/fragment/BaseF.kt`

大量页面并不直接继承 `BaseHiltActivity` / `BaseHiltFragment`，而是通过 `shared_base` 的 `BaseA` / `BaseF` 间接接入，因此 core_mvi 实际已经是页面基础设施链路的一部分。

## 延迟初始化机制
`holder(initialState) { ... }` 背后会创建 `LazyCreateHolderDecorator`：
- 首次订阅 state / event 或首次执行 intent 时才触发 `onCreate`
- 使用 `AtomicBoolean` 保证只执行一次

这能避免 ViewModel 一创建就立即执行初始化逻辑，使页面状态初始化更可控。

## 上游使用面
已验证直接依赖 `:core:core_mvi` 的模块包括：
- `feature_home`
- `feature_profile`
- `feature_reward`
- `feature_ability`
- `feature_mylist`
- `shared_ad`
- `shared_purchase`
- `shared_ui`
- `shared_web`
- `shared_af`
- `shared_general`
- `shared_base`
- `core_common`

这说明 core_mvi 不是某个业务模块专属能力，而是整个工程状态管理范式的基础底座。

## 依赖关系
见：
- `core/core_mvi/build.gradle.kts`

主要依赖：
- AndroidX lifecycle
- activity-compose
- Compose BOM / ui / material3
- appcompat
- Hilt Android + kapt
- hilt-navigation-compose

说明它同时服务于：
- 传统 View 页面
- Compose 页面
- Hilt 注入场景

## 风险与备注
- `MviWrapper.kt` 的 package 仍是 `com.youyue.hx.mvi.architecture`，和其余 `com.dramawave.core.mvi.architecture` 命名不一致，属于明显历史遗留
- `StateHolder` 的异常处理当前以 `Log.e` 为主，注释提到可接 APM，但现阶段并未形成统一监控闭环
- `dispatchChannel` 使用 `Channel.UNLIMITED`，高频 intent 场景理论上存在积压风险
- `eventsFlow` 的 `extraBufferCapacity = Int.MAX_VALUE`，在极端情况下可能放大内存压力
- `observeWithoutLifecycle` 不随页面生命周期自动暂停，适合特定场景，但使用不当容易让后台页面持续消费事件
- `StateHolderOwner.intent(...)` 等扩展内部默认把 owner 当作 `ViewModel` 使用，虽然接口层看起来更抽象，实际接入仍隐含“owner 应该是 ViewModel”的约束

## 相关页面
- [[dramawave-overview]]
- [[architecture-layering]]
- [[home-theater-attribution-network-mvi-overview]]
- [[feature-home-and-play-entry]]
