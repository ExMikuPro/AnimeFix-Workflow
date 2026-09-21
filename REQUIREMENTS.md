# Requirements

本文件由 `workflow_facefix_v2.json` **实际扫描**生成：逐一读取每个节点的 `type`、`properties.cnr_id`、`properties.aux_id` 与 `Node name for S&R`，再用本机安装的插件源码交叉验证节点归属。

- 扫描节点总数：**161**
- 不同节点类型：**44**
- 其中 ComfyUI 本体节点类型：**18**（另有 `PrimitiveInt` 亦属本体，见下方说明）
- 第三方插件依赖：**13 个包**

---

## ComfyUI Core

以下节点由 ComfyUI 本体提供，**不需要安装任何第三方插件**：

| Node | 用途 |
|---|---|
| `CheckpointLoaderSimple` | 加载主 Checkpoint |
| `VAELoader` | 加载 VAE |
| `CLIPTextEncode` | 提示词编码 |
| `ConditioningCombine` | 合并条件（基础提示 + 模块附加提示） |
| `EmptyLatentImage` | 生成初始空 latent |
| `ModelSamplingDiscrete` | 采样模型设置 |
| `BasicScheduler` | 调度器 |
| `KSamplerSelect` | 采样器选择 |
| `SamplerCustomAdvanced` | 主采样 |
| `RandomNoise` | 噪声 / 种子 |
| `DualCFGGuider` | 双 CFG 引导 |
| `PrimitiveInt` | 整数输入（手部修复种子） |
| `VAEDecode` / `VAEEncode` | latent ↔ 图像 |
| `ImageScale` | 图像缩放 |
| `ImageUpscaleWithModel` | 模型放大 |
| `UpscaleModelLoader` | 加载放大模型 |
| `ControlNetLoader` | 加载 ControlNet |
| `ControlNetApply` | 应用 ControlNet |
| `SetUnionControlNetType` | 设置 Union ControlNet 类型 |
| `SaveImage` | 保存图像 |

> `PrimitiveInt` 定义于 ComfyUI 的 `comfy_extras/nodes_primitive.py`，属于本体节点。
> 工作流中有一个该类型节点（#283）在保存时被 `cg-use-everywhere` 的前端迁移逻辑从旧版 `Seed Everywhere` 改写而来，因此它的 `properties` 中没有 `cnr_id`。这**不代表**需要额外安装插件。

---

## Required Custom Nodes

以下 14 个包必须安装，否则 ComfyUI 会报告 **Missing Nodes**。

### 1. ComfyUI Impact Pack

- **Nodes used:** `FaceDetailerPipe` ×2, `MaskDetailerPipe`, `ToDetailerPipe` ×2, `ToBasicPipe` ×3, `FromBasicPipe`, `SAMLoader` ×2, `ImpactKSamplerAdvancedBasicPipe` ×3, `ImpactSimpleDetectorSEGS`, `SegsToCombinedMask`
- **Used for:** FaceDetailer 流程、MaskDetailer 局部重绘、BasicPipe / DetailerPipe 封装、SAM 加载与 bbox+SAM 分割、SEGS → mask 转换
- **Repository:** https://github.com/ltdrdata/ComfyUI-Impact-Pack

### 2. ComfyUI Impact Subpack

- **Nodes used:** `UltralyticsDetectorProvider` ×3
- **Used for:** 加载 Ultralytics YOLO 检测器（脸部 / 眼睛 / 手部），输出 `BBOX_DETECTOR` 与 `SEGM_DETECTOR`
- **Repository:** https://github.com/ltdrdata/ComfyUI-Impact-Subpack

> Impact Subpack 提供 `UltralyticsDetectorProvider`，`SAMLoader` 则由 Impact Pack 提供。两者需要分别安装。

### 3. ComfyUI-KJNodes

- **Nodes used:** `SetNode` ×29, `GetNode` ×50, `LazySwitchKJ` ×3, `GrowMaskWithBlur`
- **Used for:** 跨模块资源广播（`Set / Get`）、三个修复模块开关、mask 扩张与羽化
- **Repository:** https://github.com/kijai/ComfyUI-KJNodes

> **这是最关键的依赖之一。** 工作流有 79 个 `Set / Get` 节点承担全部跨模块连接，缺少此插件会导致工作流无法正常加载。

### 4. rgthree-comfy

- **Nodes used:** `Image Comparer (rgthree)` ×4
- **Used for:** 各阶段前后对比预览
- **Repository:** https://github.com/rgthree/rgthree-comfy

### 5. Comfyroll Studio

- **Nodes used:** `CR Apply LoRA Stack` ×5
- **Used for:** 把 LoRA Stack 分发应用到基础模型、手部、面部、眼睛等各阶段
- **Repository:** https://github.com/Suzie1/ComfyUI_Comfyroll_CustomNodes

### 6. ComfyUI Inspire Pack

- **Nodes used:** `CLIPTextEncodeWithWeight //Inspire` ×2
- **Used for:** 带权重的文本编码（面部修复的正 / 负向附加提示）
- **Repository:** https://github.com/ltdrdata/ComfyUI-Inspire-Pack

### 7. smZNodes

