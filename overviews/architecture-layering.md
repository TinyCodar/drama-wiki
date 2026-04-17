---
title: 分层规则与依赖约束
created: 2026-04-16
updated: 2026-04-16
type: concept
layer: cross-cutting
status: verified
tags: [architecture, dependency-rule, feature, interface, shared, core]
sources:
  - AGENTS.md
  - CLAUDE.md
  - docs/项目结构.md
---

# 分层规则与依赖约束

## 核心规则
DramaWave 的核心依赖关系为：`app → feature → interface → shared → core`。

## 各层边界
- `app`：最终装配层，通常可以接入各层能力，是运行时装配入口
- `feature`：独立业务模块，只能依赖 `interface`、`shared`、`core`，不能直接依赖别的 feature
- `interface`：对 feature 能力进行 API 暴露，避免 feature 之间横向耦合
- `shared`：共享业务逻辑层，可依赖 `core`，也可在部分场景依赖 `interface`
- `core`：纯基础设施层，不能依赖 `shared`/`interface`

## 为什么重要
- 保证业务模块可替换与边界清晰
- 避免 feature 之间形成网状依赖
- 防止基础设施层掺入业务逻辑
- 使播放器、广告、分析、用户等共享能力可被多个 feature 复用

## 实际理解方式
- 首页播放链路通常是 `feature_home -> shared_player -> core_player_api/core_*`
- 广告业务通常是 `feature_* -> shared_ad -> core_*`
- 登录、奖励、Widget 等跨 feature 协作时，会通过 `interface_*` 暴露契约

## 与 wiki 的关系
- 编写模块页面时，应先判断该模块所处分层与允许依赖方向
- 分析改造影响时，应把“上游调用方 / 下游依赖层”作为固定章节

## 风险与备注
- `shared` 层虽然允许彼此依赖，但容易形成复杂横向耦合，应重点记录关键共享模块的边界
- 老文档里存在 `shared_view`、`core_mvvm` 等旧称，实际代码已演进为 `shared_ui`、`core_mvi`

## 相关页面
- [[dramawave-overview]]
- [[module-map]]
- [[shared-ad]]
- [[shared-player]]
