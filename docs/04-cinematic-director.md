# 电影化叙事：不是播片，是导演系统

DBH 的电影感来自一个可审计的实时导演系统，而不是单纯播放预渲染片段。

## Sequence 是核心

全量扫描里有：

- 5,479 个 `SEQUENCE`。
- 1,117,054 个 timeline chunk。
- 616,474 个 director event。
- 320,455 个 timed primary action。
- 27,959 个 shot。
- 5,283 个 `MVSHADER`。
- 30fps 时间网格比 0.999978。

这说明 sequence 不是附属格式，而是互动电影生产的中心数据结构。

## 它调度什么

从分析结果看，director timeline 能覆盖或桥接：

- animation / performance
- shot / lens / light
- camera quake / camera modifier / camera system
- dialogue / localized sound / subtitle
- sound event / RTPC
- UI prompt / flowchart / Scaleform resource
- input condition / branch condition / variable assignment
- material controller
- viewport fade / finalize / post preset
- watch/head target / IK / cloth / attachment

这就是“导演时钟”的意思：不是每个系统各自跑一套时间，而是 authored event 可以放进同一条可统计时间轴里。

## 对白为什么像表演单元

对白不是单独音频事件。证据里有 80,976 个 `FLOW_DIALOG` atom，和 sound data、sound event、animation、本地化 key 有稳定关系。

dialogue delivery window 的扫描也显示：以 timed dialogue event 为 seed，在 2 秒窗口内经常能同时看到 shot、camera、performance、sound、watch、material/post、interactive logic 等上下文。

这给自研引擎的启发是：

```text
DialogueAtom
  id
  text_key
  audio_event
  speaker
  language_variants
  facial_or_lipsync_ref
  performance_anim_ref
  subtitle_style
  fallback_policy
```

Sequence 不应该直接操作字幕 UI。更好的边界是：

```text
SubtitleService
  current_language
  fallback_language
  subtitle_resource
  play(atom, timing, style, anchor)
```

## 镜头不是相机组件参数

`MOV_SHOT` 里可以抽取 shot、shot group、shot property、camera/lens group 和 light group。camera bank 侧又有大量 camera block、feature row、camera motion event。

这说明镜头系统至少应该拆成几层：

- authored shot：构图、镜头、焦点、灯光、剪辑意图。
- camera bank：玩法/区域/角色状态下的相机行为。
- camera modifier stack：smooth、offset、auto-focus、quake、POI/ICO 等。
- sequence override：关键镜头可以短时接管。
- gameplay fallback：玩家交互后可以平滑交还。

一个普通第三人称相机很难支撑这种互动电影生产。

## UI 和分支必须显性化

DBH 的 UI/flowchart 证据说明分支复杂度不是只藏在脚本变量里。flowchart 有专门 Scaleform 资源、chapter/route code、本地化文本和全局 UI vocabulary。

对自研引擎来说，branch QA 不能只靠策划手动检查。应该自动回答：

- 某个剧情节点是否有 flowchart 表示？
- 某个结局/死亡/失败节点是否有本地化文本？
- 某个 input prompt 是否和 camera/shot/performance 同窗出现？
- 某个变量写入之后是否有后继逻辑？
- 某个 sequence 入口是否有完整资源依赖？

## 为什么它不是“cutscene player + QTE”

如果系统只是 cutscene player + QTE manager，那么镜头、对白、输入、UI、变量、材质、后处理之间的关系会散落在各自模块里。DBH 的证据更像是把它们放进可 cook 的导演事件和资源图里。

这带来的好处是：

- 同步 bug 更容易定位。
- 分支段落可以被 profile。
- 镜头和输入窗口可以一起审计。
- 对白和表演能作为一个单元 QA。
- 材质状态能跨镜头和分支保持连续。
- 渲染成本能回溯到 sequence 和 pass family。
