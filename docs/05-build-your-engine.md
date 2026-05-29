# 如果自己做引擎，应该怎么抄

不要照抄 DBH 的文件格式。文件格式是结果，不是目的。真正值得抄的是它的生产系统思想：把作者意图变成可构建、可依赖追踪、可审计、可 profile 的数据。

这一章按“自己做引擎时应该落哪些模块”来写。

## 1. 资源图先行

很多自研引擎一开始只做 pak 或 bundle：能打包、能加载、能压缩，就算资源系统完成了。这个阶段如果不保留依赖图，后面做剧情、对白、多语言、分支、shader cache、热更新、QA 都会很痛苦。

建议一开始就设计：

```text
ResourceRecord
  resource_id
  type
  source_path
  cooked_path
  package_id
  offset
  size
  hash
  dependencies
  debug_name
  owner_system
```

内部 build 一定要有 debug manifest。release 可以 strip，但构建机器、编辑器和 QA 工具必须能查：

- 这个 sequence 依赖哪些动画、声音、UI、后处理？
- 这个动画被哪些 sequence 使用？
- 这个本地化 key 是否有音频和字幕？
- 这个 shader/material controller 被哪些角色或段落使用？
- 这个资源是否只存在于本地化表里，却没有 UI/flowchart 资源？

DBH 的 idx/dep 证据说明这件事非常重要。资源图不是锦上添花，它是生产系统的地基。

## 2. Sequence 做成一等公民

如果你的目标是强剧情，不要把 sequence 当成 cutscene 插件。它应该是核心系统。

建议的数据模型：

```text
Chapter
  Scene
    Sequence
      ShotTrack
      AnimationTrack
      DialogueTrack
      AudioTrack
      CameraTrack
      InputTrack
      BranchTrack
      VariableTrack
      UITrack
      MaterialControllerTrack
      PostPresetTrack
```

每个 sequence 至少要能回答：

- 它依赖哪些资源？
- 哪些事件有时间，哪些事件无时间？
- 哪些事件会写变量？
- 哪些事件会打开输入窗口？
- 哪些事件会跳分支？
- 哪些对白有本地化、音频、口型、表演动画？
- 哪些材质 controller 会改变角色状态？
- 哪些后处理 preset 会 push/pop/blend？
- 跳过、失败、超时、死亡、角色缺席时怎么恢复？

没有这些信息，互动电影段落会很快变成“能跑，但没人敢改”。

## 3. DirectorEvent 统一事件语义

不要把 QTE、变量、UI、相机、对白都做成互不认识的模块。更好的方式是让它们都能落到统一事件表里。

```text
DirectorEvent
  event_id
  sequence_id
  time_start
  time_end
  category
  action_type
  payload_ref
  condition_ref
  resource_refs
  variable_reads
  variable_writes
  branch_target
  debug_label
```

这个表的价值不是运行时一定要照它执行，而是工具链可以用它做：

- event density heatmap
- 2 秒/4 秒 response window
- branch topology
- UI prompt window
- dialogue delivery window
- shot/editing rhythm
- material/post co-occurrence
- sequence resource composition

DBH 的证据显示，这些窗口分析非常有价值。比如 17,903 个 timed core logic seed 中，2 秒窗口有 91.83% 带 cinematic response。这个信息比“这里有个 QTE”有用得多。

## 4. DialogueAtom 统一对白

对白最容易被拆散：音频部门一个 ID，本地化一个 key，字幕一个 key，动画一个 lipsync，脚本再记一个事件名。项目小的时候能忍，项目大了就会崩。

建议做：

```text
DialogueAtom
  atom_id
  speaker
  text_key
  audio_event
  audio_by_language
  lipsync_ref
  facial_anim_ref
  body_anim_ref
  subtitle_style
  flowchart_link
  fallback_policy
```

然后 sequence 播放的是 `DialogueAtom`，而不是直接播放某个语言的音频文件。字幕、音频、表演、flowchart 都从同一个 atom 查数据。

DBH 的 `FLOW_DIALOG`、`COM_SOUND_DATA`、`COM_SOUND_EVENT_DRIVEN`、`ANIM_DATA`、本地化联表说明这条路线很适合强剧情项目。

## 5. Shot/Lens/Light 做成可审计资产

shot 不应该只是 camera cut。至少要包含：

```text
Shot
  shot_id
  time_range
  camera_transform
  lens_fov
  focus_plane
  f_stop
  light_groups
  key_fill_rim_tags
  post_hint
  audio_mix_hint
```

