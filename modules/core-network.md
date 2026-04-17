---
title: core_network deep dive
created: 2026-04-17
updated: 2026-04-17
type: module
layer: core
status: verified
tags: [core, network, cronet, quic, ddns, deep-dive]
sources:
  - core/core_network/build.gradle.kts
  - core/core_network/src/main/java/com/dramawave/core/network/RetrofitFactory.kt
  - core/core_network/src/main/java/com/dramawave/core/network/OkHttpClientManager.kt
  - core/core_network/src/main/java/com/dramawave/core/network/LogicGsonConverterFactory.kt
  - core/core_network/src/main/java/com/dramawave/core/network/interceptor/HeaderInterceptor.kt
  - core/core_network/src/main/java/com/dramawave/core/network/interceptor/DdnsInterceptor.kt
  - core/core_network/src/main/java/com/dramawave/core/network/interceptor/BackupDomainInterceptor.kt
  - core/core_network/src/main/java/com/dramawave/core/network/quic/DynamicQuicInterceptor.kt
  - app/src/main/java/com/dramawave/app/startup/component/NetworkInitializer.kt
  - app/src/main/java/com/dramawave/app/startup/component/CommonInitializer.kt
---

# core_network deep dive

## 定位
`core_network` 是 DramaWave 的统一网络底座。它以 `RetrofitFactory + OkHttpClientManager` 为中心，串联 Header、DDNS、备用域名、日志、业务错误码、QUIC/Cronet 与诊断能力，并通过 `SMAuthUtilsProxy` / `LoggingUtilProxy` 把用户态信息和埋点能力从 app 层注入到网络层。

它不是简单 Retrofit 封装，而是带有明显“网络中台”特征的基础设施模块。

## 核心职责
- 创建 Retrofit Service 与 API 专用 OkHttpClient
- 管理 `apiClient` / `shareClient` 两套基础客户端
- 统一请求头注入与授权桥接
- 动态 API 前缀、DDNS 域名替换、备用域名重试
- 业务 `code` 解析与异常转换
- QUIC/Cronet 动态接入与回退
- 网络日志上报与诊断文件生成

## 核心结构
### RetrofitFactory
- `core/core_network/src/main/java/com/dramawave/core/network/RetrofitFactory.kt`

职责：
- 模块主入口
- 懒加载 Retrofit，`baseUrl` 来自 `AppConfig.getApiHost()`
- 基于 `OkHttpClientManager.apiClient` 构建 API 请求链
- 统一创建 `createService(clazz)`
- 管理 Cronet 初始化和 `CronetEngine` 生命周期
- 作为 `DynamicQuicInterceptor.CronetEngineProvider` 提供 engine

### OkHttpClientManager
- `core/core_network/src/main/java/com/dramawave/core/network/OkHttpClientManager.kt`

职责：
- 提供 `apiClient` 和 `shareClient`
- 通过共享 Dispatcher / ThreadPoolExecutor 控制线程资源
- 供 API、图片、下载、WebView 等多个场景复用底层连接与调度配置

### LogicGsonConverterFactory
- `core/core_network/src/main/java/com/dramawave/core/network/LogicGsonConverterFactory.kt`

职责：
- 统一解析 `BaseResponse<T>`
- 若 `code != 200` 抛 `ApiException`
- `data` 为空时尝试退回按原始类型反序列化
- JSON 解析失败时抛 `LogicJsonParseException`
- 从 Retrofit 方法注解中提取 path，辅助定位异常

## 拦截器链
根据 `RetrofitFactory.createClient(...)` 的实际顺序，API 请求链如下：
1. `DdnsInterceptor`
2. `BackupDomainInterceptor`（仅 `CommonStore.enableBakDomainApi` 打开时启用）
3. `ApiPathInterceptor`
4. `HeaderInterceptor`
5. Debug 下附加：
   - `OkHttpProfilerInterceptor`
   - `HttpLoggingInterceptor`
   - `CurlLoggingInterceptor`
6. `LoggingInterceptor`
7. `ErrorCodeInterceptor`
8. `DynamicQuicInterceptor`
9. `GzipRequestInterceptor`

### DdnsInterceptor
- `interceptor/DdnsInterceptor.kt`
- 发请求前根据 `DdnsManager.replaceDomain()` 替换 host
- 替换失败则回退原始请求

### BackupDomainInterceptor
- `interceptor/BackupDomainInterceptor.kt`
- 原请求失败后，按白名单接口与备用域名列表重试
- 汇总失败链路，最终抛 `BackupDomainAllFailedException`

### ApiPathInterceptor
- `interceptor/ApiPathInterceptor.kt`
- 自动拼接 flavor 对应 API 前缀
- 例如注释中说明：`dramawave -> /dm-api`，`freereels -> /frv2-api`

### HeaderInterceptor
- `interceptor/HeaderInterceptor.kt`
- 注入统一请求头，包括：
  - 语言、国家、时区、网络类型
  - 屏幕宽高、设备内存、品牌、型号、指纹
  - session-id、device-id、app-version、app-name
  - AppsFlyer id、GAID、Firebase id
  - `Ab-Exps`
  - OAuth `Authorization`
