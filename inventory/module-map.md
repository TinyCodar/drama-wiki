---
title: 模块地图
created: 2026-04-16
updated: 2026-04-16
type: inventory
layer: cross-cutting
status: verified
tags: [inventory, map, architecture, app, feature, interface, shared, core]
sources:
  - docs/项目结构.md
  - AGENTS.md
  - CLAUDE.md
  - shared/AGENTS.md
  - core/AGENTS.md
---

# 模块地图

## 顶层目录
- `app/`：应用装配与启动入口
- `feature/`：业务模块集合
- `interface/`：跨模块接口集合
- `shared/`：共享业务能力集合
- `core/`：基础设施集合
- `docs/`：项目文档
- `scripts/`：自动化脚本、代码审查辅助脚本
- `build-logic/`、`plugins/`：Gradle convention plugin 与自定义插件

## Feature 层
已知重点模块：
- `feature_home`：首页、ForYou 播放、剧集详情、广告工具、插件系统
- `feature_theater`：剧场/榜单/混排浏览
- `feature_profile`：个人页、设置
- `feature_login`：登录与 onboarding
- `feature_reward`：奖励、任务、签到
- `feature_search`：搜索
- `feature_mylist`：收藏/观看列表
- `feature_novel`：小说阅读
- `feature_web`：业务 Web 页面
- `feature_widget`：桌面小组件
- `feature_develop`：开发辅助
- `feature_ability`：能力类模块
- `feature_ashes`：保活/实验或能力扩展相关

## Shared 层
重点共享模块：
- `shared_player`：播放器包装
- `shared_ad`：广告聚合 SDK
- `shared_af`：归因
- `shared_user`：用户状态
- `shared_purchase`：内购/订阅
- `shared_analytics`：埋点封装
- `shared_api`：业务 API / repository
- `shared_ui`：共享 UI
- `shared_models`：共享数据模型和事件
- `shared_push`：推送
- `shared_web`：业务 Web 包装
- `shared_general`：通用业务工具

## Core 层
重点基础模块：
- `core_mvi`：MVI 基础设施
- `core_network`：网络能力
- `core_router`：路由
- `core_bus`：事件总线
- `core_kv`：KV 存储
- `core_db`：数据库
- `core_config`：配置中心
- `core_apm`：性能与异常监控
- `core_log`：日志
- `core_image`：图片加载
- `core_web`：WebView 基础层
- `core_player_api`：播放器接口定义

## 使用建议
- 看某条业务链路时，先定位 feature 页面，再下钻 shared，再看 core
- 看跨业务基础能力时，优先从 shared/core 模块页开始
- 看工程规范与边界时，先读 [[architecture-layering]]

## 相关页面
- [[dramawave-overview]]
- [[architecture-layering]]
- [[docs-and-scripts-entrypoints]]
