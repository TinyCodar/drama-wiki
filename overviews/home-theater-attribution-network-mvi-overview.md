---
title: feature_home / feature_theater / shared_af / core_network / core_mvi 扩展总览
created: 2026-04-16
updated: 2026-04-16
type: overview
layer: cross-cutting
status: verified
tags: [feature, home, theater, attribution, network, mvi]
sources:
  - feature/feature_home/AGENTS.md
  - feature/feature_home/src/main/kotlin/com/dramawave/feature/home/HomeFeedFragment.kt
  - feature/feature_home/src/main/kotlin/com/dramawave/feature/home/detail/coordinator/DramaCoordinator.kt
  - feature/feature_home/src/main/kotlin/com/dramawave/feature/home/detail/coordinator/processors/DramaAttributionProcessor.kt
  - feature/feature_home/src/main/kotlin/com/dramawave/feature/home/architecture/pager/adapter/VideoPagerAdapter.kt
  - feature/feature_theater/CLAUDE.md
  - feature/feature_theater/src/main/kotlin/com/dramawave/feature/theater/TheaterHomeFragment.kt
  - feature/feature_theater/src/main/kotlin/com/dramawave/feature/theater/TheaterSubTabFragment.kt
  - feature/feature_theater/src/main/kotlin/com/dramawave/feature/theater/viewmodel/TheaterSubTabViewModel.kt
  - shared/shared_af/AGENTS.md
  - shared/shared_af/src/main/java/com/dramawave/shared/af/manager/AttributionManager.kt
  - core/core_network/AGENTS.md
  - core/core_network/src/main/java/com/dramawave/core/network/interceptor/DdnsInterceptor.kt
  - core/core_network/src/main/java/com/dramawave/core/network/interceptor/BackupDomainInterceptor.kt
  - core/core_mvi/AGENTS.md
  - core/core_mvi/src/main/java/com/dramawave/core/mvi/architecture/StateHolder.kt
  - core/core_mvi/src/main/java/com/dramawave/core/mvi/architecture/MviExt.kt
---

# feature_home / feature_theater / shared_af / core_network / core_mvi 扩展总览

## 定位
这 5 个模块共同构成 DramaWave 里“内容入口 -> 归因判断 -> 网络请求 -> 状态机驱动 UI”的高频主链路：
- `feature_home` 负责首页短剧 feed 与详情播放入口
- `feature_theater` 负责剧场主 Tab、多级分类页与新用户引导/继续播放容器
- `shared_af` 负责多来源归因汇聚、排序、兜底与事件分发
- `core_network` 负责域名替换、备用域名重试、拦截器链与底层请求容灾
- `core_mvi` 负责 ViewModel 侧状态状态机、intent 调度与 UI 观察接口

## 为什么这几块要一起看
单独阅读任何一个模块都不够：
- 播放入口页常直接依赖归因结果决定打开逻辑、福利流或新用户引导
- 归因模块会通过 bus 将结果投递给首页/剧场等业务页面
- 页面侧 ViewModel 与 Fragment 又统一建立在 `core_mvi` 的 StateHolder/intent 模型上
- 所有业务请求最终经 `core_network` 的拦截器链处理，尤其是 DDNS 与备用域名容灾

## 模块关系速览
1. 页面入口从 `feature_home` 或 `feature_theater` 发起
2. 页面/Processor/ViewModel 使用 `core_mvi` 管理状态与事件
3. 归因结果由 `shared_af` 聚合并通过 bus 广播给页面
4. 页面进一步请求接口时走 `core_network`
5. 结果回到 ViewModel，通过 `StateHolder` 更新 UI

## feature_home：播放入口与 Processor 协调层

### 关键职责
`feature_home` 不自己实现底层播放器，而是承担首页与播放详情的业务编排：
- 首页 feed 的初始化、分页、滚动与跳转
- 播放详情页 Processor 编排
- 归因事件接入与首次归因触发
- 与 `shared_player` 的 Fragment/控制器衔接

