# 证据目录：每个结论背后站着哪些来源

这一章更像索引。前面的章节已经解释了 DBH 的系统形状，这里把主要证据来源集中列出来，说明每个分析产物大概证明什么、不能证明什么。

公开仓库不会放这些原始研究产物。这里列出的 JSON/CSV/HTML 名称，是本地研究工作区里的聚合或报告文件名，用来让读者理解证据链。公开版只保留小型摘要表和图示。

## 证据链怎么闭合

这次分析最重要的不是某个单点数字，而是证据之间能互相对上。

资源索引告诉我们“有哪些资源”。DEP 图告诉我们“资源之间怎么引用”。sequence 扫描告诉我们“导演事件什么时候发生”。shot/lens/light、dialogue/audio、UI/flowchart、MVSHADER、post/finalize 等专项报告告诉我们“这些事件连接到哪些系统”。渲染 pipeline 和 descriptor 报告告诉我们“画面复杂度在 GPU 侧如何被 cook 和审计”。

如果只看一类证据，很容易误读。比如只看 `SEQUENCE`，你会以为这是 cutscene 时间线；把它和 DEP、对白、UI、材质、后处理、shader/pipeline 一起看，才会看出它更像互动电影的中心调度层。

## 核心来源总表

| 来源 | 关键数字 | 支撑的判断 |
|---|---:|---|
| `bigfile_index_summary.json` | 373,748 idx entries，41 types，57.31 GB | DBH 有细粒度全局资源索引，资源系统不是简单 pak 列表 |
| `bigfile_dep_full_export_summary.json` | 1,002,543 DEP edges | 资源依赖是显式可扫描数据，能支撑资源图分析 |
| `sequence_full_summary.json` | 5,479 sequences，1,117,054 timeline chunks | `SEQUENCE` 是高密度导演时间线资源 |
| `sequence_director_event_summary.json` | 616,474 director events，590,149 timed | 多数导演事件有时间信息，适合做 timeline audit |
| `interactive_decision_window_summary.json` | 17,903 timed core logic seeds | 输入、分支、变量、UI 常常和电影化事件同窗 |
| `dialogue_delivery_window_summary.json` | 7,560 timed dialogue seeds | 对白事件周围通常有镜头、相机、表演、声音 |
| `ui_prompt_window_summary.json` | 1,414 timed UI seeds | UI prompt 多数嵌在电影化和互动逻辑上下文 |
| `rendering_architecture_dossier_summary.json` | 81,649 SPIR-V，99,453 pipeline rows | 渲染复杂度被 cook 成 shader/pipeline 可审计数据 |
| `render_descriptor_abi_summary.json` | 2,408,937 descriptor occurrences | descriptor/resource vocabulary 能支持 ABI 级分析 |
| `render_pipeline_variant_summary.json` | 42,343 shader pairs，max fanout 332 | shader pair 复用和变体压力可以离线审计 |
| `shot_lens_light_summary.json` | 27,870 shots，109,424 light groups | 镜头、镜头参数和灯光是可统计的 authored 数据 |
| `camera_bank_full_summary.json` | 68 banks，32,617 feature rows | 玩法相机、目标、modifier 有专门 bank 证据 |
| `dialogue_audio_atoms_summary.json` | 80,976 FLOW_DIALOG atoms，12 audio languages | 对白、声音、动画、本地化不是孤立系统 |
| `flowchart_narrative_ui_summary.json` | 32 flowchart UI resources，2,692 ENG node keys | flowchart 是分支叙事和 UI/本地化 QA 的一部分 |
| `shader_controller_narrative_summary.json` | 5,283 MVSHADER rows，8,909 parameter occurrences | 材质 controller 是叙事、脚本和渲染之间的桥 |
| `sequence_resource_composition_summary.json` | top 40 sequence，11,622 unique refs | 强电影化段落的资源组成跨动画、声音、对白、UI、材质 |
| `finalize_data_full_summary.json` | 64 FINALIZE_DATA | 后处理/presentation preset 被资源化 |
| `post_finalize_narrative_summary.json` | 795 viewport fades，142 finalize actions | 后处理会进入 sequence/director 时间窗 |
| `script_collection_summary.json` | 855 script collections | 脚本集合是 cooked resource，而不是散落文本 |
| `script_lua_summary.json` | 18,258 scanned chunks，8,644,607 instructions | Lua/script bytecode 规模足以支撑脚本系统存在性分析 |

## 资源索引和依赖图

`bigfile_index_summary.json` 是最底层的证据之一。它给出 373,748 个 idx 条目、41 类资源和约 57.31 GB unpadded 数据。这个规模说明 DBH 的资源组织不是“几个大包 + 文件名列表”，而是非常细的资源表。

`bigfile_dep_full_export_summary.json` 则说明资源之间有大规模显式依赖：1,002,543 条 DEP edge，117,586 个 owner 有依赖。其中 547,368 条 target 能回到 idx，455,175 条属于 internal 或 non-idx target。后者不能简单理解成坏数据，它更可能包含资源内部对象或非顶层资源引用。

