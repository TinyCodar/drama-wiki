# Wiki Log

> DramaWave 工程 LLM Wiki 操作日志。
> Format: `## [YYYY-MM-DD] action | subject`

## [2026-04-17] cleanup | 统一外部 wiki 为 DramaWave 工程知识库
- 重写 `README.md`，明确仓库定位为 DramaWave 外部 wiki，而非通用 llm-wiki 初始化仓库
- 删除通用或重复的早期页面：`concepts/llm-wiki-stack.md`、`concepts/prompt-caching.md`、`queries/wiki-bootstrap-status.md`
- 删除已被新结构替代的 DramaWave 旧版页面：构建架构、代码库总览、模块地图、启动入口、技术栈、风险观察等旧稿
- 将 `concepts/dramawave-core-deep-dive.md` 迁移到 `archive/dramawave-core-deep-dive.md`，作为历史草稿保留
- 保留正式结构：`overviews/`、`inventory/`、`modules/`、`flows/`
- 更新 `index.md`，补充 archive 入口与下一批覆盖优先级

## [2026-04-16] create | DramaWave LLM Wiki initialized
- 创建 `docs/llm-wiki/` 结构
- 新增 `SCHEMA.md`
- 新增 `index.md`
- 新增首批总览、清单、模块 deep dive、运行时链路页面
- 当前首批覆盖：工程总览、分层规则、模块地图、文档脚本入口、core_apm、shared_ad、shared_player、feature_home 播放入口、启动/APM、广告体系、播放器体系

## [2026-04-16] update | 第二批 wiki 扩展
- 新增 `docs/llm-wiki/home-theater-attribution-network-mvi-overview.md`
- `index.md` 新增跨模块总览页索引
- 本轮补充 feature_home、feature_theater、shared_af、core_network、core_mvi 之间的主链路、职责边界与关键入口