### 首页入口
`feature/feature_home/src/main/kotlin/com/dramawave/feature/home/HomeFeedFragment.kt`
是首页主 feed Fragment，能直接看出首页页层职责：
- 在 `onViewCreated()` 里初始化 `PlayParams`、`Tracer`、Pager，并先执行关键 Processor
- 在 `getProcessors()` 里装配 `ForYouDataProcessor`、`ForYouAdProcessor`、`ForYouResumeSkipPlayProcessor` 等处理器
- `configureAdapter()` 中为 `VideoPagerAdapter` 注册 load more
- `onResume()` 中会做广告拦截环境检查、恢复协调器并上报首页展示

这个类说明 `feature_home` 的核心不是“单 ViewModel 页面”，而是“页面 + Coordinator + Processor 集群”的编排模型。

### Processor 协调器
`feature/feature_home/src/main/kotlin/com/dramawave/feature/home/detail/coordinator/DramaCoordinator.kt`
把详情页 Processor 当作一组可分批初始化的组件管理：
- `dispatcher.ready { hasReadyProcessors }` 将已就绪 Processor 列表暴露给 ViewModel 中心
- `dispatch(onlyCritical)` 支持先初始化关键处理器，再延迟初始化非关键处理器
- `initializeProcessor()` 先把 Processor 放进 ready 列表，再执行 `ready()` 和 `onCreate()`，保证 Processor 间可互访
- `onResume/onPause/onDestroy/onPageChanged()` 等生命周期统一下发给所有 Processor

它本质上是首页/详情复杂逻辑的“局部 runtime 容器”。

### 归因处理器接入点
`feature/feature_home/src/main/kotlin/com/dramawave/feature/home/detail/coordinator/processors/DramaAttributionProcessor.kt`
是 `shared_af` 接入详情播放场景的关键桥：
- `onCreate()` 会检查剪切板场景，处理 Deeplink + clipboard 的二次拉活冲突
- `initBus()` 订阅 `AttributionResultEvent`
- 当首页降级来源为 `HOME_DDL_FALLBACK` 且是首启时，会把新归因结果回灌给 `hostLinker`
- 首次网络数据回来后，通过 `viewModel.attributionWhenPageChanged(initialPosition)` 触发归因判定
- 翻页时对非本地数据再次检查归因，说明归因不是一次性行为，而是和剧集数据刷新/翻页联动

### ViewPager 适配器的真实复杂度
`feature/feature_home/src/main/kotlin/com/dramawave/feature/home/architecture/pager/adapter/VideoPagerAdapter.kt`
不是普通 adapter，而是承担视频页生命周期调度：
- 使用 `ConcurrentHashMap<Long, IPagerProtocol>` 维护 fragment 缓存，解决快速切换时 `getFragment()` 为空问题
- 只 attach 当前页与下一页，其他页执行 detach，形成“当前页 + 预加载一页”的资源调度策略
- 保存 `pendingAttachPositions` 和 `pendingResetMap`，兼容 fragment 尚未创建时的数据源重置
- `stableId()` 是 fragment 重用与缓存裁剪的核心锚点

因此首页/详情播放问题通常要同时看 `HomeFeedFragment`、`DramaCoordinator`、`DramaAttributionProcessor`、`VideoPagerAdapter`。

## feature_theater：剧场容器、动态 Tab 与首启引导承接页

### 关键职责
`feature_theater` 是更偏“浏览型内容首页”的容器模块：
- 服务端动态配置一级/二级 Tab
- 每个子 Tab 内承载瀑布流、多类型 header/feed 组合
- 剧场页承接继续播放入口、新用户引导、搜索建议、VIP/福利入口
- 与归因系统联动，在首启时决定新手引导显隐与跳转时机

