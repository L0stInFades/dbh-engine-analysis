# 电影化叙事：不是播片，是实时导演系统

DBH 的电影化叙事并不是“播一段视频，然后中间插几个 QTE”。从数据上看，它更像一个实时导演系统：镜头、动画、对白、声音、输入、分支、UI、材质、后处理都能进入同一条时间线。

这个判断很关键。因为如果你把它理解成 cutscene player，你会只去做镜头轨、动画轨和跳转。如果你把它理解成导演系统，你就会从一开始考虑资源依赖、事件分类、输入窗口、对白 atom、分支 QA、相机抢权、材质状态、后处理恢复和 profiler。

## `SEQUENCE` 是中心资源

全量扫描结果：

| 指标 | 数量 |
|---|---:|
| `SEQUENCE` | 5,479 |
| sequence 总字节 | 360.68 MB |
| parse error | 0 |
| timeline chunk | 1,117,054 |
| action marker | 809,404 |
| director event | 616,474 |
| timed director event | 590,149 |
| 30fps start grid ratio | 0.999998 |

这些数字说明 sequence 不是附属数据。它是大量互动电影内容的中心载体。

更重要的是，director event 的类别很广：

| 类别 | 数量 |
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

这就是“导演时间线”的含义。它不是只调动画，而是把很多系统都放进同一套时间组织里。

## 镜头：shot、lens、light 都是数据

`MOV_SHOT` 证据显示，镜头不是相机组件上的几个临时参数。

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

这说明 shot 里不只是“切到哪个相机”。它还携带 lens、focus、F-stop、light group、key/fill/rim 等电影摄影信息。

自研引擎里的 shot 系统应该至少能表达：

```text
Shot
  time_range
  camera_transform
  lens_fov
  focus_plane_or_target
  f_stop
  light_groups
  post_preset
  audio_mix_hint
  subtitle_anchor_hint
```

如果 shot 只是一个 camera cut，你就很难把镜头语言、灯光、对白和后处理统一 QA。

## Camera bank：玩法相机也要被导演化

camera bank 证据显示，DBH 的相机系统不只是 cutscene 相机。

| 指标 | 数量 |
|---|---:|
| `CAMERA_SYSTEM_BANK` | 68 |
| parsed camera block | 1,862 |
| feature rows | 32,617 |
| modifier features | 16,070 |
| target features | 14,033 |
| noise profiles | 1,936 |
| camera system director actions | 2,019 |

常见 modifier 包括 smooth、ICO、deadzone、offset、auto-focus、multi-noise、advanced framing、spring 等。target 里大量出现 `player`、`player.M_Head`、不同高度偏移和场景/区域名。

这说明玩法相机和导演相机之间不是完全割裂的。一个互动电影引擎需要能在玩家控制、区域相机、角色目标、sequence override 之间平滑抢权和交还。

## 对白：不是音频事件，而是表演 atom

对白证据非常强：

| 指标 | 数量 |
|---|---:|
| `COM_SOUND_DATA` | 80,976 |
| unique sound base | 6,748 |
| 每个 sound base 语言数 | 12 |
| `FLOW_DIALOG` atom | 80,976 |
| atoms with anim data | 74,364 |
| atoms with sound event base match | 80,946 |
| `COM_SOUND_EVENT_DRIVEN` | 6,747 |

这说明对白不是一个裸音频文件。它更像一个 atom：声音、本地化、事件、动画和表演上下文都围绕它组织。

自研引擎可以设计成：

```text
DialogueAtom
  id
  text_key
  speaker
  audio_event_by_language
  lipsync_ref
  facial_anim_ref
  body_anim_ref
  subtitle_style
  fallback_policy
```

然后 sequence 只调 `DialogueAtom`，字幕服务、音频服务、表演系统各自消费同一个 atom。这样多语言、字幕、口型、表演动画才不会各维护一套 ID。

## 互动逻辑：分支和输入必须进入导演时钟

互动逻辑不是额外挂在 cutscene 外面的 QTE manager。证据里有 17,903 个 timed core logic seed，来自 branch、input、variable、time control、UI 等类别。

2 秒窗口内，这些 timed logic seed 的同窗情况如下：

