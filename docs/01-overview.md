# DBH 引擎通俗总览

## 一句话

DBH PC 版最适合被理解成一个 **围绕互动电影生产的 cooked asset-graph cinematic engine**。

更通俗地说：它不是“普通游戏引擎里塞了很多播片”，而是从资源构建、依赖关系、渲染预编译、镜头调度、对白、动画、UI、输入、分支，到调试审计工具，都围绕“如何稳定生产可交互电影段落”来组织。

## 它和普通项目的差别

很多 UE/Unity 项目会先有场景、对象、组件、脚本，再把 cutscene、QTE、字幕、UI、相机、材质参数、后处理分别接上去。这样也能做游戏，但系统之间容易变成一堆运行时回调。

DBH 的证据更像另一种路线：

- 先有全局资源索引和依赖图。
- 大部分内容都被 cook 成可扫描、可引用、可统计的数据。
- `SEQUENCE` 像导演时钟，能调度动画、镜头、声音、对白、UI、输入、变量、分支、材质和后处理。
- 渲染侧不是临场拼 shader，而是有大规模预编译 shader、pipeline state、descriptor/resource ABI 和 variant 数据。
- 工具链可以从资源图、时间线、渲染、对白、UI 等多个角度反查一个段落为什么复杂、为什么像电影。

## 三个底层支柱

### 1. 资源图

资源索引里有 373,748 个条目、41 类资源，完整依赖图有 1,002,543 条边。这个资源图让引擎能回答：

- 一个 sequence 依赖哪些动画、对白、声音、UI 和后处理资源？
- 一个资源是顶层 idx-backed resource，还是内部对象引用？
- cook 后资源大小、类型、包位置和依赖关系是否可审计？

对自研引擎来说，这比“把资源塞进 pak 里”更重要。pak 是存储格式，资源图是生产系统。

### 2. 导演时间线

全量扫描恢复出 5,479 个 `SEQUENCE`，1,117,054 个 timeline chunk，616,474 个 director event，320,455 个 timed primary action。时间网格高度贴近 30fps。

这说明很多“电影感”不是运行时临时配合出来的，而是大量 authored intent 被放进同一套时钟里：镜头什么时候切、角色什么时候动、对白什么时候出、声音和 UI prompt 在哪里、玩家输入窗口什么时候打开、材质和后处理什么时候变。

### 3. 渲染工程化

渲染侧有 81,649 个 SPIR-V module、99,453 条 pipeline-state record、42,343 个 shader pair、127,939 个 texture resource。报告里看到的 pass family、descriptor/resource ABI、shader-pair variant pressure，都指向同一个事实：DBH 的渲染复杂度被提前 cook、分类和审计。

这给自研引擎的启发是：不要只追求“效果列表”，还要有 pipeline cache builder、shader reflection、resource ABI manifest、variant atlas 和渲染审计工具。

## 架构图

![DBH vs UE/Unity architecture](../assets/dbh_vs_ue_unity_architecture.svg)

## 最该带走的设计思想

电影化不是一个播放器功能，而是一套生产系统。真正值得借鉴的是：

- 资源依赖图。
- Sequence/Director Timeline。
- Shot/Lens/Light 数据化。
- DialogueAtom 和 SubtitleService 分离。
- Camera bank 和 camera motion language。
- Material controller 跨脚本、时间线、资产、渲染。
- Post/finalize stack 资源化。
- Flowchart/UI/branch review。
- Pipeline cache 和 rendering dossier。
- Sequence profiler 和 resource composition review。
