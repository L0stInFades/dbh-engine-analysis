# 证据边界

这份分析坚持一个原则：能从当前证据证明的，就写成结论；只能间接支持的，就写成 candidate/evidence；没有证明的，不假装知道。

## 不是官方文档

本仓库不是 Quantic Dream 官方资料，也没有源码、编辑器工程或内部字段名。文档中的系统名主要是工程归纳名，不等于官方类名。

## 已经能较强支持的判断

- DBH PC 版有全局资源索引和完整依赖图。
- 大量 sequence、timeline chunk、director event 能被离线扫描和统计。
- sequence 与动画、对白、声音、UI、后处理、材质 controller 等资源存在稳定引用关系。
- 渲染侧存在大规模 SPIR-V、pipeline state、descriptor/resource vocabulary、shader pair 和 texture resource 证据。
- Dialogue/Audio/Localization/UI/Flowchart 不是完全孤立系统。
- material controller 至少在 sequence、script、asset/material、render ABI 之间形成了强桥接证据。

## 只作为候选解释的判断

以下内容适合用于架构学习，但不能当作官方字段名、运行时因果链或 exact same-frame sync：

- pass family candidate
- resource vocabulary evidence
- descriptor ABI evidence
- pipeline-state tuple candidate
- shader key tuple candidate
- render-state tuple candidate
- verified shader reference tuple
- cinematic-render bridge evidence
- director-event response-window evidence
- director-branch topology evidence
- dialogue-delivery window evidence
- UI/action-prompt window evidence
- sequence timeline browser evidence
- full-DEP sequence resource graph evidence
- sequence resource composition evidence

## 明确没有完成的部分

这些仍然需要更多研究，当前文档不把它们写成确定事实：

- `QZIP` 内部字段完整语义。
- `DATA_CONTAINER/segs` 完整压缩/块格式。
- 所有曲线 payload 语义。
- `dep_token/dep_slot` 精确字段含义。
- 长 property graph 的真实字段名。
- `SCRIPT_COLLECTION/SBCHK___` 的完整图结构和变量槽。
- 每个 director event 的最终编辑器类型名。
- 真实 runtime load order。
- 真实 runtime control-flow graph。

## 公开发布边界

为了避免发布不适合公开的材料，本仓库不包含：

- 原始游戏资产。
- 二进制样本。
- 大型原始扫描表。
- 提取出的音频、贴图、动画、脚本 bytecode。
- 对白或本地化全文。

公开版只保留聚合指标、通俗解释、工程启发和无资产图示。