还要做 shot review 工具：

- 每个 sequence 有多少 shot。
- shot 平均/中位时长是多少。
- 哪些 shot 没有 lens。
- 哪些 shot 没有 light group。
- 哪些 shot 有 key/fill/rim。
- shot 周围 2 秒是否有 dialogue、sound、camera、performance。

DBH 的 shot/lens/light 证据说明，镜头语言可以被大量数据化。自研引擎不要只在 DCC 里看镜头，进引擎后也要能审计。

## 6. Camera bank 解决玩法和导演抢权

互动电影不是纯播片。玩家可能移动、观察、输入、失败、超时，导演镜头和玩法相机要不断抢权。

建议做：

```text
CameraBank
  scene_or_zone
  behavior
  target_rules
  modifier_stack
  noise_profile
  fallback_camera
  sequence_override_policy
```

modifier 至少要考虑：

- smooth
- offset
- deadzone
- auto-focus
- advanced framing
- noise/handheld
- POI/interest target
- spring
- camera shake

不要只做一个“电影相机轨”。你真正需要的是 gameplay camera、sequence camera、zone camera、target camera 之间的过渡和恢复策略。

## 7. MaterialController 做成跨系统接口

这是最建议自研引擎提前设计的部分。

```text
MaterialController
  name
  type
  default_value
  range
  unit
  remap_curve
  target_material_slots
  script_api
  sequence_binding
  gpu_layout_binding
```

它要同时服务四层：

| 层 | 用途 |
|---|---|
| 资产/材质图 | 声明 controller，定义范围和目标 slot |
| 脚本/运行时 | 维护长期角色状态 |
| Sequence | 按镜头精确动画 |
| 渲染 | 执行 cooked layout 和 buffer update |

比如受伤、血迹、雨水、泥、眼泪、LED、仿生人皮肤收缩，这些都不应该只是一个 shader 参数。它们是剧情状态。

## 8. PostPreset 和 PostStack 资源化

后处理不要散在相机脚本里。做成资源：

```text
PostPreset
  id
  color_grading
  grain
  exposure
  dof
  bloom
  ssr
  motion_blur
  version
  fingerprint
```

运行时做 stack：

```text
PostStack
  gameplay_layer
  camera_layer
  sequence_layer
  cinematic_override_layer
  debug_layer
  blend_policy
  restore_policy
```

这样 sequence 结束、分支跳转、玩家打断、死亡重来时，后处理状态可以恢复，不会残留。

## 9. Flowchart 当成 QA 系统

如果你的游戏有复杂分支，flowchart 不只是给玩家看的菜单。它应该反过来帮助开发：

- 每个剧情节点是否有标题。
- 每个节点是否有多语言文本。
- 死亡、失败、结局、隐藏路径是否有状态。
- checkpoint 是否能回到正确变量状态。
- 分支条件是否能追踪到脚本变量。
- flowchart UI asset 和 localization key 是否互相匹配。

DBH 的 flowchart 证据里有 32 个 flowchart UI resource、2,692 个英文 node key、26 种本地化语言。这说明分支 review 是生产系统的一部分。

## 10. 渲染工具链尽早做

至少做这些报告：

| 工具 | 解决的问题 |
|---|---|
| Pipeline manifest | 知道有哪些 pipeline state |
| Shader pair report | 找出变体压力 |
| Descriptor ABI report | 看清资源绑定和 layout |
| Pass family report | 按渲染路径分类 pipeline |
| Texture streaming report | 管理贴图体量和分块 |
| Material controller report | 追踪叙事状态进入材质 |
| Post preset report | 追踪后处理状态 |
| Rendering audit viewer | 把上述证据串起来 |

这些工具不是“高级项目才需要”。如果目标是电影化互动游戏，它们应该比很多效果功能更早出现。

## 11. 最小可落地路线

如果团队资源有限，可以按这个顺序做：

1. 资源 ID + dependency manifest。
2. Sequence event table。
3. DialogueAtom。
4. ShotTrack + CameraBank。
5. Variable/Branch/Input event 进入 DirectorEvent。
6. MaterialController 统一命名和绑定。
7. PostPreset 资源化。
8. Sequence profiler。
9. Pipeline/descriptor/shader pair 报告。
10. Flowchart QA。

不要等所有系统都成熟再做审计。审计工具越晚做，越只能服务排错；越早做，越能反过来塑造内容生产方式。