- **Nodes used:** `smZ CLIPTextEncode` ×2
- **Used for:** 提示词解析 / 权重语法处理
- **Repository:** https://github.com/shiimizu/ComfyUI_smZNodes

### 8. tinyterraNodes

- **Nodes used:** `ttN text` ×7
- **Used for:** 多行文本输入节点（各模块的附加正 / 负向提示词）
- **Repository:** https://github.com/TinyTerra/ComfyUI_tinyterraNodes

### 9. ComfyUI Detail Daemon

- **Nodes used:** `DetailDaemonSamplerNode` ×1 — **当前为 bypass 状态**
- **Used for:** 采样过程中的细节控制
- **Repository:** https://github.com/Jonseed/ComfyUI-Detail-Daemon

### 10. efficiency-nodes-comfyui

- **Nodes used:** `LoRA Stacker` ×1
- **Used for:** 配置 LoRA 列表与权重
- **Repository:** https://github.com/jags111/efficiency-nodes-comfyui

### 11. ComfyUI AutomaticCFG

- **Nodes used:** `Automatic CFG` ×2
- **Used for:** 自动 CFG 处理
- **Repository:** https://github.com/Extraltodeus/ComfyUI-AutomaticCFG

### 12. cg-use-everywhere

- **Nodes used:** 无专属节点类型；以 `properties.ue_properties` 与 `extra.ue_links` 形式作用于全部节点
- **Used for:** Set/Get 广播机制的运行时支撑
- **Repository:** https://github.com/chrisgoringe/cg-use-everywhere

> 工作流中每个节点都带有 `ue_properties` 元数据。缺少此插件虽不一定直接报 Missing Node，但广播行为会失效。

### 13. comfyui_controlnet_aux

- **Nodes used:** `MeshGraphormer+ImpactDetector-DepthMapPreprocessor` ×1
- **Used for:** MeshGraphormer 手部 mask 分支（由 `hand mask source` 开关切换；该节点当前输出**仅用于 mask，未接入任何 ControlNet**）
- **Repository:** https://github.com/Fannovel16/comfyui_controlnet_aux

---

以上 **13 个第三方包**即为 `workflow_facefix_v2.json` 的全部外部依赖。若你的 ComfyUI 仍报告缺失节点，请用 **ComfyUI Manager → Install Missing Custom Nodes** 自动比对。

> 无法可靠确认仓库地址的插件，请直接在 ComfyUI Manager 中搜索该包名安装。

---

## Bypassed / legacy nodes

以下节点在工作流中处于 **bypass（mode = 4）** 状态，当前不参与执行：

| Node | Type | 说明 |
|---|---|---|
| #207 | `GetNode` | `[Legacy / Bypassed]` — 无消费者 |
| #209 | `SetNode` | `[Legacy / Bypassed]` — 无消费者 |
| #210 | `SaveImage` | 已旁路 |
| #215 | `VAEEncode` | 已旁路 |
| #216 | `GetNode` | 已旁路 |
| #217 | `SetNode` | 已旁路 |
| #231 | `DetailDaemonSamplerNode` | 已旁路（Detail Daemon 被绕过） |

**即使处于 bypass 状态，其对应的插件仍然是加载依赖** —— ComfyUI 在读取工作流时依然需要认识这些 node type。请不要因为节点被旁路而跳过安装对应插件。

---

## Tested configuration / workflow metadata

以下信息来自工作流 JSON 自身记录的元数据（**这不是最低版本要求**）：

| 项目 | 值 |
|---|---|
| Workflow format `version` | `0.4` |
| `last_node_id` | `296` |
| `last_link_id` | `433` |
| ComfyUI core nodes `ver` | `0.3.26` |
| ComfyUI Impact Pack `ver` | `8.14.2` |
| ComfyUI Impact Subpack `ver` | `1.3.2` |
| ComfyUI-KJNodes `ue_properties.version` | `7.8` |
| ComfyUI Inspire Pack `ver` | `0f38db4180ce7836a80765111b87d5b4376a7a45` |
| Comfyroll `ver` | `d78b780ae43fcf8c6b7c6505e6ffb4584281ceca` |
| ComfyUI-AutomaticCFG `ver` | `2e395317b65c05a97a0ef566c4a8c7969305dafa` |
| Detail Daemon `ver` | `f391accbda2d309cdcbec65cb9fcc80a41197b20` |
| efficiency-nodes `ver` | `3ead4afd120833f3bffdefeca0d6545df8051798` |
| rgthree-comfy `ver` | `ab37a0bd377a4443d04896b34a9491ddb1cb014b` |
| tinyterraNodes `ttNnodeVersion` | `1.0.0` |

这些版本号是 ComfyUI 在保存工作流时自动写入的，仅用于说明作者保存该文件时的环境。**作者未验证这些是否为最低可用版本**，请尽量使用各插件的较新版本。

---

## Third-party licenses

本仓库**只引用**这些第三方工具，**不重新分发其源代码**。各插件遵循其原始许可证（GPL-3.0 / AGPL-3.0 / Apache-2.0 / MIT 等），请以各自项目仓库为准。无法确认许可证的项目，请参考其原始项目。

**Third-party software, custom nodes and model files remain subject to their respective licenses.**
