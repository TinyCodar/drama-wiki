---
title: DramaWave 代码库风险与复杂度观察
created: 2026-04-16
updated: 2026-04-16
type: query
tags: [project, query, security, architecture, dependency, research]
sources: [raw/articles/dramawave-codebase-scan-2026-04-16.md]
---

# DramaWave 代码库风险与复杂度观察

## 1. 构建配置中存在供应链安全风险
`settings.gradle.kts` 中的 GitHub Maven 仓库配置包含用户名与密码字段，这意味着构建凭据曾被直接放入仓库配置。

风险点：
- 凭据可能进入 Git 历史
- 本地开发、CI、外部协作者之间权限边界模糊
- 依赖拉取链路带来额外审计成本

建议方向：
- 改为环境变量或凭据文件注入
- 清理仓库历史中的敏感信息
- 为私服/包仓访问建立最小权限策略

## 2. MainActivity 已经成为超级协调器
`app/src/main/java/com/dramawave/app/MainActivity.kt` 约 2370 行，import 涉及首页、剧场、奖励、归因、广告、事件、播放器、通知、消息等多个域。

风险点：
- 主壳层职责过多
- 任意改动都可能触发广泛回归
- 业务状态切换逻辑容易继续堆积

建议方向：
- 逐步把横切逻辑下沉到 coordinator / manager / delegate
- 把 tab 级职责拆到独立 orchestrator
- 为主壳层建立专门的结构化文档与测试清单

## 3. 构建模型复合度高
工程同时受以下变量影响：
- productFlavor
- buildType
- buildEnv
- CI / 本地环境
- flavor-aware library 配置

风险点：
- 新模块接入门槛高
- 三方库迁移与兼容性测试成本高
- 某些问题可能只在特定 flavor/env/buildType 组合下暴露

## 4. 第三方 SDK 集成密度极高
广告、归因、播放器、Firebase、支付、Web、推送等多系统并存，特别是 `shared_ad` 非常重。

风险点：
- 初始化时序复杂
- 依赖冲突概率高
- 升级单一 SDK 可能牵动多平台 adapter

## 5. 共享层依赖面大，容易形成“业务中台巨石”
`shared` 层模块众多且业务密集，虽然分模块，但从实际依赖关系看，部分模块已经接近平台级业务底座。

风险点：
- 共享模块继续膨胀后，边界会模糊
- 一个 shared 模块的改动可能影响多个 feature
- 接口抽象不足时，shared 容易承接本应属于 feature 的逻辑

## 6. UI 技术栈处于混合期
ViewBinding、DataBinding、Compose 并存。

风险点：
- 同类页面实现风格不统一
- 新功能落地时技术选型容易摇摆
- 可复用组件和主题体系可能出现双轨维护成本

## 当前判断
DramaWave 最大的风险不是“代码写得老”，而是：
- 商业化集成很多
- 主壳与共享层承担了过多系统性职责
- 构建/依赖/初始化三条链路都很重

这类项目最怕的是缺少结构化知识地图，因此 codebase wiki 的价值会非常高。

## 关联页面
- [[dramawave-codebase-overview]]
- [[dramawave-build-and-architecture]]
- [[dramawave-startup-and-entry]]
- [[dramawave-module-map]]
- [[dramawave-technology-stack]]
