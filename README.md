# DBH Engine Analysis

一份面向自研游戏引擎开发者的通俗版研究笔记，整理自对《Detroit: Become Human》PC 版资源包、依赖图、渲染管线、导演时间线、对白/动画/UI 等系统的离线分析。

这不是源码逆向，也不是官方文档。它更像一份工程读书笔记：从已验证的资源结构和聚合数据里，解释 DBH 为什么不像“普通游戏引擎 + 播片系统”，而更像一台围绕互动电影生产的 **cooked asset-graph cinematic engine**。

## 先看结论

DBH 的核心不是单个炫技 shader，也不是一个 cutscene player。它把剧情、镜头、动画、对白、声音、字幕、UI、输入、分支、材质状态、后处理和渲染预编译，全部放进可 cook、可依赖追踪、可 profile 的数据系统里。

对自研引擎最重要的启发是：

1. 先做资源图，不要只做一个 pak。
2. 把 Sequence/Director Timeline 当成一等公民，不要把剧情散落在脚本回调里。
3. 对白、动画、镜头、UI prompt、分支条件要能在同一个时间窗里审计。
4. 渲染管线要有 pipeline cache、descriptor/resource ABI、pass family、shader variant 的离线报告。
5. 材质/角色状态 controller 要连通脚本、时间线、资产和渲染，不要各做一套孤岛 API。
6. 工具链和 profiler 决定电影化上限，runtime 只是其中一部分。

## 阅读顺序

- [docs/01-overview.md](docs/01-overview.md)：最短路径，讲清楚 DBH 的整体架构。
- [docs/02-evidence-map.md](docs/02-evidence-map.md)：关键证据和指标，说明哪些结论比较硬。
- [docs/03-rendering.md](docs/03-rendering.md)：渲染为什么强，重点是工程化而不是单个效果。
- [docs/04-cinematic-director.md](docs/04-cinematic-director.md)：电影化叙事为什么不是播片。
- [docs/05-build-your-engine.md](docs/05-build-your-engine.md)：如果自己做引擎，应该抄哪些思想。
- [docs/06-boundaries.md](docs/06-boundaries.md)：证据边界，哪些不能过度解读。

## 公开版包含什么

本仓库只包含适合公开阅读的文档、聚合指标和无原始资产的图示：

- `docs/`：通俗版分析正文。
- `data/key_metrics.json`：脱敏后的聚合指标。
- `assets/dbh_vs_ue_unity_architecture.svg`：架构对照图。

不包含：

- 游戏原始文件、二进制样本、提取的贴图/音频/动画/脚本。
- 大型 CSV 原始扫描表。
- 对白、本地化全文或可还原游戏内容的大段导出。
- Quantic Dream、Sony 或其他权利方的专有代码/资产。

## License

本文档和本仓库自写内容使用 JSON License，见 [LICENSE](LICENSE)。

注意：JSON License 包含 “Good, not Evil” 条款，在某些严格开源生态里可能被认为不符合 OSI/自由软件兼容预期。这里按原始整理需求采用它。

## Disclaimer

Detroit: Become Human is a trademark and copyrighted work of its respective owners. This repository is an independent educational analysis and is not affiliated with or endorsed by Quantic Dream, Sony, or the game's publishers.
