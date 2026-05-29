# 渲染：强在工程化

DBH 的渲染强，不只是因为用了某几个高级效果。更重要的是，它把 shader、pipeline、descriptor、材质、贴图、后处理和 sequence controller 放进了可 cook、可统计、可审计的系统。

## 关键规模

| 指标 | 数量 |
|---|---:|
| SPIR-V module | 81,649 |
| pipeline-state record | 99,453 |
| unique shader pair | 42,343 |
| state-expanded shader pair | 38,477 |
| QDIF/SPIR-V matched descriptor binding | 729,418 |
| texture resource | 127,939 |
| FINALIZE_DATA | 64 |
| timed MVSHADER | 5,283 |

这些数字说明渲染不是“运行时按需随便拼”，而是有大量离线编译和状态枚举。

## Pipeline cache 不是附属工具

自研引擎常见问题是：shader 写完了、效果跑起来了，但 pipeline state、variant、descriptor layout、资源绑定和平台差异都没有形成可审计资产。

DBH 的证据更像是把这些东西都显式化：

- shader library：SPIR-V module 和 metadata 可扫描。
- pipeline-state manifest：每条 pipeline record 有稳定统计口径。
- descriptor/resource ABI：材质、贴图、geometry buffer、pass texture、shadow texture 等资源角色可归类。
- pass family candidate：clustered lighting、material、shadow/depth、post process 等 pipeline 能被分簇分析。
- shader-pair variant atlas：能知道哪些 shader pair 被多少 pipeline state 复用，哪里 variant pressure 高。

这套东西的意义是让渲染复杂度可管理，而不是让渲染工程师靠记忆维护几万种状态组合。

## 后处理是资源和时间线动作

`FINALIZE_DATA` 不是一个全局开关。它更像资源化的 presentation/post preset：

- 64 个 `FINALIZE_DATA`。
- 64 个含 `COLOR_GR`。
- 63 个含 `GRAIN___`。
- sequence 侧有 viewport fade、invoke finalize 和 `SEQUENCE -> FINALIZE_DATA` 引用。
- post/finalize director event 还能和 2 秒 beat bucket、rendering-narrative bucket 对齐。

对自研引擎来说，后处理应该做成：

- preset 资源，有版本、字段、hash 和 diff。
- sequence action，可以 push/pop/blend。
- runtime stack，可以接收 gameplay、camera、sequence 多方 override。
- profiler，可以知道哪些段落用了哪些 preset，哪些参数变化最大。

## Material controller 是叙事接口

最值得抄的不是某个具体 shader，而是 material controller 的跨系统边界。

DBH 的证据显示：

- sequence 里有 5,283 个 `MVSHADER`，全部可解析，并能 join 到带时间的 director event。
- 脚本常量里有 `ApplyShaderController`、`SetShaderControllerValue` 等 controller 调用痕迹。
- 材质 payload 里能看到 controller/value/material instance 相关结构。
- shader descriptor/resource vocabulary 里有 material buffer、material instance buffer、texture buffer 等资源角色。

工程含义是：角色受伤、雨水、污渍、Android LED、眼泪、服装状态、道具液体等叙事状态，不应该只存在脚本变量里，也不应该只存在材质参数里。它们应该是跨工具链的一等接口。

一个自研模型可以长这样：

```text
MaterialController
  name
  type
  default_value
  range
  unit
  remap_or_curve
  target_material_slots
  script_api_name
  sequence_track_binding
  gpu_layout_binding
```

这样脚本能长期维护角色状态，sequence 能短时精确排演，renderer 只负责执行已 cook 的布局和 buffer 更新。

## 对自研渲染的建议

1. 每次构建都输出 pipeline manifest。
2. 每个 shader 变体都要能追踪到 pass family、descriptor layout 和资源角色。
3. pipeline cache builder 要输出 warmup plan、失败报告和 variant pressure。
4. 后处理 preset 要资源化，不能散落在 camera script。
5. 材质 controller 要跨脚本、时间线、资产和渲染。
6. 渲染审计工具要能回答“这段 sequence 为什么贵”，而不是只显示平均帧率。