- 依赖 `CommonStore`、`UserStore`、`DeviceLocale`、`AppsFlyerUtil`、`SMAuthUtilsProxy`

### LoggingInterceptor
- `interceptor/LoggingInterceptor.kt`
- 统计请求耗时、响应码、响应体大小、host、`X-Key-Uri`
- 通过 `LoggingUtilProxy.logCallback` 上报网络事件

### ErrorCodeInterceptor
- `interceptor/ErrorCodeInterceptor.kt`
- 解析响应头和响应体内业务 code
- 同步 `Ab-Exps` 到 `CommonStore`
- 命中特定设备错误码时发 `DeviceRemoveEvent`
- 读取 body 后会重建 response body 供下游继续消费

### DynamicQuicInterceptor
- `quic/DynamicQuicInterceptor.kt`
- 动态决定是否走 QUIC/Cronet
- QUIC 失败后回退普通 HTTP/2/HTTP
- 路径级失败次数过多时，5 分钟内跳过 QUIC
- 当前明确跳过 POST 请求

### GzipRequestInterceptor
- `interceptor/GzipRequestInterceptor.kt`
- 对有 body 且未指定 `Content-Encoding` 的请求做 gzip 压缩
- 注释明确要求它位于 QUIC 之后

## 配置与诊断能力
### DDNS 管理
- `ddns/DdnsManager.kt`
- `ddns/DdnsStorage.kt`

职责：
- 管理 replaceDomains 与 backupDomains
- 从 MMKV 缓存的 sysConfig 中读取 DDNS 配置
- 对外提供 `replaceUrl()`、`replaceDomain()`、`getBackupDomains()`、`updateConfig()` 等能力

### 认证与日志桥接
- `utils/SMAuthUtilsProxy.kt`
- `utils/LoggingUtilProxy.kt`

作用：
- 避免 core_network 直接依赖 app 或具体业务实现
- 由 app 启动阶段把 OAuth、cookie、UA、GAID、签名、日志上报能力注入进来

### 网络诊断
- `diagnosis/NetDiagnosisUtils.kt`

作用：
- 生成网络诊断报告到应用文件目录
- 输出路径为 `files/report_data/net_diagnosis_report.data`

## 启动接入与上游使用
### app 启动接入
- `app/src/main/java/com/dramawave/app/startup/component/NetworkInitializer.kt`
- `app/src/main/java/com/dramawave/app/startup/component/CommonInitializer.kt`

关键点：
- `NetworkInitializer` 会在 Google Play Services 可用且开关满足条件时调用 `RetrofitFactory.initializeCronet(context)`
- `CommonInitializer` 会注入 `LoggingUtilProxy.logCallback` 和 `SMAuthUtilsProxy.setAuthProxy(...)`

如果 app 层没有完成这些初始化，网络层的日志、用户态 header、cookie、GAID、签名能力都会打折。

### 系统配置下发
- `shared/shared_general/src/main/java/com/dramawave/shared/general/global/GlobalViewModel.kt`

`loadSysConfig()` 成功后会调用 `DdnsManager.updateConfig(result.data)`，说明 DDNS / 备用域名能力依赖系统配置接口返回与本地缓存同步。

### 已验证的上游使用方
多个模块通过 `RetrofitFactory.createService(...)` 或 `OkHttpClientManager.shareClient/apiClient` 复用该模块能力，包括：
- `app`
- `shared_user`
- `shared_general`
- `shared_im`
- `shared_purchase`
- `shared_web`
- `shared_push`
- `shared_analytics`
- `shared_af`
- `feature_home`
- `feature_profile`
- `feature_novel`
- `feature_widget`
- `feature_search`
- `feature_theater`
- `core_image`
- `core_web`

其中 `shareClient` 还会被图片加载、下载器、WebView HttpClient 等场景继续派生使用。

## 依赖关系
直接模块依赖见：
- `core/core_network/build.gradle.kts`

主要依赖：
- Retrofit / Gson Converter / RxJava2 Adapter
- OkHttp / logging-interceptor
- Cronet GMS / cronet-okhttp
- Chucker
- `core_common`
- `core_kv`
- `core_device_locale`
- `core_json`
- `core_config`
- `core_apm`
- `core_bus`

## 风险与备注
- `HeaderInterceptor` 在 debug 下写死 `internal-user-code`，调试行为与线上可能不一致
- Authorization 签名规则依赖服务端约定，并非典型标准 OAuth 签名串
- QUIC 当前明确跳过 POST 请求，因此并非所有请求都能享受 Cronet/QUIC 收益
- `ErrorCodeInterceptor` 会完整读取并重建 body，对大响应体有额外成本
- DDNS 与备用域名能力依赖系统配置下发与 MMKV 缓存，同步时序需要结合启动流程理解
- `RetrofitFactory` 中 API 备用域名白名单为硬编码，新接口若需支持要改代码而非纯配置
- `DdnsInterceptor.kt` 使用的 `BuildConfig` 包名前缀与主工程命名不完全一致，阅读时要注意历史遗留差异

## 相关页面
- [[dramawave-overview]]
- [[architecture-layering]]
- [[home-theater-attribution-network-mvi-overview]]
- [[player-system-overview]]
