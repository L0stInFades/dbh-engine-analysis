# 证据边界：哪些话能说，哪些话要收住

这份整理稿不是为了把所有猜测都包装成结论。它更像一份工程读书笔记：把能从资源索引、依赖图、时间线、shader/pipeline 表、对白/UI 联表里稳定看到的东西讲清楚；把只能间接支持的推断标成 candidate 或 evidence；把还没证明的部分留白。

这个边界很重要。DBH 的数据规模很大，确实能支撑很多工程判断，但它仍然不是源码、不是官方编辑器工程、也不是运行时调试器录制。所以读这份文档时，应该把它当作“离线证据驱动的系统复盘”，而不是官方内部架构说明。

## 它不是什么

首先，这不是 Quantic Dream 的官方文档。文档里使用的很多名字，比如“导演时间线”“资源图”“descriptor ABI”“pass family candidate”“material controller bridge”，都是为了方便工程讨论而做的归纳名，不等于官方 C++ 类名、编辑器面板名或内部枚举名。

其次，这不是源码级逆向。研究没有拿到引擎源码，也没有证明每个字段在运行时的 exact control flow。文档会讨论“这些资源之间形成了强证据关系”，但不会把它写成“运行时一定按这个顺序执行”。

最后，这也不是内容还原项目。公开仓库不发布游戏原始资产，不发布二进制样本，不发布对白全文，不发布提取出的贴图、音频、动画或脚本 bytecode。保留下来的只有聚合指标、解释性文字、图示和小型公开安全数据表。

## 可以较强支持的判断

下面这些判断有比较硬的结构化证据支持，适合作为本仓库的主结论：

- PC 版资源包里存在大规模全局资源索引：373,748 个 idx 条目、41 类资源、约 57.31 GB unpadded 数据。
- 完整依赖导出里有 1,002,543 条 DEP edge，其中 547,368 条能回到 idx-backed target，455,175 条更像 internal 或 non-idx target。
- `SEQUENCE` 是重要的一等资源：全量扫描得到 5,479 个 sequence、1,117,054 个 timeline chunk、616,474 个 director event。
- 大多数 director event 带时间信息：590,149 个 timed event，足以支撑“导演时间线可以被离线审计”的判断。
- director event 不只覆盖动画和镜头，还覆盖声音、对白、输入、分支、变量、UI、材质、后处理等类别。
- 渲染侧存在大规模 cooked 证据：81,649 个 SPIR-V module、99,453 条 pipeline-state record、42,343 个 unique shader pair。
- descriptor/resource vocabulary 能被离线观察和统计，例如 geometry buffer、material/texture、cluster/light、pass texture、shadow texture、material instance 等资源角色。
- shot/lens/light 数据规模很大：27,870 个 shot instance、109,424 个 light group，镜头语言明显不是一个简单 camera cut。
- `FLOW_DIALOG`、声音、动画、本地化和 UI/flowchart 之间存在稳定联表证据，对白更像表演 atom，而不是孤立文本或音频。
- `MVSHADER` 与 director event、脚本 controller、材质 payload、shader/material ABI 之间有强桥接证据，适合讨论“叙事状态如何进入材质和渲染”。

这些话仍然要注意措辞。比如“存在 descriptor ABI evidence”是合理的；“官方就叫 DescriptorABI”是不合理的。前者是在描述证据，后者是在伪造命名。

## 只能作为候选解释的判断

下面这些概念对引擎设计很有启发，但不能当作官方事实、真实字段名或 exact same-frame runtime sync：