### 主容器页
`feature/feature_theater/src/main/kotlin/com/dramawave/feature/theater/TheaterHomeFragment.kt`
是剧场主 Tab 外壳，几个点尤其关键：
- 使用 `activityViewModels()` 同时持有 `TheaterHomeViewModel`、`LastPlayViewModel`、`AttributionViewModel`
- `observeAttributionState()` 直接观察 `attributionViewModel.holder.stateFlow`，根据 `isNewUserGuideVisible` 和 `countDownSeconds` 控制新用户引导 UI
- 新手引导点击是否可跳过由 `CommonStore.enableNewUserGuideSkip` 控制，跳过行为再回调 `attributionViewModel.onUserSkipCountDown()`
- `initContinuePlayView()` 与新手引导逻辑显式分离：首启时不展示继续播放，非首启才根据 `LastPlayViewModel` 数据展示
- 顶部 Tab 变化时会追踪曝光/点击、切换小说配置、更新 ViewPager2 可滑动状态

这说明 Theater 不是单纯列表页，而是“剧场内容 + 首启引导/归因后处理 + 继续播放”的综合容器。

### 子 Tab 内容页
`feature/feature_theater/src/main/kotlin/com/dramawave/feature/theater/TheaterSubTabFragment.kt`
是服务端驱动的子 Tab 页面：
- 根据 feed 样式动态决定 `GridLayoutManager(3)` 或 `StaggeredGridLayoutManager(2)`
- `HeaderAdapter` 承担 banner、顶部运营区等 header 组合
- `dynamicInsertFeed()` 会在热门 Tab 且瀑布流场景把动态插入数据插到可见尾部附近
- `handleIntentEvent()` 区分首屏 header 数据、feed 数据、加载失败
- 首刷失败且首启时会发送 `CloseNewUserGuideEvent`，说明 Theater 子页加载状态会反向影响首启引导关闭时机

### 子 Tab ViewModel 的数据裁剪方式
`feature/feature_theater/src/main/kotlin/com/dramawave/feature/theater/viewmodel/TheaterSubTabViewModel.kt`
体现出 Theater 页并非简单直接渲染后端结构，而是做了一层页面化拆分：
- `loadFirstPage()` 拉 tab 首页数据后，先识别 `FeedStyle`，再抽出推荐流模块 `recommendHomeItemData`
- `processItem()` 按 `TheaterDataType` 把后端模块转换为 header/title/column/feed 等多段 UI 数据
- 缓存采用 `SubTabTheaterStore.putTabData(cacheKey, cacheData.toJson())`，连 `hasMore` 一起缓存
- `loadFeedData()` 只继续拉取 feed 部分，不重复拉 header 结构
- `requestInsertFeedData()` 用于动态插入单剧集推荐卡

另外它直接 import `AttributionManager`，说明剧场页虽然以内容浏览为主，仍与归因链路耦合。

## shared_af：归因聚合中心

### 关键职责
`shared_af` 负责把 AppsFlyer、Meta、Install Referrer、TikTok、剪切板、Server Prob 等来源统一收口，再把结果广播给业务页。

根据 `shared/shared_af/AGENTS.md`，它是 task-driven 架构，核心对象包括：
- `AttributionTaskManager`：协调各归因任务完成、超时、状态
- `AttributionTask`：单任务包装
- `AttributionManager`：保存结果、排序、修正时间戳、暴露最高优先级结果
- `AttributionViewModel`：负责倒计时、新手引导、打开剧集等 UI 层控制

### 结果容器与广播中心
`shared/shared_af/src/main/java/com/dramawave/shared/af/manager/AttributionManager.kt`
是这个模块最值得优先读的类：
- `addAttr()` 会先执行 `repairMetaTimestamp()`，再加入 `_attrResults`，随后发出 `AttributionResultEvent`
- 事件注释明确写了 TheaterHomeFragment、PlayDetailFragment 等页面会直接消费 bus
- 对 `MetaLink` 与 `MetaInstallReferrer` 的时间修正采用“新增时修正 + 排序时动态 fallback”的双策略，避免回溯改历史结果
- `getTopAttrResult()` 根据 `CommonStore.enableAttrLastClickStrategy` 在 PRESET / LAST_CLICK 两种排序策略间切换
- `shouldShowForYouFlow` 基于当前 top attribution 判断是否应该走 ForYou 流
- Server Prob 不是立即入列表，而是先缓存为 `_pendingServerProbAttr`，只有在没有其他归因结果时才 `applyPendingServerProbAttr()` 作为兜底
- `executeServerProbAttribution()` 会构建设备信息请求，调用服务端模糊归因接口，并记录 success/failure 埋点

