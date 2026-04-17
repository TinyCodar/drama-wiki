---
title: 文档与脚本入口清单
created: 2026-04-16
updated: 2026-04-16
type: inventory
layer: docs
status: verified
tags: [docs, script, inventory, guide]
sources:
  - docs/项目结构.md
  - docs/支付说明.md
  - docs/打包环境.md
  - scripts/code-review/README.md
  - scripts/i18n/README.md
---

# 文档与脚本入口清单

## 文档目录观察
仓库下存在较多可复用资料，适合后续扩展 wiki：
- `docs/项目结构.md`：旧版结构说明，适合做全局总览基线
- `docs/支付说明.md`：支付链路背景
- `docs/打包环境.md`：构建/打包环境说明
- `docs/Matrix能力映射清单.md`、`docs/Matrix集成执行基线.md`：当前 Matrix 相关研究成果
- `docs/novel/`：小说场景详细文档
- `shared/*/README.md`：模块级使用说明
- `feature/feature_home/.../playstats/*.md`：播放统计专题资料

## 脚本目录观察
- `scripts/code-review/README.md`：代码审查辅助脚本说明
- `scripts/code-review/prompts/README.md`：代码审查 prompt 资源
- `scripts/i18n/README.md`：国际化相关脚本说明

## 如何继续扩展 wiki
后续增量建页时，优先从下列资料采源：
1. 模块根目录下 `AGENTS.md` / `CLAUDE.md`
2. 模块 `README.md`
3. `docs/` 专题文档
4. 关键源码入口类

## 推荐下一批页面
- `shared-af`
- `feature-home`
- `feature-theater`
- `core-network`
- `core-mvi`
- `purchase-and-reward-overview`

## 相关页面
- [[dramawave-overview]]
- [[module-map]]
- [[startup-and-apm-init-flow]]
