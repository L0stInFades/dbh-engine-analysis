# 证据地图：哪些结论是怎么来的

这一章专门解释证据。它的目的不是堆数字，而是让读者知道每个工程判断背后大概站着哪些数据。

公开版只放聚合结果，不放原始资产和大表。这里提到的 CSV/JSON/HTML 名称，是原研究工作区里的分析产物名，用来说明证据来源；仓库里只保留了适合公开的摘要数据。

## 怎么读这些证据

这份分析有三种证据强度。

第一种是强证据。比如 idx 条目数、DEP edge 数、sequence 数量、SPIR-V module 数量、pipeline-state record 数量。这类数字来自结构化解析和全量扫描，适合支撑“系统规模”和“资源关系”的判断。

第二种是桥接证据。比如 `MVSHADER` 同时能和 director event、脚本 controller 名称、材质 payload、shader/material ABI 对上。这类证据不能证明官方内部类名，但能证明系统之间不是完全孤立的。

第三种是候选解释。比如 pass family candidate、cinematic-render bridge evidence、dialogue-delivery window evidence。它们适合指导引擎设计，但不能当作“官方字段名”或“精确运行时因果链”。

## 顶层指标

| 方向 | 指标 | 工程含义 |
|---|---:|---|
| BigFile idx 条目 | 373,748 | 资源索引非常细，不是粗粒度文件包 |
| 资源类型 | 41 | 内容被分成多类运行时资源 |
| unpadded 数据量 | 57.31 GB | 资源规模足以要求严格构建和审计 |
| DEP edge | 1,002,543 | 依赖关系是显式数据 |
| 有依赖的 owner | 117,586 | 大量资源能追踪依赖 |
| `SEQUENCE` | 5,479 | 剧情/导演段落是一等资源 |
| timeline chunk | 1,117,054 | 时间线内容密度很高 |
| director event | 616,474 | 可折叠为导演事件的行为规模很大 |
| timed director event | 590,149 | 大多数事件有时间信息 |
| SPIR-V module | 81,649 | 渲染 shader library 被 cook 成可扫描资产 |
| pipeline-state record | 99,453 | pipeline 状态不是临时拼出来的 |
| texture resource | 127,939 | 贴图/streaming 是核心资源压力 |
| `FLOW_DIALOG` atom | 80,976 | 对白不是孤立文本，而是数据 atom |
| `ANIM_DATA` | 24,562 | 表演资源规模很大 |
| `UI_MENU_RESOURCE` | 128 | UI/Scaleform 资源可追踪 |
| `FINALIZE_DATA` | 64 | 后处理/presentation preset 资源化 |

## 资源类型证据

资源类型分布能直接告诉我们这个引擎主要在服务什么内容。最大的几类不是随便的杂项，而是容器、贴图、声音、动画、sequence、UI、脚本、相机等。

| 资源类型 | 数量 | 大小 | 读法 |
|---|---:|---:|---|
| `DATA_CONTAINER` | 3,265 | 26.28 GB | 大量内容被装进容器/streaming 块 |
| `ENGINE_TEXTURE_FILE_RAW_LOD` | 78,923 | 12.30 GB | LOD 贴图资源占比极高 |
| `COM_SOUND_DATA` | 80,976 | 11.70 GB | 多语言语音/音频是核心体量 |
| `ANIM_DATA` | 24,562 | 6.84 GB | 表演动画资源占比很高 |
| `ENGINE_TEXTURE_FILE_RAW` | 49,016 | 3.33 GB | 非 LOD 贴图仍然很大 |
| `SEQUENCE` | 5,479 | 360.68 MB | 导演段落本身是资源 |
| `UI_MENU_RESOURCE` | 128 | 303.51 MB | Scaleform/UI 资源体量不小 |
| `SCRIPT_COLLECTION` | 855 | 185.05 MB | 脚本集合被 cook 成资源 |
| `CAMERA_SYSTEM_BANK` | 68 | 9.08 MB | 相机行为有专门资源 bank |
| `FLOW_DIALOG` | 80,976 | 6.99 MB | 对白 atom 很小，但数量极多 |

这张表对自研引擎很有启发：如果你的目标是强剧情和强表演，资源系统不能只为 mesh/texture 服务。对白、动画、sequence、UI、相机、后处理都要成为可追踪资源。

## 资源图和 DEP 证据

完整 DEP 导出显示：

| 指标 | 数量 |
|---|---:|
| DEP edge | 1,002,543 |
| idx-backed target edge | 547,368 |
| internal/non-idx target edge | 455,175 |
| bad owner header | 0 |

这里最重要的是区分两类引用。idx-backed target 是能回到顶层资源表的依赖；internal/non-idx target 更像资源内部对象引用。它们都不是简单的“坏引用”。

对 sequence 的验证更直接：选取 top 40 cinematic sequence 后，完整 DEP 里有 8,239 条 sequence owner edge，8,239 条都是 idx-backed，并且 40/40 与外层 `COM_CONT` 引用对齐。这个证据支持一个很重要的判断：至少对这些强电影化段落来说，sequence 的外层资源依赖能被 DEP 图完整追踪。

## 导演时间线证据

`sequence_director_event_summary.json` 折叠出 616,474 个 director event，其中 590,149 个带时间，timed share 约 95.73%。这意味着绝大部分导演事件不是无时序的杂项，而是可以放进时间轴里分析。