### 归因结果如何影响页面
从现有源码可以看到两个直接消费点：
- `feature_home` 的 `DramaAttributionProcessor` 订阅 `AttributionResultEvent`
- `feature_theater` 的 `TheaterHomeFragment` 通过 `AttributionViewModel` 观察首启引导状态

因此 `shared_af` 在实际运行时一半是“归因结果仓库”，一半是“页面行为决策上游”。

## core_network：请求容灾与拦截器链底座

### 关键职责
`core_network` 统一封装 OkHttp + Retrofit + Cronet，并把“域名替换、备用域名、请求头、业务错误、QUIC/Gzip”等能力前置到拦截器层。

根据 `core/core_network/AGENTS.md`，典型拦截器顺序包括：
- `DdnsInterceptor`
- `BackupDomainInterceptor`
- `ApiPathInterceptor`
- `HeaderInterceptor`
- `LoggingInterceptor`
- `ErrorCodeInterceptor`
- `DynamicQuicInterceptor`
- `GzipRequestInterceptor`

### DDNS 替换层
`core/core_network/src/main/java/com/dramawave/core/network/interceptor/DdnsInterceptor.kt`
职责单一但关键：
- 读取请求 host
- 通过 `DdnsManager.replaceDomain(oldDomain)` 替换 host
- 若替换后的请求成功，无论 2xx/3xx/4xx 都直接返回给业务层
- 只有发生真正网络异常时才走回原请求兜底

注释强调：不能因为返回码非 2xx 就再回退原请求，否则业务层会把业务错误误判成网络错误。

### 备用域名重试层
`core/core_network/src/main/java/com/dramawave/core/network/interceptor/BackupDomainInterceptor.kt`
复杂度比 DDNS 高很多：
- 先校验 path 是否在白名单里，不在则直接放行
- 先尝试原始请求；若 HTTP 失败或网络异常，再按备用域名列表重试
- 对取消类异常（如 `CancellationException`）直接抛出，不重试
- 对连接、超时、SSL、协议等网络异常进行兜底重试
- 每次失败会记录 `RetryAttempt` 链，全部失败时抛 `BackupDomainAllFailedException`
- 同时能记录 backup API 成功/失败埋点

它与 `DdnsInterceptor` 的关系是：前者做主域名替换，后者对已处理后的 URL 再做备用 host 级重试。

## core_mvi：统一状态机与页面观察模型

### 关键职责
`core_mvi` 是 DramaWave 页面层的基础设施，几乎所有 ViewModel 状态更新都在这里建立约束。

`core/core_mvi/AGENTS.md` 给出的基本对象是：
- `StateHolder<STATE, EVENT>`：状态与事件载体
- `HolderContext`：intent 上下文，提供 `reduce/postEvent/getState`
- `StateHolderOwner`：通常由 ViewModel 实现
- `observe/observeWithoutLifecycle`：页面观察接口

### StateHolder 的调度模型
`core/core_mvi/src/main/java/com/dramawave/core/mvi/architecture/StateHolder.kt`
展示了内部实现：
- 状态由 `MutableStateFlow` 承载，一次性事件由 `MutableSharedFlow` 承载
- intent 会写入 `Channel<Pair<CompletableJob, suspend ...>>`
- `initLaunch()` 启动一个事件循环，从 dispatchChannel 取出 intent，再在 `intentJob + INTENT_COROUTINE_CONTEXT + exceptionHandler` 中运行
- `CoroutineExceptionHandler` 会统一捕获未处理协程异常，避免 MVI 链路直接把异常炸到页面
- `joinIntents()`、`cancel()` 用于等待和回收意图任务

