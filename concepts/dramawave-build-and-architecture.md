---
title: DramaWave 构建与工程架构
created: 2026-04-16
updated: 2026-04-16
type: concept
tags: [project, android, gradle, architecture, module, flavor, dependency, security]
sources: [raw/articles/dramawave-codebase-scan-2026-04-16.md]
---

# DramaWave 构建与工程架构

## 总体结论
DramaWave 使用 Gradle Kotlin DSL 管理的大型 Android 多模块工程，构建策略强调：
- 统一版本目录
- 强依赖边界约束
- flavor 感知模块配置
- included build 形式的构建逻辑与插件复用

关键文件：
- `settings.gradle.kts`
- `build.gradle.kts`
- `app/build.gradle.kts`
- `gradle/libs.versions.toml`
- `gradle.properties`
- `build-logic/build.gradle.kts`

## 构建栈
依据 `gradle/libs.versions.toml`：
- AGP：8.8.0
- Kotlin：2.1.0
- compileSdk：35
- targetSdk：35
- minSdk：23

依赖管理采用 version catalog，避免在各模块 `build.gradle.kts` 中散落硬编码版本。

## settings.gradle.kts 观察
`settings.gradle.kts` 做了三件非常关键的事：

1. 声明 included build
- `includeBuild("build-logic")`
- `includeBuild("plugins/view-monitor")`

这意味着工程把“模块约定”和“调试插件”提升到了独立构建逻辑层，而不是简单写在根脚本里。

2. 仓库治理
- `dependencyResolutionManagement { repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS) }`
- 这是强治理模式，阻止子模块私自声明仓库，确保供应链与依赖来源集中管理。

3. 模块注册
- `app`
- 13 个 `feature` 模块
- 11 个 `interface` 模块
- 22 个 `shared` 模块
- 20 个 `core` 模块
- 额外还有 `baselineprofile`、`plugins:view-monitor-runtime`、若干 `libraries:utils:*`

## 根构建脚本
`build.gradle.kts` 负责：
- 统一声明插件但默认 `apply false`
- 通过 `gradle/subproject.gradle`、`gradle/version_loader.gradle`、`gradle/third_party_config.gradle` 下放公共规则
- 在 `allprojects.configurations.all.resolutionStrategy.eachDependency` 中做跨层依赖校验

### 依赖边界规则
根脚本里有显式约束：
- 非 `app` 模块禁止依赖 `feature_*`
- `core_*` 禁止依赖 `shared_*` 与 `interface_*`
- `shared_* -> shared_*`、`interface_* -> shared_*` 虽允许，但会打印警告

这说明该项目不是靠约定俗成维护架构，而是用 Gradle 在构建期直接做防线。

## build-logic
`build-logic/build.gradle.kts` 注册了几个重要约定插件：
- `dramawave.android.application`
- `dramawave.android.library`
- `dramawave.shared.buildconfig`
- `dramawave.flavor.aware`

其中最关键的是：
- `dramawave.shared.buildconfig`：统一 BuildConfig 字段约定
- `dramawave.flavor.aware`：把 app 的 flavor 维度约束传播到 library 模块

## app/build.gradle.kts
`app` 模块是工程装配中心，具备以下特点：

### 插件组合
- Android Application
- Kotlin Android / Compose / Kapt / Parcelize
- Hilt
- Google Services
- Firebase Perf / Crashlytics
- TheRouter
- Baseline Profile
- 自定义 flavor aware / shared buildconfig 插件

### 产品形态
存在两个 flavor：
- `dramawave`
- `freereels`

并且构建中还额外引入 `buildEnv` 概念：
- 本地默认 `dev`
- CI 测试包倾向 `intl`
- 发版分支倾向 `prod`

也就是说，这个工程并不是“仅 flavor 区分产品”，而是“flavor + buildType + buildEnv + market”的复合构建模型。

### BuildConfig 字段很重
`app/build.gradle.kts` 会动态生成：
- `BUILD_ENV`
- `APP_MARKET`
- `API_VERSION_NAME`
- `BUILD_TIME`
- `GIT_COMMIT_ID`
- `GIT_BRANCH`
- `ALLOW_SCREENSHOT`
- `VIEW_MONITOR_ENABLED`

这说明 BuildConfig 被广泛用作运行时行为切换开关，而不只是版本信息容器。

### UI 技术混合
`app` 同时开启：
- `viewBinding = true`
- `dataBinding = true`
- `compose = true`
- `buildConfig = true`

结论是：UI 体系处于“传统 ViewBinding / DataBinding 与 Compose 并存”的阶段，而不是纯 Compose 工程。

## gradle.properties
工程全局构建参数显示出“大仓库优化”倾向：
- `org.gradle.jvmargs=-Xmx10g ...`
- `org.gradle.parallel=true`
- `org.gradle.daemon=true`
- `org.gradle.configureondemand=true`
- `kapt.incremental.apt=true`
- `kapt.use.worker.api=true`
- `kotlin.incremental=true`
- `org.gradle.caching=false`

值得注意：
- 给了非常大的 Gradle 堆内存，说明模块数量和注解处理负担都不轻。
- 虽然开启了并行与增量，但缓存是关闭的，可能和 CI 稳定性或历史构建问题有关。
- `firebasePerformanceInstrumentationEnabled=false`，说明禁用了 Firebase Perf Gradle 插桩，性能链路主要走业务层显式埋点与包装层。

## 关键风险与治理结论
### 明文凭据风险
`settings.gradle.kts` 中 GitHub Maven 仓库配置出现了用户名与密码字段，属于显著供应链安全风险。无论密钥是否已失效，这种写法都意味着：
- 构建凭据进入代码仓库历史
- 凭据轮换困难
- 本地与 CI 权限边界不清晰

### 构建模型复杂度高
工程同时受以下因素共同影响：
- flavor
- buildType
- buildEnv
- CI/本地环境
- included build 插件
- flavor aware library 配置

这会提高新增模块、迁移旧模块和引入外部库时的适配成本。

## 关联页面
- [[dramawave-codebase-overview]]
- [[dramawave-startup-and-entry]]
- [[dramawave-module-map]]
- [[dramawave-codebase-risks]]
- [[dramawave-technology-stack]]