它们共同支撑的工程判断是：DBH 可以从构建阶段追踪“某个段落需要什么资源”。这也是后面所有 sequence resource composition、dialogue link、material bridge、post/finalize link 能成立的基础。

## Sequence 和 director event

`sequence_full_summary.json` 给出 5,479 个 `SEQUENCE`、1,117,054 个 timeline chunk、809,404 个 action marker。`sequence_director_event_summary.json` 又把这些结构折叠成 616,474 个 director event，其中 590,149 个带时间。

更关键的是类别分布。animation 有 242,821 个，camera_quake 有 162,999 个，sound 有 60,537 个，后面还有 shot、camera_modifier、watch、branch_condition、dialogue、haptic、material、input_condition、variable、ui、post_finalize 等类别。

这个证据支撑一个核心判断：`SEQUENCE` 不是单纯动画轨，也不是普通 cutscene 播放器。它更像一条导演时间线，把很多系统放进同一个时间组织里。

它不能证明的事情也很明确：它不能告诉我们官方编辑器里每个 event 的准确类型名，也不能证明 runtime 每帧执行顺序。

### 时间窗专项

三个 window summary 负责回答更具体的问题：某类事件前后 2 秒，同一条 sequence 时间线里还有什么。

`interactive_decision_window_summary.json` 看到 17,903 个 timed core logic seed。2 秒窗口里，16,441 个 seed 有 cinematic response，占 91.83%；14,409 个有 performance，占 80.48%；13,175 个有 dialogue/sound，占 73.59%。

`dialogue_delivery_window_summary.json` 看到 7,560 个 timed dialogue seed。2 秒 around 窗口里，7,472 个有 delivery context，占 98.84%；7,416 个有 performance，占 98.10%；7,161 个有 camera，占 94.72%。

`ui_prompt_window_summary.json` 看到 1,414 个 timed UI seed。2 秒 around 窗口里，1,403 个有 cinematic context，占 99.22%；1,090 个有 logic partner，占 77.09%；1,227 个有 shot，占 86.78%。

这些数字不能证明因果绑定，但它们能证明一个工程事实：互动逻辑、对白、UI 并不是散落在导演系统外面的孤岛。

## 渲染 pipeline 和 descriptor

`rendering_architecture_dossier_summary.json`、`render_descriptor_abi_summary.json` 和相关 shader/pipeline 报告，是渲染章节的主要来源。

最硬的数字包括：81,649 个 SPIR-V module、99,453 条 pipeline-state record、42,343 个 unique shader pair、729,418 条 QDIF/SPIR-V matched descriptor binding、2,408,937 次 pipeline descriptor occurrence。

这些数字支持三个判断。

第一，DBH PC 版渲染不是运行时临时猜 shader 状态，而是有大规模 cooked shader library 和 pipeline-state record。

第二，资源绑定不是黑箱。geometry buffer、material/texture、cluster/light/octree、pass texture、shadow/pass texture、material instance 等 resource vocabulary 能被重复观察。

第三，pass family 归类是有工程意义的。clustered lighting、simple texture/material、default material forward/gbuffer、material instance state、shadow/depth、post process/pass texture 这些 candidate，能帮助开发者理解 pipeline 压力来自哪里。

需要收住的是：pass family 是候选归类，不是官方 pass 名；descriptor ABI 是证据归纳，不是源码结构体。

`render_pipeline_variant_summary.json` 进一步补上变体压力。42,343 个 shader pair 里，38,477 个被复用或有状态展开；每个 shader pair 的 pipeline 数 p50 是 2，p95 是 4，最大是 332。这个最大值来自少数长尾，正是渲染工具应该自动暴露的风险点。

## Shot、lens、light 和 camera bank

`shot_lens_light_summary.json` 给出 27,870 个 shot instance、27,959 个 timed shot event、27,953 个 camera/lens group、109,424 个 light group。FOV p50 是 33.60，F-stop p50 是 4.0，timed shot duration p50 是 2,200 ms。

这组证据说明 shot 不是只有 camera cut。它携带镜头、焦点、光圈、灯光组和 key/fill/rim 等信息，足以支撑“镜头语言被数据化”的判断。

`camera_bank_full_summary.json` 则从另一个方向补上玩法相机证据：68 个 `CAMERA_SYSTEM_BANK`、1,862 个 parsed block、32,617 行 feature、16,070 个 modifier、14,033 个 target。常见 modifier 包括 smooth、deadzone、offset、auto-focus、multi-noise、advanced framing 等。

这说明 DBH 的相机系统不能只理解成“过场镜头”。它还有 camera bank、target、modifier 和 sequence override 之间的配合。

