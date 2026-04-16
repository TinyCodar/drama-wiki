# DramaWave Codebase Scan Raw Notes

- 扫描时间：2026-04-16
- 工程路径：`/Users/tinycoder/Documents/workspace/dramawave`
- 扫描方式：读取 `settings.gradle.kts`、`build.gradle.kts`、`app/build.gradle.kts`、`gradle.properties`、`gradle/libs.versions.toml`、`AndroidManifest.xml`、`DramaApp.kt`、`ApplicationLoader.kt`、`CommonInitializer.kt`、`MainActivity.kt`，并统计顶层模块文件分布。

## 关键观察
- 项目是大型 Android 多模块工程，分层为 `app → feature → interface → shared → core`。
- 构建栈：AGP 8.8.0、Kotlin 2.1.0、compileSdk/targetSdk 35、minSdk 23。
- `settings.gradle.kts` 使用 `RepositoriesMode.FAIL_ON_PROJECT_REPOS`，并引入 `build-logic` 与 `plugins/view-monitor` included build。
- `app` 同时开启 ViewBinding、DataBinding、Compose、BuildConfig。
- 产品风味至少包含 `dramawave` 与 `freereels`。
- 启动入口为 `DramaApp`，其内通过 `StartUpManager` + `ApplicationLoader` 编排多个 `AndroidStartup` 任务。
- APM/性能链路当前以 `PerformanceHelper`、Firebase Perf、CrashReporter、StartupPerformanceTracker 为主。
- `MainActivity.kt` 文件体量极大（约 2370 行），承担主导航、业务事件汇聚与多模块协调职责。
- 存在构建配置安全风险：`settings.gradle.kts` 的 GitHub Maven 仓库配置出现明文凭据痕迹，wiki 页面中只记录风险，不保留密钥正文。

## 粗粒度文件统计
- `app`: 114 文件，72 个 Kotlin，21 个 XML
- `feature`: 3147 文件，1542 个 Kotlin，979 个 XML
- `interface`: 112 文件，44 个 Kotlin
- `shared`: 2661 文件，1318 个 Kotlin，674 个 XML
- `core`: 622 文件，413 个 Kotlin，77 个 Java
- `plugins`: 85 文件，59 个 Kotlin

## 扩展名分布（粗略）
- `.kt`: 3457
- `.xml`: 1721
- `.webp`: 597
- `.md`: 464
- `.png`: 455
- `.pro`: 124
- `.java`: 104
- `.kts`: 83

该原始记录仅作为 wiki 页面生成时的参考摘要，后续正式知识沉淀以 `concepts/` 与 `queries/` 下的页面为准。
