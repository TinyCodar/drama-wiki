---
title: DramaWave 工程总览
created: 2026-04-16
updated: 2026-04-16
type: overview
layer: app
status: verified
tags: [app, architecture, build, overview, deep-dive]
sources:
  - AGENTS.md
  - CLAUDE.md
  - docs/项目结构.md
---

# DramaWave 工程总览

## 定位
DramaWave 是一个 Android 短剧应用工程，包含 `dramawave` 与 `freereels` 两个 product flavor，采用 Kotlin + 模块化架构 + MVI。

## 顶层认知
- 构建主命令：`./gradlew assembleDramawaveDebug`、`assembleFreereelsDebug`
- 环境切换通过 `local.properties` 中的 `buildEnv=dev|prod`
- 依赖层次为 `app → feature → interface → shared → core`
- 版本与依赖集中在 `gradle/libs.versions.toml`
- 采用 Gradle Kotlin DSL + convention plugins（`build-logic/`）
- 核心技术栈包括 Hilt、TheRouter、OkHttp/Retrofit/Cronet、MMKV、Room、Compose、ViewBinding/DataBinding

## 分层结构
- `app/`：最终装配层，负责 Application、Activity、startup 和 flavor 相关接入
- `feature/`：业务模块，如首页、剧场、登录、奖励、搜索、小说、Widget
- `interface/`：跨 feature 通信桥接层
- `shared/`：共享业务能力层，如播放器、广告、用户、支付、归因、分析
- `core/`：基础设施层，如 MVI、路由、网络、KV、DB、日志、APM、Web

## 工程特征
- 对依赖方向有强约束，违规会触发 `GradleException`
- `core` 明确禁止依赖 `shared` 与 `interface`
- `feature` 之间禁止直接互相依赖，只能借助 `interface`
- `shared` 承载大量跨业务公共能力，是理解播放、广告、归因、支付等主题的关键入口
- 项目内存在大量模块级 `AGENTS.md`/`CLAUDE.md`，可视为局部知识索引

## 重点业务域
- 播放：`shared/shared_player`
- 广告：`shared/shared_ad`
- 用户与登录：`shared/shared_user`、`feature/feature_login`
- 奖励与任务：`feature/feature_reward`、`interface/interface_reward`
- 归因：`shared/shared_af`
- 监控：`core/core_apm`
- 首页/详情播放主场景：`feature/feature_home`

## 推荐阅读顺序
1. [[architecture-layering]]
2. [[module-map]]
3. [[core-apm]]
4. [[shared-player]]
5. [[shared-ad]]
6. [[startup-and-apm-init-flow]]

## 风险与备注
- 文档 `docs/项目结构.md` 有部分模块命名沿用旧称，实际代码应以仓库目录与模块级文档为准
- 当前 wiki 首批页面以高价值导航为主，还未覆盖全部 feature/interface/shared/core 模块

## 相关页面
- [[architecture-layering]]
- [[module-map]]
- [[docs-and-scripts-entrypoints]]
