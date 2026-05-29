# 公开数据怎么读

这一章只讲仓库里的 `data/` 文件。它的目的很简单：如果读者不想只相信正文，可以直接打开这些小表，看到每个判断背后的聚合数字。

这些表都经过压缩和筛选，只保留公开安全的指标。它们不包含原始游戏资产，不包含二进制样本，不包含对白全文，也不包含能还原剧情内容的大型明细行。

## 先看哪几张表

如果你只想快速判断这份分析有没有证据，建议按这个顺序看：

1. `key_metrics.json`：总览数字。它告诉你资源、sequence、渲染、对白、UI、脚本和窗口分析的大致规模。
2. `system_evidence_matrix.csv`：每个系统对应哪些证据、能说明什么、不能说明什么。
3. `resource_type_summary.csv`：资源类型分布。它能解释为什么这不是一个只围绕 mesh/texture 的项目。
4. `director_event_category_summary.csv`：导演事件类别。它能证明 `SEQUENCE` 调度的不只是动画。
5. `interaction_window_summary.csv`、`dialogue_delivery_window_summary.csv`、`ui_prompt_window_summary.csv`：三张“时间窗”表。它们解释为什么我说 DBH 更像实时导演系统。
6. `shot_lens_light_summary.csv`、`camera_bank_summary.csv`：镜头、灯光和 camera bank 证据。
7. `material_controller_family_summary.csv`：材质 controller family 分布。
8. `render_family_profiles.csv`：渲染 pipeline 的候选 family 和变体压力。
9. `sequence_resource_composition_summary.csv`：top 40 cinematic sequence 的资源组成。

## 时间窗表是什么意思

三张 window 表都在回答同一个问题：

> 一个事件发生前后 2 秒，同一条 sequence 时间线里还有什么？

这不是在证明“运行时一定按这个顺序调用函数”。它证明的是另一件事：这些事件在作者时间线上经常被安排在同一个上下文里。

比如 `interaction_window_summary.csv` 里有 17,903 个 timed core logic seed。它们包括 branch、input、variable、time control、UI。2 秒窗口里，16,441 个 seed 旁边有 cinematic response，占 91.83%；12,196 个旁边有 shot，占 68.12%；13,175 个旁边有 dialogue/sound，占 73.59%。

这说明互动逻辑不是孤零零挂在 cutscene 外面的 QTE manager。它常常和镜头、表演、声音、材质、UI 放在同一条导演时间线上。

`dialogue_delivery_window_summary.csv` 也类似。7,560 个 timed dialogue seed 里，2 秒 around 窗口有 7,472 个带 delivery context，占 98.84%；7,416 个带 performance，占 98.10%；7,161 个带 camera，占 94.72%。这就是为什么正文把对白叫作 `DialogueAtom`：对白不是一行字幕，也不是一个音频事件，它往往连着表演、镜头、声音和互动逻辑。

`ui_prompt_window_summary.csv` 则说明 UI prompt 不是悬浮按钮。1,414 个 timed UI seed 里，2 秒窗口有 1,403 个带 cinematic context，占 99.22%；1,227 个带 shot，占 86.78%；1,090 个带 logic partner，占 77.09%。也就是说，提示出现的位置大多处在镜头、表演、对白/声音和输入/分支逻辑交汇的地方。

## 镜头和材质表怎么读

`shot_lens_light_summary.csv` 适合验证“镜头语言是数据”这句话。它里面有 27,870 个 shot instance、27,953 个 camera/lens group、109,424 个 light group。86.77% 的 shot 有 lens evidence，92.58% 有 light evidence，30.29% 有 key/fill/rim 证据。这个比例足以说明：镜头不是切一下相机就完了，它还带着镜头参数和灯光组织。

`camera_bank_summary.csv` 则看 gameplay camera 和 director camera 的连接。68 个 camera bank 里有 1,862 个 parsed camera block，16,070 个 modifier feature，14,033 个 target feature，1,936 个 noise profile；sequence 侧还有 2,019 条 camera action，覆盖 1,233 个 sequence。这些数字不能还原完整抢权规则，但能说明 camera bank 是一个真实的 authored subsystem。

`material_controller_family_summary.csv` 适合验证“材质 controller 是叙事状态接口”这句话。damage/blood/wound 有 4,052 次参数出现，android LED 有 1,977 次，weather/fluid/dirt 有 1,586 次。它们都不是纯技术分类，而是直接贴着角色状态、环境状态和身份表现。

## 渲染 family 表怎么读

`render_family_profiles.csv` 不是官方 pass 列表。它是根据 shader、pipeline state 和 resource vocabulary 做出的候选归类。

最明显的是 `clustered_lighting`：48,718 条 pipeline，占 48.99%。这说明 clustered lighting 相关资源词汇在 pipeline 中非常常见。其次是 `simple_texture_material`，23,198 条，占 23.33%。`material_instance_state` 有 10,135 条，占 10.19%，说明 material instance buffer 不是偶然出现的资源名，而是能形成独立分析层。

表里还有 p95 和 max。比如 `shadow_or_depth` 只有 2,906 条 pipeline，但 max pipelines per shader pair 是 332。这个数字很适合做工具标红：总量不大，不代表没有极端变体压力。

## sequence 资源组成表怎么读

`sequence_resource_composition_summary.csv` 只看 top 40 cinematic sequence。它不是全量资源图，也不证明运行时加载顺序。

但它很能说明“电影化段落为什么复杂”。这 40 个 sequence 一共有 11,622 个 nonexclusive unique resource refs，平均每个 290.55 个；named sequence ref rows 有 28,833 行，action direct ref rows 有 34,117 行，dialogue flow link rows 有 3,200 行，action marker 里还有 1,981 行 `MVSHADER`。

top ref type 也很清楚：`ANIM_DATA` 48,095 行，`COM_SOUND_EVENT` 22,843 行，`FLOW_DIALOG` 3,200 行，`LOCALIZATION_CONTAINER` 451 行，`UI_MENU_RESOURCE` 70 行，`FINALIZE_DATA` 3 行。

这张表要表达的不是“每个资源都同一帧加载”。它表达的是：强电影化段落的资源组成天然跨系统，动画、声音、对白、本地化、UI、材质和后处理都能进入同一个 sequence 证据链。

## 这些表不能证明什么

公开数据仍然有边界：

- 它不能证明官方字段名。
- 它不能证明源码里的类结构。
- 它不能证明真实 runtime load order。
- 它不能证明 exact same-frame sync。
- 它不能还原对白文本、本地化文本、动画、音频、贴图或脚本源码。

所以正确读法是：把这些表当成“工程证据摘要”。它们足以支撑系统设计讨论，但不应该被当成官方实现文档。

## 对自研引擎最有用的读法

如果你正在做自己的引擎，不需要照搬这些数字。真正该学的是这些表背后的审计方式：

1. 资源要能按类型和依赖导出。
2. sequence 要能导出事件类别、时间窗和资源组成。
3. 对白、UI、输入、分支要能做同窗分析。
4. 渲染 pipeline 要能按 family、shader pair、descriptor/resource vocabulary 和变体压力导出。
5. 每个候选结论都要有边界说明，避免把推断写成官方事实。

这就是公开数据的价值：它不炫技，也不还原内容，只让工程判断有地方落脚。
