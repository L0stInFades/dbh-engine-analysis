# 渲染：强在可预编译、可分类、可审计

DBH 的画面强，当然和美术、灯光、材质、后处理都有关系。但从工程角度看，更值得学的是另一件事：它把渲染复杂度提前变成了可统计的数据。

很多项目的渲染系统到后期会变成黑箱：shader 越来越多，变体越来越多，pipeline state 越来越多，某个场景为什么卡只能靠抓帧和经验猜。DBH 的证据说明，它至少在 PC 版里保留了大量可以离线分析的 shader、pipeline、descriptor、pass family 和材质状态信息。

## 渲染不是一堆 shader 文件

先看规模：

| 指标 | 数量 |
|---|---:|
| SPIR-V module | 81,649 |
| shader records | 81,649 |
| pipeline-state record | 99,453 |
| created pipeline count from stats | 99,453 |
| unique shader pair | 42,343 |
| reused shader pair | 38,477 |
| QDIF metadata rows | 1,025,191 |
| QDIF/SPIR-V matched rows | 729,418 |
| texture resources | 127,939 |
| FINALIZE_DATA resources | 64 |

这组数字的意思是：DBH 的渲染管线不是运行时临时把几个 shader 拼起来，而是有一个相当完整的离线状态空间。shader module、pipeline state、descriptor metadata、resource vocabulary 都能被扫描和交叉验证。

对自研引擎来说，这个思路很实际。项目越大，越不能只关心“shader 能不能编译过”。你还要知道：

- 有多少 pipeline state。
- 哪些 shader pair 被大量复用。
- 哪些资源绑定在绝大多数 pipeline 中出现。
- 哪些 pass family 占了最大比例。
- 哪些 shader pair 的变体压力最高。
- 哪些 sequence 或镜头段落会触发某些材质/后处理状态。

## pass family：不是官方名，但很有工程价值

分析把 99,453 条 pipeline-state record 分到几个 pass family candidate 里。这里的 candidate 很重要：它不是官方 pass 名，也不是源码里的 enum，而是根据 shader/resource/pipeline evidence 做出的保守归类。

| pass family candidate | pipeline 数 | 比例 | 读法 |
|---|---:|---:|---|
| clustered lighting | 48,718 | 48.99% | 大量 pipeline 绑定 cluster/light/octree 相关资源 |
| simple texture/material | 23,198 | 23.33% | 贴图/材质常规路径 |
| default material forward/gbuffer | 13,006 | 13.08% | 默认材质或 forward/gbuffer 候选路径 |
| material instance state | 10,135 | 10.19% | 明确出现 material instance buffer |
| shadow/depth | 2,906 | 2.92% | 深度/阴影候选路径 |
| post process/pass texture | 1,490 | 1.50% | pass texture/post 相关路径 |

这个表对开发者的意义很直接：你的 pipeline cache builder 不应该只输出“总共有多少 pipeline”。它应该输出 pass family、资源绑定、shader pair、变体压力。这样美术、TA、渲染工程师和性能 QA 才能讨论同一张表。

## descriptor/resource ABI：资源角色要能被看见

descriptor evidence 里最有用的是资源角色。比如：

| resource role | 典型资源名 | 非独占 pipeline count |
|---|---|---:|
| geometry buffer vocabulary | `g_rbVertices_Buffer`, `g_rbObjects_Buffer`, `g_rbTBNs_Buffer`, `g_rbNormals_Buffer` | 257,888 |
| cluster/light/octree vocabulary | `g_rbCubeTextures`, `g_rbOctreeTexCoords_Buffer`, `g_rb3DTextures` | 232,400 |
| material/texture vocabulary | `g_rb2DTextures`, `g_rbMaterials_Buffer` | 174,984 |
| pass texture vocabulary | `g_rb2DPassTextures`, `g_rb3DPassTextures` | 82,169 |
| shadow/pass texture vocabulary | `g_rb2DShadowPassTextures` | 63,213 |
| material instance vocabulary | `g_rbMaterialInstances_Buffer` | 10,136 |

注意这里仍然是 vocabulary evidence，不是官方 C++ struct 名。但它已经足以说明渲染资源不是随便绑的。几何、材质、贴图、pass texture、shadow texture、cluster/light 数据都有可重复观察的绑定模式。

自研引擎应该学的是：descriptor layout 和资源角色必须能导出成人能读懂的报告。否则 shader 复杂起来以后，任何重构都会变得危险。

## shader pair 和变体压力

DBH PC 版有 42,343 个 unique shader pair，38,477 个 state-expanded shader pair。每个 shader pair 对应的 pipeline 数量中位数是 2，p95 是 4，最大值是 332。