| 候选概念 | 正确读法 |
|---|---|
| pass family candidate | 基于 pipeline/shader/resource evidence 的离线归类，不是官方 pass enum |
| descriptor ABI evidence | 资源绑定、descriptor key、binding key、resource name 的稳定观察结果，不是已知源码结构体 |
| pipeline-state tuple candidate | 可以从状态表中抽出的 pipeline 状态组合，不代表官方 cache key |
| shader key tuple candidate | 用于分析 shader pair/variant 的候选 key，不代表官方 shader 编译 key |
| render-state tuple candidate | 离线整理出的 render-state 组合，不等于运行时完整状态对象 |
| verified shader reference tuple | 能交叉验证的 shader 引用关系，不等于完整 runtime shader graph |
| cinematic-render bridge evidence | sequence/material/post 与渲染证据之间的桥接，不证明每帧因果顺序 |
| director-event response-window evidence | 事件同窗统计，说明共现关系强，不直接证明玩家感知因果 |
| director-branch topology evidence | 分支/变量/输入事件结构证据，不等于完整 runtime control-flow graph |
| dialogue-delivery window evidence | 对白、声音、表演和 UI 的时间窗共现证据，不等于官方对白系统设计文档 |
| UI/action-prompt window evidence | UI prompt 与镜头/输入/对白的共现证据，不证明每个提示的最终规则 |
| sequence timeline browser evidence | 离线时间线浏览器可以重建的结构，不代表官方编辑器界面 |
| full-DEP sequence resource graph evidence | DEP 图能追踪 sequence 资源关系，不代表真实 runtime load order |
| sequence resource composition evidence | sequence 引用组成统计，不代表每个引用都在同一帧加载或播放 |

候选解释不是“不能用”。它们非常适合指导自研引擎设计，因为工程上最有价值的往往不是字段名，而是系统关系。但它们必须保持候选身份。

## 明确还没有证明的部分

下面这些内容仍然需要更多研究。当前公开文档不会把它们写成确定事实：

- `QZIP` 内部字段的完整语义。
- `DATA_CONTAINER/segs` 的完整压缩、块布局和 streaming 行为。
- 所有曲线 payload 的字段含义。
- `dep_token`、`dep_slot` 等字段的精确语义。
- 长 property graph 的真实字段名和编辑器来源。
- `SCRIPT_COLLECTION/SBCHK___` 的完整图结构、变量槽和执行语义。
- 每个 director event 对应的官方编辑器类型名。
- 真实 runtime load order。
- 真实 runtime control-flow graph。
- 每个 shader/pipeline tuple 在运行时的创建时机和复用策略。
- 每个 UI、对白、输入提示的最终 gameplay 规则。

这些空白不会削弱已有聚合证据。它们只是提醒我们：这份文档能讨论系统形状和工程启发，但不能冒充运行时源码注释。

## 公开发布边界

公开版遵守一个简单原则：可以发布“别人无法还原游戏内容的聚合解释”，不能发布“能替代或重建原始内容的数据”。

因此本仓库不会包含：

- 原始游戏资产。
- `.big`、`.idx`、`.dep` 等原始包或索引文件。
- 二进制样本。
- 大型原始扫描表。
- 提取出的音频、贴图、动画、模型或脚本 bytecode。
- 对白全文、本地化全文或可还原剧情内容的大段导出。
- 任何 Quantic Dream、Sony 或其他权利方的专有代码/资产。

仓库里的 `data/` 只保留小型聚合表，例如资源类型总览、系统证据矩阵和证据来源索引。它们用于说明“结论从哪里来”，不用于发布原始研究材料。

## 为什么要这么保守

做引擎分析最容易犯两个错误。

第一个错误是过度自信：看到几个字段、几张表、几个字符串，就把它写成官方架构。这样短期读起来很爽，但对开发者没有帮助，因为它把证据和想象混在一起。

第二个错误是过度保守：只要不是源码证明，就什么都不敢说。这样也没有帮助，因为资源索引、依赖图、时间线和渲染 manifest 本来就能说明很多系统性事实。

这份文档选择中间路线：强证据写成结论，桥接证据写成 evidence，推断写成 candidate，未知部分直接承认未知。对自研引擎来说，这种写法更有用。你可以放心吸收系统设计思路，同时知道哪些地方不能当成 DBH 官方实现细节。
