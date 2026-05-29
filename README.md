# DBH Engine Analysis

这是一份面向游戏引擎开发者的中文整理稿。它把我对《Detroit: Become Human》PC 版资源包、依赖图、渲染管线、导演时间线、对白/动画/UI 等系统的离线分析，整理成更容易读的版本。

我想回答的不是“DBH 某个文件格式怎么解”，而是一个更适合做引擎的人关心的问题：

> 如果我们想做一个强剧情、强表演、强分支的游戏引擎，DBH 这套生产系统到底有什么值得学？

这里的结论不是官方文档，也不是源码级逆向。更准确地说，它是一份工程复盘：从可验证的资源索引、依赖表、时间线扫描、shader/pipeline 统计、对白和 UI 联表里，推导出 DBH 的系统组织方式，并把它翻译成自研引擎可以吸收的设计建议。

## 最短结论

DBH 不是“普通游戏引擎 + 播片系统”。它更像一套围绕互动电影生产定制的 cooked asset-graph cinematic engine。

这句话听起来很硬，换成通俗说法就是：

- 资源不是简单打包进一个大文件，而是进入全局资源表和依赖图。
- 剧情段落不是散落在脚本回调里，而是进入 `SEQUENCE` 这类导演时间线。
- 镜头、动画、对白、声音、UI 提示、输入窗口、变量、分支、材质状态和后处理，都能被放到同一个时间窗里检查。
- 渲染不是运行时临时拼 shader，而是提前 cook 出 shader library、pipeline-state record、descriptor/resource ABI 和 pass family 证据。
- 工具链可以反查一个段落为什么复杂、为什么像电影、为什么会贵、为什么会难维护。

所以 DBH 真正值得学的不是某个具体 shader，也不是某个二进制格式，而是它把“作者意图”变成可追踪、可统计、可审计的数据。

## 一眼看证据

| 方向 | 关键证据 |
|---|---:|
| 资源索引 | 373,748 个 idx 条目，41 类资源，约 57.31 GB unpadded 数据 |
| 依赖图 | 1,002,543 条 DEP edge，117,586 个 owner 有依赖 |
| Sequence | 5,479 个 `SEQUENCE`，1,117,054 个 timeline chunk |
| 导演事件 | 616,474 个 director event，其中 590,149 个带时间 |
| 镜头 | 27,959 个 timed shot，109,424 个 light group |
| 对白 | 80,976 个 `FLOW_DIALOG` atom，6,748 个多语言对白 base |
| 渲染 | 81,649 个 SPIR-V module，99,453 条 pipeline-state record |
| 材质控制 | 5,283 个 timed `MVSHADER`，8,909 次参数引用 |
| UI/Flowchart | 128 个 UI resource，32 个 flowchart UI resource，2,692 个英文 flowchart node key |
| 互动时间窗 | 17,903 个 timed core logic seed，2 秒内 91.83% 有 cinematic response |
| 对白时间窗 | 7,560 个 timed dialogue seed，2 秒内 98.84% 有 delivery context |
| UI 时间窗 | 1,414 个 timed UI seed，2 秒内 99.22% 有 cinematic context |
| Top 40 sequence 资源组成 | 11,622 个 nonexclusive unique resource refs，平均每个 sequence 290.55 个 |

这些都是公开版保留的聚合证据。仓库不会上传原始游戏资产、二进制样本、对白全文、本地化全文或大型扫描表。

## 怎么读

如果你只想快速理解整体架构，先读：

- [docs/01-overview.md](docs/01-overview.md)

如果你想看“证据从哪里来”，读：

- [docs/02-evidence-map.md](docs/02-evidence-map.md)
- [docs/07-evidence-catalog.md](docs/07-evidence-catalog.md)
- [docs/08-public-data-guide.md](docs/08-public-data-guide.md)

如果你关心渲染，读：

- [docs/03-rendering.md](docs/03-rendering.md)

如果你关心剧情、分支、镜头、对白、UI，读：

- [docs/04-cinematic-director.md](docs/04-cinematic-director.md)

如果你想把这些东西转成自己的引擎设计，读：

- [docs/05-build-your-engine.md](docs/05-build-your-engine.md)

如果你想知道哪些说法不能过度解读，读：

- [docs/06-boundaries.md](docs/06-boundaries.md)

## 仓库内容

- `docs/`：通俗版正文和证据说明。
- `data/key_metrics.json`：关键聚合指标。
- `data/resource_type_summary.csv`：公开安全的资源类型概览。
- `data/system_evidence_matrix.csv`：系统、证据、指标和工程含义的对照表。
- `data/evidence_sources.json`：本整理稿使用的主要原始研究产物索引。
- `data/interaction_window_summary.csv`：互动逻辑事件周围 2 秒的同窗证据。
- `data/dialogue_delivery_window_summary.csv`：对白事件周围 2 秒的镜头/相机/表演/声音证据。
- `data/ui_prompt_window_summary.csv`：UI prompt 周围 2 秒的输入、分支和电影化上下文证据。
- `data/director_event_category_summary.csv`：导演事件类别和 timed event 对照。
- `data/shot_lens_light_summary.csv`：shot、镜头参数和灯光组的聚合证据。
- `data/camera_bank_summary.csv`：camera bank 的 modifier、target、noise profile 聚合证据。
- `data/material_controller_family_summary.csv`：`MVSHADER` controller family 的参数出现次数。
- `data/render_family_profiles.csv`：渲染 pass family candidate 的 pipeline、shader pair 和变体压力概览。
- `data/sequence_resource_composition_summary.csv`：top 40 cinematic sequence 的资源组成聚合证据。
- `assets/dbh_vs_ue_unity_architecture.svg`：DBH 与常见 UE/Unity 项目组织方式的架构对照图。
- `assets/evidence_chain.svg`：从资源包到工程结论的证据链图。

## 不包含什么

这个仓库刻意不包含以下内容：

- 游戏原始文件。
- 提取出的贴图、音频、动画、脚本 bytecode。
- 二进制样本。
- 大型 CSV 原始扫描表。
- 对白、本地化全文或可还原游戏内容的大段导出。
- Quantic Dream、Sony 或其他权利方的专有代码/资产。

公开版只保留聚合指标、解释性文字、证据索引和自研引擎设计建议。

## License

本文档和本仓库自写内容使用 JSON License，见 [LICENSE](LICENSE)。

注意：JSON License 带有 “The Software shall be used for Good, not Evil.” 条款，在某些严格开源生态中可能存在兼容性争议。本仓库按最初整理需求采用它。

## Disclaimer

Detroit: Become Human is a trademark and copyrighted work of its respective owners. This repository is an independent educational analysis and is not affiliated with or endorsed by Quantic Dream, Sony, or the game's publishers.
