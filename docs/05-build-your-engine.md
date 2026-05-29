# 如果自己做引擎，应该抄什么

不要照抄 DBH 文件格式。真正值得抄的是生产系统思想。

## 1. 资源图是一等公民

每个 cooked resource 都应该有：

- resource id
- type id/name
- source asset path
- cooked offset/size/hash
- dependency list
- localization key links
- sequence links
- audio/animation/camera links
- debug name

release build 可以 strip debug 信息，但内部 build 必须保留 debug manifest。没有资源图，后面的 sequence profiler、branch QA、rendering audit 都会缺底座。

## 2. Sequence 是 authoritative timeline

不要把剧情写成散落脚本。建议从一开始就设计：

```text
Chapter
  Scene
    Sequence
      Shot
      Dialogue
      Animation
      Camera
      Sound
      UI Prompt
      Input Window
      Branch Condition
      Variable Write
      Material Controller
      Post Preset
```

每个节点都要能回答：

- 前置条件是什么？
- 修改哪些变量？
- 播放哪些动画/声音/字幕？
- 使用哪个镜头、DOF、后处理？
- 失败、超时、死亡、缺席角色怎么办？
- flowchart 如何显示？

## 3. 互动逻辑进入导演时钟

不要拆成：

- cutscene player
- QTE manager
- variable system
- UI prompt manager
- subtitle manager
- camera override script

更好的做法是统一成：

```text
DirectorEvent
  time_range
  category
  condition_ref
  variable_writes
  input_prompt
  failure_target
  timeout_target
  cooccurrence_tags
```

这样工具能生成 event row、bucket profile、response window、branch topology、UI prompt window，而不是让 QA 靠录屏猜同步问题。

## 4. 对白做成 DialogueAtom

对白节点应该同时绑定：

- 字幕 key
- 多语言音频
- speaker
- event name
- lipsync/facial/performance
- fallback
- subtitle style
- flowchart/review metadata

不要让音频、本地化、字幕、口型和表演动画各自维护一套 ID。

## 5. Camera bank 比相机组件重要

互动电影项目需要 camera bank 和 camera modifier stack：

- zone/scene camera
- character target
- bone/locator target
- smooth/offset/deadzone
- auto-focus
- quake/noise
- POI/ICO
- sequence override
- gameplay fallback

目标不是做一个炫酷相机，而是能在玩家控制和导演镜头之间稳定抢权、混合、恢复。

## 6. 后处理资源化

后处理不要散落在相机脚本里。做成：

```text
PostPreset
  color_grading
  exposure
  grain
  dof
  bloom
  motion_blur
  ssr
  volume
  version
  hash
```

Sequence action 可以 push/pop/blend preset。Profiler 能回答哪些 sequence 用了哪些 preset，哪些字段变化最大。

## 7. Material controller 跨四层

角色状态 controller 要同时出现在：

| 层 | 作用 |
|---|---|
| 材质/资产 | 声明 controller 名称、类型、范围、remap、目标 shader slot |
| 脚本/运行时 | 维护长期状态，比如伤口、雨水、污渍、LED |
| Sequence | 短时精确排演，与镜头、对白、动画同步 |
| 渲染 | 执行 cooked layout 和 GPU buffer update |

这样角色状态不会在分支、镜头切换或章节衔接里丢失。

## 8. 工具链优先级高于效果列表

建议优先做这些工具：

- resource graph viewer
- sequence editor
- sequence profiler
- shot/lens/light review
- dialogue atom review
- localization/audio/subtitle review
- branch/flowchart QA
- UI prompt window audit
- material controller timeline view
- post/finalize stack review
- pipeline cache builder
- rendering audit viewer
- shader variant pressure report

电影化互动游戏的难点不是“能不能播一个镜头”，而是几千个段落、几十万事件、多语言、多分支、多角色状态能不能被持续生产和维护。