| 事件类别 | 数量 |
|---|---:|
| animation | 242,821 |
| camera_quake | 162,999 |
| sound | 60,537 |
| fx_event | 18,307 |
| shot | 17,156 |
| camera_modifier | 14,782 |
| watch | 13,145 |
| branch_condition | 12,516 |
| dialogue | 8,515 |
| haptic | 6,954 |
| material | 4,941 |
| input_condition | 4,752 |
| variable | 3,841 |
| ui | 1,786 |
| post_finalize | 882 |

这张表说明 `SEQUENCE` 不只是动画播放器。它同时调度镜头、声音、分支、输入、UI、材质、后处理等系统。

## 渲染证据

渲染侧最强的证据来自 pipeline、SPIR-V、QDIF metadata 和 descriptor/resource ABI 的联表。

| 指标 | 数量 |
|---|---:|
| pipeline rows | 99,453 |
| shader records / SPIR-V modules | 81,649 / 81,649 |
| QDIF metadata rows / matched | 1,025,191 / 729,418 |
| unique shader pairs | 42,343 |
| state-expanded shader pairs | 38,477 |
| descriptor ABI pipeline rows | 99,453 |
| pipeline descriptor occurrences | 2,408,937 |
| unique descriptor keys | 33 |
| unique binding keys | 57 |
| unique resource names | 25 |

pass family candidate 的分布也很清楚：

| pass family candidate | pipeline 数 |
|---|---:|
| clustered lighting | 48,718 |
| simple texture/material | 23,198 |
| default material forward/gbuffer | 13,006 |
| material instance state | 10,135 |
| shadow/depth | 2,906 |
| post process/pass texture | 1,490 |

这些数字支持一个工程判断：DBH 的渲染不是“一堆 shader 文件”，而是 shader library、pipeline manifest、descriptor/resource vocabulary、pass family、variant pressure 共同组成的可审计系统。

## 镜头、灯光和相机证据

`MOV_SHOT` 和 camera bank 的证据说明镜头语言不是简单相机参数。

| 指标 | 数量 |
|---|---:|
| shot instances | 27,870 |
| timed shot events | 27,959 |
| camera/lens groups | 27,953 |
| light groups | 109,424 |
| shots with lens | 24,183 |
| shots with lights | 25,803 |
| shots with key/fill/rim | 8,442 |
| FOV p50 | 33.60 |
| F-stop p50 | 4.0 |
| timed shot duration p50 | 2,200 ms |

camera bank 侧还有 68 个 `CAMERA_SYSTEM_BANK`、1,862 个 parsed camera block、32,617 行 feature、16,070 个 modifier、14,033 个 target。常见 modifier 包括 smooth、ICO、deadzone、offset、auto-focus、multi-noise、advanced framing 等。

这说明它的相机不是一个“第三人称相机组件”，而是 authored shot、camera bank、modifier、目标点和 sequence 事件共同构成的系统。

## 对白、音频和本地化证据

对白系统的证据非常适合拿来做自研设计参考。

| 指标 | 数量 |
|---|---:|
| `COM_SOUND_DATA` | 80,976 |
| unique sound base | 6,748 |
| 每个 sound base 语言数 | 12 |
| `FLOW_DIALOG` atom | 80,976 |
| atoms with anim data | 74,364 |
| atoms with sound event base match | 80,946 |
| audio languages observed | 12 |

这说明对白不是“一个字幕 key + 一个音频文件”这么简单。一个对白 atom 往往同时牵涉声音、事件、动画、本地化和表演。

## UI 和 flowchart 证据

flowchart 证据说明分支叙事不是只藏在脚本里。

| 指标 | 数量 |
|---|---:|
| UI resources scanned | 128 |
| flowchart UI resources | 32 |
| expected flowchart GFX count | 32 |
| ENG flowchart node keys | 2,692 |
| global flowchart UI keys | 81 |
| localization languages | 26 |
| character portrait asset presence | Markus/Kara/Connor 各 32 |

这个系统对开发者的启发是：分支复杂度要有 review UI。玩家看见的 flowchart 不是附属界面，它反过来要求剧情图、节点文本、本地化、章节结构和 checkpoint/statistics 都能被工具链校验。

## 材质 controller 证据

材质 controller 是这次研究里最有价值的跨系统证据之一。

| 指标 | 数量 |
|---|---:|
| `MVSHADER` rows | 5,283 |
| parse ok rows | 5,283 |
| director join rows | 5,283 |
| timed `MVSHADER` rows | 5,283 |
| parameter occurrences | 8,909 |
| unique parameters | 281 |
| controller families | 9 |
| asset-backed parameters | 136 |
| expression-backed parameters | 135 |

主要 family 包括 damage/blood/wound、android LED、weather/fluid/dirt、android skin retract、cloth/hair、face/tears 等。换句话说，角色受伤、雨水、污渍、LED、眼泪、衣物等叙事状态，会进入 sequence、脚本、材质和渲染之间的桥接层。

## 本章结论

DBH 的强项不是某一个孤立系统。真正重要的是这些证据能互相闭合：

- 资源图解释了 sequence 为什么能追踪依赖。
- 导演时间线解释了镜头、对白、输入、UI、材质、后处理为什么能同步审计。
- 渲染 manifest 解释了复杂画面为什么没有完全变成运行时黑箱。
- 对白/flowchart/UI 证据解释了分支叙事为什么可以被 QA。
- material controller 证据解释了角色状态为什么能跨脚本、时间线和渲染保持连续。

这就是“更适合开发者阅读”的核心：不要只看 DBH 做了什么效果，要看它怎样让这些效果在生产系统里可管理。