虽然 `StateHolder` 的注释强调 MVI，但它本质上更接近“顺序调度 intent + StateFlow/SharedFlow 分发”的轻量状态机。

### 页面侧常用 API
`core/core_mvi/src/main/java/com/dramawave/core/mvi/architecture/MviExt.kt`
定义了实际开发中最常用的 DSL：
- `intent { ... }`：最常见的异步状态修改入口，底层使用 `viewModelScope.launch(dispatcher)`
- `awaitIntent { ... }`：需要等待执行完成时使用
- `interceptIntent { ... }`：返回布尔值，支持“继续/拦截”模式
- `intentChain()` / `awaitIntentChain()`：顺序执行多段逻辑
- `observe()`：生命周期感知的状态/事件收集
- `observeWithoutLifecycle()`：即使页面退后台也继续收集，适合总线型/跨页联动场景

这和上面两个 feature 模块是直接对得上的：
- `HomeFeedFragment` 使用 `observeWithoutLifecycle(this, handleEvent = ::handleLinkerEvent)`
- `TheaterSubTabViewModel`、`TheaterHomeViewModel` 等大量使用 `intent { reduce { ... } postEvent(...) }`
- `TheaterHomeFragment` 直接读取 `attributionViewModel.holder.stateFlow`

## 典型跨模块链路

### 链路 1：首页详情归因刷新
1. 用户从 Deeplink 或首页进入 `feature_home`
2. `DramaAttributionProcessor` 在 `onCreate()` 处理 clipboard 特殊逻辑
3. 网络数据返回后，Processor 调 `viewModel.attributionWhenPageChanged()`
4. `shared_af` 将归因结果写入 `AttributionManager` 并发 `AttributionResultEvent`
5. 页面继续据此处理 TrialVip、ForYou 流等分支

### 链路 2：剧场首启新手引导
1. 用户打开 `TheaterHomeFragment`
2. 页面持有 `AttributionViewModel`
3. `observeAttributionState()` 根据 `isNewUserGuideVisible/countDownSeconds` 驱动 `NewUserGuideView`
4. 子 Tab 首刷失败时 `TheaterSubTabFragment` 会发 `CloseNewUserGuideEvent`
5. 引导关闭、跳过或倒计时结束后，剧场页再回归常规继续播放与内容浏览逻辑

### 链路 3：网络请求容灾
1. 业务 ViewModel 通过 repository 发请求
2. 请求进入 `core_network`
3. `DdnsInterceptor` 先尝试主域名替换
4. 如失败，`BackupDomainInterceptor` 视白名单与异常类型决定是否切备用域名重试
5. 响应回到 repository，再通过 `core_mvi` 的 `intent/reduce/postEvent` 反馈给页面

## 风险、约束与阅读建议
- `feature_home` 的复杂度不在单一 ViewModel，而在 Coordinator + Processor 的职责切分
- `feature_theater` 的复杂度不在普通列表渲染，而在“动态 Tab + 首启引导 + 继续播放 + 多类型模块拆分”叠加
- `shared_af` 同时承载排序规则、兼容补丁、实验开关、页面广播，阅读时要重点区分“结果存储”和“页面消费”
- `core_network` 的 DDNS 与备用域名逻辑不能孤立看，二者串联才是完整容灾链路
- `core_mvi` 默认 intent dispatcher 代码注释与实际常量命名存在轻微不一致时，应以源码实现为准；页面是否受生命周期影响要区分 `observe()` 与 `observeWithoutLifecycle()`

## 相关页面
- [[feature-home-and-play-entry]]
- [[player-system-overview]]
- [[startup-and-apm-init-flow]]
- [[architecture-layering]]
