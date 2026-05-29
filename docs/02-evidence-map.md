# 证据地图

这份公开版只保留聚合指标和工程结论，不发布原始样本、二进制或大规模导出表。

## 顶层指标

| 方向 | 指标 |
|---|---:|
| BigFile idx 条目 | 373,748 |
| 资源类型 | 41 |
| 资源 unpadded 总量 | 57.31 GB |
| 完整 DEP edge | 1,002,543 |
| 有依赖的 owner | 117,586 |
| SEQUENCE 数量 | 5,479 |
| timeline chunk | 1,117,054 |
| director event | 616,474 |
| timed primary action | 320,455 |
| 30fps 时间网格比 | 0.999978 |
| SPIR-V module | 81,649 |
| pipeline-state record | 99,453 |
| unique shader pair | 42,343 |
| texture resource | 127,939 |
| FLOW_DIALOG atom | 80,976 |
| ANIM_DATA | 24,562 |
| UI_MENU_RESOURCE | 128 |
| FINALIZE_DATA | 64 |

## 12 个系统块

研究整理出的架构可以拆成 12 个系统块：

1. `asset_index_and_dependency_graph`
2. `container_and_streaming`
3. `rendering_pipeline`
4. `shader_material_controller`
5. `post_finalize_stack`
6. `sequence_director_timeline`
7. `shot_camera_lighting`
8. `camera_system_bank`
9. `animation_performance`
10. `dialogue_audio_localization`
11. `script_logic_variables`
12. `ui_scaleform_flowchart_subtitle`

这些名字不是官方类名，而是根据资源、依赖、时间线和渲染证据归纳出的工程视角。

## 强证据

这些结论比较硬，可以作为架构判断基础：

- `BigFile_PC.idx` 和 `BigFile_PC.dep` 形成全局资源索引和依赖图。
- `SEQUENCE` 外层引用与 DEP 对齐；top cinematic sequence 的 outer dependency 在 DEP 中有完整 idx-backed 证据。
- `SEQUENCE` 全量扫描没有 parse error，能恢复大量 timeline chunk、action marker、shot 和 reference。
- `MOV_SHOT` 能抽取镜头、镜头组、镜头属性和灯光/镜头参数。
- `FLOW_DIALOG` 与声音、事件、动画、本地化存在稳定联表。
- Scaleform UI/flowchart/subtitle 资源和本地化 key 有可见组织。
- 渲染侧存在大规模 SPIR-V、pipeline-state manifest、descriptor/resource vocabulary 和 shader-pair reuse/variant evidence。
- `MVSHADER`、脚本 controller 名称、材质 payload、shader/material ABI 之间有多层桥接证据。

## 保守候选

这些结论适合指导设计，但不应该写成官方字段名或 exact runtime causality：

- pass family candidate
- descriptor/resource ABI evidence
- pipeline-state tuple candidate
- shader key tuple candidate
- render-state tuple candidate
- cinematic-render bridge evidence
- director-event response-window evidence
- director-branch topology evidence
- dialogue-delivery window evidence
- UI/action-prompt window evidence
- sequence resource composition evidence

简言之：它们说明“工程上很可能是这样组织/协作”，但不代表我们拿到了 Quantic Dream 内部源码或编辑器字段名。

## 为什么这些证据对开发者有用

因为它们不是孤立亮点，而是能互相闭合：

- 资源图能解释 sequence 为什么能追踪依赖。
- 时间线能解释镜头、对白、动画、UI、输入为什么能同步。
- 渲染 manifest 能解释画面复杂度为什么没有完全变成运行时混乱。
- 对白/字幕/UI/flowchart 证据能解释分支叙事为什么能被 QA。
- material controller 证据能解释角色状态为什么能跨脚本、镜头和渲染保持连续。

这才是对自研引擎真正有价值的部分。