| 2 秒窗口同窗项 | seed 数 | 比例 |
|---|---:|---:|
| cinematic response | 16,441 | 91.83% |
| shot | 12,196 | 68.12% |
| camera | 13,338 | 74.50% |
| performance | 14,409 | 80.48% |
| dialogue/sound | 13,175 | 73.59% |
| material/post | 5,572 | 31.12% |
| UI | 4,572 | 25.54% |

这说明很多输入/分支/变量事件并不是孤立发生的。它们旁边常常有镜头、相机、表演、声音、材质或后处理。

所以自研引擎不要把互动逻辑拆成一堆互不认识的系统。更好的模型是：

```text
DirectorEvent
  time_range
  category              # shot, dialogue, input, branch, material...
  condition_ref
  variable_writes
  input_prompt
  timeout_target
  fail_target
  branch_target
  cooccurrence_tags
```

工具链要能输出 response window、branch topology、UI prompt window，而不是让 QA 看录屏猜“为什么这个按键提示出现得不舒服”。

## UI prompt：提示不是悬浮按钮

UI prompt window 的证据也很清楚。1,414 个 timed UI seed 中，2 秒 around 窗口里：

| 同窗项 | 数量 | 比例 |
|---|---:|---:|
| logic partner | 1,090 | 77.09% |
| cinematic context | 1,403 | 99.22% |
| shot | 1,227 | 86.78% |
| camera | 1,313 | 92.86% |
| performance | 1,291 | 91.30% |
| dialogue/sound | 1,082 | 76.52% |
| material/post | 562 | 39.75% |

这说明 UI prompt 常常嵌在电影化上下文里。它不是简单画一个按钮，而是要和镜头、动作、声音、表演、输入窗口对齐。

## Flowchart：分支复杂度要被玩家和工具看见

flowchart UI 证据：

| 指标 | 数量 |
|---|---:|
| UI resources scanned | 128 |
| flowchart UI resources | 32 |
| expected flowchart GFX count | 32 |
| ENG flowchart node keys | 2,692 |
| localization languages | 26 |
| global flowchart UI keys | 81 |
| character portrait asset presence | Markus/Kara/Connor 各 32 |

这说明 flowchart 不是临时 UI。它和章节/路线、本地化、角色头像、checkpoint、统计、路径可见性、legend 文本都有关系。

自研引擎如果做分支叙事，也应该把 flowchart 当成 QA 工具，而不是只当玩家菜单。它应该能发现：

- 某个剧情节点没有本地化。
- 某个死亡/失败/结局节点没有 flowchart 文本。
- 某个章节有 asset-only 或 localization-only 异常。
- 某个分支目标没有 sequence 入口。
- 某个 checkpoint 节点缺资源或状态。

## 顶级 sequence 的资源组成

top 40 cinematic sequence 的 resource composition 很能说明问题：

| 指标 | 数量 |
|---|---:|
| selected sequences | 40 |
| total unique resource refs, nonexclusive | 11,622 |
| avg unique refs per sequence | 290.55 |
| named sequence ref rows | 28,833 |
| outer ref rows | 8,239 |
| nested ref rows | 20,594 |
| action direct ref rows | 34,117 |
| action child ref rows | 11,886 |
| dialogue flow link rows | 3,200 |
| MVSHADER rows in action markers | 1,981 |

top ref types 也很有意思：

| ref type | rows |
|---|---:|
| `ANIM_DATA` | 48,095 |
| `COM_SOUND_EVENT` | 22,843 |
| `FLOW_DIALOG` | 3,200 |
| `LOCALIZATION_CONTAINER` | 451 |
| `GMK` | 143 |
| `UI_MENU_RESOURCE` | 70 |
| `FINALIZE_DATA` | 3 |

这说明强电影化段落不是单一轨道强，而是动画、声音、对白、本地化、UI、材质、后处理共同参与。

## 本章结论

DBH 的电影感来自“可组织的复杂度”。它不是靠一个播放器，也不是靠某个镜头系统单独完成，而是把大量作者意图放进资源图和导演时间线，再用工具链去审计。

对自研引擎来说，最该学的是：

1. sequence 是 authoritative timeline。
2. shot/lens/light 是数据，不是临时相机参数。
3. 对白要做成 atom，连通音频、本地化、字幕和表演。
4. UI prompt 和输入窗口要进入导演时间线。
5. flowchart 是分支 QA 的一部分。
6. resource composition 和 response window 是调试电影化段落的核心工具。
