# AnimeFix-Workflow

**A Modular Anime Generation & Refinement Workflow for ComfyUI**

面向 **SDXL / Illustrious** 二次元图像生成的模块化 ComfyUI 工作流。

主要处理链：

```
Txt2Img → Upscale Refiner → Hand Repair → Face Repair → Eye Detailer → Output
```

> Version: **2.x (FaceFix V2)** · Workflow file: `workflow_facefix_v2.json`

---

## 简介

AnimeFix-Workflow 是一份单一文件的 ComfyUI 工作流，把"生成 → 放大 → 局部修复"拆成若干可独立开关的模块，用 `Set / Get` 广播节点在各模块之间传递模型、CLIP、VAE、LoRA、latent 与条件，从而避免长距离连线并把各阶段解耦。

它不是"一键完美出图"的方案。工作流只是把常见问题的**处理机会**前置：手部畸形、脸部崩坏、眼睛细节不足。是否改善、改善多少，取决于底模、LoRA、提示词、分辨率与随机种子。

## Features

以下均为 `workflow_facefix_v2.json` 中**实际存在**的功能：

- **SDXL / Illustrious oriented workflow**
- **Txt2Img generation** — `EmptyLatentImage` + `SamplerCustomAdvanced`，默认 832×1216
- **Multi-stage sampling** — 主采样与放大重绘分离，各自独立的步数 / CFG / sampler / scheduler / denoise
- **High-resolution upscale + refinement** — `ImageUpscaleWithModel`（ESRGAN）后 `ImageScale` 到 1248×1824，再低 denoise 重绘
- **Automatic hand detection** — `UltralyticsDetectorProvider`（YOLO 手部检测）
- **Hand local inpainting** — `SAMLoader` + `ImpactSimpleDetectorSEGS` + `SegsToCombinedMask` + `GrowMaskWithBlur` + `MaskDetailerPipe`，逐手裁剪局部重绘后羽化合成
- **Face detection and FaceDetailer** — `FaceDetailerPipe` + `face_yolov8m` + SAM
- **Eye Detailer** — 独立的眼睛检测器（`Eyeful_v2-Individual`）与 detailer 分支
- **SAM assisted masks** — 由 bbox 提示生成手部 / 脸部轮廓 mask
- **LoRA Stack** — `LoRA Stacker`（Efficiency Nodes）配置多 LoRA 及权重，经 `CR Apply LoRA Stack` 分发到各阶段
- **Modular Set/Get routing** — 83 个 `Set / Get` 广播节点完成跨模块资源传递
- **Optional repair switches** — 3 个 `LazySwitchKJ` 可独立旁路 Hand Repair / Face Repair，并切换手部 mask 来源
- **Before / after comparers** — 5 个 `Image Comparer (rgthree)` 节点，逐阶段对比
- **Modular workflow layout** — 12 个 Group 分区：提示词 / 功能开关 / Txt2Img / Upscale refiner / HAND REPAIR / FACE REPAIR / Face detailer / Eye Detailer 等

## Quick Start

