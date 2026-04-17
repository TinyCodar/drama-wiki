# DramaWave LLM Wiki Schema

## Domain
DramaWave Android 短剧应用工程知识库。覆盖 app、feature、interface、shared、core、build-logic、plugins、scripts、docs，以及运行时链路、模块依赖、关键业务域、性能/广告/播放器/归因/登录/支付/推送等核心主题。目标读者是需要快速理解 DramaWave 代码库、定位实现入口、评估改造影响、沉淀跨会话知识的 AI agent 与工程师。

## Conventions
- Wiki 根目录固定为 `docs/llm-wiki/`
- 文件名使用小写英文和连字符，不使用空格，例如 `shared-ad.md`
- 每个 wiki 页面必须包含 YAML frontmatter
- 使用 `[[wikilinks]]` 表示内部关联，单页至少保留 2 个出链（总览页可例外）
- 页面内容以“是什么 / 入口在哪 / 依赖谁 / 被谁使用 / 风险与备注”为核心结构
- 代码事实优先来源于仓库源码、AGENTS.md、CLAUDE.md、README、docs；推断必须明确标注
- 更新页面时必须同步更新 `index.md` 与 `log.md`
- 一个主题如果只是一处偶发提及，不单独建页，优先补充到已有总览页
- 页面内容优先使用中文，保留必要英文类名/路径/命令
- 面向代码导航的页面必须给出精确模块路径或类路径

## Frontmatter
```yaml
---
title: 页面标题
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: overview | module | concept | flow | comparison | query | inventory
layer: app | feature | interface | shared | core | build | docs | cross-cutting
status: draft | active | verified
tags: [tag1, tag2]
sources:
  - path/or/source
---
```

## Tag Taxonomy
- 分层：app, feature, interface, shared, core, build, docs, cross-cutting
- 技术：mvi, hilt, router, network, mmkv, room, compose, viewbinding, databinding, cronet
- 业务：player, ad, analytics, attribution, login, profile, reward, purchase, push, web, theater, home, novel, widget
- 工程：architecture, dependency-rule, version-catalog, convention-plugin, startup, config, quality, testing, script
- 质量：apm, crash, perf, memory, download, cache, state-machine, event-bus
- 产物：inventory, map, deep-dive, guide

使用新 tag 前，先补充到本文件。

## Page Thresholds
- 满足以下任一条件即可单独建页：
  - 是顶层模块或关键业务模块
  - 是跨模块共享的关键概念/链路
  - 是多处文档或源码共同引用的主题
  - 是后续改造、排障、集成时高频查询的知识点
- 否则合并到总览页或分层索引页

## Page Types
- `overview`: 工程总览、分层总览、导航页
- `module`: 单模块 deep dive
- `concept`: 跨模块概念，如 MVI、事件总线、依赖规则
- `flow`: 运行时链路，如启动、播放、广告、APM 初始化
- `comparison`: 对比类主题
- `query`: 特定问题的结论沉淀
- `inventory`: 清单类页面，如模块目录、文档入口、脚本入口

## Recommended Structure
每个页面尽量按以下顺序组织：
1. 定位 / 一句话说明
2. 关键职责
3. 目录与入口
4. 关键类 / 关键文件
5. 依赖关系（依赖谁 / 被谁依赖）
6. 运行时链路或典型用法
7. 风险、约束、坑点
8. 相关页面

## Update Policy
当新信息与既有内容冲突时：
1. 优先以当前仓库源码为准
2. 若文档与源码不一致，页面中同时保留并注明“文档滞后”
3. 必要时在 `log.md` 记录冲突与处理方式

## Initial Wiki Scope
当前首批页面应至少覆盖：
- 工程总览与分层规则
- 模块地图（app / feature / interface / shared / core）
- deep dive：`core_apm`、`shared_ad`、`shared_player`
- 运行时专题：启动与 APM 初始化、播放器体系、广告体系
- 文档与脚本入口清单