公开数据里对应两张表：`shot_lens_light_summary.csv` 和 `camera_bank_summary.csv`。前者保留 shot、lens、light 的关键聚合指标；后者保留 camera bank 的 parsed block、modifier、target、noise profile 和 sequence-side camera action 数量。两张表合起来看，比单独说“相机系统复杂”更有说服力。

## 对白、声音、本地化和表演

`dialogue_audio_atoms_summary.json` 是对白系统最重要的证据来源。它显示 80,976 个 `FLOW_DIALOG` atom、6,748 个 unique base、12 个 audio language；并且 80,946 个 atom 能和 sound event base match，74,364 个 atom 带 anim data。

这个联表非常关键。它说明对白不是单独的字幕 key，也不是裸音频事件。更合理的工程抽象是 `DialogueAtom`：一个 atom 同时连接声音、本地化、口型/表演动画和 sequence 事件。

这里仍然不能过度解读。我们不能从聚合表反推出完整对白文本，也不能把 atom 的内部字段语义全部写死。公开仓库也不会发布对白全文或本地化全文。

## UI 和 flowchart

`flowchart_narrative_ui_summary.json` 支撑 flowchart 章节。它显示 128 个 UI resource、32 个 flowchart UI resource、32 个 expected flowchart GFX、2,692 个 ENG flowchart node key、26 个 localization language，以及 Markus/Kara/Connor portrait asset presence 各 32。

这说明 flowchart 不是临时菜单。它和章节结构、节点文案、本地化、角色头像、路径可见性、checkpoint/statistics 等内容有稳定关系。

对自研引擎来说，这个证据最有价值的地方是 QA 反向约束：分支叙事如果能被玩家看到，就必须能被工具链检查。flowchart UI 不只是显示结果，它还倒逼剧情图、变量、checkpoint、本地化和资源引用保持一致。

## 材质 controller

`shader_controller_narrative_summary.json` 是跨系统证据最强的来源之一。它给出 5,283 条 `MVSHADER` row，全部 parse ok，全部能 join 到 director event，全部是 timed row；另有 8,909 次参数出现、281 个 unique parameter、9 个 controller family。

主要 family 包括 damage/blood/wound、android LED、weather/fluid/dirt、android skin retract、cloth/hair、face/tears/scan 等。

这个证据支撑的结论是：材质 controller 不是普通 shader 参数面板，而是叙事状态进入渲染的接口。角色受伤、血迹、雨水、污渍、LED、眼泪等状态，需要跨 sequence、脚本、材质和 GPU layout 保持一致。

它不能证明每个参数的官方名称和运行时更新函数，但足以支撑引擎设计建议：MaterialController 应该成为跨系统数据模型，而不是临时脚本调用。

`material_controller_family_summary.csv` 把 family 分布压成公开表。damage/blood/wound 有 4,052 次参数出现，android LED 有 1,977 次，weather/fluid/dirt 有 1,586 次。这个分布让“叙事状态进入材质”这句话更具体：它主要不是抽象颜色动画，而是角色受伤、仿生人身份状态、雨水污渍和面部状态这些会跨镜头延续的东西。

## 后处理和 finalize

`finalize_data_full_summary.json` 和 `post_finalize_narrative_summary.json` 共同支撑后处理章节。

`FINALIZE_DATA` 有 64 个资源，64 个都有 `COLOR_GR` block，63 个有 `GRAIN___` block。sequence 侧能看到 210 条 sequence -> finalize ref row，795 个 raw viewport fade action，142 个 raw invoke finalize action，882 条 post director event row。

这些证据说明后处理/presentation preset 被资源化，并且会进入导演时间窗。它不是相机脚本里几个临时参数。

对自研引擎来说，最直接的设计建议是 PostPreset + PostStack：preset 做成资源，运行时用层级和恢复策略管理 gameplay、camera、sequence、debug override。

## 脚本和 Lua

`script_collection_summary.json` 与 `script_lua_summary.json` 说明脚本系统也被 cook 成资源。公开指标包括 855 个 script collection、18,258 个 scanned chunk、8,644,607 条 instruction。

这些数字能支撑“脚本集合存在且规模很大”的判断，也能帮助解释为什么变量、branch、input、material controller 不应该只看 sequence。互动叙事的状态一定还和脚本层有关。

但这部分边界也最清楚：当前文档不证明完整 runtime control-flow graph，不还原变量槽真实语义，也不发布脚本 bytecode。

## 读者应该怎样使用这个目录

如果你是引擎开发者，建议把这一章当成“结论可信度表”：

1. 看到强数字，例如 idx 条目数、DEP edge、sequence 数、pipeline rows，可以把它当成系统规模证据。
2. 看到跨表联结，例如 MVSHADER join director event、FLOW_DIALOG match sound event、sequence -> finalize ref，可以把它当成系统关系证据。
3. 看到 candidate/evidence 词，例如 pass family candidate、descriptor ABI evidence、response-window evidence，要把它当成工程归纳，而不是官方命名。

这也是整个公开版的写法原则：少一点神秘感，多一点可检查的证据链。