1. Install [ComfyUI](https://github.com/comfyanonymous/ComfyUI)
2. Install the required custom nodes listed in [REQUIREMENTS.md](REQUIREMENTS.md)
3. Download the required models listed in [MODELS.md](MODELS.md)
4. Put each model into its corresponding `ComfyUI/models/` subfolder
5. Download `workflow_facefix_v2.json` from this repository
6. Drag `workflow_facefix_v2.json` into the ComfyUI canvas (or use **Workflow → Open**)
7. Resolve any **Missing Nodes** reported by ComfyUI Manager
8. Select your own compatible Checkpoint / LoRA if necessary
9. Run the workflow

> 本仓库不提供任何自动安装脚本。ComfyUI 与其插件的安装方式会随版本变化，请以各自官方文档为准。

## Usage

推荐使用流程：

1. 修改 **Positive Prompt**（`提示词 → 正向提示词` 分组）
2. 修改 **Negative Prompt**（`反向提示词` 分组）
3. 选择 **Checkpoint**
4. 配置 **LoRA**（`LoRA Stacker` 节点，默认 2 条）
5. 在 `功能开关` 分组确认 Hand Repair / Face Repair / 手部 mask 来源三个开关的状态
6. **Queue Prompt**
7. 依次经过：Txt2Img → Upscale Refiner → Hand Repair → Face Repair → Eye Detailer
8. 结果由 `SaveImage` 节点保存

### 关于开关

`功能开关` 分组中的三个 `LazySwitchKJ` 节点：

| 开关 | 作用 |
|---|---|
| **hand repair enable** | OFF 时原始图像直接通过，跳过手部修复 |
| **hand mask source** | OFF = YOLO + SAM（本地检测）；ON = MeshGraphormer 分支 |
| **face repair enable** | OFF 时 HAND FIXED 结果直接通过，跳过面部修复 |

关闭某一模块只会跳过该阶段，不会影响其它阶段的连接。

### 关于模型选择

工作流中当前写入的 Checkpoint 与 LoRA 只是**作者开发测试时使用的配置**，不是必需品。

- 你可以替换为其它兼容的 SDXL / Illustrious 模型。
- 不同模型、不同 LoRA 组合会显著改变输出效果，需要重新调整提示词与 denoise 参数。
- 无法保证所有 SDXL 系列模型都能得到相同或理想的结果。
- 替换模型后请重新检查 LoRA 的触发词（trigger words）。

## Hardware

本仓库**不声明最低显卡要求**，也不声明任何硬件"一定可用"。显存占用取决于：

- 所选 Checkpoint 的精度与体积
- 生成分辨率与放大目标分辨率
- 启用了哪些修复模块（Hand Repair / Face Repair / Eye Detailer 都会额外占用显存）
- 各 custom node 自身的实现与依赖

建议根据自己的硬件逐步测试：先只跑 Txt2Img，再逐个打开修复模块。

## Third-party licenses

本仓库**只包含工作流 JSON 配置与文档**，不重新分发任何第三方软件或模型。工作流引用了以下第三方项目，它们各自遵循其原始许可证：

| Project | License (as observed locally) |
|---|---|
| [ComfyUI](https://github.com/comfyanonymous/ComfyUI) | GPL-3.0 |
| [ComfyUI Impact Pack](https://github.com/ltdrdata/ComfyUI-Impact-Pack) | GPL-3.0 |
| [ComfyUI Impact Subpack](https://github.com/ltdrdata/ComfyUI-Impact-Subpack) | AGPL-3.0 |
| [ComfyUI-KJNodes](https://github.com/kijai/ComfyUI-KJNodes) | GPL-3.0 |
| [ComfyUI Inspire Pack](https://github.com/ltdrdata/ComfyUI-Inspire-Pack) | GPL-3.0 |
| [efficiency-nodes-comfyui](https://github.com/jags111/efficiency-nodes-comfyui) | GPL-3.0 |
| [ComfyUI_tinyterraNodes](https://github.com/TinyTerra/ComfyUI_tinyterraNodes) | GPL-3.0 |
| [ComfyUI_smZNodes](https://github.com/shiimizu/ComfyUI_smZNodes) | GPL-3.0 |
| [rgthree-comfy](https://github.com/rgthree/rgthree-comfy) | MIT |
| [ComfyUI-Detail-Daemon](https://github.com/Jonseed/ComfyUI-Detail-Daemon) | MIT |
| [comfyui_controlnet_aux](https://github.com/Fannovel16/comfyui_controlnet_aux) | Apache-2.0 |
| [cg-use-everywhere](https://github.com/chrisgoringe/cg-use-everywhere) | Apache-2.0 |
| [ComfyUI-AutomaticCFG](https://github.com/Extraltodeus/ComfyUI-AutomaticCFG) | MIT |
| [ComfyUI_Comfyroll_CustomNodes](https://github.com/Suzie1/ComfyUI_Comfyroll_CustomNodes) | Please refer to the original project for license terms |

上表许可证信息来自作者本机安装的对应项目副本，仅供参考，**不构成法律结论**。请以各项目官方仓库当前声明为准。

**Third-party software, custom nodes and model files remain subject to their respective licenses.**

## Disclaimer

- This repository does **not** redistribute any third-party model files.
- Users must obtain all models from their original distribution sources and comply with their respective licenses.
- Third-party custom nodes and models are governed by their own licenses; this repository's MIT license does not extend to them.
- AI output is non-deterministic / stochastic — identical settings do not guarantee identical results.
- Results may change between model, ComfyUI and custom-node versions.
- Future plugin updates may introduce compatibility issues.
- 本仓库不保证修复效果，不保证输出一致性，也不保证跨版本兼容性。
- 使用本工作流产生的内容及其合规性，由使用者自行负责。

本仓库的 MIT 许可证**仅覆盖**作者编写的 workflow JSON 配置与文档，详见 [LICENSE](LICENSE)。

---

## English Introduction

**AnimeFix-Workflow** is a single-file, modular ComfyUI workflow aimed at SDXL / Illustrious anime image generation.

Pipeline: `Txt2Img → Upscale Refiner → Hand Repair → Face Repair → Eye Detailer → Output`.

Each stage is an independently switchable module, wired together through `Set / Get` broadcast nodes rather than long cables. It includes ESRGAN upscaling with low-denoise refinement, YOLO + SAM assisted hand detection and local inpainting, `FaceDetailerPipe` based face repair, a dedicated eye detailer, an Efficiency-Nodes LoRA stack, and five before/after comparers.

**This repository contains only workflow configuration and documentation.** No third-party models, no custom-node source code, and no ComfyUI distribution are included. See [REQUIREMENTS.md](REQUIREMENTS.md) for the custom nodes used and [MODELS.md](MODELS.md) for the models you need to obtain yourself.

AI generation is stochastic. This workflow does not guarantee any particular repair quality or style consistency across models, versions, or seeds.
