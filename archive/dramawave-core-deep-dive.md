---
title: DramaWave Core 基础设施深挖（历史草稿）
created: 2026-04-16
updated: 2026-04-17
type: concept
layer: core
status: draft
tags: [core, architecture, network, startup, storage, archive]
sources: [raw/articles/dramawave-codebase-scan-2026-04-16.md]
---

# DramaWave Core 基础设施深挖（历史草稿）

## 说明
这是早期整理的 core 综合分析草稿，保留在 archive 中供参考。

当前正式知识应优先阅读并逐步收敛到以下页面：
- [[dramawave-overview]]
- [[architecture-layering]]
- [[module-map]]
- [[core-apm]]
- [[startup-and-apm-init-flow]]

后续如果继续补齐 `core_network`、`core_mvi`、`core_router`、`core_kv`、`core_db` 等正式页面，本草稿应进一步被拆分吸收，而不是继续扩写。

## 历史摘要
`core` 层是 DramaWave 的技术基座，不承载具体业务故事线，但决定了整个工程的运行方式、依赖边界、状态管理、存储方式与启动效率。

早期关注的核心模块包括：
- `core_network`
- `core_router`
- `core_mvi`
- `core_kv`
- `core_db`
- `core_bus`
- `core_startup`
- `core_apm`

## 当前有效结论
- `core_network` 是带有域名治理、备份域名、错误处理、QUIC/Cronet 能力的统一网络层
- `core_router` 是 TheRouter 之上的统一 deeplink 与跳转拦截体系
- `core_mvi` 是项目级状态管理基础设施
- `core_kv` / `core_db` 分别承担轻量状态持久化与结构化本地存储
- `core_startup` / `app` 启动编排决定了全局初始化顺序
- `core_apm` 是当前可观测性体系的重要落点

## 使用建议
如果只是导航代码或评估改造影响，不要从这份草稿开始，而应从正式索引页进入：
- [[dramawave-overview]]
- [[home-theater-attribution-network-mvi-overview]]
- [[player-system-overview]]
- [[ad-system-overview]]