这个数字很有用。它说明绝大多数 shader pair 变体压力不算夸张，但也存在少数极端 pair。对引擎工具来说，最应该自动标红的正是这些极端点。

建议自研 pipeline cache builder 至少输出：

```text
ShaderPairProfile
  vertex_shader_id
  fragment_shader_id
  pipeline_count
  pass_family
  descriptor_roles
  render_state_variants
  material_feature_flags
  warmup_priority
```

这样你能知道某个 shader pair 是因为 blend/depth/raster state 变化多，还是因为材质 feature、descriptor layout、pass family 分裂导致变体多。

## 后处理不是相机脚本里的几个参数

`FINALIZE_DATA` 证据说明 DBH 的后处理/presentation 也是资源化的。

| 指标 | 数量 |
|---|---:|
| `FINALIZE_DATA` resource | 64 |
| parse ok | 64 |
| `COLOR_GR` block | 64 |
| `GRAIN___` block | 63 |
| color grading fingerprint | 51 |
| grain fingerprint | 7 |
| sequence -> finalize ref rows | 210 |
| raw viewport fade action | 795 |
| raw invoke finalize action | 142 |
| post director event rows | 882 |
| timed post director event rows | 624 |

这说明后处理不是“某个相机上有个开关”。它有资源、有引用、有时间线 action，也能和 shot、sound、camera、material 等同窗分析。

自研引擎里，后处理最好做成：

```text
PostPreset
  color_grading
  grain
  exposure
  dof
  bloom
  motion_blur
  ssr
  version
  hash

PostStack
  gameplay_layer
  camera_layer
  sequence_layer
  debug_override_layer
  blend_policy
  restore_policy
```

这样 sequence 可以 push/pop/blend preset，camera 可以提出临时 override，gameplay 可以保留长期状态，调试工具也能知道最终画面为什么是这个样子。

## 材质 controller：渲染和叙事的接口

这一块是整份研究里最值得开发者注意的地方。

`MVSHADER` 不是孤立的材质动画。它和 director event、脚本 controller、材质 payload、shader/material ABI 都能形成桥接证据。

| 指标 | 数量 |
|---|---:|
| `MVSHADER` rows | 5,283 |
| parse ok rows | 5,283 |
| director join rows | 5,283 |
| timed `MVSHADER` rows | 5,283 |
| parameter occurrences | 8,909 |
| unique parameters | 281 |
| family count | 9 |
| asset-backed parameters | 136 |
| expression-backed parameters | 135 |

常见 family 包括：

| family | 参数出现次数 | 读法 |
|---|---:|---|
| damage/blood/wound | 4,052 | 受伤、血迹、伤口状态 |
| android LED | 1,977 | Android LED 颜色/速度/状态 |
| weather/fluid/dirt | 1,586 | 雨、雪、水、泥、污渍 |
| android skin retract | 299 | 仿生人皮肤收缩 |
| cloth/hair | 144 | 衣物/头发状态 |
| face/tears/scan | 110 | 眼泪、面部细节、扫描感 |

这意味着角色状态不是只存在 gameplay 变量里，也不是只存在 shader 参数里。更合理的模型是四层打通：

| 层 | 应该做什么 |
|---|---|
| 资产/材质图 | 声明 controller 名称、类型、范围、remap、目标 shader slot |
| 脚本/运行时 | 维护长期状态，比如受伤、雨水、污渍、LED |
| Sequence | 按镜头/对白/动画精确排演短时变化 |
| 渲染 | 只执行 cooked layout 和 GPU buffer 更新 |

这套设计的价值不是“暴露更多参数”，而是保证叙事状态连续。角色受伤、淋雨、眼泪、LED 状态，如果跨分支、跨镜头、跨章节都要可信，就不能靠临时脚本硬改材质。

## 自研渲染工具链建议

如果你在做自己的引擎，建议尽早做这些输出：

1. `PipelineManifest`：每条 pipeline 的 shader、state、descriptor、pass family。
2. `ShaderPairReport`：每个 shader pair 的复用次数和变体压力。
3. `DescriptorAbiReport`：资源绑定、descriptor set/binding、resource role。
4. `RenderPassFamilyReport`：候选 pass family 的 pipeline 数、资源角色、典型 shader pair。
5. `TextureStreamingReport`：贴图大小、分块、mip、压缩、streaming 状态。
6. `MaterialControllerReport`：controller 名称、来源、sequence 使用、脚本使用、GPU layout。
7. `PostPresetReport`：后处理 preset、字段变化、sequence 引用、blend 行为。
8. `RenderAuditViewer`：把上面这些报告放到一个可以筛选和追踪的 UI 里。

不要等项目后期才补这些工具。越晚补，越容易变成只能抓帧、猜测和口头经验。
